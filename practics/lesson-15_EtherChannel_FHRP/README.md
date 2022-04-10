# Занятие №16. Практическое занятие. Организация сети с избыточностью.

## 1. Создание топологии данной сети в программе cisco packet tracer. 

![](lesson_15_net_topology.png)

## 2. Выполнение настроек коммутаторов SW1 - SW3.
По условиям занятия у нас есть 3 VLAN:
- VLAN10 - USERS
- VLAN99 - MANAGEMENT
- VLAN100 - NATIVE

Для автоматизации настройки, разделим команды настроек на блок общих настроек для всех трех коммутаторов, и на индивидуальны блок.
Блок общих настроек включает в себя:
1. Создание VLAN 
2. Создание 2-х Ether-channel интерфейсов, настроенных как транки 3х созданных ранее VLAN.
3. Настройка интерфейса Gi0/1, смотрящего на роутер, в режим транка 3х созданных ранее VLAN.

Команды блока общих настроек (для SW1): 

```
SW1> enable
SW1#configure terminal
SW1(config)#vlan 10
SW1(config-vlan)#name USERS
SW1(config-vlan)#exit
SW1(config)#vlan 99
SW1(config-vlan)#name MANAGEMENT
SW1(config-vlan)#exit
SW1(config)#vlan 100
SW1(config-vlan)#name NATIVE
SW1(config-vlan)#exit
SW1(config)#interface port-channel 1
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport trunk allowed vlan 10,99,100
SW1(config-if)#switchport trunk native vlan 100
SW1(config-if)#exit
SW1(config)#interface port-channel 2
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport trunk allowed vlan 10,99,100
SW1(config-if)#switchport trunk native vlan 100
SW1(config-if)#exit
SW1(config)#interface Gi0/1
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport trunk allowed vlan 10,99,100
SW1(config-if)#switchport trunk native vlan 100
SW1(config-if)#exit
SW1(config)#exit
SW1#wr
SW1#exit

```

Текст для вставки с командную строку коммутатора
```
en
configure terminal
vlan 10
name USERS
exit
vlan 99
name MANAGEMENT
exit
vlan 100
name NATIVE
exit
interface port-channel 1
switchport mode trunk
switchport trunk allowed vlan 10,99,100
switchport trunk native vlan 100
exit
interface port-channel 2
switchport mode trunk
switchport trunk allowed vlan 10,99,100
switchport trunk native vlan 100
exit
interface Gi0/1
switchport mode trunk
switchport trunk allowed vlan 10,99,100
switchport trunk native vlan 100
exit
exit 
wr
exit

```

Блок команд, уникальных для коммутатора SW1:
1. Включение физических интерфейсов fa0/1-3 в port-channel 1 без протокола агрегации.
2. Включение физических интерфейсов fa0/6-7 в port-channel 2 с типом протокола PaGP.
3. Настройка интерфейса VTY коммутатора для MANAGEMENT VLAN: 172.17.0.1/17

Выполним необходимые настройки:
```
SW1> enable
SW1#configure terminal
SW1(config)#interface range fa0/1-3
SW1(config-if-range)#channel-group 1 mode on
SW1(config-if-range)#exit
SW1(config)#interface range fa0/6-7
SW1(config-if-range)#channel-group 2 mode desirable
SW1(config-if-range)#exit
SW1(config)#interface vlan 10
SW1(config-if)#ip address 172.17.0.1 255.255.128.0
SW1(config-if)#no shutdown
SW1(config-if)#exit
SW1(config)#exit
SW1#wr
SW1#exit
```

Текст для вставки с командную строку коммутатора:
```
en
configure terminal
interface range fa0/1-3
channel-group 1 mode on
exit
interface range fa0/6-7
channel-group 2 mode desirable
exit
interface vlan 10
ip address 172.17.0.1 255.255.128.0
no shutdown
exit
exit
wr
exit

```

Блок команд, уникальных для коммутатора SW2:
1. Включение физических интерфейсов fa0/1-3 в port-channel 1 без протокола агрегации.
2. Включение физических интерфейсов fa0/4-5 в port-channel 2 с типом протокола LACP.
3. Настройка интерфейса VTY коммутатора для MANAGEMENT VLAN: 172.17.0.2/17

Выполним необходимые настройки:
```
SW1> enable
SW1#configure terminal
SW1(config)#interface range fa0/1-3
SW1(config-if-range)#channel-group 1 mode on
SW1(config-if-range)#exit
SW1(config)#interface range fa0/4-5
SW1(config-if-range)#channel-group 2 mode active
SW1(config-if-range)#exit
SW1(config)#interface vlan 10
SW1(config-if)#ip address 172.17.0.2 255.255.128.0
SW1(config-if)#no shutdown
SW1(config-if)#exit
SW1(config)#exit
SW1#wr
SW1#exit
```

Текст для вставки с командную строку коммутатора:
```
en
configure terminal
interface range fa0/1-3
channel-group 1 mode on
exit
interface range fa0/4-5
channel-group 2 mode active
exit
interface vlan 10
ip address 172.17.0.2 255.255.128.0
no shutdown
exit
exit
wr
exit

```

Блок команд, уникальных для коммутатора SW3:
1. Включение физических интерфейсов fa0/6-7 в port-channel 1 с типом протокола PaGP.
2. Включение физических интерфейсов fa0/4-5 в port-channel 2 с типом протокола LACP.
3. Настройка интерфейса VTY коммутатора для MANAGEMENT VLAN: 172.17.0.3/17

Выполним необходимые настройки:
```
SW1> enable
SW1#configure terminal
SW1(config)#interface range fa0/6-7
SW1(config-if-range)#channel-group 1 mode desirable
SW1(config-if-range)#exit
SW1(config)#interface range fa0/4-5
SW1(config-if-range)#channel-group 2 mode active
SW1(config-if-range)#exit
SW1(config)#interface vlan 10
SW1(config-if)#ip address 172.17.0.3 255.255.128.0
SW1(config-if)#no shutdown
SW1(config-if)#exit
SW1(config)#exit
SW1#wr
SW1#exit
```

Текст для вставки с командную строку коммутатора:
```
en
configure terminal
interface range fa0/6-7
channel-group 1 mode desirable
exit
interface range fa0/4-5
channel-group 2 mode active
exit
interface vlan 10
ip address 172.17.0.3 255.255.128.0
no shutdown
exit
exit
wr
exit

```


## 4. Выполнение настроек маршрутизатора R1 и R2.

R1:
```
R1> enable
R1#configure terminal
R1(config)#interface GigabitEthernet0/1.10
R1(config-subif)#description USERS_VLAN
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip address 192.168.10.124 255.255.255.128
R1(config)#interface GigabitEthernet0/1.99
R1(config-subif)#description MANAGEMENT_VLAN
R1(config-subif)#encapsulation dot1Q 99
R1(config-subif)#ip address 172.17.127.252 255.255.128.0
R1(config-subif)#exit
R1(config)#interface GigabitEthernet0/1
R1(config-if)#no shut
```

R2:
```
R2> enable
R2#configure terminal
R2(config)#interface GigabitEthernet0/1.10
R2(config-subif)#description USERS_VLAN
R2(config-subif)#encapsulation dot1Q 10
R2(config-subif)#ip address 192.168.10.125 255.255.255.128
R2(config)#interface GigabitEthernet0/1.99
R2(config-subif)#description MANAGEMENT_VLAN
R2(config-subif)#encapsulation dot1Q 99
R2(config-subif)#ip address 172.17.127.253 255.255.128.0
R2(config-subif)#exit
R2(config)#interface GigabitEthernet0/1
R2(config-if)#no shut
```
