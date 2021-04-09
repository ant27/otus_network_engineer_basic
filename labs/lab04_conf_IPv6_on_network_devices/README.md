# Лабораторная работа №4. Настройка IPv6-адресов на сетевых устройствах.

###  Задание:

1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора.
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

Судя по всему 

1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора.


1.1 Создадим топологию данной сети в программе cisco packet tracer.


