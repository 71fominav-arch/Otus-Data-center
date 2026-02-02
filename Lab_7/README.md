### Лабораторная работа №7

### Задание: Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming.    

### Схема коммутации.       

![Схема коммутации](Схема_L2.jpg)      

### Схема L3.    

![Схема L3](Схема_VXLan.jpg)    


Leaf-1 и Leaf-2 конфигурации взята из предыдущей лабораторной работы. Настроен Mlag. Есть Server-1 (192.168.90.10 Vlan 90), Server-5 (192.168.100.50 Vlan 100).      

Leaf-3 и Leaf-4 настройки из предыдущих лабораторных работ. eBGP номера AS 65003 и 65004. Vxlan1 построен на Lo0. Есть два Vlan 90 и 100. Symmetric ARB (vrf VRF-Router).
Настройки Multihoming.   

Leaf-3    
!     
interface Port-Channel1   
   description to-Servers-host   
   switchport mode trunk   
   !
   evpn ethernet-segment    
      identifier 0000:0000:0000:0000:3401   
      designated-forwarder election algorithm preference 100   
      route-target import 00:00:00:00:34:01    
   lacp system-id 0000.0000.3401    
exit     

Leaf-4       

interface Port-Channel1    
   description to-Servers-host   
   switchport mode trunk    
   !    
   evpn ethernet-segment   
      identifier 0000:0000:0000:0000:3401    
      designated-forwarder election algorithm preference 90    
      route-target import 00:00:00:00:34:01     
   lacp system-id 0000.0000.3401    
   exit      

[Конфигурации стройств](./CFG/)

   
