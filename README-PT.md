# zbx-tpl

Templates Zabbix que monitoram toner e suprimentos nas impressoras HP LaserJet M402n, Lexmark MX421ade e Brother MFC-L9570CDW, e os switches Datacom DM1200E 24GP+4GX e DM3324F2, via SNMP.

## Destaques

- Acompanhar o toner restante em porcentagem na HP LaserJet M402n, na Lexmark MX421ade e na Brother MFC-L9570CDW
- Descobrir suprimentos da Printer-MIB e ler o toner colorido, a belt e o drum da Brother quando a Printer-MIB deixa o nível desconhecido
- Abrir um aviso quando um suprimento cai abaixo de 10% e um problema de alta severidade abaixo de 5%
- Monitorar o Datacom DM1200E 24GP+4GX e o DM3324F2 como o Cisco IOS by SNMP cobre um switch: alcance, CPU, memória, temperatura, ventiladores, energia e interfaces
- Descobrir portas ethernet, portas PoE, sensores de temperatura, ventiladores e fontes a partir das tabelas que este firmware realmente devolve
- Abrir problemas para CPU alta, memória alta, temperatura alta, ventilador ou fonte com falha e falha de PoE. O trigger de link down fica no template e sai desativado
- Gerar gráficos de tráfego de interface, CPU, memória, temperatura e consumo de PoE
- Vincular os templates de impressora ao Generic by SNMP; manter cada template de switch sozinho para essas checagens não serem coletadas duas vezes
- Entregar exports YAML do Zabbix 7.4 prontos para importação

## Visão Geral

Estes templates leem a tabela de suprimentos da Printer-MIB (`1.3.6.1.2.1.43.11`) e transformam contadores SNMP brutos em porcentagem de vida restante. A descoberta roda a cada hora e mantém os suprimentos que cada modelo deve alarmar: toner e cartuchos na HP e na Lexmark, e a unidade de imagem na Lexmark MX421ade.

A Brother MFC-L9570CDW informa o nível de toner como desconhecido na Printer-MIB. Esse template lê as porcentagens de preto, ciano, magenta e amarelo em `brInfoMaintenance` da Brother (`1.3.6.1.4.1.2435.2.3.9.4.2.1.5.5.8.0`) e usa a Printer-MIB para a belt e o drum.

Cada export de impressora vincula o template padrão Generic by SNMP. As checagens de interface e disponibilidade ficam no template genérico, e estes arquivos acrescentam a saúde dos suprimentos, os triggers e os gráficos.

O template do Datacom DM1200E fica sozinho, no mesmo papel do Cisco IOS by SNMP. O switch é um equipamento Broadcom FASTPATH (`sysObjectID` `1.3.6.1.4.1.4413`), então as MIBs da Cisco para CPU, memória e ambiente não se aplicam. Este template lê a IF-MIB nas 28 portas ethernet, a ENTITY-MIB no chassi, a POWER-ETHERNET-MIB no orçamento PoE e nas falhas de porta, e as MIBs de switching e box services da FASTPATH para CPU, memória, temperatura, ventiladores e fontes.

Datacom Device by SNMP é o companheiro genérico, no formato do template Network Generic Device. Ele cobre ICMP, identidade SNMP, tráfego da IF-MIB e duplex da EtherLike. No firmware 4.0.0.73 o DM1200E não implementa a HOST-RESOURCES-MIB nem `dot3StatsDuplexStatus`, então o uptime de hardware fica em zero e a descoberta de duplex fica vazia. Os contadores de interface desse arquivo são de 32 bits. Vincule um template de switch por host: eles compartilham chaves como `icmpping` e `system.name`.

O template do DM3324F2 usa a DMswitch-MIB (`1.3.6.1.4.1.3709`), não a FASTPATH. A CPU vem em centésimos de porcentagem, a memória já é porcentagem e a temperatura é um único sensor do chassi. Ventiladores e fontes alarmam só quando o estado é vermelho. Uma entrada de alarme ativada abre um aviso. Os contadores de interface são de 64 bits para `eth-1/1`–`eth-1/28`. Este modelo não tem PoE.

## Pré-requisitos

- **Zabbix 7.4+** — destino da importação; os arquivos usam a versão de export `7.4`
- **Generic by SNMP** — template padrão que os exports de impressora vinculam; acompanha o Zabbix. Os templates de switch não o vinculam
- **Acesso SNMP ao equipamento** — UDP/161 com community string ou credenciais SNMPv3

## Instalação

Clone o repositório:

```bash
git clone https://github.com/carlosrabelo/zbx-tpl.git
cd zbx-tpl
```

No frontend do Zabbix, abra **Data collection → Templates → Import**, escolha um arquivo YAML e importe:

- `printers/hp-laserjet-m402n.yaml` para a HP LaserJet M402n
- `printers/lexmark-mx421ade.yaml` para a Lexmark MX421ade
- `printers/brother-mfc-l9570cdw.yaml` para a Brother MFC-L9570CDW
- `switchs/datacom-device.yaml` para um dispositivo Datacom genérico
- `switchs/datacom-dm1200e.yaml` para o Datacom DM1200E 24GP+4GX
- `switchs/datacom-dm3324f2.yaml` para o Datacom DM3324F2

Importe um arquivo por vez. O Zabbix cria o template em **Templates/Network devices**.

## Uso

### Conferir a tabela de suprimentos

Confirme que a impressora responde à Printer-MIB antes de vincular um template:

```bash
snmpwalk -v2c -c public 192.0.2.10 1.3.6.1.2.1.43.11.1.1.6
```

Descrições que coincidem com `cartridge` ou `toner` viram itens de toner na HP e na Lexmark. Na Lexmark, descrições que coincidem com `imaging` viram itens da unidade de imagem. Na Brother, preto, ciano, magenta e amarelo são itens fixos decodificados de `brInfoMaintenance`; descrições que coincidem com `belt` ou `drum` viram itens de unidade.

### Vincular o template

1. Crie um host com uma interface SNMP apontando para a impressora
2. Defina a versão SNMP e a community (ou as credenciais SNMPv3) nessa interface
3. Vincule **HP LaserJet M402n**, **Lexmark MX421ade** ou **Brother MFC-L9570CDW**
4. Aguarde a regra de descoberta horária, ou execute-a uma vez nas regras de descoberta do host. Na Brother, execute também **Toner levels** uma vez para as quatro porcentagens de toner aparecerem sem esperar a coleta SNMP horária.

### Ler os resultados

A descoberta cria estes itens para cada suprimento correspondente:

- Nível de toner em porcentagem, além do nível bruto e da capacidade máxima
- Somente na Lexmark, nível da unidade de imagem em porcentagem, além do nível bruto e da capacidade máxima
- Somente na Brother, toner preto, ciano, magenta e amarelo como itens fixos de porcentagem, além do nível de belt e drum em porcentagem, com o nível bruto e a capacidade máxima

Triggers:

| Condição | Severidade |
|---|---|
| Nível restante abaixo de 10% | Warning |
| Nível restante abaixo de 5%, ou a impressora não tem uma leitura utilizável | High |

Um nível da Printer-MIB igual a -1, -2 ou -3 significa cartucho ausente ou quantidade desconhecida. Esses valores eram descartados, então a porcentagem ficava na leitura anterior, muitas vezes 100%. Agora eles são gravados como 0% e o trigger de nível muito baixo abre.

Na Brother, preto, ciano, magenta e amarelo vêm dos campos de vida restante em `brInfoMaintenance` (centésimos de porcentagem). O medidor grosso nesse mesmo objeto pode continuar em 100% com o cartucho vazio ou fora da impressora, então ele só é usado quando o campo de vida restante não existe. Uma cor que não vem na leitura é gravada como 0%.

O gráfico da porcentagem restante usa um eixo fixo de 0 a 100.

### Conferir o switch

Confirme que o DM1200E responde ao SNMP antes de vincular o template. A descrição do sistema no firmware 4.0.0.73 é `DM1200E 24GP+4GX Standalone, 4.0.0.73`.

```bash
snmpget -v2c -c public 192.0.2.10 1.3.6.1.2.1.1.1.0
snmpget -v2c -c public 192.0.2.10 1.3.6.1.4.1.4413.1.1.1.1.4.9.0
```

O segundo valor é uma string com as porcentagens de CPU de 5 segundos e de 60 segundos. O template armazena o valor de 60 segundos.

### Vincular o switch

1. Crie um host com uma interface SNMP apontando para o switch
2. Defina a versão SNMP e a community (ou as credenciais SNMPv3) nessa interface
3. Vincule **Datacom DM1200E by SNMP**, **Datacom DM3324F2 by SNMP** ou **Datacom Device by SNMP**, e deixe **Generic by SNMP** sem vínculo. Não vincule mais de um desses templates de switch no mesmo host
4. Aguarde as regras de descoberta horárias, ou execute uma vez no host **Network interface discovery**, **Temperature sensor discovery**, **Fan discovery**, **Power supply discovery** e **PoE port discovery**

Triggers:

| Condição | Severidade |
|---|---|
| Três checagens ICMP perdidas, coleta SNMP parada, controlador PoE desligado ou com falha, temperatura acima de 75°C, sensor de temperatura fora do normal, ventilador com falha, fonte com falha | High |
| Perda de ICMP acima de 20%, CPU acima de 90%, memória acima de 90%, utilização de interface acima de 90%, erros de interface acima de 2/s, orçamento PoE acima de 90%, falha de porta PoE | Warning |
| Uptime abaixo de 10 minutos | Info |

Os limiares são macros do template (`{$CPU.UTIL.CRIT}`, `{$MEMORY.UTIL.MAX}`, `{$TEMP.MAX}`, `{$IF.UTIL.MAX}`, `{$IF.ERRORS.WARN}`, `{$POE.UTIL.MAX}`, `{$ICMP.LOSS.WARN}`). A descoberta de interfaces mantém só portas ethernetCsmacd (`1/0/1`–`1/0/28` no DM1200E, `eth-1/1`–`eth-1/28` no 3324F2). Um vão de ventilador ou de fonte que informa "not present" não abre problema.

**Interface [{#IFNAME}]: Link down** está desativado em todos os templates de switch. Estado operacional, tráfego, utilização e taxa de erros continuam sendo coletados. Ative esse trigger no template quando os problemas de link up/down forem desejados.

No DM3324F2, execute **Network interface discovery**, **Fan discovery**, **Power supply discovery** e **Alarm discovery**. A temperatura é um item fixo. Ventilador ou fonte em vermelho tem severidade alta, e uma entrada de alarme ativada é um aviso. Não há descoberta de PoE nesse modelo.

## Estrutura do Projeto

```
printers/                           # Exports YAML do Zabbix 7.4 para impressoras
├── brother-mfc-l9570cdw.yaml       # Monitoramento de toner, belt e drum da Brother MFC-L9570CDW
├── hp-laserjet-m402n.yaml          # Monitoramento de toner da HP LaserJet M402n
└── lexmark-mx421ade.yaml           # Monitoramento de toner e da unidade de imagem da Lexmark MX421ade
switchs/                            # Exports YAML do Zabbix 7.4 para switches
├── datacom-device.yaml             # Monitoramento genérico de ICMP, SNMP e IF-MIB de um dispositivo Datacom
├── datacom-dm1200e.yaml            # Monitoramento de interface, CPU, memória, ambiente e PoE do Datacom DM1200E
└── datacom-dm3324f2.yaml     # Monitoramento de interface, CPU, memória, temperatura, ventiladores, energia e alarmes do DM3324F2
```

## Desenvolvimento

Copie um export existente para `printers/` ao adicionar uma impressora, ou para `switchs/` ao adicionar um switch. Nomeie o arquivo como `<vendor>-<model>.yaml`. Depois da cópia:

1. Defina um novo nome de template e nome visível
2. Gere um UUID novo para cada objeto do arquivo
3. Substitua o nome do template nas expressões de trigger e nos hosts dos itens de gráfico
4. Ajuste o filtro de descoberta para as descrições de suprimento daquele modelo
5. Mantenha `zabbix_export.version` em `7.4`. Os templates de impressora mantêm o vínculo com `Generic by SNMP`. Cada template Datacom fica sozinho porque já coleta disponibilidade e interfaces
6. Importe o arquivo em um servidor Zabbix de teste e confira itens, triggers e o gráfico

Liste os templates do repositório:

```bash
ls -1 printers/*.yaml switchs/*.yaml
```

## Contribuição

1. Faça um fork do repositório
2. Crie um branch de funcionalidade: `git checkout -b feat/description`
3. Faça o commit com Conventional Commits: `git commit -m "feat: add X"`
4. Envie o branch e abra um pull request

## Licença

Os termos de licença deste repositório estão indefinidos.
