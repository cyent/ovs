## **架构图**

---

物理图

![](/img/test1_phy.png)

逻辑图

![](/img/test1_logic.png)

流表图

![](/img/test1_openflow.png)

## **Node 1**

---

1. ovs.sh

	??? note "点击打开"
		```shell
		# 删除ovs桥接
		ovs-vsctl del-br br-int

		# 创建ovs桥接
		ovs-vsctl add-br br-int

		# 清空流表
		ovs-ofctl del-flows br-int

		# 创建vtep
		ovs-vsctl add-port br-int vtep137 -- set interface vtep137 type=vxlan options:remote_ip=172.16.1.137 options:ttl=64 options:in_key=flow options:out_key=flow
		ovs-vsctl add-port br-int vtep138 -- set interface vtep138 type=vxlan options:remote_ip=172.16.1.138 options:ttl=64 options:in_key=flow options:out_key=flow
		```

2. bridge.sh

	??? note "点击打开"
		```shell
		brctl delbr biv-1001100
		brctl delbr biv-1234567

		brctl addbr biv-1001100
		ip link set biv-1001100 up

		brctl addbr biv-1234567
		ip link set biv-1234567 up
		```

3. br-to-ovs.sh

	??? note "点击打开"
		```shell
		ip link delete vib-1001100
		ip link delete vib-1234567

		ip link add vib-1001100 type veth peer name vio-1001100
		brctl addif biv-1001100 vib-1001100
		ip link set vib-1001100 up
		ip link set vio-1001100 up
		ovs-vsctl add-port br-int vio-1001100

		ip link add vib-1234567 type veth peer name vio-1234567
		brctl addif biv-1234567 vib-1234567
		ip link set vib-1234567 up
		ip link set vio-1234567 up
		ovs-vsctl add-port br-int vio-1234567
		```

4. openflow.sh

	??? note "点击打开"
		```shell
		# 写openflow规则顺序：
		# 1. 从表0开始
		# 2. 为每个表之间留一些表，用于今后扩展
		# 3. 跳表时候只往高了跳，不往低了跳
		# 4. priority=1的写规则，可存在多条；priotity=0的只能有一条，用于其余数据包处理

		# 获得openflow端口
		vio1001100_ofport=`ovs-vsctl get interface vio-1001100 ofport`
		vio1234567_ofport=`ovs-vsctl get interface vio-1234567 ofport`
		vtep137_ofport=`ovs-vsctl get interface vtep137 ofport`
		vtep138_ofport=`ovs-vsctl get interface vtep138 ofport`


		# table 0
		## 从linux-bridge过来的数据包，扔给table 2处理
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vio1001100_ofport}, actions=goto_table:2"
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vio1234567_ofport}, actions=goto_table:2"

		## 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vtep137_ofport} actions=goto_table:10"
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vtep138_ofport} actions=goto_table:10"

		## 其余DROP
		ovs-ofctl add-flow br-int "table=0, priority=0, actions=drop"


		# table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow br-int "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=goto_table:20"

		## 多播和广播包，扔给table 22处理
		ovs-ofctl add-flow br-int "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=goto_table:22"

		## 其余DROP
		ovs-ofctl add-flow br-int "table=2, priority=0, actions=drop"


		# table 10
		## 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到linux-bridge端口
		## 具体学习内容:
		##   1. 包的目的mac跟当前的源mac匹配
		##   2. 将tunnel号修改为当前的tunnel号
		##   3. 从当前入口发出
		ovs-ofctl add-flow br-int "table=10, priority=1, tun_id=1001100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${vio1001100_ofport}"
		ovs-ofctl add-flow br-int "table=10, priority=1, tun_id=1234567, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${vio1234567_ofport}"

		## 其余DROP（可选）
		ovs-ofctl add-flow br-int "table=10, priority=0, actions=drop"


		# table 20
		## 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		## 其余扔给table 22处理
		ovs-ofctl add-flow br-int "table=20, priority=0, actions=goto_table:22"


		# table 22
		## 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow br-int "table=22, priority=1, in_port=${vio1001100_ofport}, actions=set_tunnel:1001100,output:${vtep137_ofport},${vtep138_ofport}"
		ovs-ofctl add-flow br-int "table=22, priority=1, in_port=${vio1234567_ofport}, actions=set_tunnel:1234567,output:${vtep137_ofport},${vtep138_ofport}"

		## 其余DROP
		ovs-ofctl add-flow br-int "table=22, priority=0, actions=drop"
		```

5. vm.sh

	??? note "点击打开"
		```shell
		ip netns del s1-vm1
		ip netns del s1-vm2
		ip netns del s2-vm1
		ip netns del s2-vm2

		ip netns add s1-vm1
		ip link add poi-s1-vm1 type veth peer name pii-s1-vm1
		ip link set pii-s1-vm1 netns s1-vm1
		ip netns exec s1-vm1 ip link set lo up
		ip netns exec s1-vm1 ip addr add 192.168.1.1/24 dev pii-s1-vm1
		ip netns exec s1-vm1 ip link set pii-s1-vm1 up
		ip netns exec s1-vm1 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s1-vm1 up
		brctl addif biv-1001100 poi-s1-vm1

		ip netns add s1-vm2
		ip link add poi-s1-vm2 type veth peer name pii-s1-vm2
		ip link set pii-s1-vm2 netns s1-vm2
		ip netns exec s1-vm2 ip link set lo up
		ip netns exec s1-vm2 ip addr add 192.168.1.2/24 dev pii-s1-vm2
		ip netns exec s1-vm2 ip link set pii-s1-vm2 up
		ip netns exec s1-vm2 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s1-vm2 up
		brctl addif biv-1001100 poi-s1-vm2

		ip netns add s2-vm1
		ip link add poi-s2-vm1 type veth peer name pii-s2-vm1
		ip link set pii-s2-vm1 netns s2-vm1
		ip netns exec s2-vm1 ip link set lo up
		ip netns exec s2-vm1 ip addr add 192.168.2.1/24 dev pii-s2-vm1
		ip netns exec s2-vm1 ip link set pii-s2-vm1 up
		ip netns exec s2-vm1 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s2-vm1 up
		brctl addif biv-1234567 poi-s2-vm1

		ip netns add s2-vm2
		ip link add poi-s2-vm2 type veth peer name pii-s2-vm2
		ip link set pii-s2-vm2 netns s2-vm2
		ip netns exec s2-vm2 ip link set lo up
		ip netns exec s2-vm2 ip addr add 192.168.2.2/24 dev pii-s2-vm2
		ip netns exec s2-vm2 ip link set pii-s2-vm2 up
		ip netns exec s2-vm2 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s2-vm2 up
		brctl addif biv-1234567 poi-s2-vm2
		```

## **Node 2**

---

1. ovs.sh

	??? note "点击打开"
		```shell
		# 删除ovs桥接
		ovs-vsctl del-br br-int

		# 创建ovs桥接
		ovs-vsctl add-br br-int

		# 清空流表
		ovs-ofctl del-flows br-int

		# 创建vtep
		ovs-vsctl add-port br-int vtep136 -- set interface vtep136 type=vxlan options:remote_ip=172.16.1.136 options:ttl=64 options:in_key=flow options:out_key=flow
		ovs-vsctl add-port br-int vtep138 -- set interface vtep138 type=vxlan options:remote_ip=172.16.1.138 options:ttl=64 options:in_key=flow options:out_key=flow
		```

2. bridge.sh

	??? note "点击打开"
		```shell
		brctl delbr biv-1001100
		brctl delbr biv-1234567

		brctl addbr biv-1001100
		ip link set biv-1001100 up

		brctl addbr biv-1234567
		ip link set biv-1234567 up
		```

3. br-to-ovs.sh

	??? note "点击打开"
		```shell
		ip link delete vib-1001100
		ip link delete vib-1234567

		ip link add vib-1001100 type veth peer name vio-1001100
		brctl addif biv-1001100 vib-1001100
		ip link set vib-1001100 up
		ip link set vio-1001100 up
		ovs-vsctl add-port br-int vio-1001100

		ip link add vib-1234567 type veth peer name vio-1234567
		brctl addif biv-1234567 vib-1234567
		ip link set vib-1234567 up
		ip link set vio-1234567 up
		ovs-vsctl add-port br-int vio-1234567
		```

4. openflow.sh

	??? note "点击打开"
		```shell
		# 写openflow规则顺序：
		# 1. 从表0开始
		# 2. 为每个表之间留一些表，用于今后扩展
		# 3. 跳表时候只往高了跳，不往低了跳
		# 4. priority=1的写规则，可存在多条；priotity=0的只能有一条，用于其余数据包处理

		# 获得openflow端口
		vio1001100_ofport=`ovs-vsctl get interface vio-1001100 ofport`
		vio1234567_ofport=`ovs-vsctl get interface vio-1234567 ofport`
		vtep136_ofport=`ovs-vsctl get interface vtep136 ofport`
		vtep138_ofport=`ovs-vsctl get interface vtep138 ofport`


		# table 0
		## 从linux-bridge过来的数据包，扔给table 2处理
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vio1001100_ofport}, actions=goto_table:2"
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vio1234567_ofport}, actions=goto_table:2"

		## 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vtep136_ofport} actions=goto_table:10"
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vtep138_ofport} actions=goto_table:10"

		## 其余DROP
		ovs-ofctl add-flow br-int "table=0, priority=0, actions=drop"


		# table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow br-int "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=goto_table:20"

		## 多播和广播包，扔给table 22处理
		ovs-ofctl add-flow br-int "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=goto_table:22"

		## 其余DROP
		ovs-ofctl add-flow br-int "table=2, priority=0, actions=drop"


		# table 10
		## 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到linux-bridge端口
		## 具体学习内容:
		##   1. 包的目的mac跟当前的源mac匹配
		##   2. 将tunnel号修改为当前的tunnel号
		##   3. 从当前入口发出
		ovs-ofctl add-flow br-int "table=10, priority=1, tun_id=1001100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${vio1001100_ofport}"
		ovs-ofctl add-flow br-int "table=10, priority=1, tun_id=1234567, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${vio1234567_ofport}"

		## 其余DROP（可选）
		ovs-ofctl add-flow br-int "table=10, priority=0, actions=drop"


		# table 20
		## 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		## 其余扔给table 22处理
		ovs-ofctl add-flow br-int "table=20, priority=0, actions=goto_table:22"


		# table 22
		## 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow br-int "table=22, priority=1, in_port=${vio1001100_ofport}, actions=set_tunnel:1001100,output:${vtep136_ofport},${vtep138_ofport}"
		ovs-ofctl add-flow br-int "table=22, priority=1, in_port=${vio1234567_ofport}, actions=set_tunnel:1234567,output:${vtep136_ofport},${vtep138_ofport}"

		## 其余DROP
		ovs-ofctl add-flow br-int "table=22, priority=0, actions=drop"
		```

5. vm.sh

	??? note "点击打开"
		```shell
		ip netns del s1-vm3
		ip netns del s1-vm4
		ip netns del s2-vm3
		ip netns del s2-vm4

		ip netns add s1-vm3
		ip link add poi-s1-vm3 type veth peer name pii-s1-vm3
		ip link set pii-s1-vm3 netns s1-vm3
		ip netns exec s1-vm3 ip link set lo up
		ip netns exec s1-vm3 ip addr add 192.168.1.3/24 dev pii-s1-vm3
		ip netns exec s1-vm3 ip link set pii-s1-vm3 up
		ip netns exec s1-vm3 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s1-vm3 up
		brctl addif biv-1001100 poi-s1-vm3

		ip netns add s1-vm4
		ip link add poi-s1-vm4 type veth peer name pii-s1-vm4
		ip link set pii-s1-vm4 netns s1-vm4
		ip netns exec s1-vm4 ip link set lo up
		ip netns exec s1-vm4 ip addr add 192.168.1.4/24 dev pii-s1-vm4
		ip netns exec s1-vm4 ip link set pii-s1-vm4 up
		ip netns exec s1-vm4 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s1-vm4 up
		brctl addif biv-1001100 poi-s1-vm4

		ip netns add s2-vm3
		ip link add poi-s2-vm3 type veth peer name pii-s2-vm3
		ip link set pii-s2-vm3 netns s2-vm3
		ip netns exec s2-vm3 ip link set lo up
		ip netns exec s2-vm3 ip addr add 192.168.2.3/24 dev pii-s2-vm3
		ip netns exec s2-vm3 ip link set pii-s2-vm3 up
		ip netns exec s2-vm3 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s2-vm3 up
		brctl addif biv-1234567 poi-s2-vm3

		ip netns add s2-vm4
		ip link add poi-s2-vm4 type veth peer name pii-s2-vm4
		ip link set pii-s2-vm4 netns s2-vm4
		ip netns exec s2-vm4 ip link set lo up
		ip netns exec s2-vm4 ip addr add 192.168.2.4/24 dev pii-s2-vm4
		ip netns exec s2-vm4 ip link set pii-s2-vm4 up
		ip netns exec s2-vm4 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s2-vm4 up
		brctl addif biv-1234567 poi-s2-vm4
		```

## **Node 3**

---

1. ovs.sh

	??? note "点击打开"
		```shell
		# 删除ovs桥接
		ovs-vsctl del-br br-int

		# 创建ovs桥接
		ovs-vsctl add-br br-int

		# 清空流表
		ovs-ofctl del-flows br-int

		# 创建vtep
		ovs-vsctl add-port br-int vtep137 -- set interface vtep137 type=vxlan options:remote_ip=172.16.1.137 options:ttl=64 options:in_key=flow options:out_key=flow
		ovs-vsctl add-port br-int vtep136 -- set interface vtep136 type=vxlan options:remote_ip=172.16.1.136 options:ttl=64 options:in_key=flow options:out_key=flow
		```

2. bridge.sh

	??? note "点击打开"
		```shell
		brctl delbr biv-1001100
		brctl delbr biv-1234567

		brctl addbr biv-1001100
		ip link set biv-1001100 up

		brctl addbr biv-1234567
		ip link set biv-1234567 up
		```

3. br-to-ovs.sh

	??? note "点击打开"
		```shell
		ip link delete vib-1001100
		ip link delete vib-1234567

		ip link add vib-1001100 type veth peer name vio-1001100
		brctl addif biv-1001100 vib-1001100
		ip link set vib-1001100 up
		ip link set vio-1001100 up
		ovs-vsctl add-port br-int vio-1001100

		ip link add vib-1234567 type veth peer name vio-1234567
		brctl addif biv-1234567 vib-1234567
		ip link set vib-1234567 up
		ip link set vio-1234567 up
		ovs-vsctl add-port br-int vio-1234567
		```

4. openflow.sh

	??? note "点击打开"
		```shell
		# 写openflow规则顺序：
		# 1. 从表0开始
		# 2. 为每个表之间留一些表，用于今后扩展
		# 3. 跳表时候只往高了跳，不往低了跳
		# 4. priority=1的写规则，可存在多条；priotity=0的只能有一条，用于其余数据包处理

		# 获得openflow端口
		vio1001100_ofport=`ovs-vsctl get interface vio-1001100 ofport`
		vio1234567_ofport=`ovs-vsctl get interface vio-1234567 ofport`
		vtep137_ofport=`ovs-vsctl get interface vtep137 ofport`
		vtep136_ofport=`ovs-vsctl get interface vtep136 ofport`


		# table 0
		## 从linux-bridge过来的数据包，扔给table 2处理
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vio1001100_ofport}, actions=goto_table:2"
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vio1234567_ofport}, actions=goto_table:2"

		## 从vtep过来的数据包，扔给table 10处理
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vtep137_ofport} actions=goto_table:10"
		ovs-ofctl add-flow br-int "table=0, priority=1, in_port=${vtep136_ofport} actions=goto_table:10"

		## 其余DROP
		ovs-ofctl add-flow br-int "table=0, priority=0, actions=drop"


		# table 2
		## 单播包，扔给table 20处理
		ovs-ofctl add-flow br-int "table=2, priority=1, dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=goto_table:20"

		## 多播和广播包，扔给table 22处理
		ovs-ofctl add-flow br-int "table=2, priority=1, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=goto_table:22"

		## 其余DROP
		ovs-ofctl add-flow br-int "table=2, priority=0, actions=drop"


		# table 10
		## 学习从vtep过来的数据包，往table 20中添加返程规则，然后输出到linux-bridge端口
		## 具体学习内容:
		##   1. 包的目的mac跟当前的源mac匹配
		##   2. 将tunnel号修改为当前的tunnel号
		##   3. 从当前入口发出
		ovs-ofctl add-flow br-int "table=10, priority=1, tun_id=1001100, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${vio1001100_ofport}"
		ovs-ofctl add-flow br-int "table=10, priority=1, tun_id=1234567, actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:${vio1234567_ofport}"

		## 其余DROP（可选）
		ovs-ofctl add-flow br-int "table=10, priority=0, actions=drop"


		# table 20
		## 有学习到规则的数据包，根据规则打上tunnel号并从对应的vtep port发出（动态，不用写规则）

		## 其余扔给table 22处理
		ovs-ofctl add-flow br-int "table=20, priority=0, actions=goto_table:22"


		# table 22
		## 匹配in_port后打上tunnel号，并从对应的vtep port发出（多条）
		ovs-ofctl add-flow br-int "table=22, priority=1, in_port=${vio1001100_ofport}, actions=set_tunnel:1001100,output:${vtep137_ofport},${vtep136_ofport}"
		ovs-ofctl add-flow br-int "table=22, priority=1, in_port=${vio1234567_ofport}, actions=set_tunnel:1234567,output:${vtep137_ofport},${vtep136_ofport}"

		## 其余DROP
		ovs-ofctl add-flow br-int "table=22, priority=0, actions=drop"
		```

5. vm.sh

	??? note "点击打开"
		```shell
		ip netns del s1-vm5
		ip netns del s1-vm6
		ip netns del s2-vm5
		ip netns del s2-vm6

		ip netns add s1-vm5
		ip link add poi-s1-vm5 type veth peer name pii-s1-vm5
		ip link set pii-s1-vm5 netns s1-vm5
		ip netns exec s1-vm5 ip link set lo up
		ip netns exec s1-vm5 ip addr add 192.168.1.5/24 dev pii-s1-vm5
		ip netns exec s1-vm5 ip link set pii-s1-vm5 up
		ip netns exec s1-vm5 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s1-vm5 up
		brctl addif biv-1001100 poi-s1-vm5

		ip netns add s1-vm6
		ip link add poi-s1-vm6 type veth peer name pii-s1-vm6
		ip link set pii-s1-vm6 netns s1-vm6
		ip netns exec s1-vm6 ip link set lo up
		ip netns exec s1-vm6 ip addr add 192.168.1.6/24 dev pii-s1-vm6
		ip netns exec s1-vm6 ip link set pii-s1-vm6 up
		ip netns exec s1-vm6 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s1-vm6 up
		brctl addif biv-1001100 poi-s1-vm6

		ip netns add s2-vm5
		ip link add poi-s2-vm5 type veth peer name pii-s2-vm5
		ip link set pii-s2-vm5 netns s2-vm5
		ip netns exec s2-vm5 ip link set lo up
		ip netns exec s2-vm5 ip addr add 192.168.2.5/24 dev pii-s2-vm5
		ip netns exec s2-vm5 ip link set pii-s2-vm5 up
		ip netns exec s2-vm5 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s2-vm5 up
		brctl addif biv-1234567 poi-s2-vm5

		ip netns add s2-vm6
		ip link add poi-s2-vm6 type veth peer name pii-s2-vm6
		ip link set pii-s2-vm6 netns s2-vm6
		ip netns exec s2-vm6 ip link set lo up
		ip netns exec s2-vm6 ip addr add 192.168.2.6/24 dev pii-s2-vm6
		ip netns exec s2-vm6 ip link set pii-s2-vm6 up
		ip netns exec s2-vm6 echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
		ip link set poi-s2-vm6 up
		brctl addif biv-1234567 poi-s2-vm6
		```
