# Протокол NAT.

## Типы адресов в NAT

1. Inside
- Inside Local - адрес узла в локальной внутренней сети, от которого и до которого идут пакеты, подлежащие nat-трансляции.
- Inside Global - адрес, в который будет транслироваться локальный адрес (глобальный адрес на устройстве, на котором заданы правила трансляции)
2. Outside
-  Outside Local - адрес в локальной внешней сети, до которого и от которого идут пакеты, подлежащие nat-трансляции.
-  Outside Global - адрес узла в глобальной внешней сети, до которого и от которого идут пакеты, подлежащие nat-трансляции. Если направление пакета Локальная Сеть -->> Глобальная сеть (from inside to outside), то это будет адрес назначения. Если направление пакета Глобальная сеть -->> Локальная Сеть (from outside to inside), то адрес источника.

## Типы NAT

- Статический NAT

Статический NAT определяет однозначное сопоставление локальных и глобальных адресов, настроенных сетевым администратором, которые остаются постоянными.
Одному локальному адресу сопоставляется один глобальный. Метод статического преобразования особенно полезен для веб-серверов или устройств, которые должны иметь постоянный адрес, доступный из Интернета — например, для веб-сервера компании.

- Динамический NAT

При динамическом преобразовании NAT используется пул публичных адресов, которые назначаются в порядке очереди («первым пришел — первым обслужили»). Когда внутреннее устройство запрашивает доступ к внешней сети, динамическое преобразование NAT назначает доступный публичный IPv4- адрес из пула.

- Преобразование адресов портов (PAT) или NAT с перегрузкой портов (overload)

Преобразование адреса и номера порта (PAT), также называемое NAT с перегрузкой, сопоставляет множество частных IPv4-адресов одному или нескольким глобальным IPv4-адресам, используя для идентификации TCP порты. 

Преобразование PAT пытается сохранить оригинальный порт источника. Если первоначальный порт источника уже используется, PAT назначает первый доступный номер порта, начиная с наименьшего в соответствующей группе портов: 0-511, 512-1,023, или 1,024-65,535.

Если доступных портов больше нет, а в пуле адресов есть нескольковнешних адресов, PAT переходит к следующему адресу, пытаясь выделить первоначальный порт источника. Данный процесс продолжается до тех пор, пока не исчерпаются как доступные порты, так и внешние IPv4-адреса в пуле адресов.

Некоторые пакеты не содержат номера порта уровня 4, например сообщения ICMPv4. Процесс преобразования PAT обрабатывает каждый из этих
протоколов по-разному. Например, сообщения запросов ICMPv4, эхо-запросы и эхо-ответы содержат идентификатор запроса (Query ID). ICMPv4 использует идентификатор запроса (Query ID), чтобы сопоставить эхо-запрос с соответствующим эхо-ответом.

## Формат команд для настройки NAT

### Настройка интерфейсов как inside и outside

Для работы NAT прежде всего надо настроить интерфейсы, которые будут участвовать в NAT преобразовании. Интерфейс маршрутизатора, подключенный к внутренней сети будет inside, к глобальной - outside.
```
R1(config)#interface fa0/0
R1(config-if)#ip nat outside
 ``` 
```
R1(config)#interface fa0/1
R1(config-if)#ip nat inside
``` 

### Cтатический NAT
```
R1(config)#ip nat  inside source  static  <Inside Local IP>  <Inside Global IP>
```
- **ip nat** - начало команды (всегда одинаковое) 	
- **inside source** - данные параметры команды означают, что транслируются адреса источника пакетов, приходящих на интерфейс, настроенный как ip nat inside.
- **static** 	- создается статическая трансляция (глобальный внутренний адрес не меняется).

Пример
```
R1(config)#ip nat inside source static 192.168.10.254 209.165.201.5
```


### Динамический NAT

- Сначала нужно определить диапазон адресов, подлежащих трансляции с помощью ACL:
```
R1(config)#ip access-list standard INSIDE-NET
R1(config-sacl)#permit 192.168.0.0 0.0.255.255
```

-Затем нужно настроить пул глобальных адресов

```
R1(config)#ip nat pool NAT-POOL 209.165.200.226 209.165.200.240 netmask 255.255.255.224
```
Затем настраиваем трансляцию
```
R1(config)#ip nat inside source list INSIDE-NET pool NAT-POOL
```
### Статический PAT.

Чтобы настроить PAT нужно просто к текущиим командам настройки статического и динамического NAT добавить ключевое слово overload


```
ip nat inside source list 1 interface serial 0/1/0 overload
```
```
R1(config)#ip nat inside source list INSIDE-NET pool NAT-POOL overload
```

show ip nat translations
show ip nat statistics
