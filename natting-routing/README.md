
# PACKET FLOW ROUTING AND NATTING FROM A DOCKER MACHINE, (ingress/egress) from docker -> vm -> baremetal -> router -> internet & back


##BAREMETAL INTERFACE AND ROUTE DETAILS 


```
[root@mayura ~]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp7s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:1f:bc:02:6f:10 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.21/24 brd 10.0.2.255 scope global noprefixroute enp7s0
       valid_lft forever preferred_lft forever
    inet6 fe80::33b3:7332:5e0e:a98a/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp8s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:1f:bc:02:6f:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.21/24 brd 192.168.2.255 scope global noprefixroute enp8s0
       valid_lft forever preferred_lft forever
    inet6 fe80::dcda:53b1:87fe:e92b/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
4: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 52:54:00:a8:d1:80 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
    inet6 2001:db8:ca2:3::1/64 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fea8:d180/64 scope link
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:a8:d1:80 brd ff:ff:ff:ff:ff:ff
6: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr0 state UNKNOWN group default qlen 1000
    link/ether fe:54:00:01:1a:a2 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::fc54:ff:fe01:1aa2/64 scope link
       valid_lft forever preferred_lft forever


[root@mayura ~]# ip route show
default via 10.0.2.1 dev enp7s0 proto static metric 100
default via 192.168.2.1 dev enp8s0 proto static metric 101
10.0.2.0/24 dev enp7s0 proto kernel scope link src 10.0.2.21 metric 100
192.168.2.0/24 dev enp8s0 proto kernel scope link src 192.168.2.21 metric 101
192.168.3.0/24 via 192.168.2.1 dev enp8s0 proto static metric 101
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1



```


##VM INTERFACE, NETWORK AND ROUTE DETAILS


```

virsh # net-dhcp-leases default
 Expiry Time          MAC address        Protocol  IP address                Hostname        Client ID or DUID
-------------------------------------------------------------------------------------------------------------------
 2021-11-15 17:09:48  52:54:00:01:1a:a2  ipv4      192.168.122.120/24        rhel8a          01:52:54:00:01:1a:a2
 2021-11-15 17:31:24  52:54:00:01:1a:a2  ipv6      2001:db8:ca2:3:1::c2/64   rhel8a          00:04:9c:3f:36:30:fa:3b:56:d3:ae:1d:3a:e3:11:af:c0:25

virsh # net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes

virsh # net-dumpxml default
<network connections='1'>
  <name>default</name>
  <uuid>cb35ff41-51ac-4cf9-b9fc-060aab583928</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:a8:d1:80'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
      <host mac='52:54:00:01:aa:00' name='rhelxx' ip='192.168.122.110'/>
      <host mac='52:54:00:01:1a:a2' name='rhel8a' ip='192.168.122.120'/>
      <host mac='52:54:00:01:1a:b3' name='rhel8b' ip='192.168.122.130'/>
      <host mac='52:54:00:01:1a:c4' name='rhel8c' ip='192.168.122.140'/>
      <host mac='52:54:00:01:1a:d5' name='rhel8d' ip='192.168.122.150'/>
      <host mac='52:54:00:01:1b:aa' name='rhel8i' ip='192.168.122.125'/>
      <host mac='52:54:00:01:1b:bb' name='rhel8j' ip='192.168.122.135'/>
      <host mac='52:54:00:01:1b:cc' name='rhel8k' ip='192.168.122.145'/>
      <host mac='52:54:00:01:1b:dd' name='rhel8l' ip='192.168.122.155'/>
      <host mac='52:54:00:b2:5e:a6' name='rhel8p' ip='192.168.122.60'/>
      <host mac='52:54:00:b2:5e:b7' name='rhel8q' ip='192.168.122.70'/>
      <host mac='52:54:00:b2:5e:c8' name='rhel8r' ip='192.168.122.80'/>
    </dhcp>
  </ip>
  <ip family='ipv6' address='2001:db8:ca2:3::1' prefix='64'>
    <dhcp>
      <range start='2001:db8:ca2:3:1::10' end='2001:db8:ca2:3:1::ff'/>
    </dhcp>
  </ip>
</network>

virsh # net-dhcp-leases default
 Expiry Time          MAC address        Protocol  IP address                Hostname        Client ID or DUID
-------------------------------------------------------------------------------------------------------------------
 2021-11-15 17:09:48  52:54:00:01:1a:a2  ipv4      192.168.122.120/24        rhel8a          01:52:54:00:01:1a:a2
 2021-11-15 17:31:24  52:54:00:01:1a:a2  ipv6      2001:db8:ca2:3:1::c2/64   rhel8a          00:04:9c:3f:36:30:fa:3b:56:d3:ae:1d:3a:e3:11:af:c0:25


```
