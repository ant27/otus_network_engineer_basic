## Угрозы безопасности уровня L2.

### Категории атак на безопасность сетевого уровня L2:
1. Атака на таблицу МАС - атака переполнения таблицы CAM коммутатора.
2. Атаки на сети VLAN - атаки на устройства в нативной VLAN и атаки с использованием двойного тегирования VLAN.
3. Атаки на DHCP - атаки на истощение ресурсов DHCP.
4. Атаки на протокол ARP - атаки подмены ARP и "отравление" кеша ARP.
5. Атаки с подменой адреса - атаки подмены MAC и IP адресов.
6. Атаки протокола STP - атаки путем манипуляции протокола.

### Атака на таблицу МАС

Таблицы MAC-адресов коммутаторов имеют фиксированный размер. Злоумышленник c помощью инструмента генерации фиктивных MAC-адресов (например macof) отправляет фиктивные MAC-адреса источника до тех пор, пока таблица МАС-адресов коммутатора не заполнится. Когда это происходит, коммутатор обрабатывает кадр как неизвестную одноадресную рассылку и начинает пересылать весь входящий трафик из всех портов в той же VLAN без учета таблицы MAC. Это условие теперь позволяет атакующему захватить все кадры, отправленные с одного хоста на другой в локальной сети или локальной сети VLAN.
Чтобы нейтрализовать атаки переполнения таблицы MAC-адресов, сетевые администраторы должны включать режим защиты портов (Port security). Защита порта позволит узнать только определенное количество исходных MAC-адресов порта.

### Атаки на сети VLAN

#### Атака VLAN Hopping

Атака использует функцию автоматического согласования транковых портов, включенную по умолчанию на портах коммутатора. 
Эту атаку можно провести, например с помощью утилиты Yersinia, входящей в стостав Kali Linux. 

Для борьбы с атакой необходимо отключить автосогласование портов.

#### Атака VLAN с помощью двойного тегирования.

Атака использует внедрение в кадр ethernet дополнительного VLAN тега и использование для внедрения нативной VLAN, известной злоумышленнику. После попадания кадра с двойным тегированием в магистральный порт коммутатора первый тег vlan снимается (так как он нативный) и кадр пересылается далее. Следующий коммутатор считывает уже внутренний тег и рассылает кадр в порты, соответсвующие vlan, указанному во внутреннем теге.

Для борьбы с атакой необходимо отключить автосогласование портов и сделать нативный vlan отличным от 1.

### Атаки на DHCP

#### Атака на истощение DHCP

Цель атаки с истощение DHCP - создать отказ в обслуживании (DoS) для подключения клиентов. Для атаки путем истощения ресурсов DHCP необходим специальный инструмент,например Gobbler. Gobbler способен искать все доступные для аренды IP-адреса и пытается все их арендовать. В частности, он создает сообщения DHCP Discovery с поддельными MAC-адресами.

#### Атака поддельным DHCP (spoofing)

Атака состоит в том, что к сети подключается мошеннический DHCP-сервер и предоставляет ложные параметры настройки IP легитимным клиентам. Подставной сервер может предоставлять различные неправильные сведения: шлюз по умолчанию, DNS-сервер, IP-адрес.

### Атаки на протокол ARP

#### ARP spoofing (подмена ARP)

Атака основана на уязвимости протокола ARP, который не проверяет подлинности ARP-запросов и ARP-ответов. Хост злоумышленника может послать так называемый gratuitous ARP (самообращенный ARP), в котором указывает свой MAC для целевого IP. Все хосты в сегменте, принимая данный ARP, обновляют свои ARP таблицы и начинают пересылать сообщения на хост злоумышленника.