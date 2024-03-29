# Лабораторная работа. Настройка протокола OSPFv2 для одной области.

##  Задание:

1. Создание сети и настройка основных параметров устройства
2. Настройка и проверка базовой работы протокола  OSPFv2 для одной области
3. Оптимизация и проверка конфигурации OSPFv2 для одной области

##  Решение:

### 1. Создание сети и настройка основных параметров устройств.

#### 1.1 Создадим топологию данной сети в программе cisco packet tracer. 

![](net_topology.png)

#### 1.2. Выполнение базовых настроек маршрутизаторов и коммутаторов (описание только для R1).

- Назначение имени устройства:
```
Router> enable
Router#configure terminal
Router(config)#hostname R1
```

- Отключение поиска DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

```
R1(config)#no ip domain-lookup
```

- Создадим пользоваеля admin с паролем cisco в качестве пароля.

```
R1(config)#username admin privilege 0 secret cisco
```

- Настройка использования локальной БД (с ранее заведенными пользвателем admin) для аутентификации доступа в консоль:

```
R1(config)#line console 0
R1(config-line)#login local
R1(config-line)#logging synchronous
R1(config-line)#exit
R1(config)#
```

- Настройка использования локальной БД (с ранее заведенными пользвателем admin) для аутентификации доступа к линиям VTY и отключение доступа к неактивному привилегированному режиму через заданное время:

```
R1(config)#line vty 0 15
R1(config-line)#exec-timeout 5 30
R1(config-line)#login local
R1(config-line)#exit
R1(config)#
```

- Настройка пароля для входа в привилегированный режим и настройка отображения этого пароля в неявном виде при выводе команды **show running-config**

```
R1(config)#enable secret class
R1(config)#service password-encryption
R1(config)#
```

- Настройка приветственного баннера:

```
R1(config)#banner motd $ Vy kto takie! Ya vas ne znayu! Idite naher! $
```

- Сохранение настроенной конфигурации устройства

```
R1#copy running-config startup-config
```

Для ускорения настройки соберем все команды в единый блок, который будет вставляться в консоль устройства посредством copy/paste:

```
enable
configure terminal
hostname S2
no ip domain-lookup
username admin privilege 0 secret cisco
line console 0
login local
logging synchronous
exit
line vty 0 15
exec-timeout 5 30
login local
exit
enable secret class
service password-encryption
banner motd $ Vy kto takie! Ya vas ne znayu! Idite naher! $
exit
wr
exit
```

Выполним данный блок на всех маршрутизаторах и коммутаторах нашей сети.

#### 1.3. Настройка интерфейсов маршрутизатора R1

```
R1(config)#interface loopback1
R1(config-if)#ip address 172.16.1.1 255.255.255.0 
R1(config)#interface g0/1
R1(config-if)#ip address 10.53.0.1 255.255.255.0 
R1(config-if)#no sh 
```
#### 1.4. Настройка интерфейсов маршрутизатора R2
```
R2(config)#interface loopback1
R2(config-if)#ip address 192.168.1.1 255.255.255.0 
R2(config)#interface g0/1
R2(config-if)#ip address 10.53.0.2 255.255.255.0 
R2(config-if)#no sh 
```

### 2. Настройка и проверка базовой работы протокола  OSPFv2 для одной области

#### 2.1. Настройка OSPFv2.

- Включение процесса ospf на маршрутизаторе с идентификатором процесса 56

```
R1(config)#router ospf 56
R1(config-router)#no shut
```
```
R2(config)#router ospf 56
R2(config-router)#no shut
```
- Настройка ospf идентификаторов маршрутизаторов
```
R1(config-router)#router-id 1.1.1.1
```
```
R2(config-router)#router-id 2.2.2.2
```
- Настройка сетей (интерфейсов), включенных в ospf

```
R1(config-router)#network 10.53.0.0 0.0.0.255 area 0

```
```
R2(config-router)#network 10.53.0.0 0.0.0.255 area 0

```
- Настройка интерфейса loopback на R2, включенных в ospf

```
R2(config-router)#network 192.168.1.0 0.0.0.255 area 0

```

- Альтернативный способ настройки включения интерфейсов в ospf

```
R1(config)#int f0/1
R1(config-if)#ip ospf 56 area 0 
```
```
R2(config)#int f0/1
R2(config-if)#ip ospf 56 area 0 
R2(config)#int loopback1
R2(config-if)#ip ospf 56 area 0

```

#### 2.2. Проверка настройки OSPFv2 на маршрутизаторах.

```
R1#show ip protocols 
Routing Protocol is "ospf 56"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 1.1.1.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.53.0.0 0.0.0.255 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    1.1.1.1              110      00:00:25
    2.2.2.2              110      00:00:25
  Distance: (default is 110)

R1#show ip ospf interface 
GigabitEthernet0/1 is up, line protocol is up
  Internet address is 10.53.0.1/24, Area 0
  Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1
  Transmit Delay is 1 sec, State BDR, Priority 1
  Designated Router (ID) 2.2.2.2, Interface address 10.53.0.2
  Backup Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:07
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 2.2.2.2  (Designated Router)
  Suppress hello for 0 neighbor(s)

R1#show ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:32    10.53.0.2       GigabitEthernet0/1
```

```
R2#show ip protocols 
Routing Protocol is "ospf 56"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 2.2.2.2
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.53.0.0 0.0.0.255 area 0
    192.168.1.0 0.0.0.255 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    1.1.1.1              110      00:08:21
    2.2.2.2              110      00:08:21
  Distance: (default is 110)

R2#show ip ospf interface 
GigabitEthernet0/1 is up, line protocol is up
  Internet address is 10.53.0.2/24, Area 0
  Process ID 56, Router ID 2.2.2.2, Network Type BROADCAST, Cost: 1
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 2.2.2.2, Interface address 10.53.0.2
  Backup Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:02
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 1.1.1.1  (Backup Designated Router)
  Suppress hello for 0 neighbor(s)
Loopback1 is up, line protocol is up
  Internet address is 192.168.1.1/24, Area 0
  Process ID 56, Router ID 2.2.2.2, Network Type LOOPBACK, Cost: 1
  Loopback interface is treated as a stub Host

R2#show ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:35    10.53.0.1       GigabitEthernet0/1
```

Как видно из вывода команд, смежность ospf между маршрутизаторами имеется. 

Маршрутизатором DR стал R2, BDR стал R1. Это связано с тем, что R2 имеет больший идентификатор router-id по сравнению c R1.

#### 2.3. Проверка таблицы маршрутизации на R1.

```
R1#show ip route ospf 
     192.168.1.0/32 is subnetted, 1 subnets
O       192.168.1.1 [110/2] via 10.53.0.2, 00:19:01, GigabitEthernet0/1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```

Как мы видно из вывода команд, маршрут до Loopback интерфейса R2 прописался на R1 посредством ospf. Пинг до него успешен с R1.

### 3. Оптимизация и проверка конфигурации OSPFv2 для одной области

- Сделаем R1 назначенным маршрутизатором, изменив ему приоритет.

```
R2(config)#int f0/1
R2(config-if)#ip ospf priority 50
R2(config-if)#end
R2#clear ip ospf proccess
```
- Проверим, что он стал DR

```
R1#show ip ospf interface
GigabitEthernet0/1 is up, line protocol is up
  Internet address is 10.53.0.1/24, Area 0
  Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1
  Transmit Delay is 1 sec, State DR, Priority 50
  Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:00
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 2.2.2.2
  Suppress hello for 0 neighbor(s)
```

- Установим hello-interval'ы для ospf в значение 30 (это надо сделать на обоих маршрутизаторах, иначе смежность не настроится). Показано для R1, для R2 аналогично.

```
R1(config)#int f0/1
R1(config-if)#ip ospf hello-interval 30
```

- Назначим маршрут по умолчанию на R1 через интерфейс loopback1
```
R1(config)#ip route 0.0.0.0 0.0.0.0 loopback1
R1(config)#router ospf 56
R1(config-router)#default-information originate
```
- Назначим на R2 интерфейс loopback1 как point-to-point для ospf
```
R2(config)#interface loopback 1
R2(config-if)#ip ospf network point-to-point 
```
- Назначим режим passive на интерфейсе loopback1 R2

```
R2(config)#router ospf 56
R2(config-router)#passive-interface loopback1
```
- Настроим базовую полосу пропускания в 100000 (100Gb) для ospf на обоих маршрутизаторах (показано только на R1)
```
R2(config)#router ospf 56
R2(config-router)#auto-cost reference-bandwidth 100000
% OSPF: Reference bandwidth is changed.
        Please ensure reference bandwidth is consistent across all routers.
```
- Проверка новых настроек
```
R1#show ip ospf interface g0/1
GigabitEthernet0/1 is up, line protocol is up
  Internet address is 10.53.0.1/24, Area 0
  Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1000
  Transmit Delay is 1 sec, State DR, Priority 50
  Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
  Backup Designated Router (ID) 2.2.2.2, Interface address 10.53.0.2
  Timer intervals configured, Hello 30, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:11
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 2.2.2.2  (Backup Designated Router)
  Suppress hello for 0 neighbor(s)
  ```
Как мы видим, приоритет маршрутизатора изменен на 50, поэтому R1 стал DR, hello-interval изменен на 30.
  
- Проверим прописавшиеся на R1 маршруты ospf
``` 
R1#show ip route ospf
O    192.168.1.0 [110/1012] via 10.53.0.2, 00:05:13, GigabitEthernet0/1
```
- Проверим прописавшиеся на R2 маршруты ospf
``` 
R2#show ip route ospf
O*E2 0.0.0.0/0 [110/1] via 10.53.0.1, 00:01:17, GigabitEthernet0/1
``` 
- Пропингуем интерфейс 172.16.1.1 (loopback 1 R1) из R2 для проверки ospf
``` 
R2#ping 172.16.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/2 ms
R2#traceroute 172.16.1.1
Type escape sequence to abort.
Tracing the route to 172.16.1.1

  1   10.53.0.1       0 msec    0 msec    1 msec 
``` 

Мы не включали интерейс 172.16.1.1 (loopback 1 R1) в ospf, но на R2 он доступен, так как доступен маршрут по умолчанию, распостраненный с R1.
