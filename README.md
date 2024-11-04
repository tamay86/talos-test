# Vagrant

Testumgebung für Vagrant (mit vagrant-libvirt) für Talos bestehend aus einer Controlplane und einem Worker

## Vorbereitung

* Talosctl installieren <https://www.talos.dev/v1.8/talos-guides/install/talosctl/>
* Talos ISO herunterladen

```shell
# Download Talos Iso
if [[ ! -f talos/metal-amd64.iso ]]
then
  wget https://github.com/siderolabs/talos/releases/download/v1.8.2/metal-amd64.iso -O talos/metal-amd64.iso
fi
```

## Start der VMs

```shell
vagrant up
```

Es werden 2 NICs erstellt, die erste NIC hat 192.168.121.0/24 aber keine IPv6. Habe keine Möglichkeit bisher gefunden, der ersten NIC das mitzugeben
Daher wird nur NIC2 benötigt, die sowohl IPv4 als auch IPv6 aktiviert hat. Mittels `virsh` kann die erste NIC entfernt werden.

```shell
# IP und MAC rausfinden
MAC_CP=$(virsh -c qemu:///system domifaddr --domain talos_control-plane-node-1 | grep 121 | awk '{ print $2}')
MAC_WORKER=$(virsh -c qemu:///system domifaddr --domain talos_worker-node-1 | grep 121 | awk '{ print $2}')
IP_CP=$(virsh -c qemu:///system domifaddr --domain talos_control-plane-node-1 | grep 100 | awk '{ print $4 }' | cut -f1 -d'/')
IP_WORKER=$(virsh -c qemu:///system domifaddr --domain talos_worker-node-1 | grep 100 | awk '{ print $4 }' | cut -f1 -d'/')
# NIC1 entfernen
virsh -c qemu:///system detach-interface --domain talos_control-plane-node-1 --mac ${MAC_CP} --type network
virsh -c qemu:///system detach-interface --domain talos_worker-node-1 --mac ${MAC_WORKER} --type network
```

## Talos

Jetzt kann die Talos Konfiguration erstellt und angewendet werden

```shell
# Generate talos config
talosctl gen config dev https://[fd00::10]:6443 \
  --output talos \
  --install-disk /dev/vda \
  --config-patch @talos-patches/patch.yaml \
  --config-patch-control-plane @talos-patches/patch-controlplane.yaml
# Apply Config to first Controlplane, including patch for hostname
talosctl -n ${IP_CP} apply-config \
  --insecure \
  --file talos/controlplane.yaml \
  --config-patch @talos-patches/patch-node-cp-1.yaml
talosctl -n ${IP_WORKER} apply-config \
  --insecure \
  --file talos/worker.yaml \
  --config-patch @talos-patches/patch-node-worker-1.yaml
export TALOSCONFIG=$(realpath ./talos/talosconfig)
talosctl config endpoint ${IP_CP}
talosctl -n ${IP_CP} bootstrap
talosctl -n ${IP_CP} kubeconfig ./kubeconfig
export KUBECONFIG=./kubeconfig
```

## Cilium

```shell
helm upgrade --install \
    cilium \
    cilium/cilium \
    --version 1.16.3 \
    --namespace kube-system \
    --values cilium/values.yaml
kubectl apply -f cilium/cilium-lb-l2.yaml
```

## Local path provisioner

```shell
kubectl apply -k local-path-provisioner/
```

## Test Deployment

Nginx Pod mit LoadBalancer Service

```shell
kubectl apply -f test-deployment.yaml
```

## Notizen

Cilium L2 Advertisements für IPv6 funnktionieren nicht
<https://docs.cilium.io/en/latest/network/l2-announcements/#limitations>

```text
The feature currently does not support IPv6/NDP.
```
