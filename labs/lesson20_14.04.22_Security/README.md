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
- Настройка интерфейса GigabitEthernet0/0/1:
```
R1(config)#interface GigabitEthernet0/0/1
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
S1(config)#interface range fa0/2 - 5, fa0/7 - 24, Gi0/1 - 2
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 999
S1(config-if)#shut
```

```
S1(config)#interface range fa0/2 - 17, fa0/18 - 24, Gi0/1 - 2
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 999
S1(config-if)#shut
```
- Проверим, что порты выключены

```
S1# show interfaces status
```
## 2. Дополнительные настройки безопасности коммутаторов.


