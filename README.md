# Networking

## Commands

### ip add
- `ip add 5.5.0.1 255.255.255.0 secondary`

### default gateway IP
- ip default-g 1.1.10.254 	: ip default-gateway

### VLAN
- vlan 10			: setup vlan
- name VLAN_10			: name vlan
- int range f1/1 , f1/15	: interface with non consecutive port nums
- int range f1/1 - 2		: interface with consecutive port num
- sw m a			: switch mode access	<=> no ..
- sw a v 10			: switch access vlan 10	<=> no ..
- sw t e d			: switch trunk encapsulation dot1q
- sw m tr			: switch mode trunk

### setting up vlan for routers
  1. int f0/0.10	<=> no ...
  2. en dot1q 10	<=> no ...
  3. ip add 1.1.10.254 255.255.255.0	<=> no ...

### Native VLAN
  - SW : sw tr native vlan 10	<=> no ...
  - R  : encap d 10 native 	<=> no ...

### Setting up ip routing
  - ip route (destination network ip) (broadcast) (out interface) (next ip)
  - ip route 100.30.0.0 255.255.255.0 s1/0 204.200.7.2

- ping 1.1.4.4 source 1.1.1.1

### pap
  - R1(config)# username R2 pass cisco => int s1/0 => en ppp => ppp pap sent R1 pass cisco => pp authen pap => clock rate 64000
  - R2(config)# username R1 pass cisco => int s1/0 => en ppp => ppp authen pap => ppp pap sent R2 pass cisco => clock rate 64000

### chap
  - R1(config)# username R2 pass cisco => int s1/0 => en ppp => ppp authen chap => ip add 1.1.10.1 255.255.255.0 => no sh

### L3 SW SVI
  - If set as `no ip routing`, change to `ip routing`
  - vlan 30 => vlan 40 => int f2/3 => sw m a => sw a v 30 => int f2/4 => sw m a => sw a v 40 => int vlan 30 => ip add 1.1.30.254 255.255.255.0 => int vlan 40 => ip add 1.1.40.254 255.255.255.0

### Frame relay
R1) ip route 1.1.0.0 255.255.192.0 s1/0 1.1.12.2
R2) ip route 1.1.0.0 255.255.192.0 s1/0.23 1.1.23.3
    ip route 1.1.1.0 255.255.255.0 s1/0.12 1.1.12.1
R3) ip route 1.1.4.0 255.255.255.0 s1/0.34 1.1.34.4
    ip route 1.1.0.0 255.255.224.0 s1/0.23 1.1.23.2
R4) ip route 1.1.0.0 255.255.224.0 s1/0.34 1.1.34.3 OR
(better) ip route 1.1.0.0 255.255.192.0 s1/0.34 1.1.34.3

### multi-point frame-relay
- R1) `conf t` => `int lo0` => `ip add 10.10.1.1 255.255.255.0` => `exit` => `int s1/0` => `ip add 10.10.123.1 255.255.255.0` => `fram map ip 10.10.123.2 102 b` => `fram map ip 10.10.123.3 102 b`
- R2) `conf t` => `int lo0` => `ip add 10.10.2.2 255.255.255.0` => `exit` => `int s1/0.123 m` => `ip add 10.10.123.2 255.255.255.0` => `fram map ip 10.10.123.1 201 b` => fram map ip 10.10.123.3 203 b`
- R3) `conf t` => `int lo0` => `ip add 10.10.3.3 255.255.255.0` => `exit` => int s1/0.123 m` => `ip add 10.10.123.3 255.255.255.0` => `fram map ip 10.10.123.2 302 b` => `fram map ip 10.10.123.1 302 b` => `int s1/0.34 p` => `ip add 150.1.34.1 255.255.255.0` => `fram inter 304`
- R4) `conf t` => `int lo0` => `ip add 150.1.4.4 255.255.255.0` => `exit` => `int s1/0.34 p` => `ip add 150.1.34.254 255.255.255.0` => `fram inter 403`

### route-map
- basic config
- fram map ip
- next hop ping testing
- ip routing
- ping testing
- router rip
- route-map
  - R3) `route-map STATIC` / automatically adds 10 at the end => `match ip address prefix NET1` => `set metric 3` => `route-m STATIC 20` => `match ip add pre NET2` => `set metric 2` => `route-m CONNECTED` => `match interface s1/0.123 lo0` => `set metric 1` => `ip prefix-list NET1 permit 16.16.1.0/24` => `ip pre NET2 permit 16.16.2.0/24`
- redistribute
  - R3) `router rip` => `redistribute static route STATIC` => `redistribute connected route CONNECTED`
- ping testing

### dhcp relay
- VLAN
  - SW1) `vlan 10` => `int range f1/1 - 2` => `no sh` => `sw m a` => `sw a v 10` => `exit`
  - SW1) `int vlan 10` => `ip add 1.1.10.154 255.255.255.0` => `exit`

- L2 WAN - PPP and IP
  - SW1) `int f0/0` => `no sh` => `ip add dhcp`
  - SW1) `username R1 pass cisco` => `int s2/0` => `no sh` => `en ppp` => `ppp authen chap` => `clock rate 64000` => `ip add 1.1.12.5 255.255.255.252` => `exit`
  - R1) `int f0/0` => `no sh` => `ip add 1.1.20.254 255.255.255.0` => `exit`
  - R1) `username SW1 pass cisco` => `int s1/0` => `no sh` => `en ppp` => `ppp authen chap` => `ip add 1.1.12.6 255.255.255.252`

- RIP
  - SW1) `router rip` => `ver 2` => `network 1.0.0.0` => `no auto` => `passive default` => `no passive s2/0` => `int s2/0` => `ip split`
  - SW1) `router rip` => `default-information originate` OR `redistribute static metric 1` but this is not used often and not recommended
  - R1) `router rip` => `ver 2` => `network 1.0.0.0` => `no auto` => `passive f0/0` => `int f0/0` => `ip sp`


- L3 routing protocol
  - SW1) `ip route 0.0.0.0 0.0.0.0 f0/0 10.0.0.1`

- DHCP
  - SW1) `ip dhcp pool NET10` => `network 1.1.10.0 /24` => `default-router 1.1.10.254` => `dns-server 1.1.10.100 168.126.63.1` => `lease 7` => `class 10` => `address range 1.1.10.1 1.1.10.253` => `exit` => ip dhcp exclude-address 1.1.10.254`
  - SW1) `ip dhcp pool NET20` => `network 1.1.20.0 /24` => `default-router 1.1.20.254` => `dns-server 1.1.10.100 168.126.63.1` => `l 7`
  - SW1) `ip dhcp pool NET10` => `class 10` => `address range 1.1.10.1 1.1.10.253`
  - SW1) `ip dhcp pool NET20` => `class 20` => `address range 1.1.20.1 1.1.20.253`
  - SW1) `int vlan 10` => `ip dhcp client class-id 10`
  - SW1) `int s2/0` => `ip dhcp client class-id 20`
  - R1) `int f0/0` => `ip helper-address 1.1.12.5`

- NAT
  - SW1) `access-list 1 permit 1.1.10.0 0.0.0.255`
  - SW1) `access 1 permit 1.1.20.0 0.0.0.255`
  - SW1) `access 1 permit 1.1.12.4 0.0.0.3` Only for ping test
  - SW1) `ip nat inside source list 1 int f0/0 overload`
  - SW1) `int vlan 10` => `ip nat inside`
  - SW1) `int s2/0` => `ip nat inside`
  - SW1) `int f0/0` => `ip nat outside`

### NAT_PT
- `no sh`
- `ip add dhcp` => internet facing interface
- general ip routes
- `ip route 0.0.0.0 0.0.0.0 f0/1 10.0.0.1`
- `ip route 0.0.0.0 0.0.0.0 s1/0.12 1.1.12.1`
- `ip route 0.0.0.0 0.0.0.0 s1/0.23 1.1.23.2`
- `ip route 0.0.0.0 0.0.0.0 s1/0.34 1.1.34.3`
- `access-list 10 permit 1.1.1.0 0.0.0.255` => same thing until 1.1.4.0 0.0.0.255
- `ip nat inside source list 10 int f0/1 overload`
- `int s1/0` => `ip nat inside`
- `int f0/1` => `ip nat out`
- `ping 10.0.0.1 source 1.1.4.4`

### domain
- if `no ip domain look` => change it to `ip domain look`
  - but if asked with unknown IPs/addresses, it will loop
- `ip name-server 8.8.8.8`
- `ping www.google.com`

### username / password
- `line c 0` => `login local` => `username master password pass!!` => exit and check 
- `username master password admin@$` => `line c 0` => `password cisco` => `login local`
- `username user01 password router@$` => `line vty 0 4` => `login local` => `telnet (ip)`
- `enable password router@$`
- to encrypto all passwords so that it doesn't show on running-config
  - `service password-encryption

### banner / motd
- `banner mord` or `motd banner` => `exit` out all the way to check the message of the day or banner

### stp - spanning-tree protocol
- select root switch => select root port on other switches => select one port per one segment => other ports without any roles become alternate ports and be logically blocked
- 3 switches w/ vlan 10 => check spanning-tree vlan 10 brief
- block 20s => listen 15s => learn 15s => forward
- if root or designated port, it becomes listening status immediately
- between SWs
  - `spanning uplink` => blocked link is unblocked immediately, which is 30s
  - `spanning backbonefast` on all switches => max age is gone, which is 20s
  - `int f1/1` => `spanning portfast` => looping may occur
- `spanning-tree portfast` => saves 30s for ports for end devices
- SW1) `sp vlan 10 priority 0`
- SW2) `sp vlan 10 priority 4092`
- SW3) `sp vlan 10 priority 8192` => makes the neighboring switch's port to be blocked
- `sh sp vlan 10 br` => shows the spanning-tree information
- SW4) `int f1/2` => `span vlan 10 cost 5` => makes the final cost to be lower

### vtp
- SW1) `vtp domain VTP22` => `vtp mode server
- Sw2-4) `vtp mode client`
- SW1) vlan setting => trunking mode
- SW2-4) trunking setting => access mode for SW4
- SW4) `spanning-tree portfast` on access mode ports => `span port fast bpduguard`
- if vlan doesn't update, update any vlan then erase
- SW1) `span vlan 10 root primary diameter 4` => I am root out of 4 switches => default is 7 switch based config, so in this case, the timings are shortened around aging, etc.
- SW3) `span vlan 10 root second diameter 4` => becomes the number 2 for vlan 10
- SW2) `span vlan 20 root secondary dimater 4` => becomes the number 2 for vlan 20
- for new (but old) switches to be joined in an existing network, put this switch in transparent mode to make the revision information as 0
- if you want to change from 'transparent' to 'server' mode, the port must be shut down before doing so. Or else, the server will learn clients' vlan info. Then, 
  - SW1) `vtp m t` => revision becomes 0 => `int f1/1` => `sh` => `vtp m s` => increase revision num => if it's higher than clients' num => make vlan config => `no sh` on the interface

### vtp + ripv2
- SW1-4) vtp transparent mode
- SW1-3) vlan setting => access mode
- SW1-3) config ip address => next hop ping test
- SW4) vlan setting => access mode
- PC1-2) `ip default-gateway (gateway)` => must exist for end devices
- SW1) router rip => passive int on lo0
- SW2,3) router rip => `neighbor (ip address)` ip address of destination so that it doesn't go to end devices => to make it unicast. if not, it sends multicast due to switch in the middle

### HSRP - hot standby routing protocol
- SW3) `int vlan 234` => `standby 234 ip 1.1.234.100` => `st 234 priority 110` default value is 100 => `st 234 preempt delay minimum 30` how long to make the change effective since it cannot run before rip sends out new information => `st 234 track f1/3 20` subtract 20 to make it lower than 100. higher number is more prior number
- SW2) `int vlan 234` => `st 234 ip 1.1.234.100` => `st 234 preempt`
- SW2,3) `show standby` to check standby configs
- PC1-2) `ip default-g 1.1.234.100` => ping to the vGW => `clear arp` => `clear mac` to erase cached info => `traceroute 1.1.100.100`

### mhsrp - multiple hot standby routing protocol
- SW1-4) `vtp m t` => config vlan => `sp portf` for access ports => `sp portf bpduguard` on config mode
- SW1-3) `int f1/1` => `no sw` to change L2 port to L3 port => config ip address
- SW2,3) config int vlan 10,20 and ip address
- SW1-3) next hop ping tests
- SW1-3) config routing => include neighbor config for SW2,3
- SW3) `int vlan 10` => `st 10 ip 1.1.10.100` => `st 10 pri 100` => `st 10 pre d m 30` => `st 10 tr f1/3 20` => `int vlan 20` => `st 20 ip 1.1.20.100` => `st 20 pre`
- SW2) opposite config

### vrrp
- sw4) `vtp m s`
- sw2,3) `vtp m c`
- sw4) vlan setting => `sp portf` for reducing 30s forward delay => `sp portf b` for no bpdu on portfast interfaces => `int range f1/2 , f1/4` => trunking
- sw2,3) trunking => check if vlan info has been received => if not, increase revision num for sw4
- sw1-3) ip add for `sw m a` and `no sw` and svi => next hop test => `router rip` => include `nei (destination router ip add)` => `pass default` => `no pass vlan 10` and `20` and port to SW1
- SW3) `track 1 int f1/3 line-protocol` => `int vl 10` => `vrrp 10 track 1 decrement 20 `=> `vrrp 10 pri 110` => `vrrp 10 pre d m 30` =>

### etherchannel
- dsw,asw) vtp setting => vlan setting
- asw1,2) `sw m a` => `sp portf` => `sp portf b`
- dsw,asw) use `cdp` to check what are connected => trunking => `channel-group 5 mode on`
- bb,dsw,asw) ip add + vGW
- bb,dsw) ip routing + router rip
- dsw) mhsrp
- pc) `ip default-g (vGW add)` => `ip add (ip add)`

- CentOS server dhcp
  - `rpm -qa + grep dhcp` => check firewall => turn off dhcp firewall => `ps -ef | grep dnsmasq` to kill dns => `systemctl disable dnsmasq` to disable when rebooting => `vim /etc/dhcp/dhcpd.conf`

```bash
subnet 192.168.50.0 netmask 255.255.255.0 {
}

subnet 192.168.10.64 netmask 255.255.255.224 {
        option routers 192.168.10.92;
        option domain-name "kedu.edu";
        option domain-name-servers 192.168.50.101;
        range dynamic-bootp 192.168.10.65 192.168.10.91;
        default-lease-time 10000;
        max-lease-time 50000;
}
```
  - `systemctl start dhcpd.service`
- dsw1,2) vlan interfaces => `ip help (dhcp server ip add)`
- linux
  - `vim /etc/resolv.conf` => add nameserver ip => `route` to check kernel ip routing table

### 180v
- SW1-4
  - vtp => vlan => trunking => check vlans => check trunk => check ether summ
  - L3 `no sw` => ip add
- R1-3
  - fram relay global => clock rate
  - ip add for SW facing ports
  - fram relay per router

### eigrp + rip
- fram map => ping test
  - R1) R1->R2-3
  - R2) R2->R1,R3
  - R3) R3->R1-2,R4 
  - R4) R4->R3
- eigrp => needs to have neighbor R in the eigrp topology
  - R1) `net 25.25.1.1 0.0.0.0` == 25.25.1.0 0.0.0.255 => `net 25.25.123.1 0.0.0.0`
  - R2) `net 25.25.2.2 0.0.0.0` => `net 25.25.123.2 0.0.0.0` => `int s1/0.234` => `no ip sp eigrp 25`
  - R3) `net 25.25.3.3 0.0.0.0` => `net 25.25.123.3 0.0.0.0`
- rip
  - R3) `net 150.1.0.0` => no need for passive since diff subnet
  - R4) rip config
- connecting different networks
  - R3) to bring rip info to eigrp network `router ei 25` => redistribute rip metric bandwidth delay/10 reliability load mtu `r rip m 1544 2000 255 1 1500` the metric of the entrnace to eigrp network`
  - R3) `ip prefix-list HOP1 permit 25.25.123.0/24` => `ip prefix-list HOP2 permit 25.25.2.0/24` => `ip pre HOP3 permit 25.25.1.0/24` => `route-map EIGRP_NET 10` => `match ip add prefix HOP1` => `set metric 1` => `route-map EIGRP_NET 20` => `match ip add prefix HOP2` => `set metric 2` => `route-map EIGRP_NET 30` => `match ip add prefix HOP3` => `set metric 3` => `show route-map`=> `router rip` => `redistribute digrp 25 route-map EIGRP_NET` => `sh ip route` D EX 170 are externals 
- R3) `int s1/0.34` => `ip summary-address rip 25.25.0.0 255.255.0.0` => R4) `clear ip ro\*` => `sho ip route` to check if routing table has been updated with abstracted addresses
- R3) `router ei 25` => `distribute-list prefix OVERWRITE out rip` = `ip prefix-list OVERWRITE deny 25.25.0.0/16` => `ip prefix-list OVERWRITE permit 0.0.0.0/0 le 32`

#### diffusing update algorithm - dual
- feasible distance : the best path is the path with the lowest fd value
- successor : next-hop router on the best path
- feasible successor : next-hop router on backup path
- `int s1/0.13` => `delay 2100` => `clear ip eigrp 29 neighbors` => `sh ip eigrp topology detail` to change the successor
- `router ei 29` => `variance 2` => `sh ip route` to check load balancing

### access list

#### telnet
- numbered 1
  - R1) `access-list 1 permit 29.29.4.0 0.0.0.255` => `line vty 0 4` => `access-class 1 in` => `login` => `pass cisco`
  - R4) `ip telnet source-interface f0/0` to make source starting point as f0/0, which is where a device would be => `telnet 29.29.123.1`
  - R3) `telnet 29.29.1231` to check only R4 can telnet to R1
- numbered 2
  - R1)`vty 0 4` => `no access-class 1 in` to uncheck the access-class 1
  - R3) `access-list 1 permit 29.29.44.0 0.0.0.255` => `access 1 deny any` usually this is added as default when adding access-list => `int s1/0.34` => `ip access-group 1 in` to add this group on .34 port to block .44.0 network
  - R4) `telnet 29.29.123.1` to check f0/0 is still open => `ip telnet source-interface s1/0.34` => `do telnet 29.29.123.1` to check that this port is blocked
- named 1
  - R3) `no access 1` to remove the access list => `ip access-list standard R4NET` => `deny 29.29.4.0 0.0.0.255` => `permit any` to deny 4.0 and permit all others => `int s1/0.34` => `ip access-group R4NET in`
  - R4) `telnet 29.29.123.1` => check this is allowed since telnet source-interface is still within the permitted range => `ip telnet so f0/0` to check this is denied
- named 2
  - R3) `sh ip access-lists` to check the access list and to be able to make use of the named access list which is being able to modify the access list - numbered access lists need to be removed and recreated => `ip access-list standard R4NET` => `15 deny 29.29.34.4 0.0.0.0` => `sh ip access-lists` to check the list. If saved and reboot R3, it will reorder the access list => `ip access standard R4NET` => `no 20` => `sh ip access` to check the updated list
  - R4) `tel net 29.29.123.1` to check that current source-interface is not permitted
- numbered access list - dos attack block from outside to inside
  - R3) access-list 100 deny icmp any any echo` => `access-list 100 permit ip any any` => `int s/0.34` => `ip access 100 in`
  - R4) `ping 29.29.1.1 repeat 10` to check it's blocked
- named acess list - dos attack block
  - R3) `ip access-list extended DOSATTACK` => `deny icmp any any echo` => `deny icmp any any echo-reply` => `permit ip any any` => `int s1/0.34` => `no ip access-group 100 in` => `ip access-group DOSATTACK in`
  - R4) `ping 29.29.1.1 repeat 10` to check that out to in is blocked => `ping 29.29.4.1` to check in to out is also blocked
- do it at once
  - R3) remove all access lists => `access-list 110 permit icmp any any echo` => `access-list 110 permit icmp any any echo-reply` => `int s1/0.34` => `rate-limit input access-group 110 8000 1500 2000 conform-action transmit exceed-action drop`
  - R4) `ping 29.29.1.1 repeat 100` => to check it's stopped in between

### dhcp attack
- gns
  - R1) `ip add 1.1.10.254 255.255.255.0`
  - IOU) `int e0/3` => `duplex full` in principle, it should half duplex, so use this only for this lab => `conf t` => `int range e0/0 , e0/1 , e0/2` => `sh` => `int e0/0` => `no sh` => `sw m a` => `int e0/2` => `sw m a` => `sw port-security` => `sw port-security mac-address => (kali mac address)
  - IOU) `sh errdisable detect` => `sh interface status errdisable` then you will see e0/2 err-disabled psecure-violation => remove sw port-security config => `sw port` => `sw port max 3` => `sw port violation protect` for warning. other options are restrict and shutdown => `sh errdisable detect`
  - IOU) `errdisable detect cause security-violation shutdown vlan` => `sh errdisable detect` to check it's disabled => remove the previous command and check if it's back to enable
  - IOU) `errodisable recovery cause security-violation` => `errdisable recovery internval 30` to shutdown port and no sh the port automatically after 30 sec => `int e0/2` => sh and no sh => `do sh run int e0/2`  
- kali
  - `ifconfig eth0 1.1.10.1 netmask 255.255.255.0 broadcast 1.1.10.255` => `service networking restart` => `dhcpx -i eth0 -D 1.1.10.100` to attack  
  - use stick mac address so that when the machine turned off, addresses are not erased => `sw port mac-address sticky (mac address)`

- R3) `ip access extended R4_NOWEB` => `permit udp any host 29.29.1.100 eq 53` host is 0.0.0.0 => `deny tcp host 29.29.4.4 host 29.29.1.100 eq 80` protocol start dest port-num => `permit ip any any` => `ip access R4_net in`
- R3) `ip access-list extended R4_noweb` => `20 deny tcp 29.29.4.0 0.0.0.255 host 29.29.1.100 eq 80` => `20 deny tcp 29.29.4.0 0.0.0.255 host 29.29.1.100 eq 80 time-range WORK_HOUR` => `time-range WORK_HOUR` => `periodic weekdays 09:00 to 18:00`
- R3) `clock set 15:40:00 29 august 2022` => 

### access-list extended
- R3) `ip access-list extended NORMAL_SITE` => `permit udp 29.29.4.0 0.0.0.255 host 29.29.1.100 eq 53` => `permit tcp 29.29.4.0 0.0.0.255 host 29.29.1.100 eq 80` => `permit tcp 29.29.4.0 0.0.0.255 host 29.29.1.100 eq 20` => add port 21 => => add 22 => `permit ip any any` => `int s1/0.34` => `ip access-group NORMAL_SITE in`
- R3) `ip access-list extended NORMAL_SITE` => `5 permit udp any any eq 520` => `6 permit ip host 29.29.34.4 host 224.0.0.9` => `no 70` => `70 deny ip any any`

### ospf
- R1) `router ospf 31` => `router-id 31.31.1.1` => `net 31.31.1.1 0.0.0.0 area 0` => `net 31.31.12.1 0.0.0.0 area 0` number zero is only for backbone
- do the same for R2, R3, and R4
- R1) `sh ip ospf neighbor` to check the list => `sh ip ospf int s1/0` to check the network type is non broadcast => `router ospf 31` => `neighbor 31.31.12.2`
- R2) same config as above with `nei 31.31.23.3`
- R1) `int s1/0` => `ip ospf hello-interval 20` to change from default value 30 => `clear ip ospf process` to apply the different hello interval, and it will not be neighbored with R2 because of different hello-intervals
- R1) `int s1/0` => `ip ospf network point-to-point` to reduce time the be neighbored
- R2) same as above because both routers need to have the same network type as point to point or broadcast
- R2) `int s1/0.23` => `sh` => `debug ip ospf adj` => `debug ip ospf events`
- R1-4) `ip ospf net point-to-p` to stop ospf displaying lo0 ip as /32 address

#### ospf exercise
- ospf config
- eigrp config
- R2) `int s1/0.12` => `ip ospf pri 10` => `clear ip ospf pro` to refresh the DR/BDR state, and check that R2 is DR, not R1
- R2) `redistribute eigrp 31 subnets` to redistribute eigrp to ospf network
- R2) redistribute ospf to eigrp network => `int s1/0.23` => `ip summary-address eigrp 31 31.31.0.0 255.255.240.0`
- blocking overwriting summary address
  - R2) `ip prefix-list ospfNet deny 31.31.0.0/20` => `ip prefix-list ospfNet permit 0.0.0.0/3 le 32` => `router ospf 31` => `distribute-list prefix ospfNet out eigrp 31`  - R1) `clear ip ro \*` => `sh ip route` to check that 31.31.0.0/20 is now gone
- another way
  - R2) `ip prefix-list NET3 permit 31.31.3.0/24` => `ip prefix-list NET4 permit 31.31.4.0/24` => `ip prefix-list NET23 permit 31.31.23.0/24` => `ip prefix-list NET34 permit 31.31.34.0/24` => `ip prefix-list LOOP5 permit permit 5.5.8.0/22` => `route-map eigrpNet` => `match ip add prefix-list NET3 NET4 NET23 NET34 LOOP5` => `router ospf 31` => `redistribute eigrp 31 route-map eigrpNet subnets`
  - R1) `sh ip route` to check it's not overwritten

#### ospf multiple areas
- ospf config as per different areas network => neighbor => loopback p-to-p
- R2) create lo5 with secondary addresses => `redistribute connected route-map LOOP5 subnets` => `route-map LOOP5` => `match int lo5`
- R3) `sh ip ro` to check LOOP5 network is OE2
- change from static port to dynamic port
  - R2) `redi conn subnet route-map LO5 metric-type 1`
- declare stub
  - go to ABR => R2) `area 12 stub` => go to stub => R1) `area 12 stub` => `sh ip ro` to check that it only shows directly connected parts
  - R2) `area 12 stub no-summary` => go to stub => R1) `sh ip ro` to check the summary
- nssa
  - R3) `router os 1` => `area 34 nssa default-information-originate`
  - R4) `router os 1` => `area 34 nssa
- real nssa
  - R3) `router os 1` => `area 34 nssa no-summ` + `no-redistribution`  after erasing previous config

#### ospf areas exercise
- vlan config => trunking => ether channeling => ip config => ospf config
- SW1) `int f1/0` => `ip os pri 0` so that switch doesn't participate in selecting DR device => `int vlan 100` => `ip os pri 10` so that it becomes DR between SW1 and SW3
- make sure stubs have 0 priority
- R2-3) `int s1/0.2` => `ip os net point-to-p`

- 
  - `router os 1` 
  - `auto-cost reference-bandwidth 1000` which is 10^9 to make sure that the calculation is not just based on default value of 10^8 which may not result in natural numbers but fraction numbers

### firewall
- new vm => install os later => other linux 2.6 x kernel => 10 GB => mem 512 => 6 cores per processors => sound/usb remove => select iso => start vtm
- enter => start => config multiple network adapters => regular config => ip config per network adapter => yes to additional functionality => yes to erase existing data in the disk => reboot
- root/netw0rk => root/password => windows => IE => https://192.168.10.254:4444
- interfaces => new interfaces => type ethernet standard => eth2 => 10.5.1.114 => check and refresh => make the status check so that red turns green
- UTM => `route add default gw 10.0.0.1` 
- support => tools => 10.0.0.1 check (GW IP)
- network protection => firewall => sources => new rule => folder img => internal network to source => any to service => any to destination => action allow => save
- nat
  - rule type dnat (destination) => traffic from any => using service dns => going toexternal add => action change the destination to => add network definition => name dns-svr => type host => ipv4 192.168.20.200 => save
  - new nat rule => matching condition => using service http => gonig to external (add) => action => change the detination to => add network definition => name web-svr => type host => 192.168.20.200 => save
  - green button => refresh
  - web protection => web filtering => block url


- o Access-Rule
 1. Inside -> Outside : 모든 트래픽 허용
 2. Inside -> DMZ : DNS, HTTP, HTTPs, SMTP, POP3, IMAP, FTP
 3. DMZ -> Inside : 없음(모든 트래픽 차단)
 4. DMZ -> Outside : DNS, SMTP
 5. Outside -> Inside : 없음(모든 트래픽 차단)
 6. Outside -> DMZ : DNS, HTTP, HTTPs, SMTP, FTP

### ospf project + mail server
- vtp => vlan => trunking => ip config => next hop test
- isp) `ip access-list standard standard KEDU` => `permit host 1.1.100.2` => `permit 2.2.70.0 0.0.0.255` => `ip nat in so list KEDU int f1/0 over` => `int f0/0` => `ip nat in` => `int f0/1` => `ip nat in` => `int f1/0` => `ip nat out` => `ip route 2.2.70.0 255.255.255.0 f0/1 1.1.100.6` => `ip route 1.1.200.0 255.255.255.0 f0/0 1.1.100.2` => `ip route 0.0.0.0 0.0.0.0 f1/0 10.0.0.1`
- dmz) `ip route 0.0.0.0 0.0.0.0 f1/15 1.1.100.5`
- ce) `ip access standard KEDUNET` => `permit 192.168.10.0 0.0.0.255` => `permit 192.168.60.0 0.0.0.255` => `exit` => `ip nat so li KEDUNET int f1/0 over` => `ip nat in` and `out` => `ip nat in so static 192.168.50.101` => `ip nat in so static 192.168.50.102 1.1.200.2` => `ip route 192.168.60.0 255.255.255.0 s2/0 211.104.54.2` => `ip route 0.0.0.0 0.0.0.0 f1/0 1.1.100.1`
- hqced) `ip route 0.0.0.0 0.0.0.0 s1/0 211.104.54.1`
- ce) `default-information originate always`  

### aaa server
- dsw1) `ip domain-name kedu-edu` => `crypto key generate rsa` => `768` => `ip ssh version 2` => `username admin password password` => `line vty 0 4` => `transport input ssh` => `enable secret password` => `username admin15 privilege 15 password cisco123` => `ip http server` => `ip http authentication local` => `ip http secure-server` => `aaa new-model` => `tacacs host 192.168.50.103 single-conection key cisco123` => `aaa authentication login default group tacacs local`
- authentication
  - dsw1) `aaa authen login VTY_ACC group tacacs local` => `line vty 0 4` => `login authen VTY_ACC`
- authorization
  - dsw1) `aaa author commands 5 VTY_PRI group tacas local` => `aaa author commands 15 VTY_PRI group tacacs local`
- accounting
  - dsw1) `aaa accounting exec ACC start-stop group tacacs` => `aaa account commands 15 ACC start-stop group tacacs` => `line vty 0 4` => `accounting exec ACC` => `accounting commands 15 ACC

### port forwarding
- `ip route 0.0.0.0 0.0.0.0 50.50.50.2` => `ip access standard name` => `permit 192.168.1.0 0.0.0.255` => `ip nat in so stat tcp 192.168.1.10 80 50.50.50.1 80`

### firewall
-`ip access-list INGRESS` => `permit udp any host 100.1.1.250 eq 53` => `permit tcp any host 100.1.1.250 eq 80` => `deny ip any any` => `int f0/1` => `ip access-group INGRESS in` => `21 permit ospf host 1.1.100.6 any` => `sh ip access` => `22 permit tcp any 100.1.1.0 0.0.0.255 established`
- wireshark capture => `ip.addr == 100.1.1.1 && tcp.port == 23` => win701 telnet 1.1.100.6 port23 => check wireshark
- racl - reflexive acl
  - remove existing access config => `ip access ex RACL->OUT` => `permit tcp any any reflect RACL_T` => `permit udp any any reflect RACL_T` => `permit icmp any any reflect RACL_T` => `permit ip any any` => `ip access ex RACL->IN` => `permit ospf host 1.1.100.6 any` => `permit udp any host 100.1.1.250 eq 53` => `permit tcp any host 100.1.1.250 eq 80` => `evaluate RACL_T` => `int f0/1` => `ip access RACL->OUT out` => `ip access RACL->IN in` => try to connect to web then check ip access `do sh ip access` to see numerous lists of permits created automatically
- dacl - dynamic acl
  - `ip access ex RACL->IN` => `41 permit tcp any host 1.1.100.5 eq 23` => `42 dynamic applythis permit tcp any host 1.1.100.1 eq 23` => `line vty 0 4` => `login local` => `autocommand access-enable host timeout 10` => `username reuser pass cisco123` => win702 telnet port23 1.1.100.5
- cbac - context based access control
  - `ip inspect name CBAC_T tcp` + `udp` + `icmp` => `ip access ex OUT->IN` => `permit ospf host 1.1.100.6 any` => `permit udp any host 100.1.1.250 eq 53` => `permit tcp any host 100.1.1.250 eq 80` => `permit ip any any`
  - `int f0/1` => `ip access OUT->IN in` => `ip insepct CBAC_T out`
- url filter
  - `ip urlfilter exclusive-domain deny .naver.com` => `ip urlfilter allow-mode on` => `ip urlfilter audit-trail` => `ip inspect name CBAC http urlfilter` + `https`
- firewall zone
  - `zone security Inside` => `zone security DMZ` => `zone security Outside` => `int f0/0` => `zone-member security Inside` => `int f2/0` => `zone-member security Inside` and do it for all the other ports => 

### asa firewall
- `show firewall` to check it's a router mode  => `firewall transparent` to change it to l2 mode => `no firewall transparent` to change back to l3
- `show route` to check routing table => `show nameif` to check interface names
- `hostname ASA` => `enable password cisco123` => `int g 0` => `desc ##Inside_Network##` => `ip add 200.1.1.254 255.255.255.0` => `nameif Inside` => `security-level 100` => `int g 1` => `desc ##Outside_Network##` => `ip add 1.1.100.1 255.255.255.252` => `nameif Outside` => `security-level 0` => `no sh` => `int g 2` => `desc ##DMZ\_Network##` => `ip add 100.1.1.254 255.255.255.0` => `name if DMZ` => `security-level 50` => `no sh`
- `route Inside 200.2.2.0 255.255.255.0 200.1.1.2` => `route Outside 0 0 1.1.100.2

### Debugging
- (serial interface) cdp run => int s1/0 => cdp en
- (in case of mis config of int) => no... => no ip add => sh => exit => no int s1/0.23 => int s1/0.233
- for multipoint and main interface cannot ping selves => `fram map ip 14.14.11.1 102` this is to self ping test to check if ports are on or off

#### clear
- `clear arp`
- `clear mac`
- `clear ip ospf process`

#### show
- sh ip int br
- sh fram map
- sh run int s1/0
- sh int tr
- sh ip protocol	: shows the protocol information
- sh clock
- sh ip access		: shows permits and access list 
- sh ip nat tr		: shows nat translation
- sh ip prefix		: shows prefix-list per hops
- sh route-map
- `sh spanning vlan 10 brief`
- sh arp		: show mac address table
- sh fram map
- sh fram pvc | in DLCI
- sh vlan-s		: show vlan-switch
- sh int trunk		: show interface trunk
- sh cdp n		: sh cdp (cisco discovery protocol) neighbor
- sh ip route		: show ip route
- sh controller s1/0	: check sex of interface ports
- sh fram lmi 		: CCITT => q933a
- sh ip nat trans	: show ip nat info
- sh run | begin nat	: show ip nat config
- sh vtp status`
- sh standby		: show standby vGW
- `sh ether summary` 	: show ether channel summary
- `sh route-map`
- `sh ip rip database`
- `sh ip protocol`
- `sh ip eigrp topology summary`
- `sh ip eigrp topology detail`
- `sh ip access-list`
- `sh ip access-group`
- `sh errdisable detect`
- `sh ip protocol`
- `sh ip ospf nei`
- `sh ip ospf int s1/0`
- `sh ip ospf database` +  `router` + `network` or `summary` + `asbr-summary` + `external`
- `sh zone security`
- `sh crypto isakmp sa`

#### debug
- debug arp 		: ARP packet debug on <=> no ...
- deb ip pack 		: debugging on 
- debug ppp (options)	: debug on for ppp
- debug ip rip		: uses broadcast
- un all		: undebug all
- `deb sp event` => kill interface => watch event logs
- `deb sp bpdu`
- `deb ip dhcp server events` => `deb dhcp`
