# Лабораторная работа №9. Развертывание коммутируемой сети с резервными каналами (Протокол связующего дерева - STP ).

###  Задание:

1. Создание сети и настройка основных параметров устройств.
2. Определение корневого моста
3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

## 1. Создание сети и настройка основных параметров устройств.
### 1.1 Создадим топологию данной сети в программе cisco packet tracer в соответсвии с представленной схемой. 

![](net_topology_stp.png)


### 1.2. Выполнение базовых настроек коммутаторов (описание только для S1).

```
Switch> enable
Switch#configure terminal
Switch(config)#hostname S1
S1(config)#
```

- Отключение поиска DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

```
S1(config)#no ip domain-lookup
```

- Создадим пользоваеля admin с паролем cisco в качестве пароля.

```
S1(config)#username admin privilege 0 secret cisco
```


- Настройка использования локальной БД (с ранее заведенными пользвателем admin) для аутентификации доступа в консоль:


```
S1(config)#line console 0
S1(config-line)#login local
S1(config-line)#logging synchronous
S1(config-line)#exit
S1(config)#
```

- Настройка использования локальной БД (с ранее заведенными пользвателем admin) для аутентификации доступа к линиям VTY и отключение доступа к неактивному привилегированному режиму через заданное время:

```
S1(config)#line vty 0 15
S1(config-line)#exec-timeout 5 30
S1(config-line)#login local
S1(config-line)#exit
S1(config)#
```

- Настройка пароля для входа в привилегированный режим и настройка отображения этого пароля в неявном виде при выводе команды **show running-config**

```
S1(config)#enable secret class
S1(config)#service password-encryption
S1(config)#
```

- Настройка приветственного баннера:

```
S1(config)#banner motd $ Vy kto takie! Ya vas ne znayu! Idite naher! $
```

- Настройка и активация на коммутаторе S1 интерфейса VTY:

```
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#exit
```

- Сохранение настроенной конфигурации устройства

```
S1#copy running-config startup-config
```
- Проверка доступности всех коммутаторов с S3

Интерфейсы VTY всех коммутаторов доступны.

## 2. Определение корневого моста

### 2.1 Отключение всех портов на коммутаторах (описание только для S1)

```
S1> enable
S1#configure terminal
S1(config)#interface range fastEthernet 0/1 - 24 , GigabitEthernet 0/1 - 2
S1(config-if-range)#shutdown
S1(config-if-range)#exit
S1(config)#
```

### 2.2 Настройка подключенных портов в качестве транковых (описание только для S1)

```
S1(config)#interface range fastEthernet 0/1 - 4
S1(config-if-range)#switchport mode trunk
S1(config-if-range)#switchport trunk allowed vlan 1
S1(config-if-range)#exit
```

### 2.3 Включение портов F0/2 и F0/4 на всех коммутаторах (описание только для S1)
```
S1(config)#interface range fastEthernet 0/2 , fastEthernet 0/4
S1(config-if-range)#no shutdown
S1(config-if-range)#exit
```

### 2.4 Отображение данных протокола spanning-tree.

Вывод команд show spanning-tree на всех коммутаторах:

```
S1#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0007.ECA5.7C1E
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00E0.F950.5928
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

```
S2#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0007.ECA5.7C1E
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0009.7C85.BD16
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

```
S3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0007.ECA5.7C1E
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0007.ECA5.7C1E
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

Ответы на вопросы лабраторного задания:

1. Какой коммутатор является корневым мостом?: S3
2. Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?: У него самое низкое значение MAC-адреса по сравнению с другими коммутаторами.
3. Какие порты на коммутаторе являются корневыми портами?: Для S1 и S2 - Fa0/4, 
4. Какие порты на коммутаторе являются назначенными портами? Для S2 - Fa0/2, для S3 - Fa0/2 и Fa0/4. 
5. Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?: Для S1 - Fa0/2
6. Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?

## 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов.

### 3.1 Определение коммутатора с заблокированным портом

В прошлом пункте мы определили, что заблокирован порт Fa0/2 на коммутаторе S1.

### 3.2 Изменение стоимости порта

```
S1(config)#interface fastEthernet 0/2
S1(config-if)#spanning-tree cost 18
S1(config-if)#exit
```

К сожалению в packet tracer не работает изменение стоимости портов.
Но можно предположить, что уменьшение стоимсти заблокированного порта до уровня, меньшего, чем на другом коммутаторе, приведет к тому, что ранее заблокированный порт (S1– F0/2) станет назначенным портом, а протокол spanning-tree теперь будет блокировать порт на другом коммутаторе некорневого моста (S3 – F0/4). 

### 3.3 Возвращение стоимости порта обратно

```
S1(config)#interface fastEthernet 0/2
S1(config-if)#no spanning-tree cost 18
S1(config-if)#exit
```

## 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.

### 4.1 Включение портов F0/1 и F0/3 на всех коммутаторах

```
S1(config)#interface range fastEthernet 0/1 , fastEthernet 0/3
S1(config-if-range)#no shutdown
S1(config-if-range)#exit
```

### 4.3 Отображение данных протокола spanning-tree.

```
Вывод команд show spanning-tree на всех коммутаторах:

S1#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0007.ECA5.7C1E
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00E0.F950.5928
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Altn BLK 19        128.4    P2p
Fa0/3            Root FWD 19        128.3    P2p
Fa0/1            Altn BLK 19        128.1    P2p
```

```
S2#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0007.ECA5.7C1E
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0009.7C85.BD16
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/3            Root FWD 19        128.3    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```

```
S3>en
Password: 
S3#sh
S3#show sp
S3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0007.ECA5.7C1E
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0007.ECA5.7C1E
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

Ответы на вопросы лабраторного задания:

1. Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?: На обоих некорневых коммутаторах был выбран порт Fa0/3. Из двух портов, имеющих прямое подключение к корневому коммутатору, был выбран тот, у которого был меньший порядковый номер.
2. Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?: В случае когда стоимость портов, BID их коммутатора и приоритеты портов равны, выбирается порт меньшим номером.
