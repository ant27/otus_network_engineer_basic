# Занятие №17. Лабораторная работа. Протоколы DHCPv4, SLAAC и DHCPv6.

## 1. Реализация DHCPv4.
###  Задание:

1. Создание сети и настройка параметров устройств.
2. Настройка и проверка двух серверов DHCPv4 на R1.
3. Настройка и проверка DHCP-ретрансляции на R2

## 1. Создание сети и настройка параметров устройств.
### 1.1 Создание сети.

Создадим топологию данной сети в программе cisco packet tracer в соответствии с представленной схемой.

![](lesson-17_net_topology.png)

### 1.2 Создание сети.

По условиям задания маршрутизаторам и коммутаторам небходимо выделить адреса в определенных подсетях сети 192.168.1.0/24
1. Подсеть A должна содержать 58 хостов, что соответствует подсети с маской /26, содержащей до 62 хостов.
Параметры подсети: 
- адрес подсети: 192.168.1.0/26 (255.255.255.192)
- первый IP-адрес хоста: 192.168.1.1 
- второй IP-адрес хоста: 192.168.1.2 
- последний IP-адрес хоста: 192.168.1.62 
Интерфейсу R1 G0/0/1.100 необходимо назначить первый адрес, то есть 192.168.1.1 netmask 255.255.255.192. 
Интерфейсу S1 VLAN 100 необходимо назначить второй адрес, то есть 192.168.1.2 netmask 255.255.255.192 gateway 192.168.1.1.  
2. Подсеть B должна содержать 28 хостов, что соответствует подсети с маской /27, содержащей до 30 хостов.
Параметры подсети: 
- адрес подсети: 192.168.1.64/27 (255.255.255.224)
- первый IP-адрес хоста: 192.168.1.65 
- второй IP-адрес хоста: 192.168.1.66 
- последний IP-адрес хоста: 192.168.1.94 
Интерфейсу R1 G0/0/1.200 необходимо назначить первый адрес, то есть 192.168.1.65 netmask 255.255.255.224. 
Интерфейсу S1 VLAN 200 необходимо назначить второй адрес, то есть 192.168.1.66 netmask 255.255.255.224 gateway 192.168.1.65.  
3. Подсеть C должна содержать 12 хостов, что соответствует подсети с маской /28, содержащей до 14 хостов.
Параметры подсети: 
- адрес подсети: 192.168.1.96/28 (255.255.255.240)
- первый IP-адрес хоста: 192.168.1.97
- последний IP-адрес хоста: 192.168.1.110 
Интерфейсу R2 G0/0/1 необходимо назначить первый адрес, то есть 192.168.1.97 netmask 255.255.255.240. 

- 
### 1.3. Выполнение базовых настроек маршрутизаторов и коммутаторов (описание только для R1).

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

### 1.4. Настройка интерфейсов маршрутизаторов.
- Настроим подинтерфейсы G0/1 на маршрутизаторе R1:  

```
R1> enable
R1#configure terminal
R1(config)#interface GigabitEthernet0/1.100
R1(config-subif)#description consumers_vlan
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#exit
R1(config)#interface GigabitEthernet0/1.200
R1(config-subif)#description network_management_vlan
R1(config-subif)#encapsulation dot1Q 200
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#exit
R1(config)#interface GigabitEthernet0/1.1000
R1(config-subif)#description native_vlan
R1(config-subif)#encapsulation dot1Q 1000 native
R1(config-subif)#exit
```
- Включим родительский интерфейс R1: 
```
R1(config)#interface GigabitEthernet0/1
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#exit
R1#wr
```
- Настроим интерфейс G0/0 на маршрутизаторе R1: 
```
R1(config)#interface GigabitEthernet0/0
R1(config-if)#description p2p_r1_r2
R1(config-if)#ip address 10.10.0.1 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit
```
- Настроим статический маршрут по умолчанию для R1: 
```
R1(config)#ip route 0.0.0.0 0.0.0.0 10.10.0.2
```

Блок команд:
```
configure terminal
interface GigabitEthernet0/1.100
description consumers_vlan
encapsulation dot1Q 100
ip address 192.168.1.1 255.255.255.192
exit
interface GigabitEthernet0/1.200
description network_management_vlan
encapsulation dot1Q 200
ip address 192.168.1.65 255.255.255.224
exit
interface GigabitEthernet0/1.1000
description native_vlan
encapsulation dot1Q 1000 native
exit
interface GigabitEthernet0/1
no shutdown
interface GigabitEthernet0/0
description p2p_r1_r2
ip address 10.10.0.1 255.255.255.252
no shutdown
exit
ip route 0.0.0.0 0.0.0.0 10.10.0.2
end
wr
```
- Настроим и включим интерфейс G0/1 на маршрутизаторе R2:
```
R1> enable
R1#configure terminal
R1(config)#interface GigabitEthernet0/1
R1(config-if)#ip address 192.168.1.97 netmask 255.255.255.240
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#exit
R1#wr
```
- Настроим интерфейс G0/0 на маршрутизаторе R2: 
```
R1(config)#interface GigabitEthernet0/0
R1(config-if)#description p2p_r1_r2
R1(config-if)#ip address 10.10.0.1 255.255.255.252
R1(config-if)#exit
```
- Настроим статический маршрут по умолчанию для R2: 
```
R2(config)#ip route 0.0.0.0 0.0.0.0 10.10.0.1
```

- Блок команд:
```
configure terminal
interface GigabitEthernet0/1
ip address 192.168.1.97 255.255.255.240
no shutdown
interface GigabitEthernet0/0
description p2p_r1_r2
ip address 10.10.0.2 255.255.255.252
no shutdown
exit
ip route 0.0.0.0 0.0.0.0 10.10.0.1
end
wr
```

### 1.5. Настройка VLAN и интерфесов коммутаторов.

- Настроим VLAN'ы S1
```
S1> enable
S1#configure terminal
S1(config)#vlan 999
S1(config-vlan)#name parking_lot
S1(config-vlan)vlan 100
S1(config-vlan)#name clients
S1(config-vlan)vlan 200
S1(config-vlan)#name management
S1(config-vlan)vlan 1000
S1(config-vlan)#name native
S1(config-vlan)#exit
```
- Поместим неиспользуемые интерфейсы S1 в vlan 1000 (parking_lot)
```
S1(config)#interface range fa0/1-4 , fa0/7-24 , gi0/1-2
S1(config)#switchport access vlan 999
```
- Настроим интерфейсы на коммутарое S1:  

```
S1(config)#interface vlan100
S1(config-if)#ip address 192.168.1.2 255.255.255.192
S1(config)#interface vlan200
S1(config-if)#ip address 192.168.1.66 255.255.255.224
S1(config)#interface fa0/6
S1(config-if)#description client_pc_a
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 100
S1(config-if)#interface fa0/5
S1(config-if)#description trunk_to_r1
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk allowed vlan 100,200,1000
S1(config-if)#switchport trunk native vlan 1000
```
- Общий блок команд для ввода:
```
configure terminal
vlan 999
name parking_lot
vlan 100
name clients
vlan 200
name management
vlan 1000
name native
exit
interface range fa0/1-4 , fa0/7-24 , gi0/1-2
switchport access vlan 999
interface vlan100
ip address 192.168.1.2 255.255.255.192
interface vlan200
ip address 192.168.1.66 255.255.255.224
interface fa0/6
description client_pc_a
switchport mode access
switchport access vlan 100
interface fa0/5
description trunk_to_r1
switchport mode trunk
switchport trunk allowed vlan 100,200,1000
switchport trunk native vlan 1000
end
wr
```
- Настройка VLAN и интерфесов коммутатора S2.

```
S2(config)#interface fa0/18
S2(config-if)#description client_pc_b
S2(config-if)#switchport mode access
```
-Блок команд для ввода:

```
configure terminal
interface fa0/18
description client_pc_b
switchport mode access
end
wr
```

### 1.6 Настройка и проверка двух серверов DHCPv4 на R1.
#### 1.6.1 Настройка исключения первых 5 ip-адресов из подсетей А и С на R1. 
```
R1> enable
R1#configure terminal
R1(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.5
R1(config)#ip dhcp excluded-address 192.168.1.97 192.168.1.101
```
## 1.6.2 Настройка пула для подсети А на R1 (определение подсети, шлюза по умолчанию, адресов DNS, имени домена, времени аренды адреса в днях). 
```
R1(config)#ip dhcp pool lan_poolA
R1(dhcp-config)#network 192.168.1.0 255.255.255.192
R1(dhcp-config)#default-router 192.168.1.1
R1(dhcp-config)#dns-server 192.168.1.1
R1(dhcp-config)#domain-name ccna-lab.com
R1(dhcp-config)#lease 2 12 30
```
*Примечание: по неизвестной причине команды lease нет в списке команд настройки dhcp в packet-tracer

#### 1.6.3 Настройка пула для подсети С на R1 (определение подсети, шлюза по умолчанию, адресов DNS, имени домена, времени аренды адреса в днях). 
```
R1(config)#ip dhcp pool lan_poolС
R1(dhcp-config)#network 192.168.1.96 255.255.255.240
R1(dhcp-config)#default-router 192.168.1.97
R1(dhcp-config)#dns-server 192.168.1.97
R1(dhcp-config)#domain-name ccna-lab.com
```
#### 1.6.4 Проверка выдачи адреса хоту PC-A командой show ip dhcp bindings
```
R1#show ip dhcp bindings
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      00E0.A35C.3404           --                     Automatic
```
Как мы видим, адрес был нормально выдан сервером.
*Примечание: по неизвестной причине команды show ip dhcp server statistics нет в списке команд настройки dhcp в packet-tracer

### 1.7 Настройка и проверка DHCP-ретрансляции на R2.

#### 1.7.1 Настройка команды DHCP-ретрансляции на R2
```
R2(config)#interface GigabitEthernet0/1
R2(config)#ip helper-address 10.10.0.1

```
#### 1.7.2 Проверка выдачи IP адреса хосту PC_B.

- Вывод команды ipconfig /all на PC_B
```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: ccna-lab.com
   Physical Address................: 0040.0BC7.B270
   Link-local IPv6 Address.........: FE80::240:BFF:FEC7:B270
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.102
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: ::
                                     192.168.1.97
   DHCP Servers....................: 10.10.0.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-7C-A4-11-0E-00-40-0B-C7-B2-70
   DNS Servers.....................: ::
                                     192.168.1.97
```
#### 1.7.3 Проверка выдачи адреса хоту PC-B командой show ip dhcp bindings

```
R1#show ip dhcp binding 
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      00E0.A35C.3404           --                     Automatic
192.168.1.102    0040.0BC7.B270           --                     Automatic
```

## 2. Реализация DHCPv6.

### 2.1 Создание сети.

Создадим топологию данной сети в программе cisco packet tracer в соответствии с представленной схемой. Она не отличается от прошлой схемы.

![](lesson-17_net_topology.png)

### 2.2. Выполнение базовых настроек маршрутизаторов и коммутаторов.
Выполняется аналогично пункту 1.1. данного задания.

### 2.3. Настройка маршрутизатора R1.
- Настроим подинтерфейсы G0/0 и G0/1 на маршрутизаторе R1:
```
R1> enable
R1#configure terminal
R1(config)#interface GigabitEthernet0/0
R1(config-if)#description connect_to_R2
R1(config-if)#ipv6 address 2001:db8:acad:2::1/64
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#no shut
R1(config-subif)#exit
R1(config)#interface GigabitEthernet0/1
R1(config-if)#description connect_to_S1
R1(config-if)#ipv6 address 2001:db8:acad:1::1/64
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#no shut
R1(config-subif)#exit
```

- Включим поддержку IPv6 маршрутизации, которая по умолчанию выключена:
```
R1(config)#ipv6 unicast-routing
```

- Настроим маршрутизацию по умолчанию на маршрутизаторе R1 для перенаправления пакетов на R2 с его адресом 2001:db8:acad:2::2 :
```
R1(config)#ipv6 route ::/0 2001:db8:acad:2::2
```
- Набор команд:

```
configure terminal
interface GigabitEthernet0/0
description connect_to_R2
ipv6 address 2001:db8:acad:2::1/64
ipv6 address fe80::1 link-local
no sh
exit
interface GigabitEthernet0/1
description connect_to_S1
ipv6 address 2001:db8:acad:1::1/64
ipv6 address fe80::1 link-local
no sh
exit
ipv6 unicast-routing
ipv6 route ::/0 2001:db8:acad:2::2
end
wr
```

### 2.4. Настройка маршрутизатора R2.
- Настроим подинтерфейсы G0/0 и G0/1 на маршрутизаторе R2:
```
R2> enable
R2#configure terminal
R2(config)#interface GigabitEthernet0/0
R2(config-if)#description connect_to_R1
R2(config-if)#ipv6 address 2001:db8:acad:2::2/64
R2(config-if)#ipv6 address fe80::2 link-local
R2(config-if)#no shut
R2(config-subif)#exit
R2(config)#interface GigabitEthernet0/1
R2(config-if)#description connect_to_S2
R2(config-if)#ipv6 address 2001:db8:acad:3::1/64
R2(config-if)#ipv6 address fe80::1 link-local
R2(config-if)#no shut
R2(config-subif)#exit
```
- Включим поддержку IPv6 маршрутизации, которая по умолчанию выключена:
```
R2(config)#ipv6 unicast-routing
```

- Настроим маршрутизацию по умолчанию на маршрутизаторе R2 для перенаправления пакетов на R1 с его адресом 2001:db8:acad:2::1 :
```
R2(config)#ipv6 route ::/0 2001:db8:acad:2::1
```

- Набор команд:

```
configure terminal
interface GigabitEthernet0/0
description connect_to_R1
ipv6 address 2001:db8:acad:2::2/64
ipv6 address fe80::2 link-local
no sh
exit
interface GigabitEthernet0/1
description connect_to_S2
ipv6 address 2001:db8:acad:3::1/64
ipv6 address fe80::1 link-local
no sh
exit
ipv6 unicast-routing
ipv6 route ::/0 2001:db8:acad:2::1
end
wr
```
### 2.5. Убедимся, что PC_A присваивается IPv6 адерс по SLAAC:

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 00E0.B0CC.2EC1
   Link-local IPv6 Address.........: FE80::2E0:B0FF:FECC:2EC1
   IPv6 Address....................: 2001:DB8:ACAD:1:2E0:B0FF:FECC:2EC1
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-E2-25-BA-B9-00-E0-B0-CC-2E-C1
   DNS Servers.....................: ::
                                     0.0.0.0
```
Примечание: часть адреса с идентификатором хоста формируется на основе MAC адреса интерфейса.

### 2.6. Настройка и проверка сервера DHCPv6 на R1

### 2.6.1. Настройка R1 для предоставления stateless DHCPv6 (DHCPv6 без запоминания состояний или SLAAC+DHCPv6) для PC-A.
```
R1(config)# ipv6 dhcp pool R1-STATELESS
R1(config-dhcp)# dns-server 2001:db8:acad::254
R1(config-dhcp)# domain-name STATELESS.com
R1(config)# interface g0/1
R1(config-if)# ipv6 nd other-config-flag 
R1(config-if)# ipv6 dhcp server R1-STATELESS
```

- Убедимся, что PC_A присваивается IPv6 адерс по SLAAC, а также адрес DNS и суффикс DNS:

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 00E0.B0CC.2EC1
   Link-local IPv6 Address.........: FE80::2E0:B0FF:FECC:2EC1
   IPv6 Address....................: 2001:DB8:ACAD:1:2E0:B0FF:FECC:2EC1
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 2044494489
   DHCPv6 Client DUID..............: 00-01-00-01-E2-25-BA-B9-00-E0-B0-CC-2E-C1
   DNS Servers.....................: 2001:DB8:ACAD::254
                                     0.0.0.0
```
### 2.6.2. Настройка R1 для предоставления statelfull DHCPv6 (DHCPv6 c запоминанием состояний) для PC-A.

```
R1(config)# ipv6 dhcp pool R2-STATEFUL
R1(config-dhcp)# address prefix 2001:db8:acad:3:aaa::/80
R1(config-dhcp)# dns-server 2001:db8:acad::254
R1(config-dhcp)# domain-name STATEFUL.com
R1(config)# interface g0/0
R1(config-if)# ipv6 dhcp server R2-STATEFUL
```

#### 5. Файл лабораторной работы в программе cisco packet tracer. 

![lesson-17.pkt](lesson-17.pkt)
