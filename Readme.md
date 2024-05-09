# deploy cluster
```
kcli create cluster openshift --pf ./sno.yml
kcli scale kube openshift -P workers=2 --paramfile sno.yaml sno
$ oc get nodes
NAME                               STATUS     ROLES                         AGE     VERSION
sno-ctlplane-0.5g-deployment.lab   Ready      control-plane,master,worker   4h5m    v1.27.12+7bee54d
sno-worker-0.5g-deployment.lab     Ready      worker                        3h46m   v1.27.12+7bee54d
sno-worker-1.5g-deployment.lab     Ready      worker                        3h46m   v1.27.12+7bee54d

```

# configure ovs and bond
```
sudo dnf install openvswitch libibverbs
sudo systemctl start openvswitch
sudo systemctl status openvswitch

sudo virsh net-define vlan-net.xml
sudo virsh net-start vlan119
sudo virsh net-autostart vlan119
sudo ovs-vsctl add-br virbr119
sudo ovs-vsctl set port virbr119 tag=119
ovs-vsctl add-br virbr119
sudo kcli update vm -P nets=[5gdeploymentlab,vlan119,vlan119] sno-worker-{0..1}
sudo ovs-vsctl show
85a3c733-7838-48ee-9f7c-c72eeaaeba9b
    Bridge "virbr119"
        Port "vnet71"
            tag: 119
            Interface "vnet71"
        Port "vnet75"
            tag: 119
            Interface "vnet75"
        Port "vnet74"
            tag: 119
            Interface "vnet74"
        Port "virbr119"
            tag: 119
            Interface "virbr119"
                type: internal
        Port "vnet72"
            tag: 119
            Interface "vnet72"
    ovs_version: "2.11.8"

$ cat ens8.yaml 
[connection]
id=tenant1-1
interface-name=ens8
master=15a54ad9-d8a8-5859-9902-fda097cc3b54
slave-type=bond
autoconnect=true
type=802-3-ethernet
autoconnect-priority=1

[ethernet]
auto-negotiate=true
mtu=9126

$ cat ens9.yaml
[connection]
id=tenant1-2
interface-name=ens9
master=15a54ad9-d8a8-5859-9902-fda097cc3b54
slave-type=bond
autoconnect=true
type=802-3-ethernet
autoconnect-priority=1

[ethernet]
auto-negotiate=true
mtu=9126

$ cat bond0.yaml 
[connection]
autoconnect=true
autoconnect-slaves=1
id=tenant-bond1
interface-name=tenant-bond1
uuid=15a54ad9-d8a8-5859-9902-fda097cc3b54
type=bond
autoconnect-priority=1

[bond]
miimon=100
mode=active-backup

[ipv4]
method=disabled

[ipv6]
method=disabled

[ethernet]
mtu=9126

$ cat mc-bond.yaml 
---
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 11-worker-bond0
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
      - contents:
          source: data:;base64,W2Nvbm5lY3Rpb25dCmF1dG9jb25uZWN0PXRydWUKYXV0b2Nvbm5lY3Qtc2xhdmVzPTEKaWQ9dGVuYW50LWJvbmQxCmludGVyZmFjZS1uYW1lPXRlbmFudC1ib25kMQp1dWlkPTE1YTU0YWQ5LWQ4YTgtNTg1OS05OTAyLWZkYTA5N2NjM2I1NAp0eXBlPWJvbmQKYXV0b2Nvbm5lY3QtcHJpb3JpdHk9MQoKW2JvbmRdCm1paW1vbj0xMDAKbW9kZT1hY3RpdmUtYmFja3VwCgpbaXB2NF0KbWV0aG9kPWRpc2FibGVkCgpbaXB2Nl0KbWV0aG9kPWRpc2FibGVkCgpbZXRoZXJuZXRdCm10dT05MTI2Cg==
        path: /etc/NetworkManager/system-connections/bond0.nmconnection
        filesystem: root
        mode: 0600
      - contents:
          source: data:;base64,W2Nvbm5lY3Rpb25dCmlkPXRlbmFudDEtMQppbnRlcmZhY2UtbmFtZT1lbnM4Cm1hc3Rlcj0xNWE1NGFkOS1kOGE4LTU4NTktOTkwMi1mZGEwOTdjYzNiNTQKc2xhdmUtdHlwZT1ib25kCmF1dG9jb25uZWN0PXRydWUKdHlwZT04MDItMy1ldGhlcm5ldAphdXRvY29ubmVjdC1wcmlvcml0eT0xCgpbZXRoZXJuZXRdCmF1dG8tbmVnb3RpYXRlPXRydWUKbXR1PTkxMjYK
        path: /etc/NetworkManager/system-connections/ens8.nmconnection
        filesystem: root
        mode: 0600
      - contents:
          source: data:;base64,W2Nvbm5lY3Rpb25dCmlkPXRlbmFudDEtMgppbnRlcmZhY2UtbmFtZT1lbnM5Cm1hc3Rlcj0xNWE1NGFkOS1kOGE4LTU4NTktOTkwMi1mZGEwOTdjYzNiNTQKc2xhdmUtdHlwZT1ib25kCmF1dG9jb25uZWN0PXRydWUKdHlwZT04MDItMy1ldGhlcm5ldAphdXRvY29ubmVjdC1wcmlvcml0eT0xCgpbZXRoZXJuZXRdCmF1dG8tbmVnb3RpYXRlPXRydWUKbXR1PTkxMjYK
        path: /etc/NetworkManager/system-connections/ens9.nmconnection
        filesystem: root
        mode: 0600

oc create -f mc-bond.yaml

ip a | grep tenant
3: ens8: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 9126 qdisc fq_codel master tenant-bond1 state UP group default qlen 1000
4: ens9: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 9126 qdisc fq_codel master tenant-bond1 state UP group default qlen 1000
9: tenant-bond1: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 9126 qdisc noqueue state UP group default qlen 1000
10: tenant-vlan.119@tenant-bond1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9126 qdisc noqueue state UP group default qlen 1000

```
# install nmstate operator
```
oc create -f nmstate.yaml
```

# install nncp
```
oc patch scheduler cluster --type merge -p '{"spec":{"mastersSchedulable":false}}'  / https://access.redhat.com/solutions/4564851
oc create -f nncp.yaml 
oc get nnce;oc get nncp
NAME                                                    STATUS      STATUS AGE   REASON
sno-worker-0.5g-deployment.lab.tenant-multus-vlan-119   Available   4m21s        SuccessfullyConfigured
sno-worker-1.5g-deployment.lab.tenant-multus-vlan-119   Available   2m12s        SuccessfullyConfigured
NAME                     STATUS      REASON
tenant-multus-vlan-119   Available   SuccessfullyConfigured
```
# deploy application
```
oc create -f nad.yaml
oc get network-attachment-definitions.k8s.cni.cncf.io
NAME       AGE
ctrlpub0   40s

oc create -f deployment.yaml
oc get pod
NAME                                    READY   STATUS    RESTARTS   AGE
samplepod-deployment-5c84c45b75-v8mds   1/1     Running   1          8m34s
```

# how to reproduce
```
oc create -f machineconfig_dummy.yaml
oc exec samplepod-deployment-5c84c45b75-v8mds  -- ip a s | grep pub | grep mtu
3: ctrlpub0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 9126 qdisc noqueue state UNKNOWN 
```
after restart the pod
```
oc delete pod samplepod-deployment-5c84c45b75-v8mds
$ oc exec samplepod-deployment-5c84c45b75-lpkgc -- ip a s | grep pub | grep mtu
3: ctrlpub0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UNKNOWN

```
