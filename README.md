# Nokia ISAM 7360 FX — Zabbix + Grafana Monitoring

Complete monitoring for the **Nokia 7360 ISAM FX** OLT (OLT level), over SNMP
using Nokia proprietary OIDs (enterprise `.637`) plus standard MIB-II — with
Zabbix **7.0** templates and a portable Grafana dashboard.

Two Zabbix templates: the main **Nokia ISAM 7360** template (required) and an
**optional** add-on, **Nokia ISAM 7360 - Uplink (ihub)**, for physical uplink-port
traffic/errors — optional because it requires a dedicated SNMP context (`ihub`) on
the OLT. See [`zabbix/README.md`](zabbix/README.md) for import and setup.

## Tested environment

| Field           | Value               |
| --------------- | -------------------- |
| Hardware        | Nokia 7360 ISAM FX  |
| Chassis         | FX4, FX8            |
| Controller (NT) | FANT-F              |
| Boards (LT)     | FGLT-D, FGLT-B      |
| Firmware        | R6.2.04             |
| ONUs tested     | ~3000               |
| Zabbix          | 7.0 LTS             |

Only tested on the hardware/firmware combinations above so far. If you run this
on other firmware versions or boards, please open an issue or PR to report
compatibility and we'll add it here.

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
| **PON port status & ONU inventory** | Per-port oper/admin status (IF-MIB) and provisioned-ONU count for **all** PON ports (including empty ones). Alerts on ports near the 128-ONU limit, and on a port down with 2+ ONUs (real fault). | None |
| **Board temperature**   | Current temperature (°C), TCA (alarm) high/low and shutdown high/low thresholds for NT-A, NT-B and LT boards.       | None                                                        |
| **Board memory**        | Total and used memory (bytes) for NT-A, NT-B and LT boards. Usage % is evaluated inline in the memory triggers.      | None                                                        |
| **Board CPU**           | CPU load average (%) and monitor status for NT-A, NT-B and LT boards.                                                | `cpu-load <board> monitor start`                            |

## Prerequisites

**On Zabbix:**

- Zabbix 7.0 or newer
- Host with an SNMP interface (UDP 161) pointing to the OLT management IP
- `{$SNMP_COMMUNITY}` macro set on the host

**On the OLT (CLI):** see [`zabbix/README.md`](zabbix/README.md) for the exact
`cpu-load ... monitor start` and `pon ... pm-enable` commands, plus the
`ihub`-context SNMP community setup required only for the optional uplink template.

## Repository layout

```
nokia-isam-7360-monitoring/
├── README.md
├── LICENSE
├── .gitignore
├── zabbix/
│   ├── nokia_isam_7360.yaml             # Zabbix 7.0 template (required)
│   ├── nokia_isam_7360_uplink_ihub.yaml # Zabbix 7.0 template (optional, uplink/ihub)
│   └── README.md                        # Import, macros, CLI prerequisites, tags
└── grafana/
    ├── dashboards/
    │   └── nokia-isam-7360.json # Portable dashboard (${DS_ZABBIX} input)
    ├── provisioning/
    │   ├── datasources/zabbix.yml
    │   └── dashboards/nokia.yml
    └── README.md
```

## License

MIT — see [LICENSE](LICENSE).

## Credits

- Author: Rafael Bastos
- Created with Claude Code (Anthropic)
