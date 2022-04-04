## Занятие 16. Избыточность линков (Etherchannel) и шлюзов (FHRP)

Дата занятия: 31.03.22

### Определение технологии Etherchannel.

Технология Etherchannel необходима для создания избыточности каналов связи между сетевыми устройствами без их блокировки протоколом STP.
Etherchannel объединяет несколько физических портов в один логический

### Ограничения технологии Etherchannel.

- Можно использовать только одинаковые типы интерфейсов: FastEthernet и GigabitEthernet интерфейсы в одном EtherChannel использовать нельзя.
- В один логический интерфейс можно объединить до 8 физических интерфейсов.
- На одном устройстве с Cisco IOS можно создать до 6 логических каналов.
- Созданные логические интерфейсы с двух сторон должны быть настроены или как транки, или находиться в одном VLAN.
- Изменения над логическим интерфейсом приводят к измениям над входящим в него физическим. 

### Типы и режимы протоколов Etherchannel.

- PaGP - проприетарный от Cisco, имеет три режима: 
1. On
2. Desirable
3. Auto
Для организации канала настройки канала на противоположных сторонах должны быть: Desirable/Auto или Desirable/Desirable.
- LACP - мультивендорный протокол, также имеет три режима:
1. On
2. Active
3. Passive
Для организации канала настройки канала на противоположных сторонах должны быть: Active/Passive или Active/Active.
Также для работы канала необходимы:
1. Одинаковая скорость и режим дуплекса
2. Интерефесы должны относиться к одному VLAN или настроены как trunk.
3. Trunk'и должны быть настроены под одинаковы диапазон VLAN. 

### Настройка Etherchannel.

```
Switch(config)#inteface range interface-range
Switch(config-if-range)#channel-group 1 mode active
Switch(config-if-range)#interface port-channel 1
Switch(config-if-range)#switchport mode trunk
Switch(config-if-range)#switchport trunk allowed vlan 1,2,20
```  

### Вывод информации о настроенных Etherchannel.

```
Switch#show erherchannel summary
Switch#show erherchannel port-channel
Switch#show inteface port-channel 1
Switch#show intefaces f0/1 etherchannel
```

### Протокол резервирования первого перехода.
