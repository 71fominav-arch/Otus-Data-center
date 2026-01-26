Домашнее задание
VxLAN. L3VNI

Цель:
Настроить маршрутизацию в рамках Overlay между клиентами.
В данной работе хочется реализовать два варианта Asymmetric IRB и Symmetric IRB.     

![Схема коммутации](Схема_L2.jpg)     
![Схема L3](Схема_VXLan.jpg)      

В POD1 реализуется  Asymmetric IRB.    
### UNDERLAY строим на OSPF
Пример для P1-Leaf-1    
router ospf 100  
   router-id 10.16.1.250  
   bfd default  
   passive-interface default  
   no passive-interface Ethernet5  
   no passive-interface Ethernet6  
   network 10.16.1.250/32 area 0.0.0.0  
Интерфейсы Ethernet5 и Ethernet6 на P1-Spine-1 и P1-Spine-2.       

### OVERLAY строим на iBGP 
Пример для P1-Leaf-1     
router bgp 65001
   router-id 10.16.1.250  
   timers bgp 3 9  
   maximum-paths 2 ecmp 2  
   neighbor UNDERLAY peer group  
   neighbor UNDERLAY remote-as 65001   
   neighbor UNDERLAY update-source Loopback0   
   neighbor UNDERLAY send-community extended   
   neighbor 10.16.17.250 peer group UNDERLAY   
   neighbor 10.16.18.250 peer group UNDERLAY  
exit   

### После построения L3 связности между Leaf и Spine настраиваем VXLan

vlan 90    (создаем Vlan)       
   name Servers-1     
exit  

interface Vxlan1  ( создаем Vxlan )    
   vxlan source-interface Loopback0   
   vxlan udp-port 4789   
   vxlan vlan 90 vni 10090 (настраиваем vlan и связываем с vni)     
exit  
router bgp 65001 ( добавляем информацию о vlan в bgp )      
vlan 90   
      rd auto   
      route-target both 90:10090   
      redistribute learned   
   !    
   address-family evpn   
      neighbor UNDERLAY activate  
exit     

Убеждаемся, что все работает.
Взамодействие между хостами  192.168.90.10 и 192.168.90.30 Vlan90, 192.168.100.20 и 192.168.100.40.

### Настройка L3VNI.     
vlan 100    (Добавляе Vlan)
   name Servers-2    
exit   

ip virtual-router mac-address 02:00:00:00:00:00    
interface Vlan90   (Поднимаем маршрутизацию)      
   ip address 192.168.90.1/24   
   ip virtual-router address 192.168.90.254/24    
exit   

interface Vlan100   (Поднимаем маршрутизацию)      
   ip address 192.168.100.1/24    
   ip virtual-router address 192.168.100.254/24      
 exit     
 
[Конфигурации оборудования](./POD1_CFG).    

Убеждаемся что есть взаимодействие между хостами. 

P1-Leaf-1#show bgp evpn route-type mac-ip   
BGP routing table information for VRF default    
Router identifier 10.16.1.250, local AS number 65001   
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP    
                    c - Contributing to ECMP, % - Pending BGP convergence    
Origin codes: i - IGP, e - EGP, ? - incomplete    
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop   
   
          Network                Next Hop              Metric  LocPref Weight  Path    
 * >      RD: 10.16.1.250:90 mac-ip 0050.7966.680d    
                                 -                     -       -       0       i    
 * >      RD: 10.16.1.250:90 mac-ip 0050.7966.680d 192.168.90.10 -                     -       -       0       i     
 * >Ec    RD: 10.16.2.250:100 mac-ip 0050.7966.680e     
   >                             10.16.2.250           -       100     0       i Or-ID: 10.16.2.250 C-LST: 10.16.18.250   
 *  ec    RD: 10.16.2.250:100 mac-ip 0050.7966.680e   
                                 10.16.2.250           -       100     0       i Or-ID: 10.16.2.250 C-LST: 10.16.17.250    
 * >Ec    RD: 10.16.2.250:100 mac-ip 0050.7966.680e 192.168.100.20   
                                 10.16.2.250           -       100     0       i Or-ID: 10.16.2.250 C-LST: 10.16.18.250   
 *  ec    RD: 10.16.2.250:100 mac-ip 0050.7966.680e 192.168.100.20    
                                 10.16.2.250           -       100     0       i Or-ID: 10.16.2.250 C-LST: 10.16.17.250    
 * >Ec    RD: 10.16.3.250:90 mac-ip 0050.7966.680f    
                                 10.16.3.250           -       100     0       i Or-ID: 10.16.3.250 C-LST: 10.16.18.250    
 *  ec    RD: 10.16.3.250:90 mac-ip 0050.7966.680f    
                                 10.16.3.250           -       100     0       i Or-ID: 10.16.3.250 C-LST: 10.16.17.250   
 * >Ec    RD: 10.16.3.250:90 mac-ip 0050.7966.680f 192.168.90.30
                                 10.16.3.250           -       100     0       i Or-ID: 10.16.3.250 C-LST: 10.16.18.250    
 *  ec    RD: 10.16.3.250:90 mac-ip 0050.7966.680f 192.168.90.30    
                                 10.16.3.250           -       100     0       i Or-ID: 10.16.3.250 C-LST: 10.16.17.250    
 * >Ec    RD: 10.16.3.250:100 mac-ip 0050.7966.6810    
                                 10.16.3.250           -       100     0       i Or-ID: 10.16.3.250 C-LST: 10.16.17.250    
 *  ec    RD: 10.16.3.250:100 mac-ip 0050.7966.6810    
                                 10.16.3.250           -       100     0       i Or-ID: 10.16.3.250 C-LST: 10.16.18.250     
 * >Ec    RD: 10.16.3.250:100 mac-ip 0050.7966.6810 192.168.100.40    
 Or-ID: 10.16.3.250 C-LST: 10.16.17.250    
 *  ec    RD: 10.16.3.250:100 mac-ip 0050.7966.6810 192.168.100.40     
                                 10.16.3.250           -       100     0       i Or-ID: 10.16.3.250 C-LST: 10.16.18.250


### ping
    













   
