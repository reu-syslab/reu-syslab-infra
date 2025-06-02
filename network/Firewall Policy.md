# Fortinet 80F — Базовые политики безопасности (Firewall Policy)

## 1. Зачем нужны политики?
- Разграничивают трафик между VLAN и внешними сетями.
- Защищают внутренние сервисы от несанкционированного доступа.
- Контролируют интернет-доступ для каждой зоны.

---

## 2. Типовые политики (пример)

| №  | Source Interface | Source Address     | Destination Interface | Destination Address    | Service | Action  | Comments                   |
|----|------------------|-------------------|----------------------|-----------------------|---------|---------|----------------------------|
| 1  | vlan10_mgmt      | all (10.0.0.0/24) | WAN                  | all                   | ALL     | ACCEPT  | Интернет для ПК            |
| 2  | vlan20_srv       | all (10.0.1.0/24) | WAN                  | all                   | ALL     | ACCEPT  | Интернет для серверов      |
| 3  | vlan30_lab       | all (10.0.2.0/24) | WAN                  | all                   | ALL     | ACCEPT  | Интернет для лаборатории   |
| 4  | vlan10_mgmt      | all               | vlan20_srv           | all                   | SSH, RDP| ACCEPT  | Доступ админов к серверам  |
| 5  | vlan20_srv       | all               | vlan30_lab           | all                   | ICMP    | ACCEPT  | Серверы видят лабораторию  |
| 6  | *any*            | *any*             | *any*                | *any*                 | ALL     | DENY    | Блокировка всего остального|

---

## 3. Пример настройки через GUI

1. **Policy & Objects → IPv4 Policy → Create New**
2. Указать Source/ Destination interface и адреса
3. Выбрать сервис (например, ALL, ICMP, SSH)
4. Action: ACCEPT или DENY
5. Enable NAT (для выхода в интернет)
6. Порядок политики имеет значение — сверху вниз!

---

## 4. Пример для CLI

```shell
config firewall policy
    edit 1
        set srcintf "vlan10_mgmt"
        set dstintf "wan1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
    next
    edit 2
        set srcintf "vlan10_mgmt"
        set dstintf "vlan20_srv"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set service "SSH"
    next
    edit 99
        set srcintf "any"
        set dstintf "any"
        set action deny
        set service "ALL"
    next
end
```

Все политики идут сверху вниз — разрешающее правило должно быть выше запрещающего.

Всегда делай “Deny All” в конце списка.

Для доступа к административным интерфейсам (FortiGate, Proxmox) создавай отдельные правила и ограничивай источники.

Ведите документацию по каждой политике (зачем, когда добавлена, кем)

[[vlan-topology.md]]
[[dns-setup.md]]





