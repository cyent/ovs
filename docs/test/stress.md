## **环境说明**

---

服务器：Dell R730，网卡为板载集成电口万兆（支持udp offload），os为centos 7.3

交换机：Dell万兆交换机

kvm：qemu-kvm 1.5.3，网卡virtio，os为centos 7.3 4核8G

内核参数：物理机、netns、kvm均不改变任何内核参数，并且iptables、ebtables、selinux均处于disable状态

测试方法：使用iperf分别测试tcp和udp（只测试单播），每一个测试完成后均重启物理机

测试命令：

- tcp

	client: iperf -b 10G -t 300 -P 3 -c 目标ip

	server: iperf -s

- udp

	client: iperf -u -b 10G -t 300 -P 3 -c 目标ip

	server: iperf -u -s

测试结果：client为平均每秒出站流量，server为平均每秒入站流量

## **同宿主**

---

### **bridge**

- ns - ns（veth）

	```text
	tcp:
		client 10.7Gb/s
		server 10.7Gb/s
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

- kvm - kvm

	```text
	tcp:
		client 5.57Gb/s
		server 5.57Gb/s
	udp:
		client 5.58Gb/s
		server 5.39Gb/s
	```

### **ovs**

- ns - ns（veth）

	```text
	tcp:
		client 10.7Gb/s
		server 10.7Gb/s
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

- ns - ns（patch-port）

	```text
	tcp:
		client 10.7Gb/s
		server 10.7Gb/s
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

- kvm - kvm

	```text
	tcp:
		client 5.83Gb/s
		server 5.83Gb/s
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

## **不同宿主**

---

### **基准**

- em1 - em1

	```text
	tcp:
		client 7.02Gb/s
		server 7.02Gb/s
	udp:
		client 8.14Gb/s
		server 8.08Gb/s
	```

- bridge->em1 - em1<-bridge（bridge为3层）

	```text
	tcp:
		client 6.62Gb/s
		server 6.62Gb/s
	udp:
		client 8.19Gb/s
		server 7.94Gb/s
	```

### **无封装**

#### **bridge**

- ns - ns（veth）

	```text
	tcp:
		client 6.24Gb/s
		server 6.24Gb/s
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

- kvm - kvm

	```text
	tcp:
		client 5.94Gb/s
		server 5.94Gb/s
	udp:
		client 6.41Gb/s
		server 4.93Gb/s
	```

#### **ovs**

- ns - ns（veth）

	```text
	tcp:
		client 6.09Gb/s
		server 6.09Gb/s
	udp:
		client 5.94Gb/s
		server 5.60Gb/s
	```

	??? note “环境”
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

- ns - ns（patch-port）

	```text
	tcp:
		client 6.28Gb/s
		server 6.28Gb/s
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

- kvm - kvm

	```text
	tcp:
		client 5.50Gb/s
		server 5.50Gb/s
	udp:
		client 8.50Gb/s
		server 5.90Gb/s
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
