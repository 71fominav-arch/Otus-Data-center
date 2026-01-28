### Лабораторная работа 6. Вариант 2.

Целью данной работы стало изучение технологии VXLAN, EVPN-VXLAN, L3VNI, MLAG.    
Оборудование связано между собой через eBGP, EVPN через eBGP. Leaf-1 и Leaf-2 связаны через iBGP.      
Mlag Настроен на Leaf-1 и Leaf-2.      
Server-1 в Vlan90   
Server-3 в Vlan90 /проверить L2vpn /        
Server-4 в Vlan100 /проверить L3vni схема symmetric ARB/

![Схема коммутации](Схема_L2.jpg)        



![Схема VXLan](Схема_VXLan.jpg).      

####  Описание схемы.

### UNDERLAY
Строим на протоколе eBGP.    
Пример Leaf-1.     
router bgp 65001   
   router-id 10.16.1.250   
   no bgp default ipv4-unicast   
   timers bgp 3 9    
   distance bgp 20 200 200    
   maximum-paths 2 ecmp 2     
   neighbor UNDERLAY peer group    
   neighbor UNDERLAY remote-as 65501     
   neighbor UNDERLAY out-delay 0      
   neighbor UNDERLAY-MLAG peer group / к соседу Leaf-2 /    
   neighbor UNDERLAY-MLAG remote-as 65001    
   neighbor UNDERLAY-MLAG next-hop-self     
   neighbor 10.16.1.246 peer group UNDERLAY-MLAG      
   neighbor 10.16.17.10 peer group UNDERLAY    
   neighbor 10.16.18.10 peer group UNDERLAY      
   address-family ipv4     
      neighbor UNDERLAY activate    
      neighbor UNDERLAY-MLAG activate    
      network 10.16.1.250/32   
      network 10.16.1.251/32   
end    
### OVERLAY  
interface Loopback1    
   description VXLAN-VTEP      
   ip address 10.16.1.251/32    
   exit     
    vrf VRF-Router  / vrf для маршрутизации между Vlan в одном VRF  /     
      rd 65001:11     
      route-target import evpn 11:111111    
      route-target export evpn 11:111111    
      redistribute connected     
      exit    
  interface Vlan90   / Vlan клиента /     
   vrf VRF-Router    
   ip address 192.168.90.1/24    
   ip virtual-router address 192.168.90.254/24   
   exit
   ip virtual-router mac-address 02:00:00:00:00:00 / виртуальный mac /             
interface Vxlan1    
   vxlan source-interface Loopback1   /строим от Lo1 /
   vxlan udp-port 4789   
   vxlan vlan 90 vni 10090   
   vxlan vrf VRF-Router vni 111111   
   exit     
router bgp 65001     
   neighbor OVERLAY peer group    
   neighbor OVERLAY remote-as 65501     
   neighbor OVERLAY out-delay 0    
   neighbor OVERLAY update-source Loopback0    
   neighbor OVERLAY ebgp-multihop 2    
   neighbor OVERLAY send-community extended    
   neighbor 10.16.17.250 peer group OVERLAY    
   neighbor 10.16.18.250 peer group OVERLAY    
   !       
   vlan 90     
      rd auto    
      route-target both 90:10090 / L2vpn /    
      redistribute learned     
   !       
   address-family evpn     
      neighbor OVERLAY activate    
   !      
   vrf VRF-Router /L3vni/     
      rd 65001:11    
      route-target import evpn 11:111111    
      route-target export evpn 11:111111     
      redistribute connected    
end        
     
interface Port-Channel78  /интерфейс для peerlink между Leaf-1 и Leaf-2/   
   description MLAG-PEERLINK    
   switchport mode trunk    
   switchport trunk group MLAG-PEERLINK    
   spanning-tree link-type point-to-point     
exit     

vrf instance keeplive / для keepalive линка  /     
interface Ethernet9   
   no switchport   
   vrf keeplive   
   ip address 192.168.0.1/30   
exit
interface Vlan4094 / для peerlink Vlan для передачи данных mlag между Leaf-1 и Leaf-2 /       
   no autostate    
   ip address 10.16.1.245/30   
exit     
mlag configuration    
   domain-id Leaves-1-2    
   local-interface Vlan4094    
   peer-address 10.16.1.246   / сосед /   
   peer-address heartbeat 192.168.0.2 vrf keeplive    / сосед /   
   peer-link Port-Channel78   
   dual-primary detection delay 1 action errdisable all-interfaces   
exit     

interface Port-Channel1 /в сторону клиента/    
   description -Link-to-Server-1-    
   switchport trunk allowed vlan 90    
   switchport mode trunk    
   mlag 1 /ссылка на созданный mlag/    
exit         

///////////////////////////////////////////////////////////////////////        
    
    
###  Результат настройки 

[Конфигурация оборудования](./CFG).     
      
### Server-4   ping Server-1 /Server-1 подключен через mlag /      
Server-4> show ip    
    
NAME        : Server-4[1]      
IP/MASK     : 192.168.100.40/24     
GATEWAY     : 192.168.100.254    
DNS         :    
MAC         : 00:50:79:66:68:09    
LPORT       : 20000    
RHOST:PORT  : 127.0.0.1:30000     
MTU         : 1500    
    
Server-4> ping 192.168.90.10 -t    

84 bytes from 192.168.90.10 icmp_seq=1 ttl=62 time=818.699 ms   
192.168.90.10 icmp_seq=2 timeout    
84 bytes from 192.168.90.10 icmp_seq=3 ttl=62 time=149.917 ms    
84 bytes from 192.168.90.10 icmp_seq=4 ttl=62 time=435.830 ms    
84 bytes from 192.168.90.10 icmp_seq=5 ttl=62 time=185.990 ms    
84 bytes from 192.168.90.10 icmp_seq=6 ttl=62 time=228.137 ms    
84 bytes from 192.168.90.10 icmp_seq=7 ttl=62 time=153.888 ms    
84 bytes from 192.168.90.10 icmp_seq=8 ttl=62 time=618.643 ms    
192.168.90.10 icmp_seq=9 timeout    
84 bytes from 192.168.90.10 icmp_seq=10 ttl=62 time=448.327 ms     
84 bytes from 192.168.90.10 icmp_seq=11 ttl=62 time=290.994 ms     
84 bytes from 192.168.90.10 icmp_seq=12 ttl=62 time=185.845 ms     
84 bytes from 192.168.90.10 icmp_seq=13 ttl=62 time=329.991 ms     
84 bytes from 192.168.90.10 icmp_seq=14 ttl=62 time=266.479 ms      
84 bytes from 192.168.90.10 icmp_seq=15 ttl=62 time=417.185 ms     
84 bytes from 192.168.90.10 icmp_seq=16 ttl=62 time=158.384 ms     
84 bytes from 192.168.90.10 icmp_seq=17 ttl=62 time=279.300 ms     
84 bytes from 192.168.90.10 icmp_seq=18 ttl=62 time=194.736 ms      
^C
#### Server-3 ping Server-1            
NAME        : Server-3[1]    
IP/MASK     : 192.168.90.30/24    
GATEWAY     : 192.168.90.254   
DNS         :   
MAC         : 00:50:79:66:68:06   
LPORT       : 20000   
RHOST:PORT  : 127.0.0.1:30000   
MTU         : 1500    

84 bytes from 192.168.90.10 icmp_seq=45 ttl=64 time=678.766 ms    
84 bytes from 192.168.90.10 icmp_seq=46 ttl=64 time=314.633 ms    
84 bytes from 192.168.90.10 icmp_seq=47 ttl=64 time=126.522 ms    
84 bytes from 192.168.90.10 icmp_seq=48 ttl=64 time=253.643 ms    
84 bytes from 192.168.90.10 icmp_seq=49 ttl=64 time=207.427 ms    
84 bytes from 192.168.90.10 icmp_seq=50 ttl=64 time=216.199 ms    
84 bytes from 192.168.90.10 icmp_seq=51 ttl=64 time=155.981 ms    
84 bytes from 192.168.90.10 icmp_seq=52 ttl=64 time=346.562 ms    
84 bytes from 192.168.90.10 icmp_seq=53 ttl=64 time=145.374 ms    
84 bytes from 192.168.90.10 icmp_seq=54 ttl=64 time=206.668 ms    
84 bytes from 192.168.90.10 icmp_seq=55 ttl=64 time=300.361 ms    
84 bytes from 192.168.90.10 icmp_seq=56 ttl=64 time=231.799 ms    
84 bytes from 192.168.90.10 icmp_seq=57 ttl=64 time=575.522 ms    
84 bytes from 192.168.90.10 icmp_seq=58 ttl=64 time=270.958 ms   
84 bytes from 192.168.90.10 icmp_seq=59 ttl=64 time=229.695 ms       


### Диагностические команды.       

[Команды диагностики](./test/).      

По результатам настройки оборудования - все клиенты доступны, как в одном Vlan так и между Vlan90 и Vlan100.     

