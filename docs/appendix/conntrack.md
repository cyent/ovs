```text
> lsmod | grep -i conntrack
nf_conntrack_ipv6      18894  1
nf_conntrack_ipv4      19108  1
nf_defrag_ipv4         12729  1 nf_conntrack_ipv4
nf_defrag_ipv6         35104  2 openvswitch,nf_conntrack_ipv6
nf_conntrack          111302  6 openvswitch,nf_nat,nf_nat_ipv4,nf_nat_ipv6,nf_conntrack_ipv4,nf_conntrack_ipv6

> lsmod | grep -i nat
nf_nat_ipv6            14131  1 openvswitch
nf_nat_ipv4            14115  1 openvswitch
nf_nat                 26147  3 openvswitch,nf_nat_ipv4,nf_nat_ipv6
nf_conntrack          111302  6 openvswitch,nf_nat,nf_nat_ipv4,nf_nat_ipv6,nf_conntrack_ipv4,nf_conntrack_ipv6

> lsmod | grep -i openvswitch
openvswitch           106775  4 vport_vxlan
nf_nat_ipv6            14131  1 openvswitch
nf_nat_ipv4            14115  1 openvswitch
nf_defrag_ipv6         35104  2 openvswitch,nf_conntrack_ipv6
nf_nat                 26147  3 openvswitch,nf_nat_ipv4,nf_nat_ipv6
nf_conntrack          111302  6 openvswitch,nf_nat,nf_nat_ipv4,nf_nat_ipv6,nf_conntrack_ipv4,nf_conntrack_ipv6
libcrc32c              12644  3 xfs,bnx2x,openvswitch
```
