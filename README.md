# OTUS_DC_L3VNI-routing
## Задание
- Разместите двух "клиентов" в разных VRF в рамках одной фабрики.
- Настроите маршрутизацию между клиентами через внешнее устройство (граничный роутер\фаерволл\etc)

![alt-dtp](https://github.com/vk1391/OTUS_DC_L3VNI-routing/blob/main/L3vni%20routing%202.png)
underlay - ibgp

1.Конфигурация Leaf:
 - Leaf-1:
```
vlan 10-11
!
vrf instance L3VNI
!
interface Port-Channel1
   shutdown
!
interface Ethernet1
   no switchport
   ip address 10.1.11.2/30
!
interface Ethernet2
   no switchport
   ip address 10.2.11.2/30
!
interface Ethernet3
   switchport access vlan 11
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 11.11.11.11/32
!
interface Loopback1
   ip address 101.11.11.11/32
!
interface Management1
!
interface Vlan11
   vrf L3VNI
   ip address 10.0.0.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 100
   vxlan vlan 11 vni 111
   vxlan vrf L3VNI vni 1000
!
ip routing
ip routing vrf L3VNI
!
route-map redist permit 10
   match interface Loopback0
!
route-map redist permit 20
   match interface Loopback1
!
router bgp 101
   router-id 11.11.11.11
   timers bgp 3 9
   maximum-paths 2
   neighbor 10.1.11.1 remote-as 101
   neighbor 10.1.11.1 next-hop-self
   neighbor 10.1.11.1 bfd
   neighbor 10.2.11.1 remote-as 101
   neighbor 10.2.11.1 next-hop-self
   neighbor 10.2.11.1 bfd
   neighbor 101.12.12.12 remote-as 101
   neighbor 101.12.12.12 update-source Loopback1
   neighbor 101.12.12.12 send-community extended
   neighbor 101.13.13.13 remote-as 101
   neighbor 101.13.13.13 update-source Loopback1
   neighbor 101.13.13.13 send-community extended
   redistribute connected route-map redist
   !
   vlan 10
      rd 101.13.13.13:1
      route-target both 101:1
      redistribute learned
   !
   vlan 11
      rd 101.13.13.13:11
      route-target both 101:11
      redistribute learned
   !
   address-family evpn
      no neighbor 10.1.11.1 activate
      no neighbor 10.2.11.1 activate
      no neighbor 101.1.1.1 activate
      no neighbor 101.2.2.2 activate
      neighbor 101.12.12.12 activate
      neighbor 101.13.13.13 activate
      no neighbor 101.22.22.22 activate
      no neighbor 101.33.33.33 activate
   !
   address-family ipv4
      neighbor 10.1.11.1 activate
      neighbor 10.2.11.1 activate
      no neighbor 101.1.1.1 activate
      no neighbor 101.2.2.2 activate
      no neighbor 101.12.12.12 activate
      no neighbor 101.13.13.13 activate
   !
   vrf L3VNI
      rd 101.11.11.11:1000
      route-target import 101:1000
      route-target import evpn 101:1000
      route-target export evpn 101:1000
      timers bgp 60 180 min-hold-time 3
      maximum-paths 1 ecmp 128
      !
      address-family ipv4
         redistribute connected
!
end
```
- Leaf2:
```
vlan 10-12
!
vrf instance L3VNI2
!
interface Port-Channel1
!
interface Port-Channel2
!
interface Ethernet1
   no switchport
   ip address 10.1.12.2/30
!
interface Ethernet2
   no switchport
   ip address 10.2.12.2/30
!
interface Ethernet3
   switchport access vlan 12
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 12.12.12.12/32
!
interface Loopback1
   ip address 101.12.12.12/32
!
interface Management1
!
interface Vlan12
   vrf L3VNI2
   ip address 12.0.0.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 12 vni 102
   vxlan vrf L3VNI2 vni 1001
!
ip virtual-router mac-address aa:11:22:33:44:cc
ip virtual-router address subnet-routes
!
ip routing
ip routing vrf L3VNI2
!
route-map redist permit 10
   match interface Loopback0
!
route-map redist permit 20
   match interface Loopback1
!
router bgp 101
   router-id 12.12.12.12
   timers bgp 3 9
   maximum-paths 2
   neighbor 10.1.12.1 remote-as 101
   neighbor 10.1.12.1 bfd
   neighbor 10.2.12.1 remote-as 101
   neighbor 10.2.12.1 bfd
   neighbor 101.11.11.11 remote-as 101
   neighbor 101.11.11.11 update-source Loopback1
   neighbor 101.11.11.11 send-community extended
   neighbor 101.13.13.13 remote-as 101
   neighbor 101.13.13.13 update-source Loopback1
   neighbor 101.13.13.13 send-community extended
   redistribute connected route-map redist
   !
   vlan 12
      rd 101.13.13.13:102
      route-target both 101:102
      redistribute learned
   !
   address-family evpn
      neighbor 101.11.11.11 activate
      neighbor 101.13.13.13 activate
   !
   address-family ipv4
      no neighbor 101.11.11.11 activate
      no neighbor 101.13.13.13 activate
   !
   vrf L3VNI2
      rd 101.11.11.11:1001
      route-target import evpn 101:1001
      route-target export evpn 101:1001
      !
      address-family ipv4
         redistribute connected
!
end
```
- Leaf3:
```
vlan 10,12,100,120
!
vrf instance L3VNI
!
vrf instance L3VNI2
!
interface Port-Channel1
   shutdown
!
interface Port-Channel2
!
interface Ethernet1
   no switchport
   ip address 10.1.13.2/30
!
interface Ethernet2
   no switchport
   ip address 10.2.13.2/30
!
interface Ethernet3
   switchport trunk allowed vlan 100,120
   switchport mode trunk
!
interface Ethernet4
   switchport access vlan 12
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 13.13.13.13/32
!
interface Loopback1
   ip address 101.13.13.13/32
!
interface Management1
!
interface Vlan11
!
interface Vlan100
   vrf L3VNI
   ip address 100.0.0.1/30
!
interface Vlan120
   vrf L3VNI2
   ip address 120.0.0.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vrf L3VNI vni 1000
   vxlan vrf L3VNI2 vni 1001
!
ip virtual-router mac-address aa:11:22:33:44:dd
!
ip routing
ip routing vrf L3VNI
ip routing vrf L3VNI2
!
ip prefix-list Hosts
   seq 10 permit 0.0.0.0/0 ge 32
!
route-map R9 deny 10
   match ip address prefix-list Hosts
!
route-map R9 permit 20
!
route-map redist permit 10
   match interface Loopback0
!
route-map redist permit 20
   match interface Loopback1
!
router bgp 101
   router-id 13.13.13.13
   timers bgp 3 9
   maximum-paths 2
   neighbor 10.1.13.1 remote-as 101
   neighbor 10.1.13.1 bfd
   neighbor 10.2.13.1 remote-as 101
   neighbor 10.2.13.1 bfd
   no neighbor 100.0.0.2 allowas-in
   neighbor 101.11.11.11 remote-as 101
   neighbor 101.11.11.11 next-hop-self
   neighbor 101.11.11.11 update-source Loopback1
   neighbor 101.11.11.11 send-community extended
   neighbor 101.12.12.12 remote-as 101
   neighbor 101.12.12.12 next-hop-self
   neighbor 101.12.12.12 update-source Loopback1
   neighbor 101.12.12.12 send-community extended
   redistribute connected route-map redist
   !
   vlan 100
      rd auto
      route-target both 101:1000
      redistribute learned
   !
   vlan 120
      rd auto
      route-target both 101:1001
      redistribute learned
   !
   address-family evpn
      neighbor 101.11.11.11 activate
      neighbor 101.12.12.12 activate
   !
   address-family ipv4
      no neighbor 100.0.0.2 activate
      no neighbor 101.11.11.11 activate
      no neighbor 101.12.12.12 activate
      no neighbor 120.0.0.2 activate
   !
   vrf L3VNI
      rd 101.13.13.13:1000
      route-target import evpn 101:1000
      route-target export evpn 101:1000
      neighbor 100.0.0.2 remote-as 102
      no neighbor 100.0.0.2 allowas-in
      neighbor 100.0.0.2 route-map R9 out
      !
      address-family ipv4
         neighbor 100.0.0.2 activate
   !
   vrf L3VNI2
      rd 101.13.13.13:1001
      route-target import evpn 101:1001
      route-target export evpn 101:1001
      neighbor 120.0.0.2 remote-as 102
      no neighbor 120.0.0.2 allowas-in
      neighbor 120.0.0.2 route-map R9 out
      !
      address-family ipv4
         neighbor 120.0.0.2 activate
   !
   vrf l
!
end
```
2. Конфигурация роутера, на котором будет производиться vrf leaking:
- R9:
```
interface Loopback0
 no shutdown
 ip address 8.8.8.8 255.255.255.255
!
interface Ethernet0/0
 no shutdown
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/0.100
 no shutdown
 encapsulation dot1Q 100
!
interface Ethernet0/0.120
 no shutdown
 encapsulation dot1Q 120
!
interface Ethernet0/1
 no shutdown
!
interface Ethernet0/2
 no shutdown
!
interface Ethernet0/3
 no shutdown
!
interface Vlan100
 no shutdown
 ip address 100.0.0.2 255.255.255.252
!
interface Vlan120
 no shutdown
 ip address 120.0.0.2 255.255.255.252
!
router bgp 102
 bgp log-neighbor-changes
 neighbor 100.0.0.1 remote-as 101
 neighbor 100.0.0.1 as-override
 neighbor 120.0.0.1 remote-as 101
 neighbor 120.0.0.1 as-override
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
!
!
!
!
!
control-plane
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
!
!
end
```
3. проверка
- Route type 5 LEAF1:
```
localhost#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 11.11.11.11, local AS number 101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 101.11.11.11:1000 ip-prefix 10.0.0.0/30
                                 -                     -       -       0       i
 * >      RD: 101.13.13.13:1000 ip-prefix 10.0.0.0/30
                                 101.13.13.13          -       100     0       102 102 i
 * >      RD: 101.13.13.13:1001 ip-prefix 10.0.0.0/30
                                 101.13.13.13          -       100     0       102 102 i
 * >      RD: 101.11.11.11:1001 ip-prefix 12.0.0.0/30
                                 101.12.12.12          -       100     0       i
 * >      RD: 101.13.13.13:1000 ip-prefix 12.0.0.0/30
                                 101.13.13.13          -       100     0       102 102 i
 * >      RD: 101.13.13.13:1001 ip-prefix 12.0.0.0/30
                                 101.13.13.13          -       100     0       102 102 i
```
- Route type 5 LEAF2:
```
localhost#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 12.12.12.12, local AS number 101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 101.11.11.11:1000 ip-prefix 10.0.0.0/30
                                 101.11.11.11          -       100     0       i
 * >      RD: 101.13.13.13:1000 ip-prefix 10.0.0.0/30
                                 101.13.13.13          -       100     0       102 102 i
 * >      RD: 101.13.13.13:1001 ip-prefix 10.0.0.0/30
                                 101.13.13.13          -       100     0       102 102 i
 * >      RD: 101.11.11.11:1001 ip-prefix 12.0.0.0/30
                                 -                     -       -       0       i
 * >      RD: 101.13.13.13:1000 ip-prefix 12.0.0.0/30
                                 101.13.13.13          -       100     0       102 102 i
 * >      RD: 101.13.13.13:1001 ip-prefix 12.0.0.0/30
                                 101.13.13.13          -       100     0       102 102 i
```
- таблица маршрутизации leaf1:
```
localhost#sh ip route vrf L3VNI 

VRF: L3VNI
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 C        10.0.0.0/30 is directly connected, Vlan11
 B I      12.0.0.0/30 [200/0] via VTEP 101.13.13.13 VNI 1000 router-mac 50:75:74:f3:21:4b local-interface Vxlan1
```
- таблица маршрутизации leaf2:
```
localhost#sh ip route vrf L3VNI2 

VRF: L3VNI2
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B I      10.0.0.0/30 [200/0] via VTEP 101.13.13.13 VNI 1001 router-mac 50:75:74:f3:21:4b local-interface Vxlan1
 C        12.0.0.0/30 is directly connected, Vlan12
```
- трасировка 10.0.0.2 - 12.0.0.2
```
VPCS> sh ip

NAME        : VPCS[1]
IP/MASK     : 10.0.0.2/30
GATEWAY     : 10.0.0.1
DNS         : 
MAC         : 00:50:79:66:68:08
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> trace 12.0.0.2
trace to 12.0.0.2, 8 hops max, press Ctrl+C to stop
 1   10.0.0.1   7.532 ms  6.976 ms  9.010 ms
 2   100.0.0.1   30.182 ms  25.073 ms  27.507 ms
 3   100.0.0.2   35.816 ms  44.143 ms  33.417 ms
 4   120.0.0.1   44.888 ms  50.484 ms  38.037 ms
 5   12.0.0.1   71.289 ms  74.238 ms  61.969 ms
 6   *12.0.0.2   106.673 ms (ICMP type:3, code:3, Destination port unreachable)
```
