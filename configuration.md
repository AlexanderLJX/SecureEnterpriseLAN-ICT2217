## JumpHost
```
route add 192.168.100.0 mask 255.255.255.0 192.168.100.253 metric 1 if 49
route add 0.0.0.0 mask 0.0.0.0 192.168.3.1 metric 1 if 13
```
! `route print` to view interface number

## Router3
```
! Internal Router Configuration

Host R3

flow record FLOW-RECORD-1
 match ipv4 source address
 match ipv4 destination address
 match ipv4 protocol
 match transport source-port
 match transport destination-port
 collect counter bytes long
 collect counter packets long

flow exporter FLOW-EXPORTER-1
 destination 192.168.3.2
 transport udp 9996
 source G0/1/0
 export-protocol netflow-v9

flow monitor FLOW-MONITOR-1
 record FLOW-RECORD-1
 exporter FLOW-EXPORTER-1
 cache timeout active 60
 cache timeout inactive 15

interface GigabitEthernet0/0/1
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet0/1/0
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet0/0/0
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

logging trap debugging
logging host 192.168.100.218 vrf Mgmt-intf

ip name-server 8.8.8.8
ntp master 3
ntp authenticate
ntp authentication-key 1 md5 NTPauth123
ntp trusted-key 1
ntp server 0.sg.pool.ntp.org
ntp source GigabitEthernet0

clock timezone SGT 8

vrf definition Mgmt-intf
 description Management VRF
 rd 1:1
 !
 address-family ipv4
 exit-address-family

interface GigabitEthernet0
 description Management Interface
 vrf forwarding Mgmt-intf
 ip address 192.168.100.230 255.255.255.252
 no shutdown

ip route vrf Mgmt-intf 0.0.0.0 0.0.0.0 192.168.100.229

enable secret faith

username Lmanager privilege 1 secret LManager!!
username Lstaff privilege 5 secret LStaff!!
username Lnmanager privilege 10 secret LNetworkManager!!
username Ladmin privilege 15 secret LAdmin!!



aaa new-model

aaa authentication login default group TACACSVR local
aaa authorization exec default group TACACSVR local
aaa accounting exec default start-stop group TACACSVR
aaa accounting commands 15 default stop-only group TACACSVR

tacacs server TACACSVR
 address ipv4 192.168.100.218
 key grp7


aaa group server tacacs+ TACACSVR
 server-private 192.168.100.218 key grp7
 ip vrf forwarding Mgmt-intf
 ip tacacs source-interface GigabitEthernet0


line vty 0 4
 transport input ssh
line vty 5 15
 transport input ssh
 
ip domain-name grp7
crypto key generate rsa modulus 2048
ip ssh version 2

ip dhcp pool LAB
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.10
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

interface GigabitEthernet0/1/0
ip address 192.168.10.2 255.255.255.252
no shut

interface GigabitEthernet0/0/0
ip address 192.168.10.9 255.255.255.252
no shut

interface GigabitEthernet0/0/1
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

flow record FLOW-RECORD-1
 match ipv4 source address
 match ipv4 destination address
 match ipv4 protocol
 match transport source-port
 match transport destination-port
 collect counter bytes long
 collect counter packets long

flow exporter FLOW-EXPORTER-1
 destination 192.168.3.2
 transport udp 9996
 source G0/0
 export-protocol netflow-v9

flow monitor FLOW-MONITOR-1
 record FLOW-RECORD-1
 exporter FLOW-EXPORTER-1
 cache timeout active 60
 cache timeout inactive 15

interface GigabitEthernet0/0
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet0/1
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

logging trap debugging
logging host 192.168.100.218 vrf Mgmt-intf

ntp authenticate
ntp authentication-key 1 md5 NTPauth123
ntp trusted-key 1
ntp server vrf Mgmt-intf 192.168.100.230 key 1

clock timezone SGT 8

vrf definition Mgmt-intf
 description Management VRF
 rd 1:1
 !
 address-family ipv4
 exit-address-family

interface GigabitEthernet0/2
 description Management Interface
 vrf forwarding Mgmt-intf
 ip address 192.168.100.226 255.255.255.252
 no shutdown

ip route vrf Mgmt-intf 0.0.0.0 0.0.0.0 192.168.100.225


enable secret faith

username Lmanager privilege 1 secret LManager!!
username Lstaff privilege 5 secret LStaff!!
username Lnmanager privilege 10 secret LNetworkManager!!
username Ladmin privilege 15 secret LAdmin!!

aaa new-model

aaa authentication login default group TACACSVR local
aaa authorization exec default group TACACSVR local
aaa accounting exec default start-stop group TACACSVR
aaa accounting commands 15 default stop-only group TACACSVR


aaa group server tacacs+ TACACSVR
 server-private 192.168.100.218 key grp7
 ip vrf forwarding Mgmt-intf
 ip tacacs source-interface GigabitEthernet0

tacacs server TACACSVR
 address ipv4 192.168.100.218
 key grp7

username Lmanager secret LManager!
username Linstructor secret LInstructor!
username Lstaff secret LStaff!
username LnetworkManager secret LNetworkManager!
username Ladmin secret LAdmin!

line vty 0 4
 transport input ssh
line vty 5 15
 transport input ssh
 
ip domain-name grp7
crypto key generate rsa modulus 2048
ip ssh version 2

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
! Firewall Configuration (Cisco ASA)

flow-export destination jumphost 192.168.3.2 9996
flow-export template timeout-rate 1
flow-export delay flow-create 60
flow-export active refresh-interval 60

interface GigabitEthernet0/0
 flow-export enable

interface GigabitEthernet0/1
 flow-export enable

interface GigabitEthernet0/2
 flow-export enable

interface GigabitEthernet0/3
 flow-export enable

logging trap debugging
logging host management 192.168.100.218

ntp authenticate
ntp authentication-key 1 md5 NTPauth123
ntp trusted-key 1
ntp server 192.168.100.230 key 1 source management

clock timezone SGT 8

interface Management0/0
 nameif management
 security-level 100
 ip address 192.168.100.222 255.255.255.252
 no shutdown

route management 192.168.100.0 255.255.255.0 192.168.100.221

username Lmanager password LManager!! privilege 1 
username Lstaff password LStaff!! privilege 5
username Lnmanager password LNetworkManager!! privilege 10
username Ladmin password LAdmin!! privilege 15

aaa-server TACACS protocol tacacs+
aaa-server TACACS (management) host 192.168.100.218
 key 1

aaa authentication ssh console TACACS LOCAL

ssh version 2

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
 security-level 80
 ip address 192.168.3.1 255.255.255.252
 no shutdown

object network obj_any
 subnet 192.168.0.0 255.255.0.0
 nat (inside,outside) dynamic interface

object network public_pool_inside
 range 129.126.164.32 129.126.164.35
 nat (inside,outside) dynamic public_pool_inside

nat (inside,outside) source dynamic obj_any public_pool_inside

object network WEB_SERVER
  host 192.168.5.2
  nat (dmz,outside) static 129.126.164.38

! Static NAT for jumphost
object network JUMPHOST_OUTSIDE_NAT
 host 192.168.3.2
 nat (jumphost,outside) static 129.126.164.36

! Access list to allow traffic from the internal network to the outside
access-list outside_access_in extended permit tcp 172.27.47.16 255.255.255.252 host 192.168.3.2 eq ssh
access-list outside_access_in extended permit udp any any eq 53
access-list outside_access_in extended permit udp any any eq 123
access-list outside_access_in extended permit tcp any host 192.168.5.2 eq 80
access-list outside_access_in extended permit icmp any host 192.168.5.2 echo
access-list outside_access_in extended permit icmp any host 192.168.3.2 echo
access-list outside_access_in extended deny ip any any log
access-group outside_access_in in interface outside


! Access list to allow traffic from the DMZ to the outside
access-list dmz_access_in extended permit udp any host 8.8.8.8 eq 53
access-list dmz_access_in extended permit tcp any any eq 80
access-list dmz_access_in extended permit icmp any any echo-reply
access-list dmz_access_in extended permit udp host 192.168.10.6 host 192.168.3.2 eq 9996
access-list dmz_access_in extended deny udp any any eq 53
access-list dmz_access_in extended deny ip any any log
access-group dmz_access_in in interface dmz
!
access-list out_access_in extended permit udp any host 8.8.8.8 eq 53
access-list out_access_in extended permit tcp any host 192.168.5.2 eq 80
access-list out_access_in extended permit tcp 192.168.40.0 255.255.255.0 host 192.168.3.2 eq 22
access-list out_access_in extended permit tcp 192.168.30.0 255.255.255.0 any eq 53
access-list out_access_in extended permit tcp 192.168.40.0 255.255.255.0 any eq 53
access-list out_access_in extended permit tcp 192.168.30.0 255.255.255.0 any eq 80
access-list out_access_in extended permit tcp 192.168.40.0 255.255.255.0 any eq 80
access-list out_access_in extended permit icmp any host 192.168.5.2 echo
access-list out_access_in extended permit udp host 192.168.10.2 host 192.168.3.2 eq 9996
access-list out_access_in extended deny udp any any eq 53
access-list out_access_in extended deny ip any any log
access-group out_access_in in interface inside

! Access list for ping traffic
access-list traffic_out permit icmp any any
access-list traffic_in permit icmp any any
access-list traffic_dmz permit icmp any any
access-list traffic_jumphost permit icmp any any
access-list traffic_jumphost extended permit icmp any any echo-reply
access-list traffic_jumphost extended permit udp host 192.168.3.2 any eq 53
access-list traffic_jumphost extended permit tcp host 192.168.3.2 any eq 443
access-list traffic_jumphost extended permit udp host 192.168.3.2 any eq 443
access-group traffic_out in interface outside
access-group traffic_in in interface inside
access-group traffic_dmz in interface dmz
access-group traffic_jumphost in interface jumphost

access-list INSIDE_OUT extended deny ip 192.168.20.0 255.255.255.0 any
access-list INSIDE_OUT extended permit ip any any
access-group INSIDE_OUT out interface inside

access-list OUTSIDE_IN extended permit tcp any any established
access-list OUTSIDE_IN extended permit udp any any eq 53
access-list OUTSIDE_IN extended permit icmp any any echo-reply
access-group OUTSIDE_IN in interface outside

! Define an access list for NetFlow traffic
access-list NETFLOW_TRAFFIC extended permit udp 192.168.0.0 255.255.0.0 host 192.168.3.2 eq 9996
access-list NETFLOW_TRAFFIC extended permit udp 192.168.5.0 255.255.255.0 host 192.168.3.2 eq 9996

! Apply the access list to the inside interface (inbound direction)
access-group NETFLOW_TRAFFIC in interface inside

! Apply the access list to the dmz interface (inbound direction)
access-group NETFLOW_TRAFFIC in interface dmz

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


! Additional security features

ip verify reverse-path interface outside

threat-detection basic-threat
threat-detection statistics
threat-detection statistics host
threat-detection scanning-threat shun

! SYN flood protection
tcp-map TCP_MAP
 tcp-options range 6 7 allow
 tcp-options range 9 18 allow
 tcp-options range 20 255 allow
 urgent-flag allow
 syn-data allow
 seq-past-window allow
policy-map GLOBAL_POLICY
 class TCP_CLASS
  set connection advanced-options TCP_MAP

! botnet traffic filter
dynamic-filter enable
dynamic-filter use-database

! packet capture for troubleshooting
capture TRAFFIC interface inside match ip any any
capture TRAFFIC interface outside match ip any any

! Implement timeouts for idle connections
timeout conn 1:00:00 half-closed 0:10:00 udp 0:02:00 icmp 0:00:02
timeout sunrpc 0:10:00 h323 0:05:00 h225 1:00:00 mgcp 0:05:00 mgcp-pat 0:05:00
timeout sip 0:30:00 sip_media 0:02:00 sip-invite 0:03:00 sip-disconnect 0:02:00
timeout uauth 0:05:00 absolute

!Enable logging
logging enable
logging timestamp
logging buffer-size 1048576
logging console warnings
logging monitor warnings
logging trap informational
logging asdm informational
logging facility 22
logging host management 192.168.100.218

! Implement MPF (Modular Policy Framework) for inspection
class-map INSPECT_CLASS
 match default-inspection-traffic
policy-map GLOBAL_POLICY
 class INSPECT_CLASS
  inspect dns
  inspect ftp
  inspect h323 h225
  inspect h323 ras
  inspect rsh
  inspect rtsp
  inspect sqlnet
  inspect skinny
  inspect sunrpc
  inspect xdmcp
  inspect sip
  inspect netbios
  inspect tftp
  inspect ip-options

```

## Switch1 (L3S1) [Layer 3]
```
Host L3S1

vrf definition NETVRF
 description Management VRF
 rd 1:2
 !
 address-family ipv4
 exit-address-family


enable secret faith

username Lmanager privilege 1 secret LManager!!
username Lstaff privilege 5 secret LStaff!!
username Lnmanager privilege 10 secret LNetworkManager!!
username Ladmin privilege 15 secret LAdmin!!

aaa group server tacacs+ TACACSVR
 server-private 192.168.100.218 key grp7
 ip vrf forwarding NETVRF
 ip tacacs source-interface GigabitEthernet1/0/3
!
aaa authentication login default group TACACSVR local
aaa authorization exec default group TACACSVR local
aaa accounting exec default start-stop group TACACSVR
aaa accounting commands 15 default stop-only group TACACSVR

flow record FLOW-RECORD-1
 match ipv4 source address
 match ipv4 destination address
 match ipv4 protocol
 match transport source-port
 match transport destination-port
 collect counter bytes long
 collect counter packets long

flow exporter FLOW-EXPORTER-1
 destination 192.168.100.254 vrf NETVRF
 transport udp 9996
 source G1/0/3
 export-protocol netflow-v9

flow monitor FLOW-MONITOR-1
 record FLOW-RECORD-1
 exporter FLOW-EXPORTER-1
 cache timeout active 60
 cache timeout inactive 15

interface GigabitEthernet1/0/22
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet1/0/23
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet1/0/24
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet1/0/1
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet1/0/2
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

logging trap debugging
logging host 192.168.100.218 vrf NETVRF

ntp authenticate
ntp authentication-key 1 md5 NTPauth123
ntp trusted-key 1
ntp server vrf NETVRF 192.168.100.230 key 1

clock timezone SGT 8

ip routing


tacacs server TACACSVR
 address ipv4 192.168.100.218
 key grp7


line vty 0 4
 transport input ssh
line vty 5 15
 transport input ssh
 
ip domain-name grp7
crypto key generate rsa modulus 2048
ip ssh version 2

vrf definition MGMT
 description Management VRF
 rd 1:1
 !
 address-family ipv4
 exit-address-family

interface GigabitEthernet0/0
 vrf forwarding Mgmt-vrf
 shutdown

ip route vrf NETVRF 0.0.0.0 0.0.0.0 192.168.100.233

interface GigabitEthernet1/0/1
no switchport
ip address 192.168.10.10 255.255.255.252
no shutdown

interface GigabitEthernet1/0/21
switchport mode access
switchport access vlan 100
no shut

interface GigabitEthernet1/0/3
no switchport
vrf forwarding NETVRF
ip address 192.168.100.234 255.255.255.252
no shutdown

interface range GigabitEthernet1/0/4-20
shutdown

interface GigabitEthernet1/0/2
switchport mode trunk
no shut

interface GigabitEthernet1/0/11
 no switchport
 description Connection to Jump Host
 vrf forwarding MGMT
 ip address 192.168.100.253 255.255.255.252
 no shutdown

interface GigabitEthernet1/0/12
 no switchport
 description Connection to TACACS
 vrf forwarding MGMT
 ip address 192.168.100.217 255.255.255.252
 no shutdown

interface GigabitEthernet1/0/13
 no switchport
 description Connection to Firewall
 vrf forwarding MGMT
 ip address 192.168.100.221 255.255.255.252
 no shutdown

interface GigabitEthernet1/0/14
 no switchport
 description Connection to R2
 vrf forwarding MGMT
 ip address 192.168.100.225 255.255.255.252
 no shutdown

interface GigabitEthernet1/0/15
 no switchport
 description Connection to R3
 vrf forwarding MGMT
 ip address 192.168.100.229 255.255.255.252
 no shutdown

interface GigabitEthernet1/0/16
 no switchport
 description Connection to S1 (Itself)
 vrf forwarding MGMT
 ip address 192.168.100.233 255.255.255.252
 no shutdown

interface GigabitEthernet1/0/17
 no switchport
 description Connection to S2
 vrf forwarding MGMT
 ip address 192.168.100.237 255.255.255.252
 no shutdown

interface GigabitEthernet1/0/18
 no switchport
 description Connection to S3
 vrf forwarding MGMT
 ip address 192.168.100.241 255.255.255.252
 no shutdown

interface GigabitEthernet1/0/19
 no switchport
 description Connection to S4
 vrf forwarding MGMT
 ip address 192.168.100.245 255.255.255.252
 no shutdown

interface GigabitEthernet1/0/20
 no switchport
 description Connection to S5
 vrf forwarding MGMT
 ip address 192.168.100.249 255.255.255.252
 no shutdown

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

router ospf 1
network 192.168.10.8 0.0.0.3 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
network 192.168.40.0 0.0.0.255 area 0
default-information originate

privilege exec level 5 ping
privilege exec level 10 enable
privilege exec level 15 configure terminal
privilege exec level 15 configure
privilege exec level 10 reload
privilege exec level 15 show running-config
privilege exec level 1 show

line con 0
 password team
 stopbits 1
line aux 0
 stopbits 1
line vty 0 4
 password seven
 transport input ssh
line vty 5 15
 password seven
 transport input ssh

```

## Switch2 (L3S2) [Layer 3]

```
Host L3S2

vrf definition NETVRF
 description Management VRF
 rd 1:2
 !
 address-family ipv4
 exit-address-family

flow record FLOW-RECORD-1
 match ipv4 source address
 match ipv4 destination address
 match ipv4 protocol
 match transport source-port
 match transport destination-port
 collect counter bytes long
 collect counter packets long

flow exporter FLOW-EXPORTER-1
 destination 192.168.100.254 vrf NETVRF
 transport udp 9996
 source G1/0/3
 export-protocol netflow-v9

flow monitor FLOW-MONITOR-1
 record FLOW-RECORD-1
 exporter FLOW-EXPORTER-1
 cache timeout active 60
 cache timeout inactive 15

interface GigabitEthernet1/0/22
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet1/0/23
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet1/0/24
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet1/0/1
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

interface GigabitEthernet1/0/2
 ip flow monitor FLOW-MONITOR-1 input
 ip flow monitor FLOW-MONITOR-1 output

logging trap debugging
logging host 192.168.100.218 vrf NETVRF

ntp authenticate
ntp authentication-key 1 md5 NTPauth123
ntp trusted-key 1
ntp server vrf NETVRF 192.168.100.230 key 1

clock timezone SGT 8

ip routing


enable secret faith

username Lmanager privilege 1 secret LManager!!
username Lstaff privilege 5 secret LStaff!!
username Lnmanager privilege 10 secret LNetworkManager!!
username Ladmin privilege 15 secret LAdmin!!

aaa new-model

aaa authentication login default group TACACSVR local
aaa authorization exec default group TACACSVR local
aaa accounting exec default start-stop group TACACSVR
aaa accounting commands 15 default stop-only group TACACSVR

tacacs server TACACSVR
 address ipv4 192.168.100.218
 key grp7

line con 0
 password team
 stopbits 1
line aux 0
 stopbits 1
line vty 0 4
 password seven
 transport input ssh
line vty 5 15
 password seven
 transport input ssh
 
ip domain-name grp7
crypto key generate rsa modulus 2048
ip ssh version 2

vrf definition MGMT
 description Management VRF
 rd 1:1
 !
 address-family ipv4
 exit-address-family

interface GigabitEthernet0/0
 vrf forwarding Mgmt-vrf
 shutdown

ip route vrf NETVRF 0.0.0.0 0.0.0.0 192.168.100.237

interface GigabitEthernet1/0/1
no switchport
ip address 192.168.10.14 255.255.255.252
no shutdown

interface GigabitEthernet1/0/3
no switchport
vrf forwarding NETVRF
ip address 192.168.100.238 255.255.255.252
no shutdown

interface range GigabitEthernet1/0/4-20
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

logging trap debugging
logging host 192.168.100.218

ntp authenticate
ntp authentication-key 1 md5 NTPauth123
ntp trusted-key 1
ntp server 192.168.100.230 key 1 source Vlan100

clock timezone SGT 8

enable secret faith

username Lmanager privilege 1 secret LManager!!
username Lstaff privilege 5 secret LStaff!!
username Lnmanager privilege 10 secret LNetworkManager!!
username Ladmin privilege 15 secret LAdmin!!

aaa new-model

aaa authentication login default group TACACSVR local
aaa authorization exec default group TACACSVR local
aaa accounting exec default start-stop group TACACSVR
aaa accounting commands 15 default stop-only group TACACSVR

tacacs server TACACSVR
 address ipv4 192.168.100.218
 key grp7

 line con 0
 password team
 stopbits 1
line aux 0
 stopbits 1
line vty 0 4
 password seven
 transport input ssh
line vty 5 15
 password seven
 transport input ssh

privilege exec level 5 ping
privilege exec level 10 enable
privilege exec level 15 configure terminal
privilege exec level 15 configure
privilege exec level 10 reload
privilege exec level 15 show running-config
privilege exec level 1 show
 
ip domain-name grp7
crypto key generate rsa modulus 2048
ip ssh version 2

interface FastEthernet0
 no ip address
 no ip route-cache

interface range GigabitEthernet1/0/1-13
 switchport access vlan 20
 switchport mode access

interface range GigabitEthernet1/0/14-21
 shutdown

interface GigabitEthernet1/0/22
 description Uplink to S1
 switchport mode access
 switchport access vlan 100
 no shutdown

interface range GigabitEthernet1/0/23-28
 shutdown

interface Vlan1
 no ip address

vlan 20
name LAB

vlan 100
interface Vlan100
 description Management Interface
 ip address 192.168.100.242 255.255.255.252
 no shut

ip default-gateway 192.168.100.241

#ACL
ip access-list extended ALLOWED_SUBNETS
 permit ip 192.168.20.0 0.0.0.255 any
 permit ip 192.168.30.0 0.0.0.255 any
 permit ip 192.168.40.0 0.0.0.255 any
 permit ip 192.168.100.0 0.0.0.255 any
 permit ip 192.168.3.0 0.0.0.255 any
 permit ip 192.168.5.0 0.0.0.255 any
 permit ip 192.168.10.0 0.0.0.255 any
 deny   ip any any
ip access-list extended BLOCK_INTERNET_VLAN20
 deny   ip 192.168.10.0 0.0.0.255 any
 permit ip any any
int ran g1/0/1-24
ip access-group ALLOWED_SUBNETS in

int vlan20
ip access-group BLOCK_INTERNET_VLAN20 out


#DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 20
int range g1/0/1 - 22 
ip dhcp snooping limit rate 3
int range g1/0/23 - 24
ip dhcp snooping trust

#Dynamic ARP inspection (DAI)
int range g1/023 - 24
ip arp inspection trust



```

## Switch 4 (L2S4) [Layer 2]

```
hostname L2S4

logging trap debugging
logging host 192.168.100.218

ntp authenticate
ntp authentication-key 1 md5 NTPauth123
ntp trusted-key 1
ntp server 192.168.100.230 key 1 source Vlan100

clock timezone SGT 8

enable secret faith

username Lmanager privilege 1 secret LManager!!
username Lstaff privilege 5 secret LStaff!!
username Lnmanager privilege 10 secret LNetworkManager!!
username Ladmin privilege 15 secret LAdmin!!

aaa new-model

aaa authentication login default group tacacs+ local
aaa authorization exec default group tacacs+ local
aaa accounting exec default start-stop group tacacs+
aaa accounting commands 15 default stop-only group tacacs+

tacacs server TACACSVR
 address ipv4 192.168.100.218
 key grp7


privilege exec level 5 ping
privilege exec level 10 enable
privilege exec level 15 configure terminal
privilege exec level 15 configure
privilege exec level 10 reload
privilege exec level 15 show running-config
privilege exec level 1 show

line con 0
 password team
 stopbits 1
line aux 0
 stopbits 1
line vty 0 4
 password seven
 transport input ssh
line vty 5 15
 password seven
 transport input ssh

ip domain-name grp7
crypto key generate rsa modulus 2048
ip ssh version 2

interface range FastEthernet0/1-13
 switchport access vlan 20
 switchport mode access

interface range FastEthernet0/14-23
 shutdown

interface FastEthernet0/24
 description Uplink to S1
 switchport mode access
 switchport access vlan 100
 no shutdown

int ran g0/1-2
shutdown

interface Vlan1
 no ip address

vlan 20
 name LAB

vlan 100
interface Vlan100
 description Management Interface
 ip address 192.168.100.246 255.255.255.252
 no shut

ip default-gateway 192.168.100.245

#ACL
ip access-list extended ALLOWED_SUBNETS
 permit ip 192.168.20.0 0.0.0.255 any
 permit ip 192.168.30.0 0.0.0.255 any
 permit ip 192.168.40.0 0.0.0.255 any
 permit ip 192.168.100.0 0.0.0.255 any
 permit ip 192.168.3.0 0.0.0.255 any
 permit ip 192.168.5.0 0.0.0.255 any
 permit ip 192.168.10.0 0.0.0.255 any
 deny   ip any any
ip access-list extended BLOCK_INTERNET_VLAN20
 deny   ip 192.168.10.0 0.0.0.255 any
 permit ip any any
int ran g1/0/1-24
ip access-group ALLOWED_SUBNETS in

int vlan20
ip access-group BLOCK_INTERNET_VLAN20 out


#DHCP Snooping and DAI
ip dhcp snooping
ip dhcp snooping vlan 20
int range fa0/1 - 23
ip dhcp snooping limit rate 3
int range g0/1 - 2
ip dhcp snooping trust
ip arp inspection trust



```

## Switch 5 (L2S5) [Layer 2]

```
Host L2S5

logging trap debugging
logging host 192.168.100.218

ntp authenticate
ntp authentication-key 1 md5 NTPauth123
ntp trusted-key 1
ntp server 192.168.100.230 key 1 source Vlan100

clock timezone SGT 8

enable secret faith

username Lmanager privilege 1 secret LManager!!
username Lstaff privilege 5 secret LStaff!!
username Lnmanager privilege 10 secret LNetworkManager!!
username Ladmin privilege 15 secret LAdmin!!

aaa new-model

aaa authentication login default group tacacs+ local
aaa authorization exec default group tacacs+ local
aaa accounting exec default start-stop group tacacs+
aaa accounting commands 15 default stop-only group tacacs+

tacacs server TACACSVR
 address ipv4 192.168.100.218
 key grp7

 
privilege exec level 5 ping
privilege exec level 10 enable
privilege exec level 15 configure terminal
privilege exec level 15 configure
privilege exec level 10 reload
privilege exec level 15 show running-config
privilege exec level 1 show

line con 0
 password team
 stopbits 1
line aux 0
 stopbits 1
line vty 0 4
 password seven
 transport input ssh
line vty 5 15
 password seven
 transport input ssh
 
ip domain-name grp7
crypto key generate rsa modulus 2048
ip ssh version 2

int ran f0/1-12
switchport mode access
switchport access vlan 30
no shut

int ran f0/13-23
switchport mode access
switchport access vlan 40
no shut

interface FastEthernet0/24
 description Uplink to S1
 switchport mode access
 switchport access vlan 100
 no shutdown

int ran g0/1-2
shutdown

vlan 30
name STAFF

vlan 40
name MGMT

vlan 100
interface vlan 100
 description Management Interface
 ip addr 192.168.100.250 255.255.255.252
 no shutdown

ip default-gateway 192.168.100.249

#DHCP snooping and DAI
ip dhcp snooping
ip dhcp snooping vlan 30
int range fa0/1 - 12
ip dhcp snooping limit rate 3

ip dhcp snooping
ip dhcp snooping vlan 40
int range fa0/13 - 23
ip dhcp snooping limit rate 3

int range g0/1 - 2
ip dhcp snooping trust
ip arp inspection trust

```