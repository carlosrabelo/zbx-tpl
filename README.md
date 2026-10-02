# zbx-tpl

Zabbix templates that monitor toner and supply levels on HP LaserJet M402n, Lexmark MX421ade, and Brother MFC-L9570CDW printers over SNMP.

## Highlights

- Track remaining toner as a percentage on the HP LaserJet M402n, the Lexmark MX421ade, and the Brother MFC-L9570CDW
- Discover Printer-MIB supplies over SNMP and keep cartridge and toner entries
- Read Brother color toner from brInfoMaintenance when Printer-MIB leaves the level unknown
- Track the Lexmark imaging unit and the Brother belt and drum with the same percentage calculation
- Derive remaining life from the current supply level and its maximum capacity
- Drop negative and out-of-range SNMP readings before they are stored
- Open a warning when a supply falls below 10% and a high-severity problem below 5%
- Graph remaining percentage on a fixed 0–100 scale
- Link each template to Generic by SNMP so standard device checks stay available
- Ship as Zabbix 7.4 YAML exports ready to import

## Overview

These templates read the Printer-MIB supplies table (`1.3.6.1.2.1.43.11`) and turn raw SNMP counters into a remaining-life percentage. Discovery runs hourly and keeps the supplies each model should alarm on: toner and cartridges on the HP and the Lexmark, plus the imaging unit on the Lexmark MX421ade.

The Brother MFC-L9570CDW reports toner level as unknown in Printer-MIB. That template reads black, cyan, magenta, and yellow percentages from Brother `brInfoMaintenance` (`1.3.6.1.4.1.2435.2.3.9.4.2.1.5.5.8.0`) and uses Printer-MIB for the belt and the drum.

Each export links the stock Generic by SNMP template, so interface and availability checks stay with the generic template while these files add supply health, triggers, and graphs.

## Prerequisites

- **Zabbix 7.4+** — import target; the files use export version `7.4`
- **Generic by SNMP** — stock template these exports link; it ships with Zabbix
- **SNMP access to the printer** — UDP/161 with a community string or SNMPv3 credentials

## Installation

Clone the repository:

```bash
git clone https://github.com/carlosrabelo/zbx-tpl.git
cd zbx-tpl
```

In the Zabbix frontend, open **Data collection → Templates → Import**, choose one YAML file, and import it:

- `templates/hp-laserjet-m402n.yaml` for the HP LaserJet M402n
- `templates/lexmark-mx421ade.yaml` for the Lexmark MX421ade
- `templates/brother-mfc-l9570cdw.yaml` for the Brother MFC-L9570CDW

Import one file at a time. Zabbix creates the template in **Templates/Network devices**.

## Usage

### Check the supplies table

Confirm the printer answers Printer-MIB before you link a template:

```bash
snmpwalk -v2c -c public 192.0.2.10 1.3.6.1.2.1.43.11.1.1.6
```

Descriptions that match `cartridge` or `toner` become toner items. On the Lexmark, descriptions that match `imaging` become imaging-unit items. On the Brother, descriptions that match `belt` or `drum` become unit items, and waste toner is ignored.

### Link the template

1. Create a host with an SNMP interface pointed at the printer
2. Set the SNMP version and community (or SNMPv3 credentials) on that interface
3. Link **HP LaserJet M402n** or **Lexmark MX421ade**
4. Wait for the hourly discovery rule, or run it once from the host's discovery rules

### Read the results

Discovery creates these items for each matching supply:

- Toner level as a percentage, plus the raw level and maximum capacity
- On the Lexmark only, imaging-unit level as a percentage, plus the raw level and maximum capacity
- On the Brother only, belt and drum level as a percentage, plus the raw level and maximum capacity

Triggers:

| Condition | Severity |
|---|---|
| Remaining level below 10% | Warning |
| Remaining level below 5% | High |

A graph of remaining percentage uses a fixed axis from 0 to 100.

## Project Layout

```
templates/                          # Zabbix 7.4 YAML exports
├── brother-mfc-l9570cdw.yaml       # Toner, belt, and drum monitoring for the Brother MFC-L9570CDW
├── hp-laserjet-m402n.yaml          # Toner monitoring for the HP LaserJet M402n
└── lexmark-mx421ade.yaml           # Toner and imaging-unit monitoring for the Lexmark MX421ade
```

## Development

Copy an existing export into `templates/` when you add a printer. Name the file `<vendor>-<model>.yaml`. After the copy:

1. Set a new template name and visible name
2. Generate a new UUID for every object in the file
3. Replace the template name inside trigger expressions and graph item hosts
4. Adjust the discovery filter so it matches that model's supply descriptions
5. Keep `zabbix_export.version` at `7.4` and the link to `Generic by SNMP`
6. Import the file on a test Zabbix server and confirm items, triggers, and the graph

List the templates in the repository:

```bash
ls -1 templates/*.yaml
```

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/description`
3. Commit with Conventional Commits: `git commit -m "feat: add X"`
4. Push and open a pull request

## License

License terms for this repository are unset.
