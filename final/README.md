### Цели работы ###
1. Построение совмещенной техгологической и административной сети.
2. Обеспечение отказоустойчивости сети.
3. Обеспечение безопасности сети.
4. Обеспечение расширяемости сети.
5. Бэкап сетевых настроек и логов сетевых устройств.

### Что планировалось ###
1. Построение отказоустойчивой высокопроизводительной L2 сети по топологии кольцо.
2. Обеспечение отказоустойчивости доступа к WAN.
3. Обеспечение администрирования всех устройств в сети.
4. Доступность серверов из WAN.
5. Шлюз для контроля и администрирования из WAN.
6. Настройка GRE тоннелей
7. Настройка устройств Internet of Things


### Используемые технологии ###
1. VLAN
2. LACP, EtherChannel
3. DHCP
4. DNS
5. Ethernet Security
6. NAT

### Планирование VLAN ###

Предполагается, что будут следующие VLAN:
1. 333 - ParkingLot
2. 999 - Native
3. 10 - Management
4. 20 - Servers
5. 30 - Users
6. 40 - Telephones
8. 50 - Technology_service_1 (PIVP)
9. 60 - Technology_service_2 (SPPI)
10. 70  - ISP-1

### Настройка коммутаторов L2 кольца ###

Необходимо настроить коммутаторы с помощьью протоколов LACP и STP для построения отказоустойчивой СПД.

- Настройка VLAN (одинаково для всех коммутаторов)
```
SW1-OPTICAL(config)#vlan 333
SW1-OPTICAL(config-vlan)#name ParkingLot
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 999
SW1-OPTICAL(config-vlan)#name Native
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 10
SW1-OPTICAL(config-vlan)#name Management
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 20
SW1-OPTICAL(config-vlan)#name Servers
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 30
SW1-OPTICAL(config-vlan)#name Users
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 40
SW1-OPTICAL(config-vlan)#name Telephones
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 50
SW1-OPTICAL(config-vlan)#name Technology_service_1
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 60
SW1-OPTICAL(config-vlan)#name Technology_service_2
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 70
SW1-OPTICAL(config-vlan)#name ISP-1
SW1-OPTICAL(config-vlan)#exit
```
- Внесение всех портов в ParkingLot и их выключение.
```
SW1-OPTICAL(config)#interface range fa0/1 - 24, Gi0/1 - 2
SW1-OPTICAL(config-if)#switchport mode access
SW1-OPTICAL(config-if)#switchport access vlan 333
SW1-OPTICAL(config-if)#shut
```
- Настройка транкового EtherChannell для SW1--SW2 и SW1--SW3
```
SW1-OPTICAL(config)#interface port-channel 1
SW1-OPTICAL(config-if)#description ether-chanell_to_SW2-OPTICAL
SW1-OPTICAL(config-if)#switchport mode trunk
SW1-OPTICAL(config-if)#switchport trunk allowed vlan 10,20,30,40,50,60,70,333
SW1-OPTICAL(config-if)#switchport trunk native vlan 333
SW1-OPTICAL(config)#interface port-channel 2
SW1-OPTICAL(config-if)#description ether-chanell_to_SW3-OPTICAL
SW1-OPTICAL(config-if)#switchport mode trunk
SW1-OPTICAL(config-if)#switchport trunk allowed vlan 10,20,30,40,50,60,333
SW1-OPTICAL(config-if)#switchport trunk native vlan 333
```
- Настройка транкового EtherChannell между SW2--SW3
```
SW1-OPTICAL(config)#interface port-channel 3
SW1-OPTICAL(config-if)#description ether-chanell_between_SW2_SW3-OPTICAL
SW1-OPTICAL(config-if)#switchport mode trunk
SW1-OPTICAL(config-if)#switchport trunk allowed vlan 10,20,30,40,50,60,333
SW1-OPTICAL(config-if)#switchport trunk native vlan 333
```

- Соберем все в единый набор правил для применения на коммутаторах SW1, SW2
```
conf t
vlan 333
name ParkingLot
exit
vlan 999
name Native
exit
vlan 10
name Management
exit
vlan 20
name Servers
exit
vlan 30
name Users
exit
vlan 40
name Telephones
exit
vlan 50
name Technology_service_1
exit
vlan 60
name Technology_service_2
exit
vlan 70
name ISP-1
exit
interface range fa0/1 - 24, Gi0/1 - 2
switchport mode access
switchport access vlan 333
shut
exit
interface port-channel 1
description ether-chanell_to_SW2-OPTICAL
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,70,333
switchport trunk native vlan 333
interface port-channel 2
description ether-chanell_to_SW3-OPTICAL
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,333
switchport trunk native vlan 333
end
wr
```
- Этот набор дополнительно выполяется на SW2, SW3
```
interface port-channel 3
description ether-chanell_between_SW2_SW3-OPTICAL
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,333
switchport trunk native vlan 333
```


Включение портов коммутаторов в EtherChannell
```
SW1-OPTICAL(config)#interface range fa0/1 - 2
SW1-OPTICAL(config-if)#switchport mode trunk
SW1-OPTICAL(config-if)#channel-group 1 mode active
SW1-OPTICAL(config-if)#no shut
SW1-OPTICAL(config)#interface range fa0/3 - 4
SW1-OPTICAL(config-if)#switchport mode trunk
SW1-OPTICAL(config-if)#channel-group 2 mode active
SW1-OPTICAL(config-if)#no shut
```

```
SW2-OPTICAL(config)#interface range fa0/1 - 2
SW2-OPTICAL(config-if)#switchport mode trunk
SW2-OPTICAL(config-if)#channel-group 1 mode active
SW2-OPTICAL(config-if)#no shut
SW2-OPTICAL(config)#interface range fa0/3 - 4
SW2-OPTICAL(config-if)#switchport mode trunk
SW2-OPTICAL(config-if)#channel-group 3 mode active
SW2-OPTICAL(config-if)#no shut
```

```
SW3-OPTICAL(config)#interface range fa0/1 - 2
SW3-OPTICAL(config-if)#switchport mode trunk
SW3-OPTICAL(config-if)#channel-group 1 mode active
SW3-OPTICAL(config-if)#no shut
SW3-OPTICAL(config)#interface range fa0/3 - 4
SW3-OPTICAL(config-if)#switchport mode trunk
SW3-OPTICAL(config-if)#channel-group 3 mode active
SW3-OPTICAL(config-if)#no shut
```

- Настройка STP для оптимизации путей для траффика разных VLAN

Раздел в разработке

### Настройка роутеров на палке для работы сервисов Technology_service_1 (PIVP) и Technology_service_2 (SPPI) ### 

Для работы двух производственных (не административных) сервисов нужно обеспечить связность маршрутизаторов TechService1-RT, TechService2-RT, TechService1-2-RT и серверов "TCP SERVER ON PORT 8000", FTP и "FTP, TCP CLIENT". Также все три сервера должны иметь доступ к NTP-серверу, а сервер FTP должен иметь исходящий доступ к интернету. Также все сервера и маршрутизаторы должны быть доступны из Management сети для контроля.

### Конфигурация сетей и интерфейсов ###

| Устройство           | Интерфейс  |   VLAN   |   IP-адрес   | Маска подсети   | Шлюз по умолчанию |
| -------------------- | :--------: | :------- | :----------: | :-------------- | :---------------- |
| TechService2-RT      | Vlan99     | 99       | 10.1.99.111  | 255.255.255.0   | 10.1.99.254       |
|                      | Fa0/1      | 10       |              |                 |                   |
|                      | Fa0/2      | 10       |              |                 |                   |
|                      | G0/1       | Trunk    |              |                 |                   |
| MAIN-ACC-SW-02       | Vlan99     | 99       | 10.1.99.222  | 255.255.255.0   | 10.1.99.254       |
|                      | Fa0/1      | 30       |              |                 |                   |
|                      | G0/1       | Trunk    |              |                 |                   |
| MAIN-DIST-SW-01      | Vlan99     | 99       | 10.1.99.11   | 255.255.255.0   |                   |
|                      | Vlan10     | 10       | 10.1.10.11   | 255.255.255.0   |                   |


- Настройка маршрутизаторов


