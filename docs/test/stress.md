

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
	tcp:
		client 4.45Gb/s
		server 4.45Gb/s
	udp:
		client 5.36Gb/s
		server 5.36Gb/s
	```

	??? note "环境"
		```text
		Host1:
		ip link set em1 mtu 1550
		brctl addbr br1
		ip link set br1 up
		ip link add vtep-100 mtu 1500 type vxlan id 100 group 239.0.0.100 ttl 64 dev em1 dstport 4789
		ip link set vtep-100 up
		brctl addif br1 vtep-100
		ip netns add ns1
		ip link add ns1-veth-in type veth peer name ns1-veth-out
		ip link set ns1-veth-in netns ns1
		ip link set ns1-veth-out up
		ip netns exec ns1 ip link set lo up
		ip netns exec ns1 ifconfig ns1-veth-in 192.168.1.1 netmask 255.255.255.0 up
		brctl addif br1 ns1-veth-out

		Host2:
		ip link set em1 mtu 1550
		brctl addbr br1
		ip link set br1 up
		ip link add vtep-100 mtu 1500 type vxlan id 100 group 239.0.0.100 ttl 64 dev em1 dstport 4789
		ip link set vtep-100 up
		brctl addif br1 vtep-100
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
		client 7.20Gb/s
		server 7.20Gb/s
	udp:
		client 8.23Gb/s
		server 3.66Gb/s
	```

	??? note "环境"
		```text
		Host1:
		ip link set em1 mtu 1550
		brctl addbr br1
		ip link set br1 up
		ip link add vtep-100 mtu 1500 type vxlan id 100 group 239.0.0.100 ttl 64 dev em1 dstport 4789
		ip link set vtep-100 up
		brctl addif br1 vtep-100

		Host2:
		ip link set em1 mtu 1550
		brctl addbr br1
		ip link set br1 up
		ip link add vtep-100 mtu 1500 type vxlan id 100 group 239.0.0.100 ttl 64 dev em1 dstport 4789
		ip link set vtep-100 up
		brctl addif br1 vtep-100
		```

#### **ovs**

em1 - - - vtep - ovsbr

##### **key指定vni**

由ovs来做vxlan的封装和解封装

- ns - ns（veth）

	```text
	tcp:
		client 4.23Gb/s
		server 4.23Gb/s
	udp:
		client 4.55Gb/s
		server 4.30Gb/s
	```

	??? note "环境"
		```text
		Host1:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:key=100
		ip netns add ns1
		ip link add ns1-veth-in type veth peer name ns1-veth-out
		ip link set ns1-veth-in netns ns1
		ip link set ns1-veth-out up
		ip netns exec ns1 ip link set lo up
		ip netns exec ns1 ifconfig ns1-veth-in 192.168.1.1 netmask 255.255.255.0 up
		ovs-vsctl add-port ovsbr ns1-veth-out

		Host2:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:key=100
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
		client 4.65Gb/s
		server 4.65Gb/s
	udp:
		client 4.80Gb/s
		server 4.50Gb/s
	```

	??? note "环境"
		```text
		Host1:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:key=100
		ovs-vsctl add-port ovsbr ns1 -- set interface ns1 type=internal
		ip netns add ns1
		ip link set ns1 netns ns1
		ip netns exec ns1 ip link set lo up
		ip netns exec ns1 ifconfig ns1 192.168.1.1 netmask 255.255.255.0 up

		Host2:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:key=100
		ovs-vsctl add-port ovsbr ns2 -- set interface ns2 type=internal
		ip netns add ns2
		ip link set ns2 netns ns2
		ip netns exec ns2 ip link set lo up
		ip netns exec ns2 ifconfig ns2 192.168.1.2 netmask 255.255.255.0 up
		```

- kvm - kvm

	```text
	tcp:
		client 8.51Gb/s
		server 8.51Gb/s
	udp:
		client 5.82Gb/s
		server 4.31Gb/s
	```

	??? note "环境"
		```text
		Host1:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:key=100

		Host2:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:key=100
		```

##### **key为flow**

自己写openflow来做vxlan的封装和解封装

- ns - ns（veth）

	```text
	tcp:
		client 4.43Gb/s
		server 4.43Gb/s
	udp:
		client 4.75Gb/s
		server 4.42Gb/s
	```

	??? note "环境"
		```text
		Host1:
		# 设置em1的mtu
		ip link set em1 mtu 1550

		# 创建ovs桥接
		ovs-vsctl add-br ovsbr

		# 清空流表
		ovs-ofctl del-flows ovsbr

		# 创建vtep
		ovs-vsctl add-port ovsbr vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:in_key=flow options:out_key=flow

		# ns
		ip netns add ns1
		ip link add ns1-veth-in type veth peer name ns1-veth-out
		ip link set ns1-veth-in netns ns1
		ip link set ns1-veth-out up
		ip netns exec ns1 ip link set lo up
		ip netns exec ns1 ifconfig ns1-veth-in 192.168.1.1 netmask 255.255.255.0 up
		ovs-vsctl add-port ovsbr ns1-veth-out

		# openflow
		## 获得openflow端口
		ns_ofport=`ovs-vsctl get interface ns1-veth-out ofport`
		vtepHost2_ofport=`ovs-vsctl get interface vtepHost2 ofport`

		## table 0
		### 从ns过来的数据包，扔给table 2处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${ns_ofport}, actions=goto_table:2"

		### 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vtepHost2_ofport}, actions=goto_table:10"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=0, priority=0, actions=drop"

		## table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"

		## 多播包和广播包，扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=2, priority=0, actions=drop"

		## table 10
		### 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到ns端口
		### 具体学习内容:
		###  1. 包的目的mac跟当前的源mac匹配
		###  2. 将tunnel号修改为当前的tunnel号
		###  3. 从当前入口发出
		ovs-ofctl add-flow ovsbr "table=10, priority=1, tun_id=100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${ns_ofport}"

		### 其余DROP（可选）
		ovs-ofctl add-flow ovsbr "table=10, priority=0, actions=drop"

		## table 20
		### 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		### 其余扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=20, priority=0, actions=goto_table:22"

		## table 22
		### 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow ovsbr "table=22, priority=1, in_port=${ns_ofport}, actions=set_tunnel:100,output:${vtepHost2_ofport}"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=22, priority=0, actions=drop"


		Host2:
		# 设置em1的mtu
		ip link set em1 mtu 1550

		# 创建ovs桥接
		ovs-vsctl add-br ovsbr

		# 清空流表
		ovs-ofctl del-flows ovsbr

		# 创建vtep
		ovs-vsctl add-port ovsbr vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:in_key=flow options:out_key=flow

		# ns
		ip netns add ns2
		ip link add ns2-veth-in type veth peer name ns2-veth-out
		ip link set ns2-veth-in netns ns2
		ip link set ns2-veth-out up
		ip netns exec ns2 ip link set lo up
		ip netns exec ns2 ifconfig ns2-veth-in 192.168.1.2 netmask 255.255.255.0 up
		ovs-vsctl add-port ovsbr ns2-veth-out

		# openflow
		## 获得openflow端口
		ns_ofport=`ovs-vsctl get interface ns2-veth-out ofport`
		vtepHost1_ofport=`ovs-vsctl get interface vtepHost1 ofport`

		## table 0
		### 从ns过来的数据包，扔给table 2处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${ns_ofport}, actions=goto_table:2"

		### 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vtepHost1_ofport}, actions=goto_table:10"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=0, priority=0, actions=drop"

		## table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"

		## 多播包和广播包，扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=2, priority=0, actions=drop"

		## table 10
		### 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到ns端口
		### 具体学习内容:
		###  1. 包的目的mac跟当前的源mac匹配
		###  2. 将tunnel号修改为当前的tunnel号
		###  3. 从当前入口发出
		ovs-ofctl add-flow ovsbr "table=10, priority=1, tun_id=100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${ns_ofport}"

		### 其余DROP（可选）
		ovs-ofctl add-flow ovsbr "table=10, priority=0, actions=drop"

		## table 20
		### 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		### 其余扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=20, priority=0, actions=goto_table:22"

		## table 22
		### 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow ovsbr "table=22, priority=1, in_port=${ns_ofport}, actions=set_tunnel:100,output:${vtepHost1_ofport}"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=22, priority=0, actions=drop"
		```

- ns - ns（patch-port）

	```text
	tcp:
		client 4.28Gb/s
		server 4.28Gb/s
	udp:
		client 5.41Gb/s
		server 4.92Gb/s
	```

	??? note "环境"
		```text
		Host1:
		# 设置em1的mtu
		ip link set em1 mtu 1550

		# 创建ovs桥接
		ovs-vsctl add-br ovsbr

		# 清空流表
		ovs-ofctl del-flows ovsbr

		# 创建vtep
		ovs-vsctl add-port ovsbr vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:in_key=flow options:out_key=flow

		# ns到ovsbr的patch-port
		ovs-vsctl add-port ovsbr ns1 -- set interface ns1 type=internal
		ip link set ns1 up
		ip netns add ns1
		ip link set ns1 netns ns1
		ip netns exec ns1 ip link set lo up
		ip netns exec ns1 ifconfig ns1 192.168.1.1 netmask 255.255.255.0 up

		# openflow
		## 获得openflow端口
		ns_ofport=`ovs-vsctl get interface ns1 ofport`
		vtepHost2_ofport=`ovs-vsctl get interface vtepHost2 ofport`

		## table 0
		### 从ns过来的数据包，扔给table 2处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${ns_ofport}, actions=goto_table:2"

		### 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vtepHost2_ofport}, actions=goto_table:10"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=0, priority=0, actions=drop"

		## table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"

		## 多播包和广播包，扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=2, priority=0, actions=drop"

		## table 10
		### 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到ns端口
		### 具体学习内容:
		###  1. 包的目的mac跟当前的源mac匹配
		###  2. 将tunnel号修改为当前的tunnel号
		###  3. 从当前入口发出
		ovs-ofctl add-flow ovsbr "table=10, priority=1, tun_id=100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${ns_ofport}"

		### 其余DROP（可选）
		ovs-ofctl add-flow ovsbr "table=10, priority=0, actions=drop"

		## table 20
		### 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		### 其余扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=20, priority=0, actions=goto_table:22"

		## table 22
		### 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow ovsbr "table=22, priority=1, in_port=${ns_ofport}, actions=set_tunnel:100,output:${vtepHost2_ofport}"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=22, priority=0, actions=drop"


		Host2:
		# 设置em1的mtu
		ip link set em1 mtu 1550

		# 创建ovs桥接
		ovs-vsctl add-br ovsbr

		# 清空流表
		ovs-ofctl del-flows ovsbr

		# 创建vtep
		ovs-vsctl add-port ovsbr vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:in_key=flow options:out_key=flow

		# ns到ovsbr的patch-port
		ovs-vsctl add-port ovsbr ns2 -- set interface ns2 type=internal
		ip link set ns2 up
		ip netns add ns2
		ip link set ns2 netns ns2
		ip netns exec ns2 ip link set lo up
		ip netns exec ns2 ifconfig ns2 192.168.1.2 netmask 255.255.255.0 up

		# openflow
		## 获得openflow端口
		ns_ofport=`ovs-vsctl get interface ns2 ofport`
		vtepHost1_ofport=`ovs-vsctl get interface vtepHost1 ofport`

		## table 0
		### 从ns过来的数据包，扔给table 2处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${ns_ofport}, actions=goto_table:2"

		### 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vtepHost1_ofport}, actions=goto_table:10"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=0, priority=0, actions=drop"

		## table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"

		## 多播包和广播包，扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=2, priority=0, actions=drop"

		## table 10
		### 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到ns端口
		### 具体学习内容:
		###  1. 包的目的mac跟当前的源mac匹配
		###  2. 将tunnel号修改为当前的tunnel号
		###  3. 从当前入口发出
		ovs-ofctl add-flow ovsbr "table=10, priority=1, tun_id=100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${ns_ofport}"

		### 其余DROP（可选）
		ovs-ofctl add-flow ovsbr "table=10, priority=0, actions=drop"

		## table 20
		### 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		### 其余扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=20, priority=0, actions=goto_table:22"

		## table 22
		### 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow ovsbr "table=22, priority=1, in_port=${ns_ofport}, actions=set_tunnel:100,output:${vtepHost1_ofport}"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=22, priority=0, actions=drop"
		```

- kvm - kvm

	```text
	tcp:
		client 8.77Gb/s
		server 8.77Gb/s
	udp:
		client 5.43Gb/s
		server 4.52Gb/s
	```

	??? note "环境"
		```text
		Host1:
		# 设置em1的mtu
		ip link set em1 mtu 1550

		# 创建ovs桥接
		ovs-vsctl add-br ovsbr

		# 清空流表
		ovs-ofctl del-flows ovsbr

		# 创建vtep
		ovs-vsctl add-port ovsbr vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:in_key=flow options:out_key=flow


		Host2:
		# 设置em1的mtu
		ip link set em1 mtu 1550

		# 创建ovs桥接
		ovs-vsctl add-br ovsbr

		# 清空流表
		ovs-ofctl del-flows ovsbr

		# 创建vtep
		ovs-vsctl add-port ovsbr vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:in_key=flow options:out_key=flow
		```

		启动虚拟机vm1和vm3

		```text
		Host1:
		# openflow
		## 获得openflow端口
		vm_ofport=`ovs-vsctl get interface vmeth0-vm1 ofport`
		vtepHost2_ofport=`ovs-vsctl get interface vtepHost2 ofport`

		## table 0
		### 从vm过来的数据包，扔给table 2处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vm_ofport}, actions=goto_table:2"

		### 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vtepHost2_ofport}, actions=goto_table:10"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=0, priority=0, actions=drop"

		## table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"

		## 多播包和广播包，扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=2, priority=0, actions=drop"

		## table 10
		### 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到vm端口
		### 具体学习内容:
		###  1. 包的目的mac跟当前的源mac匹配
		###  2. 将tunnel号修改为当前的tunnel号
		###  3. 从当前入口发出
		ovs-ofctl add-flow ovsbr "table=10, priority=1, tun_id=100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${vm_ofport}"

		### 其余DROP（可选）
		ovs-ofctl add-flow ovsbr "table=10, priority=0, actions=drop"

		## table 20
		### 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		### 其余扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=20, priority=0, actions=goto_table:22"

		## table 22
		### 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow ovsbr "table=22, priority=1, in_port=${vm_ofport}, actions=set_tunnel:100,output:${vtepHost2_ofport}"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=22, priority=0, actions=drop"


		Host2:
		# openflow
		## 获得openflow端口
		vm_ofport=`ovs-vsctl get interface vmeth0-vm3 ofport`
		vtepHost1_ofport=`ovs-vsctl get interface vtepHost1 ofport`

		## table 0
		### 从vm过来的数据包，扔给table 2处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vm_ofport}, actions=goto_table:2"

		### 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vtepHost1_ofport}, actions=goto_table:10"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=0, priority=0, actions=drop"

		## table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"

		## 多播包和广播包，扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=2, priority=0, actions=drop"

		## table 10
		### 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到vm端口
		### 具体学习内容:
		###  1. 包的目的mac跟当前的源mac匹配
		###  2. 将tunnel号修改为当前的tunnel号
		###  3. 从当前入口发出
		ovs-ofctl add-flow ovsbr "table=10, priority=1, tun_id=100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${vm_ofport}"

		### 其余DROP（可选）
		ovs-ofctl add-flow ovsbr "table=10, priority=0, actions=drop"

		## table 20
		### 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		### 其余扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=20, priority=0, actions=goto_table:22"

		## table 22
		### 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow ovsbr "table=22, priority=1, in_port=${vm_ofport}, actions=set_tunnel:100,output:${vtepHost1_ofport}"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=22, priority=0, actions=drop"
		```

#### **ovs-bridge**

```text
em1 - - - vtep - ovsbr - bridge
```


##### **key指定vni**


- ns - ns（ovsbr和bridge之间用veth）：无意义，因为veth不仅管理麻烦，而且性能不如patch-port

- ns - ns（ovsbr和bridge之间用patch-port）

	```text
	tcp:
		client 4.03Gb/s
		server 4.03Gb/s
	udp:
		client 4.19Gb/s
		server 3.82Gb/s
	```

	??? note "环境"
		```text
		Host1:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:key=100
		brctl addbr br1
		ip link set br1 up
		ovs-vsctl add-port ovsbr patch-br -- set interface patch-br type=internal
		ip link set patch-br up
		brctl addif br1 patch-br
		ip netns add ns1
		ip link add ns1-veth-in type veth peer name ns1-veth-out
		ip link set ns1-veth-in netns ns1
		ip link set ns1-veth-out up
		ip netns exec ns1 ip link set lo up
		ip netns exec ns1 ifconfig ns1-veth-in 192.168.1.1 netmask 255.255.255.0 up
		brctl addif br1 ns1-veth-out

		Host2:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:key=100
		brctl addbr br1
		ip link set br1 up
		ovs-vsctl add-port ovsbr patch-br -- set interface patch-br type=internal
		ip link set patch-br up
		brctl addif br1 patch-br
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
		client 8.44Gb/s
		server 8.44Gb/s
	udp:
		client 5.46Gb/s
		server 3.85Gb/s
	```

	??? note "环境"
		```text
		Host1:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:key=100
		brctl addbr br1
		ip link set br1 up
		ovs-vsctl add-port ovsbr patch-br -- set interface patch-br type=internal
		ip link set patch-br up
		brctl addif br1 patch-br

		Host2:
		ip link set em1 mtu 1550
		ovs-vsctl add-br ovsbr
		ovs-vsctl add-port ovsbr vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:key=100
		brctl addbr br1
		ip link set br1 up
		ovs-vsctl add-port ovsbr patch-br -- set interface patch-br type=internal
		ip link set patch-br up
		brctl addif br1 patch-br
		```


##### **key为flow**

- ns - ns（ovsbr和bridge之间用veth）：无意义，因为veth不仅管理麻烦，而且性能不如patch-port

- ns - ns（ovsbr和bridge之间用patch-port）

	```text
	tcp:
		client 4.25Gb/s
		server 4.25Gb/s
	udp:
		client 4.29Gb/s
		server 2.88Gb/s
	```

	??? note "环境"
		```text
		Host1:
		# 设置em1的mtu
		ip link set em1 mtu 1550

		# 创建ovs桥接
		ovs-vsctl add-br ovsbr

		# 清空流表
		ovs-ofctl del-flows ovsbr

		# 创建vtep
		ovs-vsctl add-port ovsbr vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:in_key=flow options:out_key=flow

		# 创建linux bridge
		brctl addbr br1
		ip link set br1 up

		# linux bridge到ovsbr的patch-port
		ovs-vsctl add-port ovsbr patch-br -- set interface patch-br type=internal
		ip link set patch-br up
		brctl addif br1 patch-br

		# openflow
		## 获得openflow端口
		patch_ofport=`ovs-vsctl get interface patch-br ofport`
		vtepHost2_ofport=`ovs-vsctl get interface vtepHost2 ofport`

		## table 0
		### 从linux-bridge过来的数据包，扔给table 2处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${patch_ofport}, actions=goto_table:2"

		### 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vtepHost2_ofport}, actions=goto_table:10"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=0, priority=0, actions=drop"

		## table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"

		## 多播包和广播包，扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=2, priority=0, actions=drop"

		## table 10
		### 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到linux-bridge端口
		### 具体学习内容:
		###  1. 包的目的mac跟当前的源mac匹配
		###  2. 将tunnel号修改为当前的tunnel号
		###  3. 从当前入口发出
		ovs-ofctl add-flow ovsbr "table=10, priority=1, tun_id=100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${patch_ofport}"

		### 其余DROP（可选）
		ovs-ofctl add-flow ovsbr "table=10, priority=0, actions=drop"

		## table 20
		### 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		### 其余扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=20, priority=0, actions=goto_table:22"

		## table 22
		### 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow ovsbr "table=22, priority=1, in_port=${patch_ofport}, actions=set_tunnel:100,output:${vtepHost2_ofport}"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=22, priority=0, actions=drop"

		# ns
		ip netns add ns1
		ip link add ns1-veth-in type veth peer name ns1-veth-out
		ip link set ns1-veth-in netns ns1
		ip link set ns1-veth-out up
		ip netns exec ns1 ip link set lo up
		ip netns exec ns1 ifconfig ns1-veth-in 192.168.1.1 netmask 255.255.255.0 up
		brctl addif br1 ns1-veth-out


		Host2:
		# 设置em1的mtu
		ip link set em1 mtu 1550

		# 创建ovs桥接
		ovs-vsctl add-br ovsbr

		# 清空流表
		ovs-ofctl del-flows ovsbr

		# 创建vtep
		ovs-vsctl add-port ovsbr vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:in_key=flow options:out_key=flow

		# 创建linux bridge
		brctl addbr br1
		ip link set br1 up

		# linux bridge到ovsbr的patch-port
		ovs-vsctl add-port ovsbr patch-br -- set interface patch-br type=internal
		ip link set patch-br up
		brctl addif br1 patch-br

		# openflow
		## 获得openflow端口
		patch_ofport=`ovs-vsctl get interface patch-br ofport`
		vtepHost1_ofport=`ovs-vsctl get interface vtepHost1 ofport`

		## table 0
		### 从linux-bridge过来的数据包，扔给table 2处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${patch_ofport}, actions=goto_table:2"

		### 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow ovsbr "table=0, priority=1, in_port=${vtepHost1_ofport}, actions=goto_table:10"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=0, priority=0, actions=drop"

		## table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"

		## 多播包和广播包，扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=2, priority=0, actions=drop"

		## table 10
		### 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到linux-bridge端口
		### 具体学习内容:
		###  1. 包的目的mac跟当前的源mac匹配
		###  2. 将tunnel号修改为当前的tunnel号
		###  3. 从当前入口发出
		ovs-ofctl add-flow ovsbr "table=10, priority=1, tun_id=100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${patch_ofport}"

		### 其余DROP（可选）
		ovs-ofctl add-flow ovsbr "table=10, priority=0, actions=drop"

		## table 20
		### 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		### 其余扔给table 22处理
		ovs-ofctl add-flow ovsbr "table=20, priority=0, actions=goto_table:22"

		## table 22
		### 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow ovsbr "table=22, priority=1, in_port=${patch_ofport}, actions=set_tunnel:100,output:${vtepHost1_ofport}"

		### 其余DROP
		ovs-ofctl add-flow ovsbr "table=22, priority=0, actions=drop"

		# ns
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
		client 6.89Gb/s
		server 6.89Gb/s
	udp:
		client 4.72Gb/s
		server 4.55Gb/s
	```

	??? note "环境"
		和上面这个一样，只是不用# ns部分

### **vm -> bridge --veth--> br-int --patch--> br-tun -> vtep也可以测试下性能**

即openstack的性能测试

kvm - kvm

```text
tcp:
	client 8.52Gb/s
	server 8.52Gb/s
udp:
	client 4.48Gb/s
	server 3.96Gb/s
```

?? note "环境"
	```text
	Host1:
	# 设置em1的mtu
	ip link set em1 mtu 1550

	# bridge
	brctl addbr br1
	ip link set br1 up

	# veth
	ip link add veth-b type veth peer name veth-o
	ip link set veth-b up
	ip link set veth-o up
	brctl addif br1 veth-b

	# br-int
	ovs-vsctl add-br br-int
	ovs-vsctl add-port br-int veth-o
	ovs-vsctl set port veth-o tag=1

	# patch
	ovs-vsctl add-port br-int patch-tun -- set interface patch-tun type=patch options:peer=patch-int

	# br-tun
	ovs-vsctl add-br br-tun
	ovs-ofctl del-flows br-tun
	ovs-vsctl add-port br-tun patch-int -- set interface patch-int type=patch options:peer=patch-tun

	# vtep
	ovs-vsctl add-port br-tun vtepHost2 -- set interface vtepHost2 type=vxlan options:remote_ip=10.246.247.90 options:ttl=64 options:in_key=flow options:out_key=flow

	# openflow
	## 获得openflow端口
	patch_ofport=`ovs-vsctl get interface patch-int ofport`
	vtepHost2_ofport=`ovs-vsctl get interface vtepHost2 ofport`

	## table 0
	ovs-ofctl add-flow br-tun "table=0, priority=1, in_port=${patch_ofport}, actions=goto_table:2"
	ovs-ofctl add-flow br-tun "table=0, priority=1, in_port=${vtepHost2_ofport}, actions=goto_table:4"

	## table 2
	ovs-ofctl add-flow br-tun "table=2, priority=0, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"
	ovs-ofctl add-flow br-tun "table=2, priority=0, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

	## table 4
	ovs-ofctl add-flow br-tun "table=4, priority=1, tun_id=0x3e9, actions=mod_vlan_vid:1, goto_table:10"
	ovs-ofctl add-flow br-tun "table=4, priority=0, actions=drop"

	## table 10
	ovs-ofctl add-flow br-tun "table=10, priority=1, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${patch_ofport}"

	## table 20
	ovs-ofctl add-flow br-tun "table=20, priority=0, actions=goto_table:22"

	## table 22
	ovs-ofctl add-flow br-tun "table=22, priority=1, dl_vlan=1, actions=strip_vlan,set_tunnel:0x3e9,output:${vtepHost2_ofport}"


	Host2:
	# 设置em1的mtu
	ip link set em1 mtu 1550

	# bridge
	brctl addbr br1
	ip link set br1 up

	# veth
	ip link add veth-b type veth peer name veth-o
	ip link set veth-b up
	ip link set veth-o up
	brctl addif br1 veth-b

	# br-int
	ovs-vsctl add-br br-int
	ovs-vsctl add-port br-int veth-o
	ovs-vsctl set port veth-o tag=1

	# patch
	ovs-vsctl add-port br-int patch-tun -- set interface patch-tun type=patch options:peer=patch-int

	# br-tun
	ovs-vsctl add-br br-tun
	ovs-ofctl del-flows br-tun
	ovs-vsctl add-port br-tun patch-int -- set interface patch-int type=patch options:peer=patch-tun

	# vtep
	ovs-vsctl add-port br-tun vtepHost1 -- set interface vtepHost1 type=vxlan options:remote_ip=10.246.247.89 options:ttl=64 options:in_key=flow options:out_key=flow

	# openflow
	## 获得openflow端口
	patch_ofport=`ovs-vsctl get interface patch-int ofport`
	vtepHost1_ofport=`ovs-vsctl get interface vtepHost1 ofport`

	## table 0
	ovs-ofctl add-flow br-tun "table=0, priority=1, in_port=${patch_ofport}, actions=goto_table:2"
	ovs-ofctl add-flow br-tun "table=0, priority=1, in_port=${vtepHost1_ofport}, actions=goto_table:4"

	## table 2
	ovs-ofctl add-flow br-tun "table=2, priority=0, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:20"
	ovs-ofctl add-flow br-tun "table=2, priority=0, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=goto_table:22"

	## table 4
	ovs-ofctl add-flow br-tun "table=4, priority=1, tun_id=0x3e9, actions=mod_vlan_vid:1, goto_table:10"
	ovs-ofctl add-flow br-tun "table=4, priority=0, actions=drop"

	## table 10
	ovs-ofctl add-flow br-tun "table=10, priority=1, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${patch_ofport}"

	## table 20
	ovs-ofctl add-flow br-tun "table=20, priority=0, actions=goto_table:22"

	## table 22
	ovs-ofctl add-flow br-tun "table=22, priority=1, dl_vlan=1, actions=strip_vlan,set_tunnel:0x3e9,output:${vtepHost1_ofport}"
	```

### **由于ovs强制加载conntrack模块，因此需要连续打压1天来测试性能是否有衰减，然后conntrack参数调优下再做一次测试**

这个需要大量ip和端口才能测出来，只能等到上线了才知道，环境可以一边ns一边kvm

## **结论**

---

### **性能排行**

### **bridge和ovs性能对比**
