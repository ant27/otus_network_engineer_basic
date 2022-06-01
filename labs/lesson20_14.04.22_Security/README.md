# Занятие №20. Лабораторная работа. Принципы обеспечения безопасности сети.

###  Задание:

1. Создание сети и настройка параметров устройств.
2. Дополнительные настройки безопасности коммутаторов.

## 1. Создание сети и настройка параметров устройств.
### 1.1 Создание сети.

Создадим топологию данной сети в программе cisco packet tracer в соответствии с представленной схемой.

![](lesson-20_net_topology.png)

### 1.2 Настройка маршрутизатора R1.

- Исключение двух диапазонов IP-адресов из выдачи DHCP-сервера:
```
R1(config)#ip dhcp excluded-address 192.168.10.1 192.168.10.9
R1(config)#ip dhcp excluded-address 192.168.10.201 192.168.10.202
```
- Настройка DHCP-пула:
```
R1(config)#ip dhcp pool Students
R1(dhcp-config)#network 192.168.10.0 255.255.255.0
R1(dhcp-config)#default-router 192.168.10.1
R1(dhcp-config)#domain-name CCNA2.Lab-11.6.1
```
- Настройка loopback интерфейса:
```
R1(config)#interface Loopback0
R1(config-if)#ip address 10.10.1.1 255.255.255.0
```
- Настройка интерфейса GigabitEthernet0/0:
```
R1(config)#interface GigabitEthernet0/0
R1(config-if)#description Link to S1
R1(config-if)#ip dhcp relay information trusted
R1(config-if)#ip address 192.168.10.1 255.255.255.0
R1(config-if)#no shutdown
```
*Примечание: команда ip dhcp не реализована в Packet Tracer

### 1.3 Настройка коммутаторов.

- Добавление необходимых VLAN на коммутаторы S1 и S2 (показано для S1, для S2 аналогично)
```
S1#conf t
S1(config)#vlan 10
S1(config-vlan)#name Management
S1(config)#vlan 333
S1(config-vlan)#name Native
S1(config)#vlan 999
S1(config-vlan)#name ParkingLot
```
- Настройка интерфеса SVI на VLAN 10 коммутаторов S1 и S2
```
S1(config)#interface vlan 10
S1(config-if)#ip address 192.168.10.201 255.255.255.0
S1(config-if)#exit
S1(config)#ip default-gateway 192.168.10.1
```
```
S2(config)#interface vlan 10
S2(config-if)#ip address 192.168.10.202 255.255.255.0
S2(config-if)#exit
S2(config)#ip default-gateway 192.168.10.1
```
- Настройка транковых портов коммутаторов S1 и S2 (показано для S1, для S2 аналогично)
```
S1(config)#interface fa0/1
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk allowed vlan 10,333,999
S1(config-if)#switchport trunk native vlan 333
```
В качестве native vlan устанавливается специально заведенная vlan, отличная от 1, для предотвращения атаки на vlan

- Отключение согласования DTP (показано для S1, для S2 аналогично)

```
S1(config)#interface fa0/1
S1(config-if)#switchport nonegotiate
```

- Проверка сделанных настроек
```
S1#show interface trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      333

Port        Vlans allowed on trunk
Fa0/1       10,333,999

Port        Vlans allowed and active in management domain
Fa0/1       10,333,999

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,333,999

S1#show interfaces f0/1 switchport | include Negotiation
Negotiation of Trunking: Off
```
- Настройка портов доступа S1 и S2

```
S1(config)#interface range fa0/5 - 6
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 10
```

```
S1(config)#interface range fa0/18
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 10
```
- Перевод неиспользуемых портов в VLAN999 (ParkingLot) и их выключение

```
S1(config)#interface range fa0/2 - 4, fa0/7 - 24, Gi0/1 - 2
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 999
S1(config-if)#shut
```

```
S1(config)#interface range fa0/2 - 17, fa0/19 - 24, Gi0/1 - 2
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 999
S1(config-if)#shut
```
- Проверим, что порты выключены

```
S1# show interfaces status
S1(config-if-range)#do show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        connected    1          auto    auto  10/100BaseTX
Fa0/2                        disabled 999        auto    auto  10/100BaseTX
Fa0/3                        disabled 999        auto    auto  10/100BaseTX
Fa0/4                        disabled 999        auto    auto  10/100BaseTX
Fa0/5                        notconnect   10         auto    auto  10/100BaseTX
Fa0/6                        connected    10         auto    auto  10/100BaseTX
Fa0/7                        disabled 999        auto    auto  10/100BaseTX
Fa0/8                        disabled 999        auto    auto  10/100BaseTX
Fa0/9                        disabled 999        auto    auto  10/100BaseTX
Fa0/10                       disabled 999        auto    auto  10/100BaseTX
Fa0/11                       disabled 999        auto    auto  10/100BaseTX
Fa0/12                       disabled 999        auto    auto  10/100BaseTX
Fa0/13                       disabled 999        auto    auto  10/100BaseTX
Fa0/14                       disabled 999        auto    auto  10/100BaseTX
Fa0/15                       disabled 999        auto    auto  10/100BaseTX
Fa0/16                       disabled 999        auto    auto  10/100BaseTX
Fa0/17                       disabled 999        auto    auto  10/100BaseTX
Fa0/18                       disabled 999        auto    auto  10/100BaseTX
Fa0/19                       disabled 999        auto    auto  10/100BaseTX
Fa0/20                       disabled 999        auto    auto  10/100BaseTX
Fa0/21                       disabled 999        auto    auto  10/100BaseTX
Fa0/22                       disabled 999        auto    auto  10/100BaseTX
Fa0/23                       disabled 999        auto    auto  10/100BaseTX
Fa0/24                       disabled 999        auto    auto  10/100BaseTX
Gig0/1                       disabled 999        auto    auto  10/100BaseTX
Gig0/2                       disabled 999        auto    auto  10/100BaseTX
```
## 2. Дополнительные настройки безопасности коммутаторов.

### Настройка безопасности порта коммутатора S1

- Вывод команды show port-security interface коммутатора S1 с настройками по умолчанию
```
S1#show port-security interface f0/6
Port Security              : Disabled
Port Status                : Secure-down
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 1
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0000.0000.0000:0
Security Violation Count   : 0
```
Значения параметров по умолчанию: 

- Защита портов - выключена
- Максимальное количество записей MAC-адресов - 0
- Режим проверки на нарушение безопасности - shutdown
- Aging Time - 0 mins
- Aging Type - Absolute
- Secure Static Address Aging - Disabled
- Sticky MAC Address - 0

Установим для порта следующие настройки (по условию задания):
- Максимальное количество записей MAC-адресов: 3
- Режим безопасности: restrict
- Aging time: 60 мин.
- Aging type: неактивный

```
S1#conf t 
S1(config)#interface fa0/6
S1(config-if)#switchport port-security
S1(config-if)#switchport port-security maximum 3
S1(config-if)#switchport port-security violation restrict
S1(config-if)#switchport port-security aging time 10
S1(config-if)#switchport port-security aging type inactivity 
```
*Примечание: команда switchport port-security aging type в PacketTracer не реализована.


- Вывод команды show port-security interface коммутатора S1 после настройки
```
S1#show port-security interface fa0/6
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 10 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 3
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0000.0000.0000:0
Security Violation Count   : 0
```
### Настройка безопасности порта коммутатора S2

Установим для порта следующие настройки (по условию задания):
- Максимальное количество записей MAC-адресов: 2
- Тип безопасности: Protect
- Aging time: 60 мин.
- Автоматическое добавление адресов МАС, изученных на порту в конфигурацию.

```
S2#conf t 
S2(config)#interface fa0/18
S2(config-if)#switchport port-security
S2(config-if)#switchport port-security maximum 2
S2(config-if)#switchport port-security violation protect
S2(config-if)#switchport port-security aging time 60
S2(config-if)#switchport port-security mac-address sticky
```
- Вывод команды show port-security interface коммутатора S1 после настройки
```
S2(config-if)#do show port-security interface f0/18
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Protect
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 2
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0000.0000.0000:0
Security Violation Count   : 0
```
### Включение DHCP snooping

- Выполним настройки в соответствии с заданием:
```
S2#conf t 
S2(config)#ip dhcp snooping
S2(config)#ip dhcp snooping vlan 10
S2(config)#interace f0/1
S2(config-if)#ip dhcp snooping trust
S2(config-if)#interace f0/18
S2(config-if)#ip dhcp snooping limit rate 5
```

- Проверка сделанных настроек
```
S2#show ip dhcp snooping
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10
Insertion of option 82 is enabled
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
Interface                  Trusted    Rate limit (pps)
-----------------------    -------    ----------------
FastEthernet0/1            yes        unlimited       
FastEthernet0/18           no         5     
```
- 




Реализация PortFast и BPDU Guard
