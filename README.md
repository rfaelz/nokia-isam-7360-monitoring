# Nokia ISAM 7360 FX — Zabbix Monitoring

Template Zabbix 6.0 para monitoramento completo da OLT **Nokia 7360 ISAM FX** (nível de OLT),
via SNMP v2c com OIDs proprietários Nokia (enterprise `.637`) e MIB-II padrão.

## Alvo

| Campo | Valor |
|---|---|
| Hardware | Nokia 7360 ISAM FX |
| Firmware alvo | R6.2.04 |
| Protocolo | SNMP v2c |

## O que é monitorado

### Itens estáticos (coletados após vincular o template)

| Métrica | Detalhes |
|---|---|
| **Uptime de gerência** | `snmpEngineTime` — tempo do plano de gerência/SNMP da NT. Detecta reboot e restart de gerência. |
| **Versão de firmware** | Extraída do `sysDescr` (MIB-II). Alerta INFO quando muda — possível upgrade ou downgrade. |
| **Modo do FAN do chassis** | default / eco / protect / classic. Alerta INFO se sair do modo PROTECT. |
| **Ópticos dos transceivers XFP** | NTAXFP-1 a NTAXFP-4: RX/TX power (dBm), temperatura (°C), tensão (V), corrente de bias (mA), LOS, TX fault, status DDM e flags de limiar da própria OLT. |

### Descoberta automática — LLD

| Regra | O que descobre | Pré-requisito na OLT |
|---|---|---|
| **Portas PON** | Utilização DS/US (%), ONTs com tráfego, frames descartados RX, banda estimada DS/US (bps). Filtra PONs sem ONTs ativas. | `SET-PMMODE-PONUTIL` por porta (TL1) |
| **Temperatura por placa** | Temperatura atual (°C), limiares TCA (alarme) alto/baixo e shutdown alto/baixo para NT-A, NT-B e placas LT. | Nenhum |
| **Memória por placa** | Memória total e usada (bytes) para NT-A, NT-B e placas LT. | Nenhum |
| **CPU por placa** | Carga média de CPU (%), status do monitor e timestamp de início para NT-A, NT-B e placas LT. | `ED-CPULOAD::ALL::START` (TL1) |

## Pré-requisitos

**No Zabbix:**
- Zabbix 6.0 ou superior
- Host configurado com interface SNMP (UDP 161, versão 2c) apontando para o IP de gerência da OLT
- Macro `{$SNMP_COMMUNITY}` definida no host com a community correta

**Na OLT (via TL1):**

```
# Habilitar contadores PON por porta (exemplo: slot 1, porta 1):
SET-PMMODE-PONUTIL::PONUTIL-1-1-1-1::::PMSTATE=PMENABLED;

# Habilitar monitor de CPU para todas as placas:
ED-CPULOAD::ALL::START;
```

Consulte [`zabbix/README.md`](zabbix/README.md) para instruções detalhadas de importação, macros e
verificação via `snmpwalk`.

## Estrutura de pastas

```
nokia-isam-7360-monitoring/
├── README.md
├── LICENSE
├── .gitignore
├── zabbix/
│   ├── nokia_isam_7360_unico.yaml   # Template Zabbix 6.0
│   └── README.md                    # Importação, macros, pré-requisitos TL1, tags
└── grafana/
    ├── dashboards/                  # Dashboards JSON (em breve)
    ├── provisioning/
    │   ├── datasources/             # datasource YAML
    │   └── dashboards/              # provisioning YAML
    └── README.md                    # Arquitetura e instruções de provisionamento
```

## Roadmap

- [x] Template Zabbix 6.0 para Nokia 7360 ISAM FX (PON, XFP, temperatura, memória, CPU, sistema)
- [ ] Dashboard Grafana (plugin Zabbix — Alexander Zobnin) em `grafana/`
  - Painel de utilização PON (DS/US %, ONTs ativas, frames descartados)
  - Painel de ópticos XFP (RX/TX power, temperatura, LOS)
  - Painel de saúde do sistema (CPU, memória, temperatura por placa)
  - Painel de resumo geral (uptime, firmware, modo do FAN)

## Licença

MIT — veja [LICENSE](LICENSE).
