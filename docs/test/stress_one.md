## **bridge**

---

### ns - ns（veth）

```text
tcp: 10.7Gb/s
udp:
	client 4.93Gb/s
	server 4.93Gb/s
```

??? note "环境"
	```text
	brctl addbr br1
	ip link set br1 up
	ip netns add ns1
	ip netns add ns2
	ip link add ns1-veth-in type veth peer name ns1-veth-out
	ip link add ns2-veth-in type veth peer name ns2-veth-out
	ip link set ns1-veth-in netns ns1
	ip link set ns2-veth-in netns ns2
	ip link set ns1-veth-out up
	ip link set ns2-veth-out up
	ip netns exec ns1 ip link set lo up
	ip netns exec ns2 ip link set lo up
	ip netns exec ns1 ifconfig ns1-veth-in 192.168.1.1 netmask 255.255.255.0 up
	ip netns exec ns2 ifconfig ns2-veth-in 192.168.1.2 netmask 255.255.255.0 up
	brctl addif br1 ns1-veth-out
	brctl addif br1 ns2-veth-out
	```

### kvm - kvm

```text
tcp: 5.57Gb/s
udp:
	client 5.58Gb/s
	server 5.39Gb/s
```

## **ovs**

---

### ns - ns（veth）

```text
tcp: 10.7Gb/s
udp:
	client 3.85Gb/s
	server 3.85Gb/s
```

??? note "环境"
	```text
	ovs-vsctl add-br ovsbr
	ip netns add ns1
	ip netns add ns2
	ip link add ns1-veth-in type veth peer name ns1-veth-out
	ip link add ns2-veth-in type veth peer name ns2-veth-out
	ip link set ns1-veth-in netns ns1
	ip link set ns2-veth-in netns ns2
	ip link set ns1-veth-out up
	ip link set ns2-veth-out up
	ip netns exec ns1 ip link set lo up
	ip netns exec ns2 ip link set lo up
	ip netns exec ns1 ifconfig ns1-veth-in 192.168.1.1 netmask 255.255.255.0 up
	ip netns exec ns2 ifconfig ns2-veth-in 192.168.1.2 netmask 255.255.255.0 up
	ovs-vsctl add-port ovsbr ns1-veth-out
	ovs-vsctl add-port ovsbr ns2-veth-out
	```

### ns - ns（ovs-port）

ovs-port(internal)

```text
tcp: 10.7Gb/s
udp:
	client 3.91Gb/s
	server 3.91Gb/s
```

??? note "环境"
	```text
	ovs-vsctl add-br ovsbr
	ovs-vsctl add-port ovsbr ns1 -- set interface ns1 type=internal
	ovs-vsctl add-port ovsbr ns2 -- set interface ns2 type=internal
	ip netns add ns1
	ip netns add ns2
	ip link set ns1 netns ns1
	ip link set ns2 netns ns2
	ip netns exec ns1 ip link set lo up
	ip netns exec ns2 ip link set lo up
	ip netns exec ns1 ifconfig ns1 192.168.1.1 netmask 255.255.255.0 up
	ip netns exec ns2 ifconfig ns2 192.168.1.2 netmask 255.255.255.0 up
	```

### kvm - kvm

```text
tcp: 5.83Gb/s
udp:
	client 5.33Gb/s
	server 5.18Gb/s
```

??? note "xml配置"
	```xml hl_lines="4"
	<interface type='bridge'>
	  <mac address='52:54:00:c8:3b:ea'/>
	  <source bridge='ovsbr'/>
	  <virtualport type='openvswitch'/>
	  <target dev='vmeth0-vm1'/>
	  <model type='virtio'/>
	</interface>
	```
