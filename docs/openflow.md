## **查看**

---

```text
# 查看openflow端口信息
ovs-ofctl show ovs-switch

# 查看openflow端口编号
ovs-vsctl get interface p0 ofport

# 查看datapath
ovs-dpctl show

# 查看流表
ovs-ofctl dump-flows ovs-switch
```

## **流表操作**

---

### **默认规则**

每个交换机创建之后默认都有一条流表规则，用于允许所有流量通过

```text hl_lines="2"
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=1588.919s, table=0, n_packets=50, n_bytes=3260, idle_age=0, priority=0 actions=NORMAL
```

加亮这条就是默认的流表，如果这条删掉，那么当其他条目都没匹配到的时候，数据包就会被丢弃

### **添加规则**

```text
# 屏蔽所有进入 OVS 的以太网广播数据包
ovs-ofctl add-flow ovs-switch "table=0, dl_src=01:00:00:00:00:00/01:00:00:00:00:00, actions=drop"
```

### **流表参数详解**

```text
ovs-ofctl dump-flows ovs-switch
```

输出

```text
NXST_FLOW reply (xid=0x4):
cookie=0x0, duration=7.672s, table=0, n_packets=0, n_bytes=0, idle_age=7, priority=0 actions=NORMAL
```

- cookie: 不了解，官方manpage解释如下

	??? note "manpage"
		```text
		An opaque identifier called a cookie can be used as a handle to identify a set of flows:

		cookie=value
			A cookie can be associated with a flow using the add-flow, add-flows, and mod-flows commands.  value can be any 64-bit number and need not be unique among flows. If this field is omitted, a default cookie value of 0 is used.

		cookie=value/mask
			When using NXM, the cookie can be used as a handle for querying, modifying, and deleting flows. value and mask may be supplied for the del-flows, mod-flows, dump-flows, and dump-aggregate commands to limit matching cookies. A 1-bit in mask indicates that the corresponding bit in cookie must match exactly, and a 0-bit wildcards that bit. A mask of -1 may be used to exactly match a cookie.

			The  mod-flows  command can update the cookies of flows that match a cookie by specifying the cookie field twice (once with a mask for matching and once without to indicate the new value):

			ovs-ofctl mod-flows br0 cookie=1,actions=normal
				Change all flows' cookies to 1 and change their actions to normal.

			ovs-ofctl mod-flows br0 cookie=1/-1,cookie=2,actions=normal
				Update cookies with a value of 1 to 2 and change their actions to normal.

			The ability to match on cookies was added in Open vSwitch 1.5.0.
		```

- duration: 该条目创建多久了

- table: 该条目所属table

- n_packets: 匹配了多少个包，注意是匹配，即如果该条目有被查询过，但是没匹配上，那么数值是不会增加的

- n_bytes: 匹配的所有包的大小总和，注意是匹配，即如果该条目有被查询过，但是没匹配上，那么数值是不会增加的

- idle_age: 该条目多久没被匹配了，注意是匹配，即如果该条目有被查询过，但是没匹配上，那么数值是不会增加的

- priority: 表内条目优先级，0-65535，数字越过越优先，不指定优先级则为32768，相同优先级的话，先插入的优先（dump-flows结果中靠上的优先）

- actions: 匹配之后的操作

## **实验**

---

### **使用openflow转发包到多个端口**

转到任何端口的方法都是一样的，包括veth和vtep。效果等同于用linux bridge时候的bridge fdb append

```text
# 先清空默认条目，以免导致环路，也是为了更了解数据包
ovs-ofctl del-flows br-tun

# 转到openflow端口1、2、3、4
ovs-ofctl add-flow br-tun "table=0, dl_dst=01:00:00:00:00:00/01:00:00:00:00:00, actions=output:1,2,3,4"
```

上面的

```text
actions=output:1,2,3,4
```

等同于(ovs-ofctl就可以看到自动翻译成如下)

```text
actions=output:1,output:2,output:3,output:4
```
