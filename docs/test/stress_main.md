服务器：Dell R730，网卡为板载集成电口万兆（支持udp offload），os为centos 7.3

交换机：Dell万兆交换机

kvm：qemu-kvm 1.5.3，网卡virtio，os为centos 7.3 4核8G

iptables、ebtables、selinux均处于disable状态，不过openvswitch会强制加载conntrack模块

测试方法：使用iperf分别测试tcp和udp（只测试单播），每一个测试完成后均重启物理机

测试命令：

- tcp

	client: iperf -b 10G -t 300 -P 3 -c 目标ip

	server: iperf -s

- udp

	client: iperf -u -b 10G -t 300 -P 3 -c 目标ip

	server: iperf -u -s

测试结果：tcp的client和server流量相同，udp的client为平均每秒出站流量，server为平均每秒入站流量
