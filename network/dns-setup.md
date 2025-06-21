# DNS-инфраструктура reu-syslab.dev — Fortinet, Bind9, Unbound (рабочий отчёт)

---

## 1. Архитектура и исходные данные

- **Ядро сети:** Fortinet 80F (VLAN, маршрутизация, DHCP)
    
- **Внутренний DNS:** Bind9 (`10.0.2.124`), авторитет по зоне `reu-syslab.dev`
    
- **Unbound:** кеширующий резолвер (планировался как внешний резолвер)
    
- **DHCP Fortinet:** раздаёт IP и адрес DNS (10.0.2.124) всем клиентам
    
- **Внешний резолвинг:** через forwarders (Unbound/публичные DNS)
    

---

## 2. Базовая схема сети

|VLAN|Назначение|Подсеть|Gateway|Комментарий|
|---|---|---|---|---|
|vlan10_mgmt|Management/ПК|10.0.0.0/24|10.0.0.1|ПК, админка|
|vlan20_srv|Серверы/Proxmox|10.0.1.0/24|10.0.1.1|ВМ, Proxmox|
|vlan30_lab|Лаборатория/DNS|10.0.2.0/24|10.0.2.1|DNS, Zabbix|
|vlan40_wifi|WiFi через MikroTik|10.0.4.0/24|10.0.4.1|WiFi|

---

## 3. Настройка Fortinet: DHCP + DNS

- **DHCP-сервер Fortinet** для всех VLAN, выдаёт:
    
    - `Primary DNS`: 10.0.2.124 (Bind9)
        
    - `Search Domain`: reu-syslab.dev
        

---

## 4. Bind9: Авторитетный + внешний резолвинг

- **`/etc/bind/named.conf.local`** — view split-DNS:
    
    ```conf
    acl internal-net { 10.0.0.0/16; 127.0.0.1; };
    view "internal" {
        match-clients { internal-net; };
        recursion yes;
        zone "reu-syslab.dev" { type master; file "/etc/bind/db.reu-syslab.dev.internal"; };
        // ...
        forwarders { 127.0.0.2; }; // или 8.8.8.8, если Unbound не работает
        forward only;
    };
    view "external" {
        match-clients { any; };
        recursion no;
        zone "reu-syslab.dev" { type master; file "/etc/bind/db.reu-syslab.dev.external"; };
        forwarders { 127.0.0.2; };
        forward only;
    };
    ```
    
- **Зоны**: отдельные файлы для internal/external
    
- **PTR-записи**: реализованы для обратного поиска
    

---

## 5. Unbound (кеширующий резолвер):

- **Конфиг:**
    
    ```conf
    server:
      interface: 127.0.0.2
      port: 53
      do-ip4: yes
      do-udp: yes
      do-tcp: yes
      cache-max-ttl: 86400
      cache-min-ttl: 3600
    forward-zone:
      name: "."
      forward-addr: 8.8.8.8
      forward-addr: 1.1.1.1
    ```
    
- **Сервис Unbound должен быть активен** (ошибки чаще всего — из-за конфликта портов, неверного интерфейса, падения демона).
    
- **Bind9 forwarders указывают на Unbound**.
    

---

## 6. Типовые ошибки и разборы

### ❌ Ошибка 1: Нет резолвинга внешних доменов

- **Симптом:**
    
    - `nslookup ya.ru 10.0.2.124` → `Query refused` или `SERVFAIL`
        
- **Причина:**
    
    - Unbound не запущен/падает/слушает не тот порт
        
    - Bind9 не может достучаться до forwarder
        

**Решение:**

- Проверить `systemctl status unbound`
    
- Проверить, что Unbound слушает 127.0.0.2:53
    
- Проверить, что в forwarders Bind9 прописан правильный адрес
    

---

### ❌ Ошибка 2: DNS-зона резолвит неверные адреса

- **Симптом:**
    
    - `nslookup proxmox.reu-syslab.dev` возвращает не тот IP
        
- **Причина:**
    
    - Перепутаны записи A в зоне, не увеличен serial
        

**Решение:**

- Проверить `db.reu-syslab.dev.internal`
    
- Исправить адреса
    
- Увеличить serial
    
- Перезагрузить bind9
    

---

### ❌ Ошибка 3: В браузере всё работает, но в консоли — нет

- **Симптом:**
    
    - ping по FQDN работает, nslookup/консоль не видит имён
        
- **Причина:**
    
    - Кэш, локальный hosts, отсутствие search domain
        

**Решение:**

- Очистить кэш (`ipconfig /flushdns`, `systemd-resolve --flush-caches`)
    
- Проверить `/etc/hosts` и `C:\Windows\System32\drivers\etc\hosts`
    
- Убедиться, что search domain прописан
    

---

### ❌ Ошибка 4: Не работает SSH после изменений

- **Симптом:**
    
    - Не коннектится к DNS-серверу по SSH, остальные ВМ доступны
        
- **Причина:**
    
    - Неправильный IP, сетевые настройки, firewall, перезагрузка, проблемы с консолью
        

**Решение:**

- Проверить адресацию ВМ, доступность по ping
    
- Проверить статус sshd
    
- Перезагрузить ВМ через Proxmox, войти через serial console
    

---

## 7. Финальный рабочий результат

-  Вся инфраструктура на именах (FQDN) — работает
    
-  Внутренние A-записи актуальны
    
-  PTR-записи реализованы для всех важных сервисов
    
-  Клиенты Windows/Linux резолвят FQDN корректно
    
-  Split-DNS: внутренние и внешние зоны разнесены, DNS-инфраструктура “по-взрослому”
    
-  Документированы решения и типовые ошибки
    

---

## 8. Рекомендации по дальнейшему развитию

-  Восстановить работу Unbound как внешнего кеширующего резолвера (принцип: Bind9 → Unbound → интернет)
    
-  Добавить автоматизацию (ansible скрипты для .zone-файлов и backup)
    
-  Реализовать контроль и мониторинг DNS через Zabbix
    
-  Регулярно проверять консистентность zone/serial/backup
    

---

## 9. Ссылки на внутренние документы и файлы

-  `db.reu-syslab.dev.internal` — актуальная зона
    
-  `db.reu-syslab.dev.external` — публичная зона
    
-  `named.conf.local` и `named.conf.options` — структура view и forwarders
    
-  `unbound.conf` — кеширующий резолвер
    

---

**Краткий вывод:**  
_Вся цепочка ошибок, решений и финальной схемы задокументирована.  
Реализован split-DNS, продакшн-подход.  
Платформа готова к дальнейшей автоматизации и развитию._

---

> Обновлено: июнь 2025