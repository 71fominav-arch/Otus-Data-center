### Лабораторная работа №8     
### Задание.
Реализовать передачу суммарных префиксов через EVPN route-type 5.    


### Схема соединения.      

![Схема коммутации](Схема_L2.jpg).      


Предположим у нас есть три клиента, которых нужно обьеденить. Клиент 1 (Vlan 90 192.168.90.0/24, 100  192.168.100.0/24) и клиент 2 (Vlan 70 192.168.70.0/24, 71 192.168.71.0/24 ) находятся за коммутатором Switch1.    
Клиент 3 внешний за Border_router (Vlan 91, 101).
Клиент 1 находится в vrf Client-1. Vni 111111.         
Клиент 2 находится в vrf Client-2. Vni 222222.     
Клиент 3 подключен в отдельный vrf Clients-1-2 на Border_router.
Для передачи информации между клиентами будем использовать Border_router. Border_router подключен к Pod1 через Leaf-3 и Leaf-4.

Между Leaf-3 и Border_router используются два транзитных Vlan 89 (vrf Client-1) Vlan 79 (vrf Client-2).        
Leaf-3    
interface Ethernet1.1    
   encapsulation dot1q vlan 89    
   vrf Client-1     
   ip address 172.16.3.1/30    
exit       
interface Ethernet1.2    
   encapsulation dot1q vlan 79     
   vrf Client-2    
   ip address 172.16.3.5/30     
   exit      

Border_router     
interface Ethernet1.1     
   encapsulation dot1q vlan 89     
   vrf Clients-1-2      
   ip address 172.16.3.2/30    
!      
interface Ethernet1.2     
   encapsulation dot1q vlan 79       
   vrf Clients-1-2     
   ip address 172.16.3.6/30      
exit       
         

Между Leaf-4 и Border_router используются два транзитных Vlan 88 (vrf Client-1) Vlan 78 (vrf Client-2). 
Leaf-4     
interface Ethernet1.1    
   encapsulation dot1q vlan 88    
   vrf Client-1   
   ip address 172.16.4.1/30    
!     
interface Ethernet1.2     
   encapsulation dot1q vlan 78     
   vrf Client-2    
   ip address 172.16.4.5/30    
exit      

Border_router     
interface Ethernet2.1    
   encapsulation dot1q vlan 88   
   vrf Clients-1-2   
   ip address 172.16.4.2/30   
!   
interface Ethernet2.2   
   encapsulation dot1q vlan 78   
   vrf Clients-1-2   
   ip address 172.16.4.6/30   
  exit     

### Схема маршрутизации.      
![схема маршрутизации](./Схема_VXLan.jpg)
            
        
### BGP    
### Leaf-3
router bgp 65003    
  vrf Client-1      
      rd 65003:13    
      route-target import evpn 11:111111    
      route-target export evpn 11:111111    
      neighbor 172.16.3.2 remote-as 65601     
      redistribute connected     
      !    
      address-family ipv4    
         neighbor 172.16.3.2 activate     
         redistribute connected     
   !     
   vrf Client-2     
      rd 65003:23      
      route-target import evpn 11:222222     
      route-target export evpn 11:222222     
      neighbor 172.16.3.6 remote-as 65601      
      redistribute connected      
      !      
      address-family ipv4    
         neighbor 172.16.3.6 activate    
         redistribute connected     
exit    
exit        
#### Leaf-4        
router bgp 65004     
   vrf Client-1      
      rd 65004:14      
      route-target import evpn 11:111111     
      route-target export evpn 11:111111    
      neighbor 172.16.4.2 remote-as 65601    
      redistribute connected     
      !    
      address-family ipv4    
         neighbor 172.16.4.2 activate    
         redistribute connected     
   !     
   vrf Client-2      
      rd 65004:24      
      route-target import evpn 11:222222    
      route-target export evpn 11:222222     
      neighbor 172.16.4.6 remote-as 65601  / Сосед Border_router /         
      redistribute connected      
      !
      address-family ipv4         
         neighbor 172.16.4.6 activate     
         redistribute connected      
exit      
end      
### Border_router    
ip prefix-list BGP_in seq 10 permit 0.0.0.0/0 le 28   / фильтрация маршрутов /    
!
ip route vrf Clients-1-2 192.168.0.0/16 Null0  / объеденяем маршруты  /    
     
router bgp 65601    
   vrf Clients-1-2    
      router-id 172.16.16.250    
      no bgp default ipv4-unicast     
      timers bgp 3 9    
      distance bgp 20 200 200     
      maximum-paths 2 ecmp 2     
      neighbor 172.16.3.1 remote-as 65003     
      neighbor 172.16.3.5 remote-as 65003     
      neighbor 172.16.4.1 remote-as 65004     
      neighbor 172.16.4.5 remote-as 65004     
      !     
      address-family ipv4         
         neighbor 172.16.3.1 activate        
         neighbor 172.16.3.1 prefix-list BGP_in in        
         neighbor 172.16.3.5 activate       
         neighbor 172.16.3.5 prefix-list BGP_in in       
         neighbor 172.16.4.1 activate          
         neighbor 172.16.4.1 prefix-list BGP_in in       
         neighbor 172.16.4.5 activate      
         neighbor 172.16.4.5 prefix-list BGP_in in      
         network 172.16.16.250/32      
         network 172.16.91.0/24       
         network 172.16.101.0/24      
         redistribute connected       
         redistribute static      
!      
end       

##### Готовые конфигурации      

[Конфигурации устройств](./CFG)      

     
По итогу работ взаимодействие между сетями 192.168.90.0/24, 192.168.100.0/24, 192.168.70.0/24, 192.168.71.0/24, 172.16.91.0/24, 172.16.101.0/24 установлено.   

### Border-router#show ip route vrf Clients-1-2     
     
VRF: Clients-1-2     
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
        
 C        172.16.3.0/30 is directly connected, Ethernet1.1     
 C        172.16.3.4/30 is directly connected, Ethernet1.2     
 C        172.16.4.0/30 is directly connected, Ethernet2.1     
 C        172.16.4.4/30 is directly connected, Ethernet2.2       
 C        172.16.16.250/32 is directly connected, Loopback0     
 C        172.16.91.0/24 is directly connected, Vlan91      
 C        172.16.101.0/24 is directly connected, Vlan101       
 B E      192.168.70.0/24 [20/0] via 172.16.3.5, Ethernet1.2     /Client-2/        
                                 via 172.16.4.5, Ethernet2.2      
 B E      192.168.71.0/24 [20/0] via 172.16.3.5, Ethernet1.2     /Client-2/        
                                 via 172.16.4.5, Ethernet2.2      
 B E      192.168.90.0/24 [20/0] via 172.16.3.1, Ethernet1.1     /Client-1/      
                                 via 172.16.4.1, Ethernet2.1     
 B E      192.168.100.0/24 [20/0] via 172.16.3.1, Ethernet1.1    /Client-1/          
                                  via 172.16.4.1, Ethernet2.1         
 S        192.168.0.0/16 is directly connected, Null0      

Border-router#      
        
Leaf-1#show ip route vrf Client-1      
     
VRF: Client-1       
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

 B E      172.16.3.0/30 [20/0] via VTEP 10.16.3.250 VNI 111111 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1      
 B E      172.16.3.4/30 [20/0] via VTEP 10.16.4.250 VNI 111111 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1      
                               via VTEP 10.16.3.250 VNI 111111 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1         
00:f6:ad:37 local-interface Vxlan1        
 B E      172.16.4.4/30 [20/0] via VTEP 10.16.4.250 VNI 111111 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1       
                               via VTEP 10.16.3.250 VNI 111111 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1        
 B E      172.16.16.250/32 [20/0] via VTEP 10.16.4.250 VNI 111111 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1       
                                  via VTEP 10.16.3.250 VNI 111111 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1       
 B E      172.16.91.0/24 [20/0] via VTEP 10.16.4.250 VNI 111111 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1  /внешний клиент /        
                                via VTEP 10.16.3.250 VNI 111111 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1      
 B E      172.16.101.0/24 [20/0] via VTEP 10.16.4.250 VNI 111111 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1 /внешний клиент /      
                                 via VTEP 10.16.3.250 VNI 111111 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1       
 B E      192.168.70.0/24 [20/0] via VTEP 10.16.3.250 VNI 111111 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1 / Client-2 /       
 B E      192.168.71.0/24 [20/0] via VTEP 10.16.3.250 VNI 111111 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1 / Client-2 /       
 C        192.168.90.0/24 is directly connected, Vlan90        
 C        192.168.100.0/24 is directly connected, Vlan100       
 B E      192.168.0.0/16 [20/0] via VTEP 10.16.4.250 VNI 111111 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1  /Суммированный маршрут /      
                                via VTEP 10.16.3.250 VNI 111111 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1  /Суммированный маршрут /     
     
Leaf-1#       

Leaf-1#show ip route vrf Client-2      
      
VRF: Client-2       
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
      
 B E      172.16.3.0/30 [20/0] via VTEP 10.16.3.250 VNI 222222 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1        
                               via VTEP 10.16.4.250 VNI 222222 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1     
 B E      172.16.3.4/30 [20/0] via VTEP 10.16.3.250 VNI 222222 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1      
00:d7:ee:0b local-interface Vxlan1     
                               via VTEP 10.16.4.250 VNI 222222 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1      
 B E      172.16.4.4/30 [20/0] via VTEP 10.16.4.250 VNI 222222 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1      
 B E      172.16.16.250/32 [20/0] via VTEP 10.16.3.250 VNI 222222 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1     
                                  via VTEP 10.16.4.250 VNI 222222 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1     
 B E      172.16.91.0/24 [20/0] via VTEP 10.16.3.250 VNI 222222 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1    /внешний клиент /        
                                via VTEP 10.16.4.250 VNI 222222 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1        
 B E      172.16.101.0/24 [20/0] via VTEP 10.16.3.250 VNI 222222 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1   /внешний клиент /       
                                 via VTEP 10.16.4.250 VNI 222222 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1         
 C        192.168.70.0/24 is directly connected, Vlan70       
 C        192.168.71.0/24 is directly connected, Vlan71          
 B E      192.168.90.0/24 [20/0] via VTEP 10.16.3.250 VNI 222222 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1   / Client-1  /       
 B E      192.168.100.0/24 [20/0] via VTEP 10.16.3.250 VNI 222222 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1   / Client-1  /  
 B E      192.168.0.0/16 [20/0] via VTEP 10.16.3.250 VNI 222222 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1     /Суммированный маршрут /     
                                via VTEP 10.16.4.250 VNI 222222 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1    /Суммированный маршрут /     

Leaf-1#      

Ping между всеми хостами проходит. Если ронять соединение Leaf-3 или Border_router взяимодействие не теряется. Между Leaf-4 и Border_router также.    





