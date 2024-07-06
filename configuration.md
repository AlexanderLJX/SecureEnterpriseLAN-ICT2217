
## Router1
```
! Internal Router Configuration

Host R1

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
ip route 0.0.0.0 0.0.0.0 192.168.10.9
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

object network obj_any
 subnet 192.168.0.0 255.255.255.0
 nat (inside,outside) dynamic interface

! If you have a specific public IP pool, you can configure it as follows
object network public_pool
 range 129.126.164.33 129.126.164.38
 nat (inside,outside) dynamic public_pool

nat (inside,outside) source dynamic obj_any public_pool

object network dmz_network
 subnet 192.168.5.0 255.255.255.0
 nat (dmz,outside) dynamic interface

! Access list to allow traffic from the internal network to the outside
access-list outside_access_in extended permit ip any any
access-group outside_access_in in interface outside

! Access list to allow traffic from the DMZ to the outside
access-list dmz_access_in extended permit ip any any
access-group dmz_access_in in interface dmz

! Access list for ping traffic
access-list traffic_out permit icmp any any
access-list traffic_in permit icmp any any
access-group traffic_out in interface outside
access-group traffic_in in interface inside

! Route to the internal network
route outside 0.0.0.0 0.0.0.0 172.27.47.18
route inside 192.168.0.0 255.255.255.0 192.168.10.2
route dmz 192.168.5.0 255.255.255.0 192.168.10.9

! Allow DMZ to access the internal network if needed
access-list dmz_to_inside extended permit ip 192.168.5.0 255.255.255.0 192.168.0.0 255.255.255.0
access-group dmz_to_inside in interface dmz
```

## Switch1
```
Host L3S1

vlan 20
int vlan 20
des Lab

vlan 30
int vlan 30
des staff

vlan 40
int vlan 40
des management


interface GigabitEthernet1/0/1
no switchport
ip address 192.168.10.10 255.255.255.252
no shutdown

interface range GigabitEthernet1/0/2-21
shutdown

interface GigabitEthernet1/0/22
desc Admin Network
switchport mode trunk
switchport trunk allowed vlan 40
no shut

interface GigabitEthernet1/0/23
desc Staff Network
switchport mode trunk
switchport trunk allowed vlan 30
no shut

interface GigabitEthernet1/0/24
desc Lab Network
switchport mode trunk
switchport trunk allowed vlan 20
no shut

interface vlan 20
ip address 192.168.20.8 255.255.255.0
standby 20 ip 192.168.20.10
standby 20 priority 110
standby 20 preempt

interface vlan 30
ip address 192.168.30.8 255.255.255.0
standby 30 ip 192.168.30.10
standby 30 priority 110
standby 30 preempt

interface vlan 40
ip address 192.168.40.8 255.255.255.0
standby 40 ip 192.168.40.10
standby 40 priority 100
standby 40 preempt

```

## Switch2

```
Host L3S2

vlan 20
int vlan 20
des Lab

vlan 30
int vlan 30
des staff

vlan 40
int vlan 40
des management


interface GigabitEthernet1/0/1
no switchport
ip address 192.168.10.14 255.255.255.252
no shutdown

interface range GigabitEthernet1/0/2-21
shutdown

interface GigabitEthernet1/0/22
desc Admin Network
switchport mode trunk
switchport trunk allowed vlan 40
no shut

interface GigabitEthernet1/0/23
desc Staff Network
switchport mode trunk
switchport trunk allowed vlan 30
no shut

interface GigabitEthernet1/0/24
desc Lab Network
switchport mode trunk
switchport trunk allowed vlan 20
no shut

interface vlan 20
ip address 192.168.20.9 255.255.255.0
standby 20 ip 192.168.20.10
standby 20 priority 100
standby 20 preempt

interface vlan 30
ip address 192.168.30.9 255.255.255.0
standby 30 ip 192.168.30.10
standby 30 priority 100
standby 30 preempt

interface vlan 40
ip address 192.168.40.9 255.255.255.0
standby 40 ip 192.168.40.10
standby 40 priority 110
standby 40 preempt
```