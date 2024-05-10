# Crear interfaces dummy 
```
sudo modprobe dummy
sudo ip link add dummy0 type dummy
sudo ip link add dummy1 type dummy
sudo ip link set dummy0 up
sudo ip link set dummy1 up
```

# Crear el Bond
```
sudo ip link add bond0 type bond
sudo ip link set bond0 type bond mode active-backup miimon 100
sudo ip link set bond0 mtu 9126
sudo ip link set dummy0 down
sudo ip link set dummy1 down
sudo ip link set dummy0 master bond0
sudo ip link set dummy1 master bond0
sudo ip link set dummy0 up
sudo ip link set dummy1 up
sudo ip link set bond0 up
```

# Configurar la VLAN sobre el Bond
```
sudo ip link add link bond0 name bond0.119 type vlan id 119
sudo ip link set bond0.119 up
```

# Configurar la IPvlan sobre la VLAN
```
sudo ip link add name ipvlan1 link bond0.119 type ipvlan mode l2
sudo ip link set ipvlan1 mtu 1500
sudo ip addr add 192.168.119.10/24 dev ipvlan1
sudo ip link set ipvlan1 up
```

```
sudo ip addr show
sudo ip link show type bond
sudo ip link show type vlan
sudo ip link show type ipvlan
```

----------------------

```
sudo ip link set ipvlan1 down
sudo ip link delete ipvlan1
sudo ip link set bond0.119 down
sudo ip link delete bond0.119
sudo ip link set dummy0 nomaster
sudo ip link set dummy1 nomaster
sudo ip link set bond0 down
sudo ip link delete bond0 type bond
sudo ip link set dummy0 down
sudo ip link set dummy1 down
sudo ip link delete dummy0 type dummy
sudo ip link delete dummy1 type dummy
sudo rmmod dummy
sudo rmmod bonding
ip link show
```
