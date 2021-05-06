# Лабораторная работа №7. Внедрение маршрутизации между виртуальными локальными сетями.

###  Задание:

1. Создание сети и настройка основных параметров устройств.
2. Создание сетей VLAN и назначение портов коммутаторов.
3. Настройка транка 802.1Q между коммутаторами.
4. Настройка маршрутизации между сетями VLAN.
5. Проверка маршрутизация между VLAN.

###  Решение:

#### 1. Создание сети и настройка основных параметров устройств.


##### 1.1 Создадим топологию данной сети в программе cisco packet tracer. 

![](net_topology.png)


##### 1.2. Выполнение базовых настроек маршрутизатора:


- Настройка имени устройства в соответствии с топологией.

```
Router> enable
Router#configure terminal
Router(config)#hostname R1
R1(config)#
```

- Отключение поиска DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

```
R1(config)#no ip domain-lookup
```

- Создадим пользоваеля admin с паролем cisco в качестве пароля.

```
R1(config)#username admin privilege 0 secret cisco
```

- Предотвращение перебора пароля методом грубой силы

```
R1(config)#login block-for 120 attempts 3 within 60
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
R1(config)#enable secret cisco
R1(config)#service password-encryption
R1(config)#
```

- Настройка приветственного баннера:

```
R1(config)#banner motd $ Authorized Access Only! $
```

- Сохранение настроенной конфигурации устройства

```
R1#copy running-config startup-config
```


- Настройка времени на маршрутизаторе

```
R1#clock set 11:33:25 MAY 6 2021
R1#show clock set
```



##### 1.3. Выполнение базовых настроек на каждом коммутаторе (описание только для первого):


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
S1(config)#enable secret cisco
S1(config)#service password-encryption
S1(config)#
```

- Настройка приветственного баннера:

```
S1(config)#banner motd $ Authorized Access Only! $
```

- Сохранение настроенной конфигурации устройства

```
S1#copy running-config startup-config
```

##### 1.4. Выполнение базовых настроек узлов ПК.

Настроим сетевые параметры узлов ПК (IP адреса, маски и шлюзы) в соответствии с заданием.




#### 2. Создание сетей VLAN и назначение портов коммутаторов.

В соответсвии с заданием на каждом коммутаторе необходимо создать 5 VLAN: 10, 20, 30, 999 и 1000.

- Создание необходимых VLAN (выполняется на обоих коммутаторах)


```
S1(config)#vlan 10
S1(config-vlan)#name network_management
S1(config-vlan)#exit
S1(config)#vlan 20
S1(config-vlan)#name sales
S1(config-vlan)#exit
S1(config)#vlan 30
S1(config-vlan)#name operations
S1(config-vlan)#exit
S1(config)#vlan 999
S1(config-vlan)#name parking_lot
S1(config-vlan)#exit
S1(config)#vlan 1000
S1(config-vlan)#name native_vlan
S1(config-vlan)#exit
```

- Включение портов в VLAN 999 (для каждого коммутатора свой диапазон портов)

interface range fastEthernet 0/2 - 4 , fastEthernet 0/7 -24 , GigabitEthernet 0/1 - 2






























































- Настройка и активация на маршрутизаторе R1 интерфейса G0/1 в соответствии с заданием

```
R1(config)#interface ge0/1
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#exit
R1#
```

- Сохранение настроенной конфигурации устройства.

```
R1#copy running-config startup-config
```

1.3. Настройка IP-адреса и маски подсети для PC-A в соответствии с заданием.

Настроим через вкладку **Config** окна узла PC-A его IPv4 адрес **192.168.1.3** и шлюз по умолчанию **192.168.1.1**

1.4. Проверка доступности маршрутизатора R1 с узла PC-A

```
C:\>ping 192.168.1.1
Pinging 192.168.1.1 with 32 bytes of data:
Reply from 192.168.1.1: bytes=32 time=1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
```

#### 2. Настройка маршрутизатора для доступа по протоколу SSH.

2.1. Зададим доменное имя устройства

```
R1#configure terminal
R1(config)#ip domain-name R1
```
2.2. Сгенерируем ключ шифрования с указанием его длины.

```
R1(config)#crypto key generate rsa general-keys modulus 1024
The name for the keys will be: R1.R1
% The key modulus size is 1024 bits
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
*Mar 1 0:38:43.894: %SSH-5-ENABLED: SSH 1.99 has been enabled
```
2.3. Создадим пользоваеля admin с паролем Adm1n в качестве пароля.

```
R1(config)#username admin privilege 15 secret Adm1n
```

2.4.Включаем SSH сервер v2

```
R1(config)#ip ssh version 2
```
2.4. Активируем протокол SSH и telnet на линиях VTY:

```
R1(config)#line vty 0 15 
R1(config-line)#transport input all
R1(config-line)#login local
R1(config)#exit
R1#
```

2.5. Указываем маршрутизатору использовать локальную базу учетных записей для аутентификации.

```
R1(config-line)#login local
R1(config)#exit
R1#
```


#### 3. Выполнение базовых настроек коммутатора.

- Настройка имени устройства в соответствии с топологией.

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

- Назначение пароля **cisco** в качестве пароля консоли:

```
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#logging synchronous
S1(config-line)#exit
S1(config)#
```

- Назначение пароля **cisco** в качестве пароля VTY и отключение доступа к неактивному привилегированному режиму через заданное время:

```
S1(config)#line vty 0 15
S1(config)#exec-timeout 5 30
S1(config-line)#password cisco
S1(config-line)#login
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
S1(config)#banner motd $ Authorized Access Only! $
```

- Настройка и активация на коммутаторе S1 интерфейса VTY:

```
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.11 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#exit
```
- Настройка шлюза по умолчанию

```
S1(config)#ip default-gateway 192.168.1.1
```

- Сохранение настроенной конфигурации устройства.

```
S1#copy running-config startup-config
```


#### 4. Настройка коммутатора для доступа по протоколу SSH.

4.1. Зададим доменное имя устройства

```
S1#configure terminal
S1(config)#ip domain-name S1
```
4.2. Сгенерируем ключ шифрования с указанием его длины.

```
S1(config)#crypto key generate rsa general-keys modulus 1024
The name for the keys will be: S1.S1
% The key modulus size is 1024 bits
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
*Mar 1 0:20:41.373: %SSH-5-ENABLED: SSH 1.99 has been enabled
```
4.3. Создадим пользоваеля admin с паролем Adm1n в качестве пароля.

```
S1(config)#username admin privilege 15 secret Adm1n
```

4.4.Включаем SSH сервер v2

```
S1(config)#ip ssh version 2
```
4.4. Активируем протокол SSH и telnet на линиях VTY:

```
S1(config)#line vty 0 15 
S1(config-line)#transport input all
S1(config-line)#login local
S1(config)#exit
S1#
```

4.5. Указываем маршрутизатору использовать локальную базу учетных записей для аутентификации.

```
S1(config-line)#login local
S1(config)#exit
S1#
```

#### 5. Установление с коммутатора S1 соединения с маршрутизатором R1 по протоколу SSH

```
S1#ssh -l admin 192.168.1.1
Password: 

 Authorized Access Only! 

R1#
```
