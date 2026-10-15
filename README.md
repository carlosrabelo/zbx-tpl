# zbx-tpl

Zabbix templates that monitor toner and supply levels on HP LaserJet M402n, Lexmark MX421ade, and Brother MFC-L9570CDW printers, and the Datacom DM1200E 24GP+4GX and DM3324F2 switches, over SNMP.

## Highlights

- Track remaining toner as a percentage on the HP LaserJet M402n, the Lexmark MX421ade, and the Brother MFC-L9570CDW
- Discover Printer-MIB supplies and read Brother color toner, belt, and drum when Printer-MIB leaves the level unknown
- Open a warning when a supply falls below 10% and a high-severity problem below 5%
- Monitor the Datacom DM1200E 24GP+4GX and the DM3324F2 the way Cisco IOS by SNMP covers a switch: reachability, CPU, memory, temperature, fans, power, and interfaces
- Discover ethernet ports, PoE ports, temperature sensors, fans, and power supplies from the tables this firmware actually returns
- Open problems for high CPU, high memory, high temperature, a failed fan or power supply, and a PoE fault. The link-down trigger stays in the template and ships disabled
- Graph interface traffic, CPU, memory, temperature, and PoE consumption
- Link the printer templates to Generic by SNMP; keep each switch template standalone so those checks are not collected twice
- Ship as Zabbix 7.4 YAML exports ready to import

## Overview

These templates read the Printer-MIB supplies table (`1.3.6.1.2.1.43.11`) and turn raw SNMP counters into a remaining-life percentage. Discovery runs hourly and keeps the supplies each model should alarm on: toner and cartridges on the HP and the Lexmark, plus the imaging unit on the Lexmark MX421ade.

The Brother MFC-L9570CDW reports toner level as unknown in Printer-MIB. That template reads black, cyan, magenta, and yellow percentages from Brother `brInfoMaintenance` (`1.3.6.1.4.1.2435.2.3.9.4.2.1.5.5.8.0`) and uses Printer-MIB for the belt and the drum.

Each printer export links the stock Generic by SNMP template, so interface and availability checks stay with the generic template while these files add supply health, triggers, and graphs.

The Datacom DM1200E template stands on its own, in the same role as Cisco IOS by SNMP. The switch is a Broadcom FASTPATH device (`sysObjectID` `1.3.6.1.4.1.4413`), so the Cisco MIBs for CPU, memory, and environment do not apply. This template reads IF-MIB for the 28 ethernet ports, ENTITY-MIB for the chassis, POWER-ETHERNET-MIB for the PoE budget and port faults, and the FASTPATH switching and box-services MIBs for CPU, memory, temperature, fans, and power supplies.

Datacom Device by SNMP is the generic companion, built like the stock Network Generic Device template. It covers ICMP, SNMP identity, IF-MIB traffic, and EtherLike duplex. On firmware 4.0.0.73 the DM1200E does not implement HOST-RESOURCES-MIB or `dot3StatsDuplexStatus`, so hardware uptime stays at zero and duplex discovery stays empty. Its interface counters are 32-bit. Link one switch template per host: they share keys such as `icmpping` and `system.name`.

The DM3324F2 template uses DMswitch-MIB (`1.3.6.1.4.1.3709`), not FASTPATH. CPU is reported in hundredths of a percent, memory is already a percentage, and temperature is one chassis sensor. Fans and power supplies alarm only when the state is red. An activated alarm input opens a warning. Interface counters are 64-bit for `eth-1/1`–`eth-1/28`. This model has no PoE.

## Prerequisites

- **Zabbix 7.4+** — import target; the files use export version `7.4`
- **Generic by SNMP** — stock template the printer exports link; it ships with Zabbix. The switch templates do not link it
- **SNMP access to the device** — UDP/161 with a community string or SNMPv3 credentials

## Installation

Clone the repository:

```bash
git clone https://github.com/carlosrabelo/zbx-tpl.git
cd zbx-tpl
```

In the Zabbix frontend, open **Data collection → Templates → Import**, choose one YAML file, and import it:

- `printers/hp-laserjet-m402n.yaml` for the HP LaserJet M402n
- `printers/lexmark-mx421ade.yaml` for the Lexmark MX421ade
- `printers/brother-mfc-l9570cdw.yaml` for the Brother MFC-L9570CDW
- `switchs/datacom-device.yaml` for a generic Datacom device
- `switchs/datacom-dm1200e.yaml` for the Datacom DM1200E 24GP+4GX
- `switchs/datacom-dm3324f2.yaml` for the Datacom DM3324F2

Import one file at a time. Zabbix creates the template in **Templates/Network devices**.

## Usage

### Check the supplies table

Confirm the printer answers Printer-MIB before you link a template:

```bash
snmpwalk -v2c -c public 192.0.2.10 1.3.6.1.2.1.43.11.1.1.6
```

Descriptions that match `cartridge` or `toner` become toner items on the HP and the Lexmark. On the Lexmark, descriptions that match `imaging` become imaging-unit items. On the Brother, black, cyan, magenta, and yellow are fixed items decoded from `brInfoMaintenance`; descriptions that match `belt` or `drum` become unit items.

### Link the template

1. Create a host with an SNMP interface pointed at the printer
2. Set the SNMP version and community (or SNMPv3 credentials) on that interface
3. Link **HP LaserJet M402n**, **Lexmark MX421ade**, or **Brother MFC-L9570CDW**
4. Wait for the hourly discovery rule, or run it once from the host's discovery rules. On the Brother, also run **Toner levels** once so the four toner percentages fill in without waiting for the hourly SNMP poll.

### Read the results

Discovery creates these items for each matching supply:

- Toner level as a percentage, plus the raw level and maximum capacity
- On the Lexmark only, imaging-unit level as a percentage, plus the raw level and maximum capacity
- On the Brother only, black, cyan, magenta, and yellow toner as fixed percentage items, plus belt and drum level as a percentage with the raw level and maximum capacity

Triggers:

| Condition | Severity |
|---|---|
| Remaining level below 10% | Warning |
| Remaining level below 5%, or the printer has no usable reading | High |

A Printer-MIB level of -1, -2, or -3 means the cartridge is missing or the printer does not know the amount. Those values used to be discarded, so the percentage stayed at the previous reading, often 100%. They are now stored as 0% and the very-low trigger opens.

On the Brother, black, cyan, magenta, and yellow come from the remaining-life fields in `brInfoMaintenance` (hundredths of a percent). The coarse gauge in that same object can stay at 100% after a cartridge is empty or removed, so it is used only when the remaining-life field is absent. A color that is not in the reading is stored as 0%.

A graph of remaining percentage uses a fixed axis from 0 to 100.

### Check the switch

Confirm the DM1200E answers SNMP before you link the template. The system description on firmware 4.0.0.73 is `DM1200E 24GP+4GX Standalone, 4.0.0.73`.

```bash
snmpget -v2c -c public 192.0.2.10 1.3.6.1.2.1.1.1.0
snmpget -v2c -c public 192.0.2.10 1.3.6.1.4.1.4413.1.1.1.1.4.9.0
```

The second value is a string with 5-second and 60-second CPU percentages. The template stores the 60-second figure.

### Link the switch

1. Create a host with an SNMP interface pointed at the switch
2. Set the SNMP version and community (or SNMPv3 credentials) on that interface
3. Link **Datacom DM1200E by SNMP**, **Datacom DM3324F2 by SNMP**, or **Datacom Device by SNMP**, and leave **Generic by SNMP** unlinked. Do not link more than one of these switch templates to the same host
4. Wait for the hourly discovery rules, or run **Network interface discovery**, **Temperature sensor discovery**, **Fan discovery**, **Power supply discovery**, and **PoE port discovery** once from the host

Triggers:

| Condition | Severity |
|---|---|
| Three ICMP checks missed, SNMP collection stopped, PoE controller off or faulty, temperature above 75°C, temperature sensor not normal, fan failed, power supply failed | High |
| ICMP loss above 20%, CPU above 90%, memory above 90%, interface utilization above 90%, interface errors above 2/s, PoE budget above 90%, PoE port fault | Warning |
| Uptime under 10 minutes | Info |

Thresholds are template macros (`{$CPU.UTIL.CRIT}`, `{$MEMORY.UTIL.MAX}`, `{$TEMP.MAX}`, `{$IF.UTIL.MAX}`, `{$IF.ERRORS.WARN}`, `{$POE.UTIL.MAX}`, `{$ICMP.LOSS.WARN}`). Interface discovery keeps ethernetCsmacd ports only (`1/0/1`–`1/0/28` on the DM1200E, `eth-1/1`–`eth-1/28` on the 3324F2). An empty fan or power-supply bay that reports "not present" does not open a problem.

**Interface [{#IFNAME}]: Link down** is disabled on every switch template. Operational status, traffic, utilization, and error rates are still collected. Enable that trigger on the template when link-state problems should open.

On the DM3324F2, run **Network interface discovery**, **Fan discovery**, **Power supply discovery**, and **Alarm discovery**. Temperature is a fixed item. A red fan or power supply is high severity, and an activated alarm input is a warning. There is no PoE discovery on that model.

## Project Layout

```
printers/                           # Zabbix 7.4 YAML exports for printers
├── brother-mfc-l9570cdw.yaml       # Toner, belt, and drum monitoring for the Brother MFC-L9570CDW
├── hp-laserjet-m402n.yaml          # Toner monitoring for the HP LaserJet M402n
└── lexmark-mx421ade.yaml           # Toner and imaging-unit monitoring for the Lexmark MX421ade
switchs/                            # Zabbix 7.4 YAML exports for switches
├── datacom-device.yaml             # Generic ICMP, SNMP, and IF-MIB monitoring for a Datacom device
├── datacom-dm1200e.yaml            # Interface, CPU, memory, environment, and PoE monitoring for the Datacom DM1200E
└── datacom-dm3324f2.yaml     # Interface, CPU, memory, temperature, fans, power, and alarms for the DM3324F2
```

## Development

Copy an existing export into `printers/` when you add a printer, or into `switchs/` when you add a switch. Name the file `<vendor>-<model>.yaml`. After the copy:

1. Set a new template name and visible name
2. Generate a new UUID for every object in the file
3. Replace the template name inside trigger expressions and graph item hosts
4. Adjust the discovery filter so it matches that model's supply descriptions
5. Keep `zabbix_export.version` at `7.4`. Printer templates keep the link to `Generic by SNMP`. Each Datacom template stays standalone because it already collects availability and interfaces
6. Import the file on a test Zabbix server and confirm items, triggers, and the graph

List the templates in the repository:

```bash
ls -1 printers/*.yaml switchs/*.yaml
```

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/description`
3. Commit with Conventional Commits: `git commit -m "feat: add X"`
4. Push and open a pull request

## License

License terms for this repository are unset.
