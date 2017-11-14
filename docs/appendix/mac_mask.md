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
