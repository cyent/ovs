## **环境说明**

---

服务器：Dell R730，网卡为板载集成电口万兆（支持udp offload），os为centos 7.3

交换机：Dell万兆交换机

kvm：qemu-kvm 1.5.3，网卡virtio，os为centos 7.3 4核8G

内核参数：物理机、netns、kvm均不改变任何内核参数，并且iptables、ebtables、selinux均处于disable状态

测试方法：使用iperf分别测试tcp和udp（只测试单播），每一个测试完成后均重启物理机

测试命令：

- tcp

	服务端: iperf -s

	客户端: iperf -b 10G -t 300 -P 3 -c 目标ip

- udp

	服务端: iperf -u -s

	客户端: iperf -u -b 10G -t 300 -P 3 -c 目标ip

## **同宿主**

---

### **bridge**

- ns - ns（veth）

	```text
	tcp:
		ns1 10.7Gb/s
		ns2 10.7Gb/s
	udp:
		ns1 4.93Gb/s
		ns2 4.93Gb/s
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

- kvm - kvm

	```text
	tcp:
		vm1 5.94Gb/s
		vm2 5.94Gb/s
	udp:
		vm1 6.18Gb/s
		vm2 4.84Gb/s
	```

### **ovs**

- ns - ns（veth）

	```text
	tcp:
		ns1 10.7Gb/s
		ns2 10.7Gb/s
	udp:
		ns1 3.85Gb/s
		ns2 3.85Gb/s
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

- ns - ns（patch-port）

	```text
	tcp:
		ns1 10.7Gb/s
		ns2 10.7Gb/s
	udp:
		ns1 Gb/s
		ns2 Gb/s
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

- kvm - kvm

	```text
	tcp:
		vm1 Gb/s
		vm2 Gb/s
	udp:
		vm1 Gb/s
		vm2 Gb/s
	```

## **不同宿主**

---

### **基准**

- em1 - em1

	```text
	dstat稳定在1179M（2端一样）
	```

- bridge->em1 - em1<-bridge（bridge为3层）

	```text
	dstat稳定在1179M（2端一样）
	```

### **无封装**

#### **bridge**

- ns - ns（veth）

	```text
	tcp: iperf -b 10G -P 3（线程到3，带宽就打到了上限）
	宿主1 1179M（稳定），ns：1127M，ns iperf结果：9.40Gb/s
	宿主2 1179M（稳定），ns：1148M，ns iperf结果：9.39Gb/s
	udp: iperf -u -b 10G -P 2（线程到2，带宽就打到了上限）
	宿主1 1179M（稳定），ns：1626M-1738M，平均为13.8Gb/s
	宿主2 1179M（稳定），ns：927M-1174M，ns iperf结果：9.08Gb/s
	```

- kvm - kvm

	```text
	tcp: iperf -b 10G -P 2（线程到2，再往上提升不明显，有时候还会更差）
	宿主1的vm 796-971（不稳定），iperf结果：7.26Gb/s
	宿主2的vm 806-984（不稳定），iperf结果：7.25Gb/s
	```

#### **ovs**

- ns - ns（veth）

	```text

	```

- ns - ns（patch-port）

	```text

	```

- kvm - kvm

	```text

	```

### **vxlan封装**

#### **bridge**

- ns - ns（veth）

	```text

	```

- kvm - kvm

	```text

	```

#### **ovs**

em1 - - - vtep - ovsbr

- ns - ns（veth）

	```text

	```

- ns - ns（patch-port）

	```text

	```

- kvm - kvm

	```text

	```

#### **ovs-bridge**

em1 - - - vtep - ovsbr - bridge

- ns - ns（veth）

	```text

	```

- ns - ns（patch-port）

	```text

	```

- kvm - kvm

	```text

	```

## **结论**

---

### **性能排行**

### **bridge和ovs性能对比**
