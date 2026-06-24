# Grafana — Nokia ISAM 7360 FX

> **Em construção.** Os dashboards serão adicionados em breve nesta pasta.

## Arquitetura prevista

Os dashboards serão provisionados via arquivos de configuração (sem importação manual):

- **Datasource:** plugin [Zabbix for Grafana](https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app/)
  (Alexander Zobnin), conectado ao Zabbix 6.0 via API HTTP.
- **Provisionamento:** YAML para o datasource + JSON para cada dashboard, usando o mecanismo
  nativo de provisioning do Grafana.

## Estrutura de pastas

```
grafana/
├── dashboards/              # Arquivos .json dos dashboards
├── provisioning/
│   ├── datasources/         # datasource.yaml — configura o plugin Zabbix
│   └── dashboards/          # dashboard.yaml — aponta para grafana/dashboards/
└── README.md
```

## Como provisionar (instrução futura)

Quando os arquivos estiverem prontos, monte os volumes no container Grafana:

```yaml
volumes:
  - ./grafana/provisioning/datasources:/etc/grafana/provisioning/datasources
  - ./grafana/provisioning/dashboards:/etc/grafana/provisioning/dashboards
  - ./grafana/dashboards:/var/lib/grafana/dashboards
```

O Grafana carregará o datasource e os dashboards automaticamente na inicialização.

## Painéis planejados

- **Utilização das PONs** — DS/US (%), ONTs com tráfego, frames descartados RX
- **Ópticos XFP** — RX/TX power (dBm), temperatura, LOS, TX fault
- **Saúde por placa** — CPU (%), memória (%), temperatura vs. limiares TCA/shutdown
- **Resumo do sistema** — uptime de gerência, versão de firmware, modo do FAN
