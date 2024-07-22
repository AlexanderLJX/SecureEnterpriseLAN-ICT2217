
## Router1
```
! Internal Router Configuration

Host R1

ip dhcp pool LAB
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.10
 dns-server 8.8.8.8
ip dhcp pool STAFF
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.10
 dns-server 8.8.8.8
ip dhcp pool MGMT
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.10
 dns-server 8.8.8.8

ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp excluded-address 192.168.40.1 192.168.40.10

interface GigabitEthernet0/0
ip address 192.168.10.2 255.255.255.252
no shut

interface GigabitEthernet0/1
ip address 192.168.10.9 255.255.255.252
no shut

interface GigabitEthernet0/2
ip address 192.168.10.13 255.255.255.252
no shut

! Default route to the firewall
ip route 0.0.0.0 0.0.0.0 192.168.10.1


router ospf 1
 network 192.168.10.8 0.0.0.3 area 0
 network 192.168.10.12 0.0.0.3 area 0
 default-information originate

```

## Router2 (DMZ)
```
! Internal Router Configuration

Host R2

interface GigabitEthernet0/0
 ip address 192.168.10.6 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.5.1 255.255.255.0
 no shutdown

! Default route to the firewall
ip route 0.0.0.0 0.0.0.0 192.168.10.5
```


## Firewall
```
! Firewall Configuration (Example using Cisco ASA)

interface GigabitEthernet0/0
 nameif outside
 security-level 0
 ip address 172.27.47.17 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 nameif inside
 security-level 100
 ip address 192.168.10.1 255.255.255.252
 no shutdown

interface GigabitEthernet0/2
 nameif dmz
 security-level 50
 ip address 192.168.10.5 255.255.255.252
 no shutdown

interface GigabitEthernet0/3
 nameif jumphost
 security-level 100
 ip address 192.168.3.1 255.255.255.252
 no shutdown

object network obj_any
 subnet 192.168.0.0 255.255.0.0
 nat (inside,outside) dynamic interface

object network public_pool_inside
 range 129.126.164.32 129.126.164.34
 nat (inside,outside) dynamic public_pool_inside

nat (inside,outside) source dynamic obj_any public_pool_inside

object network public_pool_dmz
 range 129.126.164.35 129.126.164.37
 nat (dmz,outside) dynamic public_pool_dmz

object network dmz_network
 subnet 192.168.5.0 255.255.255.0
 nat (dmz,outside) dynamic public_pool_dmz

! Static NAT for jumphost
! object network JUMPHOST_OUTSIDE_NAT
!  host 192.168.3.2
!  nat (jumphost,outside) static 129.126.164.38

object network JUMPHOST
 host 192.168.3.2
 nat (jumphost,outside) static interface service tcp 22 22


! Access list to allow traffic from the internal network to the outside
! access-list outside_access_in extended permit tcp any host 129.126.164.38 eq ssh
! access-list outside_access_in extended permit icmp any host 129.126.164.38
access-list outside_access_in extended permit tcp 172.27.47.16 255.255.255.252 host 192.168.3.2 eq ssh
access-list outside_access_in extended permit ip any any
access-group outside_access_in in interface outside

! Access list for jumphost interface
! access-list jumphost_access_in extended permit tcp any host 192.168.3.2 eq ssh
! access-list jumphost_access_in extended permit ip 192.168.3.0 255.255.255.252 any
! access-list jumphost_access_in extended deny ip any any log
! access-group jumphost_access_in in interface jumphost

! Access list to allow traffic from the DMZ to the outside
access-list dmz_access_in extended permit udp any host 8.8.8.8 eq 53
access-list dmz_access_in extended deny udp any any eq 53
access-list dmz_access_in extended permit ip any any
access-group dmz_access_in in interface dmz
!
access-list out_access_in extended permit udp any host 8.8.8.8 eq 53
access-list out_access_in extended deny udp any any eq 53
access-list out_access_in extended permit ip any any
access-group out_access_in in interface inside

! Access list for ping traffic
access-list traffic_out permit icmp any any
access-list traffic_in permit icmp any any
access-list traffic_dmz permit icmp any any
access-list traffic_jumphost permit icmp any any
access-group traffic_out in interface outside
access-group traffic_in in interface inside
access-group traffic_dmz in interface dmz
access-group traffic_jumphost in interface jumphost

! Route to the internal network
route outside 0.0.0.0 0.0.0.0 172.27.47.18
route inside 192.168.20.0 255.255.255.0 192.168.10.2
route inside 192.168.30.0 255.255.255.0 192.168.10.2
route inside 192.168.40.0 255.255.255.0 192.168.10.2
route dmz 192.168.5.0 255.255.255.0 192.168.10.6

! Allow DMZ to access the internal network if needed
access-list dmz_to_inside extended permit ip 192.168.5.0 255.255.255.0 192.168.0.0 255.255.255.0
access-group dmz_to_inside in interface dmz

! Allow for DNS
dns domain-lookup outside
dns server-group DefaultDNS
 name-server 8.8.8.8
 name-server 8.8.4.4
```

## Switch1 (L3S1) [Layer 3]
```
Host L3S1

ip routing

interface GigabitEthernet1/0/1
no switchport
ip address 192.168.10.10 255.255.255.252
no shutdown

interface GigabitEthernet1/0/21
switchport mode access
switchport access vlan 100
no shut

interface range GigabitEthernet1/0/3-20
shutdown

interface GigabitEthernet1/0/2
switchport mode trunk
no shut

interface GigabitEthernet1/0/22
desc Admin Network
switchport mode trunk
no shut

interface GigabitEthernet1/0/23
desc Staff Network
switchport mode trunk
no shut

interface GigabitEthernet1/0/24
desc Lab Network
switchport mode trunk
no shut

vlan 20
interface vlan 20
des LAB
ip address 192.168.20.8 255.255.255.0
ip helper-address 192.168.10.9
standby version 2
standby 20 ip 192.168.20.10
standby 20 priority 50
standby 20 preempt

vlan 30
interface vlan 30
des STAFF
ip address 192.168.30.8 255.255.255.0
ip helper-address 192.168.10.9
standby version 2
standby 30 ip 192.168.30.10
standby 30 priority 50
standby 30 preempt

vlan 40
interface vlan 40
des MGMT
ip address 192.168.40.8 255.255.255.0
ip helper-address 192.168.10.9
standby version 2
standby 40 ip 192.168.40.10
standby 40 priority 100
standby 40 preempt

vlan 100
interface vlan 100
des Mgmt-intf
ip address 192.168.100.245 255.255.255.0

router ospf 1
network 192.168.10.8 0.0.0.3 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
network 192.168.40.0 0.0.0.255 area 0

```

## Switch2 (L3S2) [Layer 3]

```
Host L3S2

ip routing


interface GigabitEthernet1/0/1
no switchport
ip address 192.168.10.14 255.255.255.252
no shutdown

interface range GigabitEthernet1/0/3-20
shutdown

interface GigabitEthernet1/0/21
switchport mode access
switchport access vlan 100
no shut

interface GigabitEthernet1/0/22
desc Admin Network
switchport mode trunk
no shut

interface GigabitEthernet1/0/2
switchport mode trunk
no shut

interface GigabitEthernet1/0/23
desc Staff Network
switchport mode trunk
no shut

interface GigabitEthernet1/0/24
desc Lab Network
switchport mode trunk
no shut

interface vlan 20
des LAB
ip address 192.168.20.9 255.255.255.0
ip helper-address 192.168.10.13
standby version 2
standby 20 ip 192.168.20.10
standby 20 priority 100
standby 20 preempt

interface vlan 30
des STAFF
ip address 192.168.30.9 255.255.255.0
ip helper-address 192.168.10.13
standby version 2
standby 30 ip 192.168.30.10
standby 30 priority 100
standby 30 preempt

interface vlan 40
des MGMT
ip address 192.168.40.9 255.255.255.0
ip helper-address 192.168.10.13
standby version 2
standby 40 ip 192.168.40.10
standby 40 priority 50
standby 40 preempt

vlan 100
interface vlan 100
des Mgmt-intf
ip address 192.168.100.246 255.255.255.0


router ospf 1

 network 192.168.10.12 0.0.0.3 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 default-information originate


```
## Switch 3 (L2S3) [Layer 2]

```
hostname L2S3

enable password faith

username wendell password 0 odom
aaa new-model

aaa authentication login default group tacacs+ local

ip routing

ip domain-name grp7


interface FastEthernet0
 no ip address
 no ip route-cache

interface GigabitEthernet1/0/1
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/2
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/3
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/4
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/5
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/6
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/7
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/8
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/9
 shutdown

interface GigabitEthernet1/0/10
 shutdown

interface GigabitEthernet1/0/11
 shutdown

interface GigabitEthernet1/0/12
 shutdown

interface GigabitEthernet1/0/13
 shutdown

interface GigabitEthernet1/0/14
 shutdown

interface GigabitEthernet1/0/15
 shutdown

interface GigabitEthernet1/0/16
 shutdown

interface GigabitEthernet1/0/17
 shutdown

interface GigabitEthernet1/0/18
 shutdown

interface GigabitEthernet1/0/19
 shutdown

interface GigabitEthernet1/0/20
 shutdown

interface GigabitEthernet1/0/21
 shutdown

interface GigabitEthernet1/0/22
 shutdown

interface GigabitEthernet1/0/23
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/24
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/25
 shutdown

interface GigabitEthernet1/0/26
 shutdown

interface GigabitEthernet1/0/27
 shutdown

interface GigabitEthernet1/0/28
 shutdown

interface Vlan1
 no ip address

interface Vlan100
 ip address 192.168.100.247 255.255.255.240

ip default-gateway 192.168.100.241
ip http server
ip http secure-server

ip ssh version 2

tacacs server TACACSVR
 address ipv4 192.168.100.253
 key 1

line con 0
 password hope
line vty 0 4
 password love
 transport input ssh
line vty 5 15
 password love
 transport input ssh


```

## Switch 4 (L2S4) [Layer 2]

```
Host L2S4

vlan 30
name STAFF

vlan 40
name MGMT

vlan 100
interface vlan 100
name Mgmt-intf
ip addr 192.168.100.248 255.255.255.240

int ran f0/1-12
switchport mode access
switchport access vlan 30
no shut

int ran f0/13-24
switchport mode access
switchport access vlan 40
no shut

int ran g0/1-2
no shut
```

## Switch 5 (L2S5) [Layer 2]

```
Host L2S5

vlan 20
name LAB

vlan 100
interface vlan 100
name Mgmt-intf
ip addr 192.168.100.249

int ran f0/1-24
switchport mode access
switchport access vlan 20
no shut

int ran g0/1-2
no shut
```

vrf definition Mgmt-intf
address-family ipv4

int <port>
vrf forwarding Mgmt-intf
ip addr 192.168.100.<> 255.255.255.240

## AAA

```
Authentication

aaa authentication login default group tacacs+ group radius local

Authorization

aaa authorization exec default group tacacs+ group radius local

Accounting

aaa accounting exec default start-stop group tacacs+ group radius
aaa accounting commands all default start-stop group tacacs+


User - manager
Support - instructors, staff and admin staff
JR-Admin - network manager
Admin - network administrator

local
username USER privilege 1 secret cisco
privilege exec level 5 ping
username SUPPORT privilege 5 secret cisco5
privilege exec level 10 reload
username JR-ADMIN privilege 10 secret cisco10
username ADMIN privilege 15 secret cisco123

parser view SHOWVIEW
secret cisco
commands exec include show version
exit

parser view VERIFYVIEW
secret cisco5
commands exec include ping
exit

parser view RELOADVIEW
secret cisco10
commands exec include reload
exit

parser view CONFIGVIEW
secret cisco15
commands exec include configure terminal
commands configure include interface

parser view USER superview
secret cisco
view SHOWVIEW
exit

parser view SUPPORT superview
secret cisco1
view SHOWVIEW
view VERIFYVIEW
exit

parser view JR-ADMIN superview
secret cisco2
view SHOWVIEW
view VERIFYVIEW
view RELOADVIEW
exit

parser view ADMIN superview
secret cisco3
view SHOWVIEW
view VERIFYVIEW
view RELOADVIEW
view CONFIGVIEW
exit

