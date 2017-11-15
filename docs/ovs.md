## **同宿主，同子网的2台VM互通**

---

```text
# 建立桥接
ovs-vsctl add-br br-int

# 用veth模拟vm，将veth在宿主上的一端加入桥接
ovs-vsctl add-port br-int poi-vm1
ovs-vsctl add-port br-int poi-vm2
```

## **同宿主，不同子网之间隔离（分别桥接）**

---

```text
# 建立桥接，biv-100是子网1，biv-200是子网2
ovs-vsctl add-br biv-100
ovs-vsctl add-br biv-200

# 用veth模拟vm，2个vm在宿主上的一端分别加入2个桥接
ovs-vsctl add-port biv-100 poi-vm1
ovs-vsctl add-port biv-200 poi-vm2
```

## **同宿主，不同子网之间隔离（打tag）**

---

```text
# 建立桥接
ovs-vsctl add-br br-int

# 用veth模拟vm，2个vm在宿主上的一端加入桥接，同时打tag
ovs-vsctl add-port br-int poi-vm1 -- set port poi-vm1 tag=100
ovs-vsctl add-port br-int poi-vm2 -- set port poi-vm2 tag=200
```

## **同宿主，不同桥接之间二层互通**

---

```text
# 建立桥接
ovs-vsctl add-br biv-100
ovs-vsctl add-br biv-200

# 创建patch port，2端分别桥接到biv-100和biv-200
ovs-vsctl add-port biv-100 patch-to-200 -- set interface patch-to-200 type=patch -- set interface patch-to-200 options:peer=patch-to-100
ovs-vsctl add-port biv-200 patch-to-100 -- set interface patch-to-100 type=patch -- set interface patch-to-100 options:peer=patch-to-200

# 用veth模拟vm，vm在宿主的一端加入桥接
ovs-vsctl add-port biv-100 poi-vm1
ovs-vsctl add-port biv-200 poi-vm2
```

## **不同宿主，同子网互通这么做会环路**

---

![](/img/loop.jpg)

这样看上去好像没啥问题，但是这个图可以画成这样：

![](/img/loop1.jpg)

这样就形成了一个环路，如果宿主机就2台还不会环路，只要达到3台就会开始环路

注意：我们当前用linux bridge没有环路是因为无论是多播vtep还是单播vtep，一个桥接网卡都只桥接一个vtep，而广播包（或者叫泛洪）的特性是数据包从非入口的所有端口泛洪出去，即如果一个linux bridge桥接2个vtep，也是会造成环路的

其实openstack的网络架构图画出来看也是有环路的：

![](/img/loop2.jpg)

那么为什么没有环路呢，是因为如果br-tun只是个简单的switch/hub功能，那么是会环路的，vtep收到数据包后会从br-tun的另一个vtep泛洪出去。但br-tun是openflow控制的，其中2条规则起了关键作用：

1. br-tun从vtep收到数据包后，将vni去掉，加上vlan头，这还不是关键，关键是第2点

2. 然后将数据包从patch-int端口发出，即这个数据包只发到br-int而不会交到另一个vtep上，因此不会产生环路

## **iptables/ebtables对ovs port不起作用**

---

经测试，使用iptables和ebtables对ovs端口不起任何作用，只对linux bridge的端口起作用
