## **物理机**

---

### em1 - em1

```text
tcp: 7.02Gb/s
udp:
	client 8.14Gb/s
	server 8.08Gb/s
```

### bridge - bridge

bridge->em1 - em1<-bridge（bridge为3层）

```text
tcp: 6.62Gb/s
udp:
	client 8.19Gb/s
	server 7.94Gb/s
```

## **bridge**

---

### ns - ns（veth）

```text
tcp: 6.24Gb/s
udp:
	client 6.89Gb/s
	server 6.88Gb/s
```

??? note "环境"
	```text
	Host1:
	brctl addbr br1
	ip link set br1 up
	ifconfig em1 0.0.0.0
	brctl addif br1 em1
	ip netns add ns1
	ip link add ns1-veth-in type veth peer name ns1-veth-out
	ip link set ns1-veth-in netns ns1
	ip link set ns1-veth-out up
	ip netns exec ns1 ip link set lo up
	ip netns exec ns1 ifconfig ns1-veth-in 192.168.1.1 netmask 255.255.255.0 up
	brctl addif br1 ns1-veth-out

	Host2:
	brctl addbr br1
	ip link set br1 up
	ifconfig em1 0.0.0.0
	brctl addif br1 em1
	ip netns add ns2
	ip link add ns2-veth-in type veth peer name ns2-veth-out
	ip link set ns2-veth-in netns ns2
	ip link set ns2-veth-out up
	ip netns exec ns2 ip link set lo up
	ip netns exec ns2 ifconfig ns2-veth-in 192.168.1.2 netmask 255.255.255.0 up
	brctl addif br1 ns2-veth-out
	```

### kvm - kvm

```text
tcp: 5.94Gb/s
udp:
	client 6.41Gb/s
	server 4.93Gb/s
```

## **ovs**

---

### ns - ns（veth）

```text
tcp: 6.09Gb/s
udp:
	client 5.94Gb/s
	server 5.60Gb/s
```

??? note "环境"
	```text
	Host1:
	ovs-vsctl add-br ovsbr
	ovs-vsctl add-port ovsbr em1
	ip netns add ns1
	ip link add ns1-veth-in type veth peer name ns1-veth-out
	ip link set ns1-veth-in netns ns1
	ip link set ns1-veth-out up
	ip netns exec ns1 ip link set lo up
	ip netns exec ns1 ifconfig ns1-veth-in 192.168.1.1 netmask 255.255.255.0 up
	ovs-vsctl add-port ovsbr ns1-veth-out

	Host2:
	ovs-vsctl add-br ovsbr
	ovs-vsctl add-port ovsbr em1
	ip netns add ns2
	ip link add ns2-veth-in type veth peer name ns2-veth-out
	ip link set ns2-veth-in netns ns2
	ip link set ns2-veth-out up
	ip netns exec ns2 ip link set lo up
	ip netns exec ns2 ifconfig ns2-veth-in 192.168.1.2 netmask 255.255.255.0 up
	ovs-vsctl add-port ovsbr ns2-veth-out
	```

### ns - ns（ovs-port）

ovs-port(internal)

```text
tcp: 6.28Gb/s
udp:
	client 7.05Gb/s
	server 7.04Gb/s
```

??? note "环境"
	```text
	Host1:
	ovs-vsctl add-br ovsbr
	ovs-vsctl add-port ovsbr em1
	ovs-vsctl add-port ovsbr ns1 -- set interface ns1 type=internal
	ip netns add ns1
	ip link set ns1 netns ns1
	ip netns exec ns1 ip link set lo up
	ip netns exec ns1 ifconfig ns1 192.168.1.1 netmask 255.255.255.0 up

	Host2:
	ovs-vsctl add-br ovsbr
	ovs-vsctl add-port ovsbr em1
	ovs-vsctl add-port ovsbr ns2 -- set interface ns2 type=internal
	ip netns add ns2
	ip link set ns2 netns ns2
	ip netns exec ns2 ip link set lo up
	ip netns exec ns2 ifconfig ns2 192.168.1.2 netmask 255.255.255.0 up
	```

### kvm - kvm

```text
tcp: 5.50Gb/s
udp:
	client 8.50Gb/s
	server 5.90Gb/s
```
