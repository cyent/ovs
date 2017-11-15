## **ovs官方解释**

---

man ovs-fields

```text
Open vSwitch 1.8 and later support arbitrary masks for source and/or destination. Earlier versions only support masking the destination with the following masks:

	   01:00:00:00:00:00
			  Match  only  the  multicast  bit.  Thus,  dl_dst=01:00:00:00:00:00/01:00:00:00:00:00  matches  all  multicast  (including  broadcast)  Ethernet  packets,  and
			  dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 matches all unicast Ethernet packets.

	   fe:ff:ff:ff:ff:ff
			  Match all bits except the multicast bit. This is probably not useful.

	   ff:ff:ff:ff:ff:ff
			  Exact match (equivalent to omitting the mask).

	   00:00:00:00:00:00
			  Wildcard all bits (equivalent to dl_dst=*).
```

## **单播、多播、广播**

---

- 单播: 最左边第一个字节的最后一个bit为0

	`xxxxxxx0:xxxxxxxx:xxxxxxxx:xxxxxxxx:xxxxxxxx:xxxxxxxx`

	可以用`00:00:00:00:00:00/01:00:00:00:00:00`来表示

- 多播: 最左边第一个字节的最后一个bit为1，当所有bit都为1就是广播，即多播包括广播，就像正方形属于长方形，但正方形是独特的长方形，多播就像是长方形，广播就像是正方形

	`xxxxxxx1:xxxxxxxx:xxxxxxxx:xxxxxxxx:xxxxxxxx:xxxxxxxx`

	可以用`01:00:00:00:00:00/01:00:00:00:00:00`来表示，但不是只能这么表示，只要掩码里为0对应到的mac可以为任何数，即也可以写成`a1:bb:cc:dd:ee:ff/01:00:00:00:00:00`，或者`11:1b:cc:dd:ee:ff/11:10:08:00:00:00`，都是可以的

- 广播: 所有bit均为1，因此就是`ff:ff:ff:ff:ff:ff/ff:ff:ff:ff:ff:ff`

如果要匹配具体的mac地址，就把掩码写成`ff:ff:ff:ff:ff:ff`即可（如果不写掩码，默认就是全f）

**注意：如果想只匹配多播，但不包含广播，没有办法通过掩码做到，只能在写条目规则时候先匹配广播，再匹配多播**

## **掩码匹配算法详解**

---

例如`11:1b:cc:dd:ee:ff/11:10:08:00:00:00`表示

地址`11:1b:cc:dd:ee:ff` -> 二进制为`00010001:00011011:11001100:11011101:11101110:11111111`

掩码`11:10:08:00:00:00` -> 二进制为`00010001:00010000:00001000:00000000:00000000:00000000`

掩码中为1的，对应到地址的bit不变，掩码中为0的，对应到地址的bit可以为1也可为0，和ip/netmask的算法是类似的，因此这个例子中，可以表示的范围是：

最小：`00010001:00010000:00001000:00000000:00000000:00000000`

最大：`11111111:11111111:11111111:11111111:11111111:11111111`

转换成16进制，则是：

从`11:10:08:00:00:00`到`ff:ff:ff:ff:ff:ff`

!!! warning
	这个例子算出来的最小地址刚好和掩码一样，这只是个巧合
