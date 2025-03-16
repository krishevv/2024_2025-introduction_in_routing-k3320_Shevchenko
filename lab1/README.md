University: ITMO University

Faculty: FICT

Course: Introduction in routing

Year: 2024/2025

Group: K3320

Author: Shevchenko Kristina

Lab: Lab1

Date of create: 14.12.2024

Date of finished: 15.03.2025

# Отчет по лабораторной работе №1 "Установка ContainerLab и развертывание тестовой сети связи"

## Цель работы

Ознакомиться с инструментом ContainerLab и методами работы с ним, изучить работу VLAN, IP адресации и т.д.

## Ход работы

### 1. Создание трехуровневой сети связи



```

name: lab1

topology:
  nodes:
    R01:
      kind: vr-ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 192.168.50.2

    Linux1:
      kind: linux
      image: alpine:latest
      cmd: sleep infinity
      mgmt-ipv4: 192.168.50.3

    Linux2:
      kind: linux
      image: alpine:latest
      cmd: sleep infinity
      mgmt-ipv4: 192.168.50.4

    SW01.L3.01:
      kind: vr-ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 192.168.50.5

    SW02.L3.01:
      kind: vr-ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 192.168.50.6

    SW02.L3.02:
      kind: vr-ros
      image: vrnetlab/mikrotik_routeros:6.47.9
      mgmt-ipv4: 192.168.50.7

  links:
      - endpoints: ["R01:eth1", "SW01.L3.01:eth1"]
      - endpoints: ["SW01.L3.01:eth2", "SW02.L3.01:eth1"]
      - endpoints: ["SW02.L3.01:eth2", "Linux1:eth1"]
      - endpoints: ["SW01.L3.01:eth3", "SW02.L3.02:eth1"]
      - endpoints: ["SW02.L3.02:eth2", "Linux2:eth1"]

mgmt:
  network: static
  ipv4-subnet: 192.168.50.0/24

```

С помощью команды `sudo containerlab deploy -t lab1.yaml` развертываем сеть:

![containerlab](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab1/images/image1.png)


### 2. Настройка устройств


1) R01:


![Настройка R01](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab1/images/image2.png)

2) SW01.L3.01


![Настройка SW01.L3.01](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab1/images/image3.png)



3) SW02.L3.01
   
![Настройка SW02.L3.01](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab1/images/image4.png)

4) SW02.L3.02


![Настройка SW02.L3.02](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab1/images/image5.png)


### 3. Проверка работоспособности сети


![Проверка работоспособности](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab1/images/image6.png)

### 4. Диаграмма получившейся сети

![Диаграмма](https://github.com/krishevv/2024_2025-introduction_in_routing-k3320_Shevchenko/raw/main/lab1/images/image7.png)
