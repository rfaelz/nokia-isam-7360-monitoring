# Nokia ISAM 7360 FX — Zabbix + Grafana Monitoring

Complete monitoring for the **Nokia 7360 ISAM FX** OLT (OLT level), over SNMP v2c
using Nokia proprietary OIDs (enterprise `.637`) plus standard MIB-II — with a
Zabbix **7.0** template and a portable Grafana dashboard.

## Target

| Field         | Value              |
| ------------- | ------------------ |
| Hardware      | Nokia 7360 ISAM FX |
| Firmware      | R6.2.04            |
| Protocol      | SNMP v2c           |
| Zabbix        | 7.0 LTS            |

## What is monitored

### Static items (collected once the template is linked)

| Metric                     | Details                                                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **Management uptime**      | `snmpEngineTime` — management/SNMP plane uptime of the NT. Detects reboot and management restart.                                     |
| **Firmware version**       | Extracted from `sysDescr` (MIB-II). INFO alert when it changes — possible upgrade or downgrade.                                       |
| **Chassis FAN mode**       | default / eco / protect / classic. INFO alert when it differs from `{$FAN.MODE.EXPECTED}`.                                            |
| **XFP transceiver optics** | NTAXFP-1..4: RX/TX power (dBm), temperature (°C), voltage (V), bias current (mA), LOS, TX fault, DDM status and OLT threshold flags. |

### Low-Level Discovery (LLD)

| Rule                    | Discovers                                                                                                            | OLT prerequisite                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| **PON ports**           | Utilization DS/US (%), active ONTs, RX dropped frames, estimated DS/US bandwidth (bps). Filters PONs with no active ONTs. | `pon-pmcollect pm-enable` per port                          |
| **Board temperature**   | Current temperature (°C), TCA (alarm) high/low and shutdown high/low thresholds for NT-A, NT-B and LT boards.       | None                                                        |
| **Board memory**        | Total and used memory (bytes) for NT-A, NT-B and LT boards. Usage % is evaluated inline in the memory triggers.      | None                                                        |
| **Board CPU**           | CPU load average (%) and monitor status for NT-A, NT-B and LT boards.                                                | `cpu-load <board> monitor start`                            |

## Prerequisites

**On Zabbix:**

- Zabbix 7.0 or newer
- Host with an SNMP interface (UDP 161, v2c) pointing to the OLT management IP
- `{$SNMP_COMMUNITY}` macro set on the host

**On the OLT (CLI):** see [`zabbix/README.md`](zabbix/README.md) for the exact
`cpu-load ... monitor start` and `pon ... pm-enable` commands.

## Repository layout

```
nokia-isam-7360-monitoring/
├── README.md
├── LICENSE
├── .gitignore
├── zabbix/
│   ├── nokia_isam_7360.yaml     # Zabbix 7.0 template
│   └── README.md                # Import, macros, CLI prerequisites, tags
└── grafana/
    ├── dashboards/
    │   └── nokia-isam-7360.json # Portable dashboard (${DS_ZABBIX} input)
    ├── provisioning/
    │   ├── datasources/zabbix.yml
    │   └── dashboards/nokia.yml
    └── README.md
```

## Roadmap

- [x] Zabbix 7.0 template for Nokia 7360 ISAM FX (PON, XFP, temperature, memory, CPU, system)
- [x] Grafana dashboard (Zabbix plugin — Alexander Zobnin), with OLT selector
  * System health overview (uptime, firmware, FAN, CPU, memory %, temperature)
  * PON utilization and traffic (DS/US %, bandwidth, active ONTs, dropped frames)
  * Uplink XFP optics (RX/TX power, temperature, LOS)
- [ ] PON port status (UP/DOWN)
- [ ] XFP / VLAN traffic counters
- [ ] Provisioned-ONU inventory per port (with near-limit macro)
- [ ] LOS / DYING-GASP counters (evaluate SNMP trap vs. polling)
- [ ] Out-of-band (FUI4) Ethernet port status

## License

MIT — see [LICENSE](LICENSE).
