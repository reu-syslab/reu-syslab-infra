# VLAN-инфраструктура: FortiGate 80F + MikroTik RB5009

## Краткое описание
Архитектура построена на Fortinet 80F как ядре сети с сегментацией по VLAN через порт `internal1`. Управление Wi-Fi и разбивка на зоны реализованы с помощью MikroTik RB5009, который разносит трафик по отдельным физическим интерфейсам.

---

## Шаг 1. Создание VLAN на Fortinet 80F (internal1)

### VLAN-схема (пример):

- `vlan10_mgmt` (10.0.0.0/24) — управление и рабочие ПК
- `vlan20_srv` (10.0.1.0/24) — серверы/Proxmox
- `vlan30_lab` (10.0.2.0/24) — лаборатория/DNS/мониторинг
- `vlan40_wifi` (10.0.4.0/24) — Wi-Fi-зона через MikroTik (internal5)

Пример создания vlan через CLI console 

```bash
FortiGate-80F # config system interface

FortiGate-80F (interface) # edit "vlan10_mgmt"

FortiGate-80F (vlan10_mgmt) # show
config system interface
    edit "vlan10_mgmt"
        set vdom "root"
        set ip 10.0.0.1 255.255.255.0
        set allowaccess ping https ssh snmp http
        set device-identification enable
        set role lan
        set snmp-index 17
        set ip-managed-by-fortiipam disable
        set interface "internal1"
        set vlanid 10
    next
end

FortiGate-80F (vlan10_mgmt) #
```


Пример создание через GUI web interface

GUI->Network->Interface->Create new->Interface 
 
Name: vlan20_srv

Type: VLAN

VLAN protocol 802.1Q

Interface internal1

VLAN ID 20

IP/Netmask 10.0.1.0/255.255.255.0

Administrative Access PING 

OK

все остальное можно настроить позже


### Почему именно так:
- **Один физический uplink (internal1)**: удобно вести “транк” (trunk) — все VLAN трафик идёт по одному кабелю до коммутатора (RB5009).
- Это **облегчает масштабирование** — добавляешь новые VLAN логически, не меняя физику.
- **Изоляция**: каждая зона получает свои политики безопасности и правила межсетевого экрана (firewall).

---

## Шаг 2. Конфигурация MikroTik RB5009 (приём trunk)

- В MikroTik создаём интерфейсы VLAN для каждого тега (10, 20, 30 ) на том физическом порту, куда приходит trunk от Fortinet (например, `ether1`).
- Далее, для каждой VLAN — отдельный bridge/interface, куда “выносим” физический порт (например, `ether2` для management, `ether3` для серверов и т.д.)

### Пример конфигурации (CLI MikroTik):

```shell
# Пример для ether1 — trunk-порт с FortiGate

```bash
/interface vlan
add name=vlan10_mgmt vlan-id=10 interface=ether1
add name=vlan20_srv  vlan-id=20 interface=ether1
add name=vlan30_lab  vlan-id=30 interface=ether1
```

# Bridge для каждой VLAN (или напрямую назначить порт)
```bash
/interface bridge
add name=bridge_mgmt
add name=bridge_srv
add name=bridge_lab
```
# Привязываем физические порты к нужному bridge
```bash
/interface bridge port
add bridge=bridge_mgmt interface=vlan10_mgmt
add bridge=bridge_srv  interface=vlan20_srv
add bridge=bridge_lab  interface=vlan30_lab
```

# Добавить физические LAN-порты для устройств
```bash
add bridge=bridge_mgmt interface=ether2
add bridge=bridge_srv  interface=ether3
add bridge=bridge_lab  interface=ether4
```

Почему так:
Вся сегментация проходит через Fortinet — там проще вести firewall и межзоновые политики.

RB5009 просто “разносит” VLAN по портам — удобно подключать ПК, серверы и лабораторные устройства.

Если потребуется добавить ещё одну зону — достаточно добавить VLAN на Forti и сконфигурировать bridge на MikroTik.

Выводы / Инсайты
Один uplink (trunk) — меньше кабелей, больше порядка.

VLAN в инфраструктуре = логическая чистота, проще трассировать ошибки.

MikroTik отлично справляется с “раздачей” VLAN на разные физические устройства, а FortiGate держит логику и политику доступа.




