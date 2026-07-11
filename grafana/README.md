# Grafana dashboard — Nokia ISAM 7360

Dashboard for the Zabbix template in [`../zabbix/nokia_isam_7360.yaml`](../zabbix/nokia_isam_7360.yaml),
using the [Zabbix plugin (Alexander Zobnin)](https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app/)
as the data source.

## Prerequisite

The `alexanderzobnin-zabbix-datasource` plugin installed, with a working Zabbix
data source configured in Grafana.

## Portable data source (`${DS_ZABBIX}`)

The dashboard references its data source as the input **`DS_ZABBIX`** instead of a
hard-coded UID, so it imports on any Grafana:

- **Manual import** (Dashboards → Import): Grafana prompts once to pick the Zabbix
  data source.
- **Automatic provisioning does *not* substitute `${DS_ZABBIX}`.** If you provision
  this file, replace `${DS_ZABBIX}` with your data source UID first, or keep your
  own concrete-UID copy for provisioning and use this portable file only for sharing.

## Structure

Three rows:

| Row                             | Panels                                                                                          |
| ------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Overview - System Health**    | Uptime, firmware version, chassis FAN mode, temperature per board, CPU load per board, memory usage % per board |
| **PON - Utilization / Traffic** | DS/US utilization (%), DS/US bandwidth, active ONTs, RX dropped frames — per PON                 |
| **Optics - Uplink (XFP)**       | RX/TX power, transceiver temperature, supply voltage, bias current, LOS — per cage              |

An **Equipment (OLT)** variable at the top switches between OLTs (a custom list of
host names — edit it in Dashboard settings → Variables → `host`).

## Notes

- **Memory usage %** is computed client-side (used ÷ total) via panel transformations,
  since the Zabbix calculated item does not resolve LLD siblings in this environment.
- The **firmware** panel needs three things aligned (text query mode, an *Organize
  fields* transform keeping only the value, and Stat *All values* with `Fields = Last value`)
  because it displays a text item. Reuse this recipe for any other text item.
- Some panel-level bindings may need a one-time re-select of the Host in the query
  editor after import, depending on the plugin version. The **Item** filter of each
  panel is already correct; the exact filters are documented in the panel queries.

## How to use

**Manual import:** Dashboards → Import → paste `dashboards/nokia-isam-7360.json` →
select your Zabbix data source when prompted.

**Provisioning:** copy `provisioning/dashboards/nokia.yml` to your Grafana
provisioning path and `dashboards/nokia-isam-7360.json` to the path referenced in it
(`/etc/grafana/provisioning/dashboards/nokia-isam/`). Replace `${DS_ZABBIX}` with your
data source UID first (see above). Restart Grafana.
