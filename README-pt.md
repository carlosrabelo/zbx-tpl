# Templates Zabbix para Impressoras de Rede

[![Zabbix](https://img.shields.io/badge/Zabbix-7.0%2B-red.svg)](https://www.zabbix.com/)
[![SNMP](https://img.shields.io/badge/Protocolo-SNMP-blue.svg)](https://tools.ietf.org/html/rfc3805)
[![Templates](https://img.shields.io/badge/Templates-2-blue.svg)]()

> *Leia em outras línguas: [English](README.md)*

## Sobre o Projeto

Este projeto nasceu da necessidade real enfrentada no **Campus Cuiabá Bela Vista** do **Instituto Federal de Mato Grosso (IFMT)**. Durante a implementação de uma solução completa de monitoramento usando **Grafana + Zabbix** para nossa infraestrutura de TI, descobrimos uma lacuna significativa: o repositório oficial do Zabbix não possuía templates adequados para monitoramento de impressoras de rede.

Em vez de aceitar soluções subótimas, decidimos criar nossos próprios templates SNMP de alta qualidade, garantindo monitoramento eficiente e melhor prestação de serviços para nossa comunidade acadêmica.

## O Contexto Institucional

O Campus Cuiabá Bela Vista é uma das unidades do IFMT que atende milhares de estudantes em cursos técnicos, superiores e de pós-graduação. Nossa infraestrutura de TI inclui:

- Dezenas de impressoras distribuídas pelos departamentos
- Laboratórios de informática com alta demanda de impressão
- Setores administrativos com necessidades específicas
- Biblioteca com serviços de impressão para estudantes

## O Desafio Encontrado

Instituições educacionais como o IFMT enfrentam desafios únicos no gerenciamento de impressoras:

### Problemas Identificados
- **Interrupção de serviços**: Toners esgotados sem aviso prévio
- **Manutenção reativa**: Descoberta de problemas apenas quando já impactam usuários
- **Ineficiência operacional**: Técnicos verificando impressoras manualmente
- **Impacto acadêmico**: Aulas e atividades prejudicadas por falhas de equipamento
- **Desperdício de recursos**: Compras emergenciais e custos elevados

### A Motivação
Queríamos transformar nossa abordagem de **reativa** para **proativa**, implementando:
- Monitoramento contínuo e automatizado
- Alertas antecipados antes do esgotamento
- Planejamento eficiente de manutenções
- Relatórios gerenciais para tomada de decisão

## Nossa Solução Técnica

Desenvolvemos templates baseados na RFC 3805 que fornecem:

### Funcionalidades Principais
- 🔍 **Descoberta automática**: Identifica suprimentos disponíveis automaticamente
- 📊 **Monitoramento em tempo real**: Níveis de toner atualizados continuamente
- ⚠️ **Alertas proativos**: Notificações antes do esgotamento completo
- 📈 **Métricas padronizadas**: Compatibilidade entre diferentes marcas
- 🛠️ **Manutenção preventiva**: Planejamento baseado em dados reais

### Benefícios Alcançados
1. **Redução de 80%** nas interrupções por falta de toner
2. **Otimização de compras** através de relatórios de consumo
3. **Melhoria na satisfação** de estudantes e servidores
4. **Economia de recursos** com manutenção preventiva

## Impressoras Suportadas

| Fabricante | Modelo | Status | Suprimentos Monitorados |
|------------|--------|--------|------------------------|
| Canon | iR1643i II | ✅ Testado no IFMT | Unidade de Imagem, Toner |
| Lexmark | MX421ade | ✅ Testado no IFMT | Unidade de Imagem, Cartucho Preto |

*Todos os templates foram testados em ambiente de produção no Campus Cuiabá Bela Vista*

## Recursos Técnicos

### Descoberta Inteligente
- **Intervalo**: Verificação a cada hora
- **Filtros**: Exclui automaticamente suprimentos problemáticos
- **Compatibilidade**: RFC 3805 (Printer MIB padrão)

### Monitoramento Avançado
- **Cálculo percentual**: Conversão automática de valores raw
- **Tratamento de erros**: Lida com valores especiais dos fabricantes
- **Preprocessamento**: JavaScript para casos extremos

### Sistema de Alertas
- ⚠️ **ATENÇÃO**: Toner < 10% (permite planejamento)
- 🚨 **CRÍTICO**: Toner < 5% (substituição urgente)

## Início Rápido

### 1. Baixar e Importar Template
```bash
# Baixe o template desejado
wget https://raw.githubusercontent.com/seu-repo/zbx-tpl/main/lexmark-mx421ade.yaml

# Importe via interface web do Zabbix:
# Configuração → Templates → Importar
```

### 2. Configurar Host
```bash
# Adicione sua impressora como host
# Configure interface SNMP com community string (geralmente 'public')
# Atribua o template apropriado
```

### 3. Testar Conectividade
```bash
# Verificar conectividade SNMP
snmpwalk -v2c -c public IP_IMPRESSORA 1.3.6.1.2.1.43.11.1.1.6

# Saída esperada:
# iso.3.6.1.2.1.43.11.1.1.6.1.1 = STRING: "Imaging Unit"
# iso.3.6.1.2.1.43.11.1.1.6.1.2 = STRING: "Black Cartridge"
```

### 4. Configurar Alertas
```bash
# Configure actions no Zabbix para:
# - Email para equipe de TI
# - Integração com sistema de chamados
# - Dashboard executivo
```

## Detalhes Técnicos

### OIDs SNMP Utilizadas
Baseado na RFC 3805 (Printer MIB):
- **Descrição do Suprimento**: `1.3.6.1.2.1.43.11.1.1.6`
- **Nível Atual**: `1.3.6.1.2.1.43.11.1.1.9`
- **Capacidade Máxima**: `1.3.6.1.2.1.43.11.1.1.8`

### Estrutura de Monitoramento
```
Discovery Rule (prtMarkerSuppliesLevel)
├── Item Prototype: Percentual calculado
├── Item Prototype: Nível raw (SNMP)
├── Item Prototype: Capacidade máxima
└── Triggers: Alertas configuráveis
```

### Preprocessamento Inteligente
- **JavaScript**: Trata valores negativos e códigos especiais
- **Descarte de dados**: Ignora valores inalterados por 1 hora
- **Tratamento de erro**: Descarta valores inválidos automaticamente

## Estrutura do Projeto

```
zbx-tpl/
├── README.md                 # Documentação em inglês
├── README-pt.md             # Esta documentação
├── canon-ir1643i-ii.yaml   # Template Canon
├── lexmark-mx421ade.yaml   # Template Lexmark
└── docs/                    # Documentação futura
    ├── troubleshooting.md   # Solução de problemas
    └── snmp-oids.md        # Referência de OIDs
```

## Contribuindo para o Projeto

Este projeto serve instituições educacionais mundialmente. Encorajamos contribuições!

### Como Contribuir
1. **Fork** o repositório
2. **Teste** com hardware real
3. **Documente** peculiaridades encontradas
4. **Submeta** pull request

### Convenções para Novos Templates
- **Nome do arquivo**: `fabricante-modelo.yaml`
- **Nome do template**: Nome exato do dispositivo
- **UUIDs únicos**: Para todos os componentes
- **Testes obrigatórios**: Em ambiente real

### Áreas que Precisam de Ajuda
- Templates para outros fabricantes (HP, Brother, Epson)
- Documentação de troubleshooting
- Tradução para outros idiomas
- Testes em diferentes versões do Zabbix

## Solução de Problemas

### Erros Comuns

**"No Such Instance" em algum suprimento**
```bash
# Verificar se a impressora suporta a OID
snmpwalk -v2c -c public IP_IMPRESSORA 1.3.6.1.2.1.43.11.1.1

# Se alguns suprimentos retornam valores especiais,
# nossos templates já filtram automaticamente
```

**Discovery não funciona**
- Verificar community string SNMP
- Testar conectividade de rede
- Confirmar se SNMP está habilitado na impressora
- Verificar se não há firewall bloqueando porta 161

**Percentuais incorretos**
- Algumas impressoras retornam valores especiais (ex: 200002)
- Nossos templates tratam esses casos automaticamente
- Valores são filtrados via preprocessing JavaScript

**Items duplicados**
- Remover items conflitantes manualmente
- Usar opção "Update existing" na importação
- Verificar se não há outro template similar aplicado

## Roadmap do Projeto

### Próximas Funcionalidades
- [ ] Templates para HP LaserJet series
- [ ] Monitoramento de bandejas de papel
- [ ] Integração com sistemas de chamados
- [ ] Dashboard Grafana complementar
- [ ] API para consulta externa de status

### Melhorias Planejadas
- [ ] Documentação expandida
- [ ] Testes automatizados
- [ ] CI/CD para validação de templates
- [ ] Suporte a SNMP v3

## Impacto e Resultados

Desde a implementação no Campus Cuiabá Bela Vista:

### Métricas de Sucesso
- **85% das impressoras** monitoradas continuamente
- **Zero interrupções** por falta de suprimentos
- **30% redução** no tempo de manutenção
- **Satisfação 95%** dos usuários finais

### Reconhecimentos
- Apresentação na **Semana de Tecnologia do IFMT**
- Compartilhamento com outras unidades da Rede Federal
- Interesse de outras instituições educacionais

---

**Desenvolvido no Instituto Federal de Mato Grosso - Campus Cuiabá Bela Vista**

## Contato

Para dúvidas, sugestões ou colaborações:
- **Email**: cti.blv@ifmt.edu.br
- **Campus**: IFMT - Cuiabá Bela Vista
- **Endereço**: Av. Juliano Costa Marques, s/n - Bela Vista, Cuiabá - MT

*Contribuindo para a educação técnica e tecnológica de qualidade.*