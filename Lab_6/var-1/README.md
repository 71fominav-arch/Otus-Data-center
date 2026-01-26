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

После построения 

   
