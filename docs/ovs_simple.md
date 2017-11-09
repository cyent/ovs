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

## **不同宿主，同子网互通（不同桥接）**

---

假设一共有3台宿主，分别是172.16.1.136/137/138

172.16.1.136配置：

```text
# 建立桥接
ovs-vsctl add-br biv-100

# 创建2个vtep，分别是到137和到138的vxlan隧道，vni为100
ovs-vsctl add-port biv-100 int-v100-137 -- set interface int-v100-137 type=vxlan options:remote_ip=172.16.1.137 options:key=100
ovs-vsctl add-port biv-100 int-v100-138 -- set interface int-v100-138 type=vxlan options:remote_ip=172.16.1.138 options:key=100

# 用veth模拟vm，vm在宿主上的一端加入桥接
ovs-vsctl add-port biv-100 poi-vm1
```

172.16.1.137配置：

```text
# 建立桥接
ovs-vsctl add-br biv-100

# 创建2个vtep，分别是到136和到138的vxlan隧道，vni为100
ovs-vsctl add-port biv-100 int-v100-136 -- set interface int-v100-136 type=vxlan options:remote_ip=172.16.1.136 options:key=100
ovs-vsctl add-port biv-100 int-v100-138 -- set interface int-v100-138 type=vxlan options:remote_ip=172.16.1.138 options:key=100

# 用veth模拟vm，vm在宿主上的一端加入桥接
ovs-vsctl add-port biv-100 poi-vm2

```

172.16.1.138配置：

```text
# 建立桥接
ovs-vsctl add-br biv-100

# 创建2个vtep，分别是到136和到137的vxlan隧道，vni为100
ovs-vsctl add-port biv-100 int-v100-136 -- set interface int-v100-136 type=vxlan options:remote_ip=172.16.1.136 options:key=100
ovs-vsctl add-port biv-100 int-v100-137 -- set interface int-v100-137 type=vxlan options:remote_ip=172.16.1.137 options:key=100

# 用veth模拟vm，vm在宿主上的一端加入桥接
ovs-vsctl add-port biv-100 poi-vm3
```

## **不同宿主，同子网互通（打tag）**

---

假设一共有3台宿主，分别是172.16.1.136/137/138

172.16.1.136配置：

```text
# 建立桥接
ovs-vsctl add-br br-int

# 创建2个vtep，分别是到137和到138的vxlan隧道，vni为100
ovs-vsctl add-port br-int int-v100-137 -- set interface int-v100-137 type=vxlan options:remote_ip=172.16.1.137 options:key=100
ovs-vsctl add-port br-int int-v100-138 -- set interface int-v100-138 type=vxlan options:remote_ip=172.16.1.138 options:key=100

# 用veth模拟vm，vm在宿主上的一端加入桥接
ovs-vsctl add-port br-int poi-vm1
```

172.16.1.137配置：

```text
# 建立桥接
ovs-vsctl add-br br-int

# 创建2个vtep，分别是到136和138的vxlan桥接，vni为100
ovs-vsctl add-port br-int int-v100-136 -- set interface int-v100-136 type=vxlan options:remote_ip=172.16.1.136 options:key=100
ovs-vsctl add-port br-int int-v100-138 -- set interface int-v100-138 type=vxlan options:remote_ip=172.16.1.138 options:key=100

# 用veth模拟vm，vm在宿主上的一端加入桥接
ovs-vsctl add-port br-int poi-vm2
```

172.16.1.138配置：

```text
# 建立桥接
ovs-vsctl add-br br-int

# 创建2个vtep，分别是到136和137的vxlan隧道，vni为100
ovs-vsctl add-port br-int int-v100-136 -- set interface int-v100-136 type=vxlan options:remote_ip=172.16.1.136 options:key=100
ovs-vsctl add-port br-int int-v100-137 -- set interface int-v100-137 type=vxlan options:remote_ip=172.16.1.137 options:key=100

# 用veth模拟vm，vm在宿主上的一端加入桥接
ovs-vsctl add-port br-int poi-vm3
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
