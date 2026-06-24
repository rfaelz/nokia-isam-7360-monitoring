# Template Zabbix — Nokia ISAM 7360 FX

## Arquivo

`nokia_isam_7360_unico.yaml` — template único para Zabbix 6.0+.

## Como importar

1. Acesse **Data collection → Templates → Import**
2. Selecione o arquivo `nokia_isam_7360_unico.yaml`
3. **Primeira importação:** deixe as opções padrão e clique em **Import**
4. **Reimportação (atualização):** marque **"Delete missing"** em todos os grupos (items,
   triggers, graphs, discovery rules) para remover protótipos obsoletos da versão anterior

## Como vincular ao host

1. No Zabbix, abra o host da OLT
2. Certifique-se de que há uma **interface SNMP** configurada (UDP 161, versão 2c)
3. Na aba **Templates**, adicione `Nokia ISAM 7360`
4. Defina a macro `{$SNMP_COMMUNITY}` no host com a community correta da OLT

## Macros

| Macro | Padrão | Descrição |
|---|---|---|
| `{$SNMP_COMMUNITY}` | `public` | Community SNMP v2c da OLT |
| `{$PON.UTIL.WARN}` | `90` | Limiar de utilização de PON (%) para alerta WARNING |
| `{$PON.DROP.WARN}` | `200` | Limiar de frames descartados por intervalo de 5 min para AVERAGE |
| `{$PON.DOWN.BPS_PER_PCT}` | `24883200` | bps por 1% downstream. Padrão GPON; use `99532800` para XGS-PON |
| `{$PON.UP.BPS_PER_PCT}` | `12441600` | bps por 1% upstream. Padrão GPON; use `99532800` para XGS-PON/XG-PON |
| `{$MEM.WARN}` | `90` | Limiar de uso de memória (%) para WARNING |
| `{$MEM.CRIT}` | `95` | Limiar de uso de memória (%) para HIGH |
| `{$CPU.WARN}` | `80` | Limiar de CPU (%) para WARNING |
| `{$CPU.CRIT}` | `90` | Limiar de CPU (%) para HIGH |

Sobrescreva as macros no host para personalizar os limiares sem modificar o template.

## Pré-requisitos na OLT (TL1)

### Contadores de utilização PON

Deve ser habilitado individualmente por porta. A descoberta LLD só retorna PONs com
`PMSTATE=PMENABLED` e pelo menos uma ONT com tráfego ativo:

```
SET-PMMODE-PONUTIL::PONUTIL-1-1-<slot>-<porta>::::PMSTATE=PMENABLED;
```

Exemplo para as 4 portas do slot 1:

```
SET-PMMODE-PONUTIL::PONUTIL-1-1-1-1::::PMSTATE=PMENABLED;
SET-PMMODE-PONUTIL::PONUTIL-1-1-1-2::::PMSTATE=PMENABLED;
SET-PMMODE-PONUTIL::PONUTIL-1-1-1-3::::PMSTATE=PMENABLED;
SET-PMMODE-PONUTIL::PONUTIL-1-1-1-4::::PMSTATE=PMENABLED;
```

Confirme que os contadores estão acessíveis via SNMP:

```
snmpwalk -v2c -c <community> <ip> 1.3.6.1.4.1.637.61.1.35.21.57.1.16
```

### Monitor de CPU por placa

```
ED-CPULOAD::ALL::START;
```

Confirme via SNMP:

```
snmpwalk -v2c -c <community> <ip> 1.3.6.1.4.1.637.61.1.9.29.1.1.4
```

> O item **Status do monitor de CPU** coleta `OPERATESTATUS`. O valor `3 = PROCEEDING` indica
> que o monitor está ativo. Qualquer outro valor gera um alerta INFO — a média de CPU pode
> estar congelada no último valor coletado antes da parada do monitor.

## Esquema de tags

| Tag | Valor | Aplicação |
|---|---|---|
| `component` | `PON` | Itens de utilização e tráfego das portas PON |
| `component` | `Optics` | Itens dos transceivers XFP (NTAXFP-1..4) |
| `component` | `Temperature` | Protótipos de temperatura por placa (LLD) |
| `component` | `Memory` | Protótipos de memória por placa (LLD) |
| `component` | `CPU` | Protótipos de CPU por placa (LLD) |
| `component` | `System` | Uptime, firmware e modo do FAN |
| `pon` | ex. `1/1/1/1` | Identifica a porta PON no formato rack/frame/slot/porta |
| `board` | ex. `NT-A`, `NT-B`, `LT-1-1-2` | Identifica a placa |
| `xfp` | ex. `NTAXFP-1` | Identifica o transceiver de uplink |

Use as tags para filtrar problemas no Zabbix e como labels em painéis Grafana.
