# zbx-tpl

Templates Zabbix que monitoram toner e suprimentos em impressoras HP LaserJet M402n, Lexmark MX421ade e Brother MFC-L9570CDW via SNMP.

## Destaques

- Acompanhar o toner restante em porcentagem na HP LaserJet M402n, na Lexmark MX421ade e na Brother MFC-L9570CDW
- Descobrir suprimentos da Printer-MIB via SNMP e manter entradas de cartucho e toner
- Ler o toner colorido da Brother em brInfoMaintenance quando a Printer-MIB deixa o nível desconhecido
- Acompanhar a unidade de imagem da Lexmark e a belt e o drum da Brother com o mesmo cálculo de porcentagem
- Calcular a vida restante a partir do nível atual do suprimento e da capacidade máxima
- Descartar leituras SNMP negativas e fora da faixa antes de armazená-las
- Abrir um aviso quando um suprimento cai abaixo de 10% e um problema de alta severidade abaixo de 5%
- Gerar gráfico da porcentagem restante em uma escala fixa de 0 a 100
- Vincular cada template ao Generic by SNMP para manter as checagens padrão do dispositivo
- Entregar exports YAML do Zabbix 7.4 prontos para importação

## Visão Geral

Estes templates leem a tabela de suprimentos da Printer-MIB (`1.3.6.1.2.1.43.11`) e transformam contadores SNMP brutos em porcentagem de vida restante. A descoberta roda a cada hora e mantém os suprimentos que cada modelo deve alarmar: toner e cartuchos na HP e na Lexmark, e a unidade de imagem na Lexmark MX421ade.

A Brother MFC-L9570CDW informa o nível de toner como desconhecido na Printer-MIB. Esse template lê as porcentagens de preto, ciano, magenta e amarelo em `brInfoMaintenance` da Brother (`1.3.6.1.4.1.2435.2.3.9.4.2.1.5.5.8.0`) e usa a Printer-MIB para a belt e o drum.

Cada export vincula o template padrão Generic by SNMP. As checagens de interface e disponibilidade ficam no template genérico, e estes arquivos acrescentam a saúde dos suprimentos, os triggers e os gráficos.

## Pré-requisitos

- **Zabbix 7.4+** — destino da importação; os arquivos usam a versão de export `7.4`
- **Generic by SNMP** — template padrão que estes exports vinculam; acompanha o Zabbix
- **Acesso SNMP à impressora** — UDP/161 com community string ou credenciais SNMPv3

## Instalação

Clone o repositório:

```bash
git clone https://github.com/carlosrabelo/zbx-tpl.git
cd zbx-tpl
```

No frontend do Zabbix, abra **Data collection → Templates → Import**, escolha um arquivo YAML e importe:

- `templates/hp-laserjet-m402n.yaml` para a HP LaserJet M402n
- `templates/lexmark-mx421ade.yaml` para a Lexmark MX421ade
- `templates/brother-mfc-l9570cdw.yaml` para a Brother MFC-L9570CDW

Importe um arquivo por vez. O Zabbix cria o template em **Templates/Network devices**.

## Uso

### Conferir a tabela de suprimentos

Confirme que a impressora responde à Printer-MIB antes de vincular um template:

```bash
snmpwalk -v2c -c public 192.0.2.10 1.3.6.1.2.1.43.11.1.1.6
```

Descrições que coincidem com `cartridge` ou `toner` viram itens de toner. Na Lexmark, descrições que coincidem com `imaging` viram itens da unidade de imagem. Na Brother, descrições que coincidem com `belt` ou `drum` viram itens de unidade, e o waste toner fica de fora.

### Vincular o template

1. Crie um host com uma interface SNMP apontando para a impressora
2. Defina a versão SNMP e a community (ou as credenciais SNMPv3) nessa interface
3. Vincule **HP LaserJet M402n** ou **Lexmark MX421ade**
4. Aguarde a regra de descoberta horária, ou execute-a uma vez nas regras de descoberta do host

### Ler os resultados

A descoberta cria estes itens para cada suprimento correspondente:

- Nível de toner em porcentagem, além do nível bruto e da capacidade máxima
- Somente na Lexmark, nível da unidade de imagem em porcentagem, além do nível bruto e da capacidade máxima
- Somente na Brother, nível de belt e drum em porcentagem, além do nível bruto e da capacidade máxima

Triggers:

| Condição | Severidade |
|---|---|
| Nível restante abaixo de 10% | Warning |
| Nível restante abaixo de 5% | High |

O gráfico da porcentagem restante usa um eixo fixo de 0 a 100.

## Estrutura do Projeto

```
templates/                          # Exports YAML do Zabbix 7.4
├── brother-mfc-l9570cdw.yaml       # Monitoramento de toner, belt e drum da Brother MFC-L9570CDW
├── hp-laserjet-m402n.yaml          # Monitoramento de toner da HP LaserJet M402n
└── lexmark-mx421ade.yaml           # Monitoramento de toner e da unidade de imagem da Lexmark MX421ade
```

## Desenvolvimento

Copie um export existente para `templates/` ao adicionar uma impressora. Nomeie o arquivo como `<vendor>-<model>.yaml`. Depois da cópia:

1. Defina um novo nome de template e nome visível
2. Gere um UUID novo para cada objeto do arquivo
3. Substitua o nome do template nas expressões de trigger e nos hosts dos itens de gráfico
4. Ajuste o filtro de descoberta para as descrições de suprimento daquele modelo
5. Mantenha `zabbix_export.version` em `7.4` e o vínculo com `Generic by SNMP`
6. Importe o arquivo em um servidor Zabbix de teste e confira itens, triggers e o gráfico

Liste os templates do repositório:

```bash
ls -1 templates/*.yaml
```

## Contribuição

1. Faça um fork do repositório
2. Crie um branch de funcionalidade: `git checkout -b feat/description`
3. Faça o commit com Conventional Commits: `git commit -m "feat: add X"`
4. Envie o branch e abra um pull request

## Licença

Os termos de licença deste repositório estão indefinidos.
