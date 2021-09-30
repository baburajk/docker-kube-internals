# docker-kube-internals
Reference Notes

Reference Notes


#DRAFT

[root@rhel8a ~]# docker run -p 180:80 -p 1443:443 -d --name netops1 -it baburaj/netops
0d1b1ae4026999708a633aff36b12b102e2b7a308a56b4f9a8a1043a65225400

[root@rhel8a ~]# docker run -p 280:80 -p 2443:443 -d --name netops2 -it baburaj/netops
5008d7025acb85ee992aab30ab95811afbbd131c9065bb9f5ed1be7a6907e538


[root@rhel8a ~]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:01:1a:a2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.120/24 brd 192.168.122.255 scope global dynamic noprefixroute enp1s0
       valid_lft 2474sec preferred_lft 2474sec
    inet6 2001:db8:ca2:3:1::c2/128 scope global dynamic noprefixroute
       valid_lft 2447sec preferred_lft 2447sec
    inet6 fe80::f2c8:8009:4778:b5a1/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:b8:57:2f:e4 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:b8ff:fe57:2fe4/64 scope link
       valid_lft forever preferred_lft forever
9: veth0578815@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 5e:5c:47:cc:59:08 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::5c5c:47ff:fecc:5908/64 scope link
       valid_lft forever preferred_lft forever
11: vethdae0d70@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 0e:8c:f9:f7:f0:bd brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::c8c:f9ff:fef7:f0bd/64 scope link
       valid_lft forever preferred_lft forever



[root@rhel8a ~]#  curl -ks https://[::]:1443
Container/HostName : 0d1b1ae40269

 - Interfaces -
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever



[root@rhel8a ~]# tcpdump -qnne -i docker0 dst port 443
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on docker0, link-type EN10MB (Ethernet), capture size 262144 bytes
15:00:40.460677 02:42:b8:57:2f:e4 > 02:42:ac:11:00:02, IPv4, length 74: 172.17.0.1.34288 > 172.17.0.2.443: tcp 0
15:00:40.460721 02:42:b8:57:2f:e4 > 02:42:ac:11:00:02, IPv4, length 66: 172.17.0.1.34288 > 172.17.0.2.443: tcp 0
15:00:40.472009 02:42:b8:57:2f:e4 > 02:42:ac:11:00:02, IPv4, length 583: 172.17.0.1.34288 > 172.17.0.2.443: tcp 517
15:00:40.475174 02:42:b8:57:2f:e4 > 02:42:ac:11:00:02, IPv4, length 66: 172.17.0.1.34288 > 172.17.0.2.443: tcp 0



[root@rhel8a ~]# bridge fdb show | grep 02:42:ac:11:00:02
02:42:ac:11:00:02 dev veth0578815 master docker0


[root@rhel8a ~]# bridge fdb show | grep 02:42:b8:57:2f:e4
02:42:b8:57:2f:e4 dev docker0 vlan 1 master docker0 permanent
02:42:b8:57:2f:e4 dev docker0 master docker0 permanent


interface_name@if<peers_interface_id>



9: veth0578815@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 5e:5c:47:cc:59:08 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::5c5c:47ff:fecc:5908/64 scope link
       valid_lft forever preferred_lft forever
11: vethdae0d70@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 0e:8c:f9:f7:f0:bd brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::c8c:f9ff:fef7:f0bd/64 scope link
       valid_lft forever preferred_lft forever

[root@0d1b1ae40269 /]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever


[root@5008d7025acb /]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
10: eth0@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

