# reu-syslab-infra

Multi-zone infrastructure lab for hands-on network engineering, automation, and Fortinet NSE4 exam preparation. Includes segmentation (LAN, DMZ, Isolated), monitoring (Zabbix), automation (Ansible, Terraform), and real hardware (MikroTik, Fortinet, Proxmox). Documentation, test scenarios, and real-world troubleshooting are included.

# reu-syslab-infra

**Multi-zone infrastructure lab for hands-on network engineering, automation, and Fortinet NSE4 exam preparation.**

## Project structure

- [Hardware](docs/hardware-list.md): main goals, topology, hardware, implementation stages.
- [Zones](docs/zones.md): description of LAN, DMZ, Isolated segments.
- [Monitoring](docs/monitoring.md): Zabbix integration, checks, alerting.
- [Scenarios](docs/scenarios.md): practical tasks, troubleshooting, test cases.
- [Scripts](scripts/): automation and setup scripts (bash, PowerShell).
- [Ansible](ansible/): playbooks for automated deployment (future).
- [Terraform](terraform/): infrastructure as code configs (future).
- [Diagrams & images](img/): visual schemes, network topology.
- [DNS-infra](network/dns-setup.md):
- [Vlan](network/vlan-topology.md):

## Quickstart

1. See [docs/architecture.md](docs/architecture.md) for full description and implementation plan.
2. Use [scripts/](scripts/) for quick setup (see scripts/README.md if present).
3. All tasks and progress tracked in [docs/scenarios.md](docs/scenarios.md).

---

## О проекте

Здесь документируется и развивается архитектура лаборатории Ruven"reu-syslab.dev":  
— Сегментация сети через VLAN  
— DNS (Bind9/Unbound)  
— FortiGate + MikroTik  
— Автоматизация, мониторинг, резервное копирование  
— Чеклисты и грабли, которые реально встречаются

---

_Для связи и вопросов: [tg:@Ruven_007]_

**Lab started: 26.05.2025**

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
