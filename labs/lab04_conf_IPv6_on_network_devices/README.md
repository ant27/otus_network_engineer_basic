# Лабораторная работа №4. Настройка IPv6-адресов на сетевых устройствах.

###  Задание:

1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора.
1.3. Обновление образа IOS коммутатора до версии 15:
2. Ручная настройка IPv6-адресов.
3. Проверка сквозного соединения.

###  Решение:

1. Подготовка устройств.

В задании указано, что шаблон по умолчанию менеджера базы данных 2960 Switch Database Manager (SDM) не поддерживает IPv6. Перед назначением IPv6-адреса SVI VLAN 1 может понадобиться выполнение команды sdm prefer dual-ipv4-and-ipv6 default для включения IPv6-адресации.

Создадим в программе cisco packet tracer сетевое устройство коммутатор Cisco 2960 Switch и выполним в нем команду **show sdm prefer** для определения поддержки IPv6:

```
S1> enable
S1# show sdm prefer
 The current template is "default" template.
 The selected template optimizes the resources in
 the switch to support this level of features for
 0 routed interfaces and 255 VLANs.

  number of unicast mac addresses:                  8K
  number of IPv4 IGMP groups:                       256
  number of IPv4/MAC qos aces:                      128
  number of IPv4/MAC security aces:                 384
```

Так как нет упоминания о IPv6, cудя по всему, в SDM данной версии IOS нет его поддержки. Также в задании сказано, что в лабораторной работе должен использоваться Cisco IOS версии 15.2(2) (образ lanbasek9). 
Проверим текущую версию IOS командой **show version** или **show run**  

```
S1# show version
Cisco IOS Software, C2960 Software (C2960-LANBASE-M), Version 12.2(25)FX, RELEASE SOFTWARE (fc1)
...
```

Видимо, необходимо обновить версию IOS для поддержки IPv6. Это будет сделано далее.


2. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора.


1.1 Создадим топологию данной сети в программе cisco packet tracer. Добавим в топологию данной сети TFTP сервер (server-pt), на котором содержатся образы IOS, настроим на нем IP адрес 192.168.1.2 и подключим его в порт FE0/1 коммутатора. 

![](Net_structure.png)


1.2. Выполнение базовых настроек коммутатора:

- Настройка имени устройства в соответствии с топологией.

```
Switch> enable
Switch#configure terminal
Switch(config)#hostname S1
S1(config)#exit
```

- Назначение cisco в качестве паролей консоли, VTY и доступа к привилегированному режиму EXEC, а также отображение его в неявном виде в конфигурации и включение доступа по telnet.


```
S1#configure terminal
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#logging synchronous
S1(config-line)#exit
S1(config)#
S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#transport input telnet
S1(config-line)#exit
S1(config)#enable secret cisco
S1(config)#service password-encryption
S1(config)#exit 
S1#
```

- Настройка приветственного баннера:

```
S1#
S1#configure terminal
S1(config)#banner motd $ Authorized Access Only! $
S1(config)#exit 
S1#
```

- Настройка IPv4-адреса устройства для обновления IOS через TFTP.

```
S1#configure terminal
S1(config)#interface vlan1
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#exit
S1(config)#exit
S1#
```

- Сохранение настроенной конфигурации устройства.
```
S1#copy running-config startup-config
```


1.3. Выполнение базовых настроек маршрутизатора:


- Настройка имени устройства в соответствии с топологией.

```
Router> enable
Router#configure terminal
Router(config)#hostname R1
R1(config)#exit
```

- Назначение cisco в качестве паролей консоли, VTY и доступа к привилегированному режиму EXEC, а также отображение его в неявном виде в конфигурации и включение доступа по telnet.


```
R1#configure terminal
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#logging synchronous
R1(config-line)#exit
R1(config)#
R1(config)#line vty 0 15
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#transport input telnet
R1(config-line)#exit
R1(config)#enable secret cisco
R1(config)#service password-encryption
R1(config)#exit 
R1#
```

- Настройка приветственного баннера:

```
R1#
R1#configure terminal
R1(config)#banner motd $ Authorized Access Only! $
R1(config)#exit 
R1#
```

- Настройка IPv6-адресов устройства в соответствии с заданием.

```
R1#configure terminal
R1(config)#interface ge0/0
R1(config-if)#ipv6 enable
R1(config-if)#ipv6 address 2001:db8:acad:a::1/64
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#interface ge0/1
R1(config-if)#ipv6 enable
R1(config-if)#ipv6 address 2001:db8:acad:1::1/64
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#exit
R1#
```

- Сохранение настроенной конфигурации устройства.
```
R1#copy running-config startup-config
```

1.3. Обновление образа IOS коммутатора до версии 15:

- Проверим текущий образ и количество всей и свободной памяти на устройстве:

```
S1> enable
S1#show flash: 
Directory of flash:/

    1  -rw-     4414921          <no date>  c2960-lanbase-mz.122-25.FX.bin
    2  -rw-        1074          <no date>  config.text

64016384 bytes total (59600389 bytes free)
```
- Скопируем образ c2960-lanbasek9-mz.150-2.SE4.bin:

```
S1#copy tftp flash
Address or name of remote host []? 192.168.1.2
Source filename []? c2960-lanbasek9-mz.150-2.SE4.bin
Destination filename [c2960-lanbasek9-mz.150-2.SE4.bin]? 

Accessing tftp://192.168.1.2/c2960-lanbasek9-mz.150-2.SE4.bin...
Loading c2960-lanbasek9-mz.150-2.SE4.bin from 192.168.1.2: !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
[OK - 4670455 bytes]

4670455 bytes copied in 0.103 secs (3645511 bytes/sec)
```
- Еще раз выведем содержимое флэш памати устройства:

```
S1#show flash: 
Directory of flash:/

    1  -rw-     4414921          <no date>  c2960-lanbase-mz.122-25.FX.bin
    3  -rw-     4670455          <no date>  c2960-lanbasek9-mz.150-2.SE4.bin
    2  -rw-        1074          <no date>  config.text

64016384 bytes total (54929934 bytes free)
```

- Последним шагом укажем устройству, к какому образу обращаться при загрузке операционной системы.

```
S1#configure terminal
S1(config)#boot system flash:c2960-lanbasek9-mz.150-2.SE4.bin
S1(config)#exit
```
- Сохранение настроенной конфигурации устройства.
```
S1#copy running-config startup-config
```
- Перезагрузим устройство.
```
S1#reload
```

1.4.





S1#show sdm prefer
 The current template is "default" template.
 The selected template optimizes the resources in
 the switch to support this level of features for
 0 routed interfaces and 1024 VLANs.

  number of unicast mac addresses:                  8K
  number of IPv4 IGMP groups + multicast routes:    0.25K
  number of IPv4 unicast routes:                    0
  number of IPv6 multicast groups:                  0
  number of directly-connected IPv6 addresses:      0
  number of indirect IPv6 unicast routes:           0
  number of IPv4 policy based routing aces:         0
  number of IPv4/MAC qos aces:                      0.125k
  number of IPv4/MAC security aces:                 0.375k
  number of IPv6 policy based routing aces:         0
  number of IPv6 qos aces:                          20
  number of IPv6 security aces:                     25
