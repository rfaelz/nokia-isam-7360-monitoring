# Zabbix template — Nokia ISAM 7360 FX

Zabbix **7.0** template (`nokia_isam_7360.yaml`) for the Nokia 7360 ISAM FX OLT over
SNMP v2c. A single self-contained template: static items + four LLD rules
(PON, board temperature, board memory, board CPU).

## Import

**Data collection → Templates → Import**, select `nokia_isam_7360.yaml`.

When **re-importing** an updated version, enable **"Delete missing"** for *Items*,
*Triggers* and *Discovery rules* so that removed objects are cleaned from the
template (otherwise orphan items remain and LLD key collisions can occur).

Then, on the host:

1. Add an **SNMP interface** (UDP 161, v2c) pointing to the OLT management IP.
2. Set the `{$SNMP_COMMUNITY}` macro.
3. Link the **Nokia ISAM 7360** template.

## OLT prerequisites (CLI)

Some proprietary counters only respond after their monitor/PM is enabled on the OLT.

**Per-board CPU monitor** (required for the CPU LLD):

```
admin system cpu-load nt-a monitor start
admin system cpu-load nt-b monitor start
admin system cpu-load lt:1/1/1 monitor start
admin system cpu-load lt:1/1/2 monitor start
admin system cpu-load lt:1/1/3 monitor start
admin system cpu-load lt:1/1/4 monitor start
```

**Per-PON utilization counters** (required for PON utilization/bandwidth/drops).
A PON is *discovered* even without PM, but its utilization/bandwidth items stay
without data until PM is enabled on that port:

```
configure pon interface 1/1/1/[1...16] utilization pon-pmcollect pm-enable
configure pon interface 1/1/2/[1...16] utilization pon-pmcollect pm-enable
configure pon interface 1/1/3/[1...16] utilization pon-pmcollect pm-enable
configure pon interface 1/1/4/[1...16] utilization pon-pmcollect pm-enable
```

## Macros

| Macro                     | Default    | Purpose                                                             |
| ------------------------- | ---------- | ------------------------------------------------------------------- |
| `{$SNMP_COMMUNITY}`       | `public`   | OLT SNMP v2c community.                                             |
| `{$CPU.WARN}`             | `80`       | Per-board CPU warning threshold (%).                                |
| `{$CPU.CRIT}`             | `90`       | Per-board CPU high threshold (%).                                   |
| `{$MEM.WARN}`             | `90`       | Per-board memory usage warning (%). Evaluated inline (used/total).  |
| `{$MEM.CRIT}`             | `95`       | Per-board memory usage high (%). Evaluated inline (used/total).     |
| `{$PON.UTIL.WARN}`        | `90`       | PON DS/US utilization alert threshold (%).                          |
| `{$PON.DROP.WARN}`        | `200`      | RX dropped frames per interval alert threshold.                     |
| `{$PON.DOWN.BPS_PER_PCT}` | `24883200` | bps per 1% downstream (GPON). XGS-PON = 99532800.                   |
| `{$PON.UP.BPS_PER_PCT}`   | `12441600` | bps per 1% upstream (GPON).                                         |
| `{$PON.NOONT.TIME}`       | `1h`       | Sustained window with zero active ONTs before flagging a PON down.  |
| `{$FAN.MODE.EXPECTED}`    | `2`        | Expected FAN mode (0=default,1=eco,2=protect,3=classic).            |
| `{$PON.ONU.MAX}`          | `128`      | Max provisioned ONUs per PON port (GPON hard limit).                |
| `{$PON.ONU.WARN}`         | `112`      | Provisioned-ONU count that raises a near-limit warning.             |
| `{$PON.DOWN.TIME}`        | `15m`      | Sustained window a PON port must stay down before the port-down alert fires. |

## PON port status & ONU inventory

**Port status** comes from the standard IF-MIB (`ifOperStatus` / `ifAdminStatus`), which
does work for PON ports on this ISAM even though the uplink traffic counters are stubbed.
The PONUTIL index equals the PON `ifIndex`, so the same index decoder is reused.

Important semantics, verified against the CLI: **a PON port reads DOWN when *all* its ONTs
are down; an empty port reads UP.** On a single-ONU port, DOWN therefore just means that one
customer is offline — not a fault. For this reason the port-down alert is *gated*: it only
fires when the port has been down for `{$PON.DOWN.TIME}`, is admin-up, **and** has 2+
provisioned ONUs (i.e. several customers went down together — a real fiber/port fault).

**ONU inventory** is derived from a single bulk walk of the provisioned-ONT table
(`.637...35.11.4.1.1`, one row per provisioned ONT, indexed by `<PON ifIndex>.<ONT id>`).
A master item performs one walk; per-port counts are dependent items that count rows in
Zabbix — one SNMP walk on the OLT regardless of how many ports there are.

> **Large OLTs:** the master item runs every **1h** on purpose. ONU inventory changes slowly,
> and on OLTs with thousands of ONTs a frequent full-column walk stresses the ISAM
> control-plane CPU. Before enabling on a big OLT, verify GETBULK behaviour and watch the
> NT CPU item during the walk.

## Item tags

Every item is tagged for filtering in *Latest data* and *Problems*:

- `component` — subsystem: `PON`, `Optics`, `Temperature`, `Memory`, `CPU`, `System`
- `pon` — PON id (`1/1/x/x`) on PON items
- `board` — board name (`NT-A`, `LT-1-1-x`) on temperature/memory/CPU items
- `xfp` — cage (`NTAXFP-1..4`) on optics items

## Notes / design decisions

- **Memory usage %** is computed **inline in the memory triggers**
  (`100*last(used)/last(total)`), because a Zabbix *calculated item* referencing
  sibling LLD items does not resolve in this environment. For a **graph** of the
  percentage, use the Grafana dashboard (it computes % client-side).
- **Numeric-only OIDs** are used throughout (net-snmp on Debian disables MIB
  loading by default, so symbolic OIDs would fail).
- Empty XFP cages return non-numeric optical values; those items discard the
  reading (JavaScript `return null`) so an empty cage produces no data and no
  false alarm.

## Verify via snmpwalk

```
# PON utilization table (active ONTs column):
snmpwalk -v2c -c <community> <ip> 1.3.6.1.4.1.637.61.1.35.21.57.1.16

# Board temperature table:
snmpwalk -v2c -c <community> <ip> 1.3.6.1.4.1.637.61.1.23.10.1.2

# Board CPU load (after starting the monitor):
snmpwalk -v2c -c <community> <ip> 1.3.6.1.4.1.637.61.1.9.29.1.1.4
```
