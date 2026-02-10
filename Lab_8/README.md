### Лабораторная работа №8     
### Задание.
Реализовать передачу суммарных префиксов через EVPN route-type 5.    

### Схема соединения.      

![Схема коммутации](Схема_L2.jpg).      


Предположим у нас есть три клиента, которых нужно обьеденить. Клиент 1 (Vlan 90, 100) и клиент 2 (Vlan 70,71) находятся за коммутатором Switch1.    
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













