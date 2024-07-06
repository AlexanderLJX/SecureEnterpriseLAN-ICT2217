
## Router1
```
! Internal Router Configuration

interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.10.9 255.255.255.252
 no shutdown

interface GigabitEthernet0/2
 ip address 192.168.10.13 255.255.255.252
 no shutdown

! Default route to the firewall
ip route 0.0.0.0 0.0.0.0 192.168.10.1
```

## Router2 (DMZ)
```
! Internal Router Configuration

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

object network obj_any
 subnet 192.168.0.0 255.255.255.0
 nat (inside,outside) dynamic interface

object network public_pool_inside
 range 129.126.164.32 129.126.164.34
 nat (inside,outside) dynamic public_pool_inside

nat (inside,outside) source dynamic obj_any public_pool_inside

object network public_pool_dmz
 range 129.126.164.35 129.126.164.38
 nat (dmz,outside) dynamic public_pool_dmz

object network dmz_network
 subnet 192.168.5.0 255.255.255.0
 nat (dmz,outside) dynamic public_pool_dmz

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
route inside 192.168.20.0 255.255.255.0 192.168.10.2
route inside 192.168.30.0 255.255.255.0 192.168.10.2
route dmz 192.168.5.0 255.255.255.0 192.168.10.9

! Allow DMZ to access the internal network if needed
access-list dmz_to_inside extended permit ip 192.168.5.0 255.255.255.0 192.168.0.0 255.255.255.0
access-group dmz_to_inside in interface dmz
```

## Switch1

## Switch2
