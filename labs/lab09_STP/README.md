# Лабораторная работа №9. Развертывание коммутируемой сети с резервными каналами (Протокол связующего дерева - STP ).


### 1.1 Создадим топологию данной сети в программе cisco packet tracer в соответсвии с представленной схемой. 

![](net_topology_stp.png)


### 1.2. Выполнение базовых настроек узлов ПК.

Настроим сетевые параметры узлов ПК (IP адреса, маски и шлюзы) в соответствии со схемой.

Здесь есть противоречие, с которым, думаю нужно обратиться к преподавателю - при автоматической настройке IPv6 узлов с помощью SLAAC, адрес DNS сервера автоматически не настраивается. Для его настройки вручную необходимо выключить SLAAC.

![](pc0laptop0.png)

![](pc1laptop1.png)

![](servers.png)



### 1.2. Выполнение настроек коммутаторов.

- Настройка SW1


```
SW1(config)#vlan 22
SW1(config-vlan)#name vlan_22
SW1(config-vlan)#exit
SW1(config)#vlan 64
SW1(config-vlan)#name vlan_64
SW1(config-vlan)#exit
SW1(config)#interface fastEthernet 0/2
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 64
SW1(config-if)#exit
SW1(config)#interface fastEthernet 0/3
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 22
SW1(config-if)#exit
SW1(config)#interface fastEthernet 0/1
SW1(config-if)#switchport mode trunk
SW1(config-if)#exit
SW1#copy running-config startup-config
```

- Настройка SW2


```
SW2(config)#vlan 22
SW2(config-vlan)#name vlan_22
SW2(config-vlan)#exit
SW2(config)#vlan 64
SW2(config-vlan)#name vlan_64
SW2(config-vlan)#exit
SW2(config)#interface fastEthernet 0/2
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 64
SW2(config-if)#exit
SW2(config)#interface fastEthernet 0/3
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 22
SW2(config-if)#exit
SW2(config)#interface fastEthernet 0/1
SW2(config-if)#switchport mode trunk
SW2(config-if)#exit
SW2#copy running-config startup-config
```

- Настройка SW0


```
SW0(config)#vlan 22
SW0(config-vlan)#name vlan_22
SW0(config-vlan)#exit
SW0(config)#vlan 64
SW0(config-vlan)#name vlan_64
SW0(config-vlan)#exit
SW0(config)#interface fastEthernet 0/1
SW0(config-if)#switchport mode trunk
SW0(config-if)#exit
SW0(config)#interface fastEthernet 0/2
SW0(config-if)#switchport mode trunk
SW0(config-if)#exit
SW0(config)#interface GigabitEthernet0/1
SW0(config-if)#switchport mode trunk
SW0(config-if)#exit
SW0#copy running-config startup-config
```

- Настройка SW3


```
SW3(config)#vlan 9
SW3(config-vlan)#name vlan_9
SW3(config-vlan)#exit
SW3(config)#vlan 29
SW3(config-vlan)#name vlan_29
SW3(config-vlan)#exit
SW3(config)#interface fastEthernet 0/1
SW3(config-if)#switchport mode access
SW3(config-if)#switchport access vlan 9
SW3(config-if)#exit
SW3(config)#interface fastEthernet 0/2
SW3(config-if)#switchport mode access
SW3(config-if)#switchport access vlan 29
SW3(config-if)#exit
SW3(config)#interface GigabitEthernet0/1
SW3(config-if)#switchport mode trunk
SW3(config-if)#exit
SW3#copy running-config startup-config
```


### 1.3. Выполнение настроек маршрутизатора


```
R1(config)#interface GigabitEthernet0/0/0.22
R1(config-subif)#description vlan_22
R1(config-subif)#encapsulation dot1Q 22
R1(config-subif)#ip address 172.16.0.1 255.255.252.0
R1(config-subif)#exit
R1(config)#interface GigabitEthernet0/0/0.64
R1(config-subif)#description vlan_64
R1(config-subif)#encapsulation dot1Q 64
R1(config-subif)#ipv6 address FE80::1 link-local
R1(config-subif)#ipv6 address 2002:AAAA:BBBB::1/64
R1(config-subif)#exit
R1(config)#interface GigabitEthernet0/0/1.9
R1(config-subif)#description vlan_9
R1(config-subif)#encapsulation dot1Q 9
R1(config-subif)#ip address 10.0.0.1 255.128.0.0
R1(config-subif)#ipv6 address FE80::1 link-local
R1(config-subif)#ipv6 address 2001:EEEE:FFFF::1/64
R1(config-subif)#exit
R1(config)#interface GigabitEthernet0/0/1.29
R1(config-subif)#description vlan_29
R1(config-subif)#encapsulation dot1Q 29
R1(config-subif)#ip address 192.168.1.1 255.255.255.248
R1(config-subif)#ipv6 address FE80::1 link-local
R1(config-subif)#ipv6 address 2001:CCCC:DDDD::1/64
R1(config-subif)#exit
R1(config)#ipv6 unicast-routing
R1(config)#exit
R1#copy running-config startup-config
```

### 1.4. Внесение A-записей в DNS-сервер.

![](dns_records.png)

Здесь также есть непонятный момент: для узлов с двойным стеком по какому адресу узел будет обращаться к шлюзу и DNS-серверу? А также какой IP-адрес: IPv4 или IPv6 будет выдавать узлу DNS-сервер в ответ на его запрос о разрешении имени узла? 

### 1.5. Проверка сетевой связности.

Также можно пропинговать узлы, находящиеся в одних VLAN. При правильной настройке все должно пинговаться.
Таакже пропингуем сервера по именам, внесенным в DNS-сервер. При правильной настройке все должно пинговаться.






