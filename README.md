# Zabbix Templates for Network Printers

[![Zabbix](https://img.shields.io/badge/Zabbix-7.0%2B-red.svg)](https://www.zabbix.com/)
[![SNMP](https://img.shields.io/badge/Protocol-SNMP-blue.svg)](https://tools.ietf.org/html/rfc3805)
[![Templates](https://img.shields.io/badge/Templates-2-blue.svg)]()

> *Read this in other languages: [Português](README-pt.md)*

## About

This project was born out of necessity at **Campus Cuiabá Bela Vista**, a unit of the **Federal Institute of Mato Grosso (IFMT)**, Brazil. When implementing a comprehensive monitoring solution using **Grafana + Zabbix** for our IT infrastructure, we discovered a significant gap: the official Zabbix repository lacked proper templates for network printer monitoring.

Rather than settling for suboptimal solutions, we decided to create our own high-quality SNMP templates, ensuring efficient monitoring and improved service delivery for our campus community.

## The Challenge

Educational institutions like IFMT manage dozens of network printers across multiple departments. Without proper monitoring:
- Toner levels go unnoticed until printers stop working
- Maintenance schedules become reactive instead of proactive
- IT staff spend unnecessary time on manual checks
- Service interruptions impact academic activities

## Our Solution

We developed RFC 3805-compliant SNMP templates that provide:
- **Automatic discovery** of printer supplies
- **Real-time monitoring** of toner levels
- **Proactive alerting** before supplies run out
- **Standardized metrics** across different printer brands

## Supported Printers

| Manufacturer | Model | Status | Supplies Monitored |
|-------------|-------|--------|-------------------|
| Canon | iR1643i II | ✅ Tested | Imaging Unit, Toner |
| Lexmark | MX421ade | ✅ Tested | Imaging Unit, Black Cartridge |

## Features

- 🔍 **Auto-discovery**: Automatically detects available printer supplies
- 📊 **Percentage calculation**: Shows toner levels as percentages
- ⚠️ **Smart alerting**: Warning at 10%, Critical at 5%
- 🏷️ **Proper tagging**: Organized with component and resource tags
- 🛡️ **Error handling**: Robust preprocessing for edge cases
- 📈 **Zabbix 7.0+ compatible**: Uses latest Zabbix features

## Quick Start

### 1. Import Template
```bash
# Download the template
wget https://raw.githubusercontent.com/your-repo/zbx-tpl/main/lexmark-mx421ade.yaml

# Import via Zabbix web interface:
# Configuration → Templates → Import
```

### 2. Configure Host
```bash
# Add your printer as a host
# Set SNMP interface with community string (usually 'public')
# Assign the appropriate template
```

### 3. Test Discovery
```bash
# Verify SNMP connectivity
snmpwalk -v2c -c public PRINTER_IP 1.3.6.1.2.1.43.11.1.1.6

# Expected output:
# iso.3.6.1.2.1.43.11.1.1.6.1.1 = STRING: "Imaging Unit"
# iso.3.6.1.2.1.43.11.1.1.6.1.2 = STRING: "Black Cartridge"
```

## Technical Details

### SNMP OIDs Used
Based on RFC 3805 (Printer MIB):
- **Supply Description**: `1.3.6.1.2.1.43.11.1.1.6`
- **Current Level**: `1.3.6.1.2.1.43.11.1.1.9`
- **Max Capacity**: `1.3.6.1.2.1.43.11.1.1.8`

### Discovery Rules
- **Key**: `prtMarkerSuppliesLevel`
- **Update interval**: 1 hour
- **Filters**: Excludes problematic supplies (Maintenance Kit)

### Item Prototypes
1. **Toner level [%]**: Calculated percentage
2. **Toner level (raw)**: Raw SNMP value
3. **Toner max capacity**: Maximum supply capacity

### Triggers
- **WARNING**: Toner < 10%
- **HIGH**: Toner < 5%

## Project Structure

```
zbx-tpl/
├── README.md                 # This file (English)
├── README-pt.md             # Portuguese documentation
├── canon-ir1643i-ii.yaml   # Canon template
├── lexmark-mx421ade.yaml   # Lexmark template
└── docs/                    # Future documentation
    ├── troubleshooting.md
    └── snmp-oids.md
```

## Contributing

We welcome contributions! This project serves educational institutions worldwide. If you have templates for other printer models:

1. Fork the repository
2. Create a new template following our naming convention
3. Test with real hardware
4. Submit a pull request

### Template Naming Convention
- Filename: `manufacturer-model.yaml`
- Template name: `Manufacturer Model` (exact device name)
- Unique UUIDs for all components

## Troubleshooting

### Common Issues

**"No Such Instance" errors**
```bash
# Check if printer supports the OID
snmpwalk -v2c -c public PRINTER_IP 1.3.6.1.2.1.43.11.1.1
```

**Discovery not working**
- Verify SNMP community string
- Check network connectivity
- Ensure printer has SNMP enabled

**Incorrect percentages**
- Some printers return special values (e.g., 200002)
- Our templates handle these edge cases automatically

---

**Developed at Instituto Federal de Mato Grosso - Campus Cuiabá Bela Vista**