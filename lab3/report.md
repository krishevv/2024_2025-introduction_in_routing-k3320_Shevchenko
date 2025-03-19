University: ITMO University

Faculty: FICT

Course: Introduction in routing

Year: 2024/2025

Group: K3320

Author: Shevchenko Kristina

Lab: Lab3

Date of create: 14.12.2024

Date of finished: 19.03.2025

# Отчет по лабораторной работе №3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"
## Описание

Наша компания "RogaIKopita Games" с прошлой лабораторной работы выросла до серьезного игрового концерна, ещё немного и они выпустят свой ответ Genshin Impact - Allmoney Impact. И вот для этой задачи они купили небольшую, но очень старую студию "Old Games" из Нью Йорка, при поглощении выяснилось что у этой студии много наработок в области компьютерной графики и совет директоров "RogaIKopita Games" решил взять эти наработки на вооружение. К сожалению исходники лежат на сервере "SGI Prism", в Нью-Йоркском офисе никто им пользоваться не умеет, а из-за короновируса сотрудники офиса из Санкт-Петерубурга не могут добраться в Нью-Йорк, чтобы забрать данные из "SGI Prism". Ваша задача подключить Нью-Йоркский офис к общей IP/MPLS сети и организовать EoMPLS между "SGI Prism" и компьютером инженеров в Санк-Петербурге.

## Цель работы

Изучить протоколы OSPF и MPLS, механизмы организации EoMPLS. 



## Ход работы

### 1. Развертывание тестовой сети

```
name: lab3

mgmt:
  network: statics
  ipv4-subnet: 172.20.20.0/24

topology:

  nodes:
    R01.MSK:
      kind: vr-mikrotik_ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 172.20.20.2

    R01.SPB:
      kind: vr-mikrotik_ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 172.20.20.3

    R01.HKI:
      kind: vr-mikrotik_ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 172.20.20.4

    R01.LND:
      kind: vr-mikrotik_ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 172.20.20.5

    R01.NY:
      kind: vr-mikrotik_ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 172.20.20.6

    R01.LBN:
      kind: vr-mikrotik_ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 172.20.20.7

    PC1:
      kind: vr-mikrotik_ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 172.20.20.8

    SGI.Prism:
      kind: vr-mikrotik_ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 172.20.20.9
  

  links:
    - endpoints: ["R01.MSK:eth1","R01.SPB:eth1"]
    - endpoints: ["R01.MSK:eth2","R01.LBN:eth1"]
    - endpoints: ["R01.SPB:eth2","PC1:eth1"]
    - endpoints: ["R01.SPB:eth3","R01.HKI:eth1"]
    - endpoints: ["R01.HKI:eth2","R01.LBN:eth2"]
    - endpoints: ["R01.HKI:eth3","R01.LND:eth1"]
    - endpoints: ["R01.LND:eth2","R01.NY:eth1"]
    - endpoints: ["R01.LBN:eth3","R01.NY:eth2"]
    - endpoints: ["R01.NY:eth3","SGI.Prism:eth1"]

```

### 2. Схема связи
![схема](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab3/images/схема.png)

### 3. Настройка конфигурации устройств
R01.MSK:
```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.1.1/30 interface=ether2 network=10.10.1.0
add address=10.10.3.1/30 interface=ether3 network=10.10.3.0
add address=10.10.10.2 interface=Lo0 network=10.10.10.2
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.2
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.MSK
```


R01.SPB:
```
/interface bridge
add name=EoMPLS
add name=Lo0
/interface vpls
add cisco-style=yes cisco-style-id=666 disabled=no l2mtu=1500 mac-address=02:C3:38:F3:1C:51 name=EoMPLS_VPLS remote-peer=10.10.10.5
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.1
/interface bridge port
add bridge=EoMPLS interface=ether3
add bridge=EoMPLS interface=EoMPLS_VPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.1.2/30 interface=ether2 network=10.10.1.0
add address=10.10.2.1/30 interface=ether4 network=10.10.2.0
add address=192.168.10.1/24 interface=ether3 network=192.168.10.0
add address=10.10.10.1 interface=Lo0 network=10.10.10.1
/ip dhcp-client
add disabled=no interface=ether2
add disabled=no interface=ether1
/ip route
add distance=1 dst-address=192.168.10.0/24 gateway=ether2
add distance=1 dst-address=192.168.20.0/24 gateway=10.10.10.5
/mpls ldp
set enabled=yes transport-address=10.10.10.1
/mpls ldp interface
add interface=ether2
add interface=ether4
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB     
```
R01.HKI:
```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.2.2/30 interface=ether2 network=10.10.2.0
add address=10.10.4.2/30 interface=ether3 network=10.10.4.0
add address=10.10.6.1/30 interface=ether4 network=10.10.6.0
add address=10.10.10.3 interface=Lo0 network=10.10.10.3
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.3
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone
/system identity
set name=R01.HKI
```


R01.LND:
```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.6.2/30 interface=ether2 network=10.10.6.0
add address=10.10.7.1/30 interface=ether3 network=10.10.7.0
add address=10.10.10.4 interface=Lo0 network=10.10.10.4
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.4
/mpls ldp interface
add interface=ether3
add interface=ether2
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```
R01.NY:
```
/interface bridge
add name=EoMPLS
add name=Lo0
/interface vpls
add cisco-style=yes cisco-style-id=666 disabled=no l2mtu=1500 mac-address=02:14:45:15:45:BC name=EoMPLS_VPLS remote-peer=10.10.10.1
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.5
/interface bridge port
add bridge=EoMPLS interface=ether4
add bridge=EoMPLS interface=EoMPLS_VPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.7.2/30 interface=ether2 network=10.10.7.0
add address=10.10.5.2/30 interface=ether3 network=10.10.5.0
add address=192.168.20.1/24 interface=ether4 network=192.168.20.0
add address=10.10.10.5 interface=Lo0 network=10.10.10.5
/ip dhcp-client
add disabled=no interface=ether1
/ip route
add distance=1 dst-address=192.168.10.0/24 gateway=10.10.10.1
add distance=1 dst-address=192.168.20.0/24 gateway=ether2
/mpls ldp
set enabled=yes transport-address=10.10.10.5
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

R01.LBN:
```
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.6
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.3.2/30 interface=ether2 network=10.10.3.0
add address=10.10.4.1/30 interface=ether3 network=10.10.4.0
add address=10.10.5.1/30 interface=ether4 network=10.10.5.0
add address=10.10.10.6 interface=Lo0 network=10.10.10.6
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.6
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone
/system identity
set name=R01.LBN
```

    PC1:
```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.10/24 interface=ether2 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route
```

SGI.Prism:
  ```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.20.10/24 interface=ether2 network=192.168.20.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route
add distance=1 gateway=192.168.20.1
/system identity
set name=SGI_Prism
```

### 4. Результаты пингов

![Пинг](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab3/images/ping1.png)
![Пинг](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab3/images/ping2.png)
![Пинг](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab3/images/ping3.png)


## Вывод:

В результате выполнения лабораторной работы была развернута сеть и была изучена настройка протоколов OSPF и MPLS и механизмы организации EoMPLS.
