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

### Начальная настройка сетевых устройств ###

Начальная настройка устройств включает заведение пользователя, установку паролей на расширенный режим, включение ssh доступа.
Выполним наальную настройку на всех устройствах

```
en
conf t
username user privilege 15 secret 0 user
line console 0
login local
exit
enable secret user
service password-encryption
banner motd $ OTUS Network-engineer-basic final project. 2022 $
ip domain-name mega-company.com
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
line vty 0 15
transport input ssh
login local
end
wr
exit
```

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
| ISP-1                | 70         | 213.87.113.0 |  255.255.255.248   | 213.87.113.1      | 


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

- Настройка виртуального интерфейса management vlan и шлюза по умолчанию
```
SW1-OPTICAL(config)#interface vlan10
SW1-OPTICAL(config-if)#ip address 10.10.10.1 255.255.255.0 
SW1-OPTICAL(config-if)#exit 
SW1-OPTICAL(config)#ip default-gateway 10.10.10.100
```
```
SW2-OPTICAL(config)#interface vlan10
SW2-OPTICAL(config-if)#ip address 10.10.10.2 255.255.255.0 
SW2-OPTICAL(config-if)#exit 
SW2-OPTICAL(config)#ip default-gateway 10.10.10.100
```
```
SW3-OPTICAL(config)#interface vlan10
SW3-OPTICAL(config-if)#ip address 10.10.10.3 255.255.255.0 
SW3-OPTICAL(config-if)#exit 
SW3-OPTICAL(config)#ip default-gateway 10.10.10.100
```


- Настройка STP для оптимизации путей для траффика разных VLAN

Раздел в разработке

### Настройка роутеров на палке для работы сервисов Technology_service_1 (PIVP) и Technology_service_2 (SPPI) ### 

Есть три информационные системы (PROD1, PROD2, PROD3), находящиеся в разных частях офисного здания. Маршрутизаторы систем имеют доступ к созданной в прошлом пункте СПД.
Задачи:
1. Серверам TCP-PROD, FTP-PROD информационной системы PROD1 нужно обеспечить связность с PROD3.
2. Серверу HTTPS-PROD информационной системы PROD2 нужно обеспечить выход в интернет и видимость из интернета через статический NAT.
3. Все маршрутизаторы и серверы должны иметь доступ к NTP-серверу сети SERVERS и должны быть доступны из Management сети для контроля и управления.

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
|                      | VLAN60     | 60       | 10.10.60.2   | 255.255.255.248 |                   | Production Service 2 VLAN |
|                      | G0/0       | Trunk    |              |                 |                   | Trunk link vlan 10,20,60|
|                      | G0/1       |          | 192.168.2.1  | 255.255.255.0   |                   | RT Interface in Production Service 2 NET |
| HTTPS_PROD           | F0/1       |          | 192.168.2.2  | 255.255.255.0   | 192.168.2.1       | HTTPS SERVER |
| PROD3-RT             | VLAN10     | 10       | 10.10.10.26  | 255.255.255.0   | 10.10.10.100      | Management VLAN   |
|                      | VLAN20     | 20       | 10.10.20.26  | 255.255.255.0   | 10.10.20.100      | Servers VLAN      |
|                      | VLAN50     | 50       | 10.10.50.1   | 255.255.255.252 |                   | Production Service 1 VLAN |
|                      | VLAN60     | 60       | 10.10.60.1   | 255.255.255.248 |                   | Production Service 2 VLAN|
|                      | G0/0       | Trunk    |              |                 |                   | Trunk link vlan 10,20,50,60|
|                      | G0/1       |          | 192.168.3.1  | 255.255.255.0   |                   | RT Interface in Production Service 3 NET |
| TCP-FTP-CLIENT       | F0/1       |          | 192.168.3.2  | 255.255.255.0   | 192.168.3.1       | PROD3 SERVER |


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
PROD2-RT(config-subif)#ip address 10.10.60.2 255.255.255.248
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
PROD3-RT(config-subif)#ip address 10.10.10.26 255.255.255.0
PROD3-RT(config-subif)#end
PROD3-RT(config)#interface GigabitEthernet0/0.20
PROD3-RT(config-subif)#description servers
PROD3-RT(config-subif)#encapsulation dot1Q 20
PROD3-RT(config-subif)#ip address 10.10.20.26 255.255.255.0
PROD3-RT(config-subif)#end
PROD3-RT(config)#interface GigabitEthernet0/0.50
PROD3-RT(config-subif)#description production_service_1
PROD3-RT(config-subif)#encapsulation dot1Q 50
PROD3-RT(config-subif)#ip address 10.10.50.1 255.255.255.252
PROD3-RT(config-subif)#end
PROD3-RT(config)#interface GigabitEthernet0/0.60
PROD3-RT(config-subif)#description production_service_2
PROD3-RT(config-subif)#encapsulation dot1Q 60
PROD3-RT(config-subif)#ip address 10.10.60.1 255.255.255.248
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

### Настройка коммутаторов и роутера корпоративной сети ### 

Имееется небольшая корпоративная сеть, использующая инфраструктуру технологической. Имеется два провайдера интернета ISP-1 и ISP2 со следующими подсетями:

- ISP-1 (МТС), предоставленные IP: 213.87.113.2 - 7/29, шлюз, DNS: 213.87.113.1
- ISP-2 (Ростелеком), предоставленные IP: 109.127.128.2 - 7/29, шлюз, DNS: 109.127.128.1

Задачи:
1. Настройка CORE-RT для доступа к интернету через двух провайдеров.
2. Настройка в CORE-RT DHCP-сервера в сети USERS.
3. Настройка в CORE-RT DNS-сервера в сети USERS для локальных ресурсов WEB-CORP, FILE-BACKUP-CORP, NTP-CORP.
4. Настройка синхронизации по NTP всех устройств в MANAGEMENT сети и серверов в сети SERVERS и серверов в комплексах PROD1, PROD2, PROD3.
5. Настройка доступности серверов в комплексах PROD1, PROD2, PROD3 из сети MANAGEMENT (с сервера ZABBIX-CORP)
6. Настройка доступа в интернет с сервера HTTPS-PROD.
7. Настройка сохранения логирования и конфигураций на ftp-сервер LOG-CORP
8. Настройка доступ к WEB-CORP и HTTPS-PROD серверам из интернета по статитескому NAT.
9. Настройка доступа в интернет из сети USERS по PAT.
10. Разрешение доступа из сети USERS в сеть SERVERS только к IP серверов WEB-CORP по HTTPS, FILE-BACKUP-CORP по SAMBA, и NTP-CORP по NTP.
11. Настройка port-security на access портах сети USERS

### Конфигурация интерфейсов корпоративной сети ###

| Устройство           | Интерфейс  |   VLAN   |   IP-адрес   | Маска подсети   | Шлюз по умолчанию | Примечание |
|:------------------- | :--------: | :------- | :----------: | :-------------- | :---------------- | :---------------- |
| SW1-CORP             | VLAN333    | 333      |              |                 |                   | Parking Lot VLAN   |
|                      | VLAN999    | 999      |              |                 |                   | Native VLAN      |
|                      | VLAN10     | 10       | 10.10.10.6   | 255.255.255.0   | 10.10.10.100      | Management VLAN   |
|                      | VLAN20     | 20       |              |                 |                   | Servers VLAN       |
|                      | VLAN30     | 30       |              |                 |                   | Users VLAN       |
|                      | VLAN40     | 40       |              |                 |                   | Telephones VLAN       |
|                      | VLAN50     | 50       |              |                 |                   | Production Service 1 VLAN |
|                      | VLAN60     | 60       |              |                 |                   | Production Service 2 VLAN |
|                      | VLAN70     | 70       |              |                 |                   | Internet VLAN |
|                      | G0/1       | Trunk    |              |                 |                   | Trunk link for all vlan to CORE-RT |
|                      | F0/24      | Trunk    |              |                 |                   | Trunk link for all vlan to SW3-OPTICAL|
|                      | F0/1 - 4   | 20       |              |                 |                   | Access ports to servers interfaces|
|                      | F0/5 - 10  | 30       |              |                 |                   | Access ports to users hosts interfaces|
|                      | F0/11      | 10       |              |                 |                   | Access port to SysAdmin host interface|
| SW2-CORP             | VLAN333    | 333      |              |                 |                   | Parking Lot VLAN   |
|                      | VLAN999    | 999      |              |                 |                   | Native VLAN      |
|                      | VLAN10     | 10       | 10.10.10.7   | 255.255.255.0   | 10.10.10.100      | Management VLAN   |
|                      | VLAN20     | 20       |              |                 |                   | Servers VLAN       |
|                      | VLAN30     | 30       |              |                 |                   | Users VLAN       |
|                      | VLAN40     | 40       |              |                 |                   | Telephones VLAN       |
|                      | VLAN50     | 50       |              |                 |                   | Production Service 1 VLAN |
|                      | VLAN60     | 60       |              |                 |                   | Production Service 2 VLAN |
|                      | VLAN70     | 70       |              |                 |                   | Internet VLAN |
|                      | G0/1       | Trunk    |              |                 |                   | Trunk link for all vlan to CORE-RT |
|                      | F0/24      | Trunk    |              |                 |                   | Trunk link for all vlan to SW3-OPTICAL|
|                      | F0/1 - 4   | 20       |              |                 |                   | Access ports to servers interfaces|
|                      | F0/5 - 10  | 30       |              |                 |                   | Access ports to users hosts interfaces|
|                      | F0/11      | 10       |              |                 |                   | Access port to SysAdmin host interface|
|CORE-RT               | VLAN999    | 999      |              |                 |                   | Native VLAN      |
|                      | VLAN10     | 10       | 10.10.10.100 | 255.255.255.0   |                   | Management VLAN   |
|                      | VLAN20     | 20       | 10.10.20.100 | 255.255.255.0   |                   | Servers VLAN       |
|                      | VLAN30     | 30       | 10.10.30.100 | 255.255.255.0   |                   | Users VLAN       |
|                      | VLAN40     | 40       | 10.10.40.100 | 255.255.255.0   |                   | Telephones VLAN       |
|                      | VLAN60     | 60       | 10.10.60.3   | 255.255.255.248 |                   | Production Service 2 VLAN  |
|                      | VLAN70     | 70       | 213.87.113.2 | 255.255.255.248 |213.87.113.1       | Internet ISP-1 VLAN |
|                      | G0/0       | Trunk    |              |                 |                   | Trunk link for all vlan to SW2-CORP |
|                      | G0/1       | Trunk    |              |                 |                   | Trunk link for all vlan to SW1-CORP |
|                      | G0/2       |          | 109.127.128.2| 255.255.255.248 | 109.127.128.1     | Link to ISP-2 |
| NTP-CORP             | F0/0       |          | 10.10.20.10  |                 | 10.10.20.100      | NTP server interface 1|
| NTP-CORP             | F0/1       |          | 10.10.20.11  |                 | 10.10.20.100      | NTP server interface 2|
| LOG-CORP             | F0/0       |          | 10.10.20.12  |                 | 10.10.20.100      | LOG server interface 1|
| LOG-CORP             | F0/1       |          | 10.10.20.13  |                 | 10.10.20.100      | LOG server interface 2|
| FILE-BACKUP-CORP     | F0/0       |          | 10.10.20.14  |                 | 10.10.20.100      | FILE-BACKUP server interface 1|
| FILE-BACKUP-CORP     | F0/1       |          | 10.10.20.15  |                 | 10.10.20.100      | FILE-BACKUP server interface 2|
| WEB-CORP             | F0/0       |          | 10.10.20.16  |                 | 10.10.20.100      | WEB server interface 1|
| WEB-CORP             | F0/1       |          | 10.10.20.17  |                 | 10.10.20.100      | WEB server interface 2|
| ZABBIX-CORP          | F0/0       |          | 10.10.10.9   |                 | 10.10.10.100      | ZABBIX server interface 1|
| ZABBIX-CORP          | F0/1       |          | 10.10.10.10  |                 | 10.10.10.100      | ZABBIX server interface 2|
| USER1-CORP           | F0/0       |          | 10.10.30.1   |                 | 10.10.30.100      | User1 host interface|
| USER2-CORP           | F0/0       |          | 10.10.30.2   |                 | 10.10.30.100      | User2 host interface|

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
SW1-CORP(config)#interface rangeF0/24, G0/1
SW1-CORP(config-if)#switchport mode trunk
SW1-CORP(config-if)#switchport trunk allowed vlan 10,20,30,40,50,60,70,333
SW1-CORP(config-if)#switchport trunk native vlan 333
SW1-CORP(config-if)#no sh
```

- Включение транковых портов со стороны SW2-OPTICAL, который подключен к SW1-CORP и SW2-CORP
```
SW2-OPTICAL(config)#interface range f0/5 - 6
SW2-OPTICAL(config-if-range)#switchport trunk allowed vlan 10,20,30,40,50,60,70,333
SW2-OPTICAL(config-if-range)#no sh
```
- Настройка портов доступа на коммутаторах SW1-CORP и SW2-CORP (одинаково для обоих коммутаторов)

```
SW1-CORP(config)interface range F0/1 - 4
SW1-CORP(config-if)#switchport mode access
SW1-CORP(config-if)#switchport access vlan 20
SW1-CORP(config-if)#no sh
SW1-CORP(config)interface range F0/5 - 10
SW1-CORP(config-if)#switchport mode access
SW1-CORP(config-if)#switchport access vlan 30
SW1-CORP(config-if)#no sh
SW1-CORP(config)interface F0/11
SW1-CORP(config-if)#switchport mode access
SW1-CORP(config-if)#switchport access vlan 10
SW1-CORP(config-if)#no sh
```
- Настройка виртуального интерфейса management vlan и шлюза по умолчанию на коммутаторах SW1-CORP и SW2-CORP
```
SW1-CORP(config)#interface vlan10
SW1-CORP(config-if)#ip address 10.10.10.6 255.255.255.0 
SW1-CORP(config-if)#exit 
SW1-CORP(config)#ip default-gateway 10.10.10.100
```
```
SW2-CORP(config)#interface vlan10
SW2-CORP(config-if)#ip address 10.10.10.7 255.255.255.0 
SW2-CORP(config-if)#exit 
SW2-CORP(config)#ip default-gateway 10.10.10.100
```
#### Настройка маршрутизатора CORE-RT #### 

- Настройка VLAN на маршрутизаторе CORE-RT

```
CORE-RT(config)#interface GigabitEthernet0/0.10
CORE-RT(config-subif)#description management
CORE-RT(config-subif)#encapsulation dot1Q 10
CORE-RT(config-subif)#ip address 10.10.10.100 255.255.255.0
CORE-RT(config-subif)#end
CORE-RT(config)#interface GigabitEthernet0/0.20
CORE-RT(config-subif)#description servers
CORE-RT(config-subif)#encapsulation dot1Q 20
CORE-RT(config-subif)#ip address 10.10.20.100 255.255.255.0
CORE-RT(config-subif)#end
CORE-RT(config)#interface GigabitEthernet0/0.30
CORE-RT(config-subif)#description users
CORE-RT(config-subif)#encapsulation dot1Q 30
CORE-RT(config-subif)#ip address 10.10.30.100 255.255.255.0
CORE-RT(config-subif)#end
CORE-RT(config)#interface GigabitEthernet0/0.40
CORE-RT(config-subif)#description telephones
CORE-RT(config-subif)#encapsulation dot1Q 40
CORE-RT(config-subif)#ip address 10.10.40.100 255.255.255.0
CORE-RT(config-subif)#end
CORE-RT(config)#interface GigabitEthernet0/0.60
CORE-RT(config-subif)#description prod_service2
CORE-RT(config-subif)#encapsulation dot1Q 60
CORE-RT(config-subif)#ip address 10.10.60.3 255.255.255.248
CORE-RT(config-subif)#end
CORE-RT(config)#interface GigabitEthernet0/0.70
CORE-RT(config-subif)#description ISP-1-MTS
CORE-RT(config-subif)#encapsulation dot1Q 70
CORE-RT(config-subif)#end
CORE-RT(config)#interface GigabitEthernet0/0
CORE-RT(config-if)#no sh
CORE-RT(config)#interface GigabitEthernet0/1.10
CORE-RT(config-subif)#description management
CORE-RT(config-subif)#encapsulation dot1Q 10
CORE-RT(config)#interface GigabitEthernet0/1.20
CORE-RT(config-subif)#description servers
CORE-RT(config-subif)#encapsulation dot1Q 20
CORE-RT(config)#interface GigabitEthernet0/1.30
CORE-RT(config-subif)#description users
CORE-RT(config-subif)#encapsulation dot1Q 30
CORE-RT(config)#interface GigabitEthernet0/1.40
CORE-RT(config-subif)#description telephones
CORE-RT(config-subif)#encapsulation dot1Q 40
CORE-RT(config)#interface GigabitEthernet0/1.70
CORE-RT(config-subif)#description ISP-1-MTS
CORE-RT(config-subif)#encapsulation dot1Q 70
CORE-RT(config)#interface GigabitEthernet0/1
CORE-RT(config-if)#no sh
```
*!!!Проверить, будет ли таким образом осуществляться резервирование, когда VLAN'ы настроены на двух интерфейсах маршрутизатора.

- Настройка интерфейсов CORE-RT до провайдеров интернета (обратите внимание, что первый провайдер приходит через VLAN, потому что оборудование провайдера находится в другом от CORE-RT помещении)

```
CORE-RT(config)#interface GigabitEthernet0/0.70
CORE-RT(config-subif)#ip address 213.87.113.2 255.255.255.248
CORE-RT(config-subif)#end
CORE-RT(config)#interface GigabitEthernet0/2
CORE-RT(config-if)#ip address 109.127.128.2 255.255.255.248
CORE-RT(config-if)#no sh
CORE-RT(config-if)#end
```

- Настройка маршрутизаторов провайдеров интернета ISP-1 и ISP-2.

```
ISP-1(config)#interface GigabitEthernet0/0
ISP-1(config-if)#ip address 213.87.113.1 255.255.255.248
ISP-1(config-if)#no sh
ISP-1(config-if)#end
```
```
ISP-2(config)#interface GigabitEthernet0/0
ISP-2(config-if)#ip address 109.127.128.1 255.255.255.248
ISP-2(config-if)#no sh
ISP-2(config-if)#end
```
- Включение порта коммутатора SW1-OPTICAL, к которому подключен провайдер ISP-1
```
SW1-OPTICAL(config)#interface g0/1
SW1-OPTICAL(config-if)#switchport mode access 
SW1-OPTICAL(config-if)#switchport access vlan 70
SW1-OPTICAL(config-if)#no sh
```

### 1. Настройка CORE-RT для доступа к интернету через двух провайдеров. ### 
```
CORE-RT(config)#ip route 0.0.0.0 0.0.0.0 109.127.128.1
CORE-RT(config)#ip route 0.0.0.0 0.0.0.0 213.87.113.1
```
Такая схема из двух одновременно работающих маршрутов по умолчанию работает плохо (большие потери пакетов)

*Примечание: Вообще для резервирования интернет линков используется так называемый IP SLA и EEM. К сожалению их функционал не реализован в Packet Tracer, но на реальном Cisco 2921 все нижепреведенные настройки работают:

- Настраиваем правило тестирования доступности сервера google через ISP-2
```
CORE-RT(config)#ip sla 1
CORE-RT(config-ip-sla)#icmp-echo 8.8.8.8 source-interface GigabitEthernet0/2
CORE-RT(config-ip-sla)#timeout 1500
CORE-RT(config-ip-sla)#frequency 3
CORE-RT(config-ip-sla)#exit
```
При данных настройках происходит пинг адреса 8.8.8.8 каждые 3 секунды, при отсутствии пинга в течение 1,5 секунд правило срабатывает

Включаем выполнение тестирования
```
CORE-RT(config)#ip sla schedule 1 life forever start-time now
```
- Связываем правило с треком, который привяжем к командам, выполняемым при срабатывании правила
```
CORE-RT(config)#track 1 ip sla 1 reachability
```
- Привяжем объявление маршрута до ISP-1 к треку (обратите внимание, что административное расстояние для маршрута до ISP-2 специально сделано больше, чтобы при наличии двух ISP маршрутизатор использовал маршрут до первого)
```
ip route 0.0.0.0 0.0.0.0 109.127.128.1 10 track 1
ip route 0.0.0.0 0.0.0.0 213.87.113.1 50
```

- В конце заведем EEM апплет ISPtracking для сброса NAT-трансляций при сработке track1 (для того, чтобы при пропадании или появлении вновь линка, трансляции происходили через пул адресов работающего провайдера)
```
CORE-RT(config)#event manager applet ISPtracking
CORE-RT(config-applet)#event track 1 state any
CORE-RT(config-applet)#action 1.0 cli command "enable"
CORE-RT(config-applet)#action 1.1 cli command "clear ip nat translation *"
CORE-RT(config-applet)#action 1.2 syslog msg "ISP track state is changed"
```

### 2. Настройка в CORE-RT DHCP-сервера в сети USERS. ###

- Настройка исключения первых 10 ip-адресов из DHCP сети USERS для ручного назначения их устройствам (например, принтерам)
```
CORE-RT(config)#ip dhcp excluded-address 10.10.30.1 10.10.30.10
```
- Настройка пула для сети USERS (определение подсети, шлюза по умолчанию, адресов DNS, имени домена, времени аренды адреса в днях).
```
CORE-RT(config)#ip dhcp pool USERS_DHCP_POOL
CORE-RT(dhcp-config)#network 10.10.30.0 255.255.255.0
CORE-RT(dhcp-config)#default-router 10.10.30.100
CORE-RT(dhcp-config)#dns-server 10.10.30.100
CORE-RT(dhcp-config)#domain-name megacompany.com
CORE-RT(dhcp-config)#lease 2 12 30
```

### 3. Настройка в CORE-RT DNS-сервера в сети USERS для локальных ресурсов WEB-CORP, FILE-BACKUP-CORP, NTP-CORP. ###

- Настройка DNS-сервера на CORE-RT для разрешения локальных адресов
```
CORE-RT(config)ip dns server
CORE-RT(config)ip host web-corp.local 10.10.20.16
CORE-RT(config)ip host file-backup-corp.local 10.10.20.16
CORE-RT(config)ip host ntp-corp.local 10.10.20.16
```
К сожалению в packet-tracer не реализована команда "ip dns server".

- Настройка CORE-RT как DNS-клиента для разрешения глобальных адресов (в проекте настроен google.com)
```
CORE-RT(config)ip domain-lookup
CORE-RT(config)ip name-server 8.8.8.8
```

### 4. Настройка синхронизации по NTP всех сетевых устройств в MANAGEMENT сети с NTP-сервером в сети SERVERS ###

На предприятии имеется промышленный NTP-сервер c синхронизацией по GPS и GLONASS.

Необходимо настроить синхронизацию всех устройств в MANAGEMENT сети с NTP-сервером в сети SERVERS. 
В качестве резервного для всех устройств NTP-сервера будет использован CORE-RT, настроенный как master и тоже получающий данные от двух источников - локального NTP-сервера и глобального доступного из интернета time.google.com

- Настройка NTP-клиентов на всех сетевых устройствах (показано для SW1-OPTICAL, для других аналогично)
```
SW1-OPTICAL(config)#ntp server 10.10.20.10 prefer
SW1-OPTICAL(config)#ntp server 10.10.20.100
```
- Настройка CORE-RT как NTP-сервера
```
CORE-RT(config)#ntp master 2
CORE-RT(config)#ntp server 10.10.20.10
CORE-RT(config)#ntp server time.google.com
```
К сожалению cisco packet tracer по видимому, не поддерживает несколько записей "ntp server"


### 5. Настройка доступности серверов в комплексах PROD1, PROD2, PROD3 из сети MANAGEMENT (с сервера ZABBIX-CORP) для контроля ### 

Так как для предприятия критически важна работоспособность комплексов PROD1, PROD2, PROD3 была поставлена задача контроля серверов 24/7 с помощью развернутого сервера ZABBIX. Обеспечим доступность сетей PROD1, PROD2, PROD3 из сети MANAGEMENT (с сервера ZABBIX-CORP) и наоборот.

-Необходимо прописать статические маршруты до сетей комплексов PROD1, PROD2, PROD3 на CORE-RT

```
CORE-RT(config)#ip route 192.168.1.0 255.255.255.0 10.10.10.4
CORE-RT(config)#ip route 192.168.2.0 255.255.255.0 10.10.10.5
CORE-RT(config)#ip route 192.168.3.0 255.255.255.0 10.10.10.26
```

Маршруты наоборот, из сетей PROD1, PROD2, PROD3 в сеть MANAGEMENT прописывать не нужно, потому что они автоматически добавлены маршрутизаторами PROD1-RT, PROD2-RT, PROD3-RT как сеть MANAGEMENT непосредственно подключена к ним через подинтерфейсы VLAN10.


### 6. Настройка доступа в интернет с сервера HTTPS-PROD. ###

Для настройки выхода в интернет для сервера HTTPS-PROD нужно прописать маршрут по умолчанию на маршрутизаторе PROD2-RT. Пускай в интернет траффик в этом маршрутизаторе бегает по VLAN 60 (PROD2).

```
PROD2-RT(config)#ip route 0.0.0.0 0.0.0.0 10.10.60.3
```

### 7. Настройка доступ к WEB-CORP и HTTPS-PROD серверам из интернета по статитескому NAT. ###

- Настраиваем интрфейсы для натирования

```
CORE-RT(config)#interface GigabitEthernet0/2
CORE-RT(config-if)#ip nat outside
CORE-RT(config-subif)#interface GigabitEthernet0/0.70
CORE-RT(config-subif)#ip nat outside
CORE-RT(config-subif)#interface GigabitEthernet0/1.70
CORE-RT(config-subif)#ip nat outside
```
```
CORE-RT(config)#interface GigabitEthernet0/0.20
CORE-RT(config-subif)#ip nat inside
CORE-RT(config-subif)#interface GigabitEthernet0/0.60
CORE-RT(config-subif)#ip nat inside
CORE-RT(config-subif)#interface GigabitEthernet0/1.20
CORE-RT(config-subif)#ip nat inside
CORE-RT(config-subif)#interface GigabitEthernet0/1.60
CORE-RT(config-subif)#ip nat inside
```

- Настройка статического NAT для сервера WEB-CORP (глобальный IP берем из пула ISP-1)
```
CORE-RT(config)# ip nat inside source static 10.10.20.16 213.87.113.3
CORE-RT(config)# ip nat inside source static 10.10.20.16 109.127.128.3
```
- Настройка статического NAT для сервера HTTP-PROD (глобальный IP берем из пула ISP-2)
```
CORE-RT(config)# ip nat inside source static 192.168.2.2 213.87.113.4
CORE-RT(config)# ip nat inside source static 192.168.2.2 109.127.128.4
```

### 8. Настройка доступа в интернет из сети USERS по динамическому PAT. ### 

- Определим диапазон адресов, подлежащих трансляции с помощью стандартной ACL
```
CORE-RT(config)#ip access-list standard USERS_NET
CORE-RT(config-std-nacl)#permit 10.10.30.0 0.0.0.255
```
- Затем настроим два пула глобальных адресов от двух провайдеров

```
CORE-RT(config)#ip nat pool ISP-1-USERS-POOL 213.87.113.5 213.87.113.7 netmask 255.255.255.248
CORE-RT(config)#ip nat pool ISP-2-USERS-POOL 109.127.128.5 109.127.128.7 netmask 255.255.255.248
```
- Добавляем правила трансляции

```
CORE-RT(config)#ip nat inside source list USERS_NET pool ISP-1-USERS-POOL
CORE-RT(config)#ip nat inside source list USERS_NET pool ISP-2-USERS-POOL
```

- Включаем NAT-трансляцию на интерфейсе VLAN30

```
CORE-RT(config)#interface GigabitEthernet0/0.30
CORE-RT(config-subif)#ip nat inside
CORE-RT(config-subif)#interface GigabitEthernet0/1.30
CORE-RT(config-subif)#ip nat inside
```

### 9.Настройка сохранения логирования и конфигураций на syslog сервер LOG-CORP ###

- Настройка логирования на удаленный syslog сервер LOG-CORP (одинаково для всех устройств)

```
CORE-RT(config)#logging trap debugging
CORE-RT(config)#logging 10.10.20.12
CORE-RT(config)#logging on
```

- Настройка сохранения startup-config на удаленный ftp-сервер FILESHARE-BACKUP-CORP (одинаково для всех устройств)

```
RT-01(config)#archive
RT-01(config-archive)#log config
RT-01(config-archive)#logging enable
RT-01(config-archive)#logging persistent reload
RT-01(config-archive)#hidekeys
RT-01(config-archive)#path tftp://10.10.20.14/$H-$T
RT-01(config-archive)#write-memory
```

*К сожалению в Packet Tracer не реализовано.



### 10. Разрешение доступа из сети USERS в сеть SERVERS только к IP серверов WEB-CORP по HTTP и HTTPS, FILE-BACKUP-CORP по SAMBA, и NTP-CORP по NTP. ### 

Пользователи сети USERS (обычные юзвери) могут ходить во все VLAN'ы без ограничений за счет маршрутизации CORE-RT. Это нехорошо.
Настроим ACL для доступа из сети USERS ТОЛЬКО в сеть SERVERS ТОЛЬКО к IP серверов WEB-CORP по HTTPS, FILE-BACKUP-CORP по SAMBA, и NTP-CORP по NTP. К остальным серверам доступа у юзверей быть не должно. Также разрешается ходить в интернет. 

- Описание 1-го правила: Нужно создать access-list с правилом, разрешающим пакеты с ip источника, соответствующего диапазону сети USERS (10.10.30.0/24), ip назначения, соответствующего хосту WEB-CORP (10.10.20.16), протоколом tcp и портом 443. 
- Описание 2-го правила: Нужно создать access-list с правилом, разрешающим пакеты с ip источника, соответствующего диапазону сети USERS (10.10.30.0/24), ip назначения, соответствующего хосту FILE-BACKUP-CORP (10.10.20.14), протоколом tcp и портами 139, 445. 
- Описание 3-го правила: Нужно создать access-list с правилом, разрешающим пакеты с ip источника, соответствующего диапазону сети USERS (10.10.30.0/24), ip назначения, соответствующего хосту FILE-BACKUP-CORP (10.10.20.10), протоколом udp и портами 1023. 
- Описание 3-го правила: Нужно создать access-list с правилом, запрещающим пакеты с ip источника, соответствующего диапазону сети USERS (10.10.30.0/24), ip назначения, соответствующего диапазону сетей всех наших VLAN 10.10.0.0 0.0.255.255, чтобы юзвери не могли ходить в vlan'ы, но могли в интернет.

Весь остальной траффик разрешается.

```
CORE-RT(config)#ip access-list extended USERS_TO_SERVERS_AND_INTERNET
CORE-RT(config-std-nacl)#permit tcp 10.10.30.0 0.0.0.255 host 10.10.20.16 eq www
CORE-RT(config-std-nacl)#permit tcp 10.10.30.0 0.0.0.255 host 10.10.20.16 eq 443
CORE-RT(config-std-nacl)#permit tcp 10.10.30.0 0.0.0.255 host 10.10.20.14 eq 139
CORE-RT(config-std-nacl)#permit tcp 10.10.30.0 0.0.0.255 host 10.10.20.14 eq 445
CORE-RT(config-std-nacl)#permit tcp 10.10.30.0 0.0.0.255 host 10.10.20.10 eq 123
CORE-RT(config-std-nacl)#deny ip 10.10.30.0 0.0.0.255 10.10.0.0 0.0.255.255
CORE-RT(config-std-nacl)#permit ip any any
```
- Применение на интерфейсах GigabitEthernet0/0.30 и GigabitEthernet0/1.30 для входящего траффика.
```
CORE-RT(config)#int GigabitEthernet0/0.30
CORE-RT(config-if)#ip access-group USERS_TO_SERVERS_AND_INTERNET in
CORE-RT(config)#int GigabitEthernet0/1.30
CORE-RT(config-if)#ip access-group USERS_TO_SERVERS_AND_INTERNET in
```

###  11. Настройка port-security на access портах сети USERS ### 

- Настраиваем port-security на access портах сети USERS коммутатора SW1-CORP (на SW2-CORP аналогично)
```
SW1-CORP(config)interface range F0/5 - 10
SW1-CORP(config-if)#switchport port-security
SW1-CORP(config-if)#switchport port-security maximum 3
SW1-CORP(config-if)#switchport port-security violation restrict
SW1-CORP(config-if)#switchport port-security aging time 10
SW1-CORP(config-if)#ip dhcp snooping limit rate 5
SW1-CORP(config-if)#spanning-tree portfast
SW1-CORP(config-if)#spanning-tree bpduguard enable
```
