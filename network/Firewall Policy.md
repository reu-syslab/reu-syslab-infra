
# Политика межсетевого экрана (Firewall) — FortiGate 80F

## VLAN → Internet

| VLAN            | IP Range         | Доступ в интернет | Доступ к другим VLAN |
|-----------------|------------------|-------------------|-----------------------|
| vlan10_mgmt     | 10.0.0.0/24      |  Да             |  Да (ко всем)       |
| vlan20_srv      | 10.0.1.0/24      |  Да             |  Только от mgmt     |
| vlan30_lab      | 10.0.2.0/24      |  Да             |  Только нужные      |
| vlan40_wifi     | 10.0.4.0/24      |  Да             |  Нет                |
| vlan50_mincraft | 10.0.5.0/24      |  Да             |  Полностью изолирован|

## CLI-конфигурация изоляции vlan50_mincraft

```cli
config firewall address
    edit "vlan50_mincraft"
        set subnet 10.0.5.0 255.255.255.0
    next
    edit "vlan10_mgmt"
        set subnet 10.0.0.0 255.255.255.0
    next
    edit "vlan20_srv"
        set subnet 10.0.1.0 255.255.255.0
    next
    edit "vlan30_lab"
        set subnet 10.0.2.0 255.255.255.0
    next
    edit "vlan40_wifi"
        set subnet 10.0.4.0 255.255.255.0
    next
    edit "dmz_proxy"
        set subnet 10.0.10.0 255.255.255.0
    next
end

config firewall addrgrp
    edit "internal_subnets"
        set member "vlan10_mgmt" "vlan20_srv" "vlan30_lab" "vlan40_wifi" "dmz_proxy"
    next
end

config firewall policy
    edit 101
        set name "vlan50_deny_internal"
        set srcintf "vlan50_mincraft"
        set dstintf "any"
        set srcaddr "vlan50_mincraft"
        set dstaddr "internal_subnets"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```
