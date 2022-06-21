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

| VLAN                 | Номер      |   Сеть     |  Маска подсети   | Шлюз по умолчанию |
| -------------------- | :--------: | :--------- | :--------------: | :---------------- |
| ParkingLot           | 333        | - |  -   | -      | 
| Native               | 999        | - |  -   | -     | 
| Management           | 10         | 10.10.10.0 |  255.255.255.0   | 10.10.10.100      | 
| Servers              | 20         | 10.10.20.0 |  255.255.255.0   | 10.10.20.100      | 
| Users                | 30         | 10.10.30.0 |  255.255.255.0   | 10.10.30.100      | 
| Telephones           | 40         | 10.10.40.0 |  255.255.255.0   | 10.10.40.100      | 
| Production_service_1 | 50         | 10.10.50.0 |  255.255.255.0   | 10.10.50.100      | 
| Production_service_2 | 60         | 10.10.60.0 |  255.255.255.0   | 10.10.60.100      | 
| INTERNET             | 70         | 10.10.70.0 |  255.255.255.0   | 10.10.70.100      | 


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
SW1-OPTICAL(config-vlan)#name Production_service_1
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 60
SW1-OPTICAL(config-vlan)#name Production_service_2
SW1-OPTICAL(config-vlan)#exit
SW1-OPTICAL(config)#vlan 70
SW1-OPTICAL(config-vlan)#name INTERNET
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
name Production_service_1
exit
vlan 60
name Production_service_2
exit
vlan 70
name INTERNET
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

Есть три информационные системы (PROD1, PROD2, PROD3), находящиеся в разных частях офисного здания. Маршрутизаторы систем имеют доступ к созданной в прошлом пункте СПД.
Задачи:
1. Серверам TCP-PROD, FTP-PROD информационной системы PROD1 нужно обеспечить связность с PROD3.
2. Серверу HTTPS-PROD информационной системы PROD2 нужно обеспечить выход в интернет и видимость из интернета через статический NAT.
3. Все маршрутизаторы и серверы должны иметь доступ к NTP-серверу сети SERVERS и должны быть доступны из Management сети для контроля и управления.
4. Все маршрутизаторы должны сохранять логи и конфигурации на syslog и ftp сервер в сети SERVERS. 


### Конфигурация интерфейсов ###

| Устройство           | Интерфейс  |   VLAN   |   IP-адрес   | Маска подсети   | Шлюз по умолчанию | Примечание |
| :------------------: | :--------: | :------- | :----------: | :-------------- | :---------------- | :---------------- |
| PROD1-RT             | VLAN10     | 10       | 10.10.10.4   | 255.255.255.0   | 10.10.10.100      | Management VLAN   |
|                      | VLAN20     | 20       | 10.10.20.4   | 255.255.255.0   | 10.10.20.100      | Servers VLAN      |
|                      | VLAN50     | 50       | 10.10.50.2   | 255.255.255.252 |                   | Production Service 1 VLAN |
|                      | G0/0       | Trunk    |              |                 |                   | Trunk link vlan 10,20,50|
|                      | G0/1       |          | 192.168.1.1  | 255.255.255.0   |                   | RT Interface in Production Service 1 NET |
| TCP_PROD             | F0/1       |          | 192.168.1.2  | 255.255.255.0   | 192.168.1.1       | TCP SERVER ON PORT 8000|
| FTP_PROD             | F0/1       |          | 192.168.1.3  | 255.255.255.0   | 192.168.1.1       | FTP SERVER |
| PROD2-RT             | VLAN10     | 10       | 10.10.10.5   | 255.255.255.0   | 10.10.10.100      | Management VLAN   |
|                      | VLAN20     | 20       | 10.10.20.5   | 255.255.255.0   | 10.10.20.100      | Servers VLAN      |
|                      | VLAN60     | 60       | 10.10.60.2   | 255.255.255.252 |                   | Production Service 2 VLAN |
|                      | G0/0       | Trunk    |              |                 |                   | Trunk link vlan 10,20,60|
|                      | G0/1       |          | 192.168.2.1  | 255.255.255.0   |                   | RT Interface in Production Service 2 NET |
| HTTPS_PROD           | F0/1       |          | 192.168.2.2  | 255.255.255.0   | 192.168.2.1       | HTTPS SERVER |
| PROD3-RT             | VLAN10     | 10       | 10.10.10.6   | 255.255.255.0   | 10.10.10.100      | Management VLAN   |
|                      | VLAN20     | 20       | 10.10.20.6   | 255.255.255.0   | 10.10.20.100      | Servers VLAN      |
|                      | VLAN50     | 50       | 10.10.50.1   | 255.255.255.252 |                   | Production Service 1 VLAN |
|                      | VLAN60     | 60       | 10.10.60.1   | 255.255.255.252 |                   | Production Service 2 VLAN|
|                      | G0/0       | Trunk    |              |                 |                   | Trunk link vlan 10,20,50,60|
|                      | G0/1       |          | 192.168.3.1  | 255.255.255.0   |                   | RT Interface in Production Service 3 NET |
| TCP-FTP-CLIENT       | F0/1       |          | 192.168.3.2  | 255.255.255.0   | 192.168.3.1       | HTTPS SERVER |


- Настройка интерфейсов маршрутизаторов

```
PROD1-RT(config)#interface GigabitEthernet0/0.10
PROD1-RT(config-subif)#description management
PROD1-RT(config-subif)#encapsulation dot1Q 10
PROD1-RT(config-subif)#ip address 10.10.10.4 255.255.255.0
PROD1-RT(config-subif)#end
PROD1-RT(config)#interface GigabitEthernet0/0.20
PROD1-RT(config-subif)#description servers
PROD1-RT(config-subif)#encapsulation dot1Q 20
PROD1-RT(config-subif)#ip address 10.10.20.4 255.255.255.0
PROD1-RT(config-subif)#end
PROD1-RT(config)#interface GigabitEthernet0/0.50
PROD1-RT(config-subif)#description production_service_1
PROD1-RT(config-subif)#encapsulation dot1Q 50
PROD1-RT(config-subif)#ip address 10.10.50.2 255.255.255.252
PROD1-RT(config-subif)#end
PROD1-RT(config)#interface GigabitEthernet0/0
PROD1-RT(config-if)#no sh
PROD1-RT(config)#interface GigabitEthernet0/1
PROD1-RT(config-if)#ip address 192.168.1.1 255.255.255.0
PROD1-RT(config-if)#no sh
```

```
PROD2-RT(config)#interface GigabitEthernet0/0.10
PROD2-RT(config-subif)#description management
PROD2-RT(config-subif)#encapsulation dot1Q 10
PROD2-RT(config-subif)#ip address 10.10.10.5 255.255.255.0
PROD2-RT(config-subif)#end
PROD2-RT(config)#interface GigabitEthernet0/0.20
PROD2-RT(config-subif)#description servers
PROD2-RT(config-subif)#encapsulation dot1Q 20
PROD2-RT(config-subif)#ip address 10.10.20.5 255.255.255.0
PROD2-RT(config-subif)#end
PROD2-RT(config)#interface GigabitEthernet0/0.60
PROD2-RT(config-subif)#description production_service_2
PROD2-RT(config-subif)#encapsulation dot1Q 60
PROD2-RT(config-subif)#ip address 10.10.60.2 255.255.255.252
PROD2-RT(config-subif)#end
PROD2-RT(config)#interface GigabitEthernet0/0
PROD2-RT(config-if)#no sh
PROD2-RT(config)#interface GigabitEthernet0/1
PROD2-RT(config-if)#ip address 192.168.2.1 255.255.255.0
PROD2-RT(config-if)#no sh
```

```
PROD3-RT(config)#interface GigabitEthernet0/0.10
PROD3-RT(config-subif)#description management
PROD3-RT(config-subif)#encapsulation dot1Q 10
PROD3-RT(config-subif)#ip address 10.10.10.5 255.255.255.0
PROD3-RT(config-subif)#end
PROD3-RT(config)#interface GigabitEthernet0/0.20
PROD3-RT(config-subif)#description servers
PROD3-RT(config-subif)#encapsulation dot1Q 20
PROD3-RT(config-subif)#ip address 10.10.20.5 255.255.255.0
PROD3-RT(config-subif)#end
PROD3-RT(config)#interface GigabitEthernet0/0.50
PROD3-RT(config-subif)#description production_service_1
PROD3-RT(config-subif)#encapsulation dot1Q 50
PROD3-RT(config-subif)#ip address 10.10.50.1 255.255.255.252
PROD3-RT(config-subif)#end
PROD3-RT(config)#interface GigabitEthernet0/0.60
PROD3-RT(config-subif)#description production_service_2
PROD3-RT(config-subif)#encapsulation dot1Q 60
PROD3-RT(config-subif)#ip address 10.10.60.1 255.255.255.252
PROD3-RT(config-subif)#end
PROD3-RT(config)#interface GigabitEthernet0/0
PROD3-RT(config-if)#no sh
PROD3-RT(config)#interface GigabitEthernet0/1
PROD3-RT(config-if)#ip address 192.168.3.1 255.255.255.0
PROD3-RT(config-if)#no sh
```

-Настраиваем статическую маршрутизацию для сетевой связности серверов 

```
PROD1-RT(config)#ip route 192.168.3.0 255.255.255.0 10.10.50.1
```
```
PROD2-RT(config)#ip route 192.168.3.0 255.255.255.0 10.10.60.1
```

```
PROD3-RT(config)#ip route 192.168.1.0 255.255.255.0 10.10.50.2
PROD3-RT(config)#ip route 192.168.2.0 255.255.255.0 10.10.60.2
```




- Включаем порты на коммутаторе SW3-OPTICAL для связи с маршрутизаторами PROD1-RT и PROD2-RT
```
SW3-OPTICAL(config)#interface g0/1
SW3-OPTICAL(config-if)#switchport mode trunk 
SW3-OPTICAL(config-if)#switchport trunk allowed vlan 10,20,50
SW3-OPTICAL(config-if)#no sh
SW3-OPTICAL(config-if)#exit
SW3-OPTICAL(config)#interface g0/2
SW3-OPTICAL(config-if)#switchport mode trunk 
SW3-OPTICAL(config-if)#switchport trunk allowed vlan 10,20,60
SW3-OPTICAL(config-if)#no sh
```

- Включаем порт на коммутаторе SW2-OPTICAL для связи с маршрутизатором PROD3-RT
```
SW2-OPTICAL(config)#interface g0/1
SW2-OPTICAL(config-if)#switchport mode trunk 
SW2-OPTICAL(config-if)#switchport trunk allowed vlan 10,20,50,60
SW2-OPTICAL(config-if)#no sh
```

- Настройка сохранения логов и конфигураций на syslog и ftp серверы

Раздел в разработке

- Настройка доступа сервера HTTPS-PROD в интернет через статический NAT

Раздел в разработке

- Настройка ACL для доступа на сервер TCP-PROD только по TCP порту 8000

Раздел в разработке

*Примечание: нужно ли ставить шлюз по умолчанию на VLAN интерфейс или для маршрутизатора это не имеет смысла.

### Настройка коммутаторов и роутера корпоративной сети ### 

Имееется небольшая корпоративная сеть, использующая инфраструктуру технологической. Имеется два провайдера интернета ISP-1 и ISP2.
Задачи:
1. Обеспечить доступ из сети USERS в сеть SERVERS
2. Настроить работу DHCP-сервера в сети USERS
3. Настроить работу DNS-сервера в сети USERS для локальных ресурсов
4. Настроить работу NTP-сервера для сети USERS, MANAGEMENT и серверов в комплексах PROD1, PROD2, PROD3.
5. Настроить доступ с ноутбука SYSADM в сеть USERS, MANAGEMENT,SERVERS
6. Настроить доступ к WEB-CORP серверу из интернета по статитескому NAT.
7. Настроить доступ в интернет из сети USERS по PAT.
8. Настройка отказоустойчивости интернета.

- Настройка VLAN на коммутаторах SW1-CORP и SW2-CORP (используем предыдущие наработки)
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
name Production_service_1
exit
vlan 60
name Production_service_2
exit
vlan 70
name INTERNET
exit
interface range fa0/1 - 24, Gi0/1 - 2
switchport mode access
switchport access vlan 333
shut
```
- Настройка транковых портов на коммутаторах SW1-CORP и SW2-CORP (одинаково для обоих коммутаторов)
```
SW1-CORP(config)interface F0/24, G0/1
SW1-CORP(config-if)#switchport mode trunk
SW1-CORP(config-if)#switchport trunk allowed vlan 10,20,30,40,50,60,70,333
SW1-CORP(config-if)#switchport trunk native vlan 333
SW1-CORP(config-if)#no sh
```
- Настройка портов доступа на коммутаторах SW1-CORP и SW2-CORP (одинаково для обоих коммутаторов)

```
SW1-CORP(config)interface F0/1 - 4
SW1-CORP(config-if)#switchport mode access
SW1-CORP(config-if)#switchport access vlan 20
SW1-CORP(config-if)#no sh
SW1-CORP(config)interface F0/5 - 10
SW1-CORP(config-if)#switchport mode access
SW1-CORP(config-if)#switchport access vlan 30
SW1-CORP(config-if)#no sh
SW1-CORP(config)interface F0/11
SW1-CORP(config-if)#switchport mode access
SW1-CORP(config-if)#switchport access vlan 20
SW1-CORP(config-if)#no sh
```

```
SW1-CORP(config)interface F0/4, G0/1
SW1-CORP(config-if)#switchport mode trunk
SW1-CORP(config-if)#switchport trunk allowed vlan 10,20,30,40,50,60,70,333
SW1-CORP(config-if)#switchport trunk native vlan 333
SW1-CORP(config-if)#no sh
```


### Настройка сохранения логов и конфигураций сетевых устройств на syslog и ftp сервер в сети SERVERS ### 
9. Настроить  сохранение конфигураций



```
SW1-OPTICAL(config-if)#switchport mode trunk
SW1-OPTICAL(config-if)#switchport trunk allowed vlan 10,20,30,40,50,60,333
SW1-OPTICAL(config-if)#switchport trunk native vlan 333
```
