# 🫀 WELLI — Automação Inteligente de Atendimento Hospitalar

> Bot conversacional via WhatsApp para agendamento automatizado de consultas e exames na Rede MaterDei, com IA generativa, memória de contexto e pipeline analítico.

---

## 📋 Sobre o Projeto

O **WELLI** é um sistema de automação de atendimento hospitalar desenvolvido como projeto acadêmico na **FIAP — 2TSCOR (2025)**. O bot atua diretamente no WhatsApp do paciente, processando mensagens em linguagem natural, realizando agendamentos e integrando-se ao CRM hospitalar — 24 horas por dia, sem necessidade de atendentes humanos para triagem inicial.

**Problema endereçado:** A Rede MaterDei registrava **70% de abandono** antes da conclusão do agendamento via WhatsApp, devido ao processo manual e lento.

---

## 🏗️ Arquitetura da Solução

```
WhatsApp (Paciente)
       │
       ▼
   WAHA API  ──── Webhook ────▶  N8N (Orquestrador)
                                      │
                          ┌───────────┼───────────┐
                          ▼           ▼            ▼
                     AI Agent    Redis Memory   Switch Rules
                   (Gemini LLM)  (Contexto)    (Filtros)
                          │
                          ▼
                     API Gateway
                    /           \
               Bitrix24      PostgreSQL
              (CRM/Agenda)   (Histórico/Logs)
                                   │
                              Pipeline ETL
                            (Python/Pandas)
                                   │
                            Data Warehouse
                          (Modelagem Star Schema)
                                   │
                             Power BI Dashboard
                               (KPIs em tempo real)
```

---

## 🚀 Stack Tecnológico

| Camada | Tecnologia | Função |
|--------|-----------|--------|
| **Entrada** | WhatsApp Business API + WAHA | Canal de comunicação com o paciente |
| **Orquestração** | N8N | Automação do fluxo de atendimento |
| **IA / NLP** | Google Gemini (via N8N) | Interpretação de linguagem natural |
| **Memória** | Redis | Contexto da conversa entre mensagens |
| **CRM** | Bitrix24 | Agenda médica e gestão de atendimentos |
| **Banco de Dados** | PostgreSQL | Histórico de logs e agendamentos |
| **ETL** | Python + Pandas | Extração, limpeza e carga dos dados |
| **Data Warehouse** | Star Schema (DW) | Modelagem analítica |
| **BI** | Microsoft Power BI | Dashboards e KPIs executivos |
| **Infraestrutura** | Docker Desktop | Orquestração de containers local |

---

## 📁 Estrutura do Repositório

```
welli/
├── n8n/
│   └── Automacao_N8N_Whatsapp_WELLI.json   # Workflow completo exportado do N8N
├── docker/
│   └── docker-compose.yml                   # WAHA + PostgreSQL + Redis
├── etl/
│   ├── eda.ipynb                            # Análise Exploratória de Dados (298.511 registros)
│   └── pipeline.py                         # ETL: extração, limpeza e carga no DW
├── sql/
│   └── schema.sql                          # Modelagem Star Schema do Data Warehouse
├── docs/
│   └── arquitetura.png                     # Diagrama da arquitetura de solução
└── README.md
```

---

## ⚙️ Como Rodar Localmente

### Pré-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [N8N](https://n8n.io/) (self-hosted ou cloud)
- Conta Google com acesso à API do Gemini
- Conta WAHA (WhatsApp HTTP API)

### 1. Subir a infraestrutura

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/welli.git
cd welli

# Suba os containers
docker-compose up -d
```

Isso irá inicializar:
- `waha-1` — WhatsApp HTTP API na porta `3000`
- `postgres-1` — Banco de dados na porta `5432`
- `redis-1` — Memória de sessão na porta `6379`

### 2. Importar o Workflow no N8N

1. Acesse seu N8N
2. Vá em **Workflows → Import from file**
3. Selecione `n8n/Automacao_N8N_Whatsapp_WELLI.json`
4. Configure as credenciais:
   - **Google Gemini API Key**
   - **WAHA API** apontando para `http://localhost:3000`
   - **Redis** apontando para `localhost:6379`

### 3. Configurar o WAHA

1. Acesse `http://localhost:3000`
2. Crie uma nova sessão e escaneie o QR Code com o WhatsApp
3. Configure o webhook apontando para o endpoint do N8N

### 4. Rodar o ETL (opcional)

```bash
cd etl
pip install -r requirements.txt
python pipeline.py
```

---

## 🔄 Fluxo de Atendimento

```
Paciente envia mensagem
        │
        ▼
WAHA recebe via webhook → N8N processa
        │
   Switch (tipo de mensagem)
        │
        ├── Mensagem válida → AI Agent (Gemini) interpreta intenção
        │         │
        │    Redis mantém contexto da conversa
        │         │
        │    N8N aciona Bitrix24 (API REST) → Agenda consulta
        │         │
        │    Resposta enviada via WAHA → Paciente
        │         │
        │    Log gravado no PostgreSQL
        │
        └── Mensagem inválida / baixa confiança
                  │
             Escala para atendimento humano (Bitrix24)
```

### Tratamento de Exceções

| Cenário | Comportamento |
|---------|--------------|
| NLP com baixa confiança | Escala automática para atendente humano |
| WAHA indisponível | Retry automático (3x) + alerta no N8N |
| Bitrix24 fora do ar | Fila no Redis + reprocessamento automático |
| Timeout de resposta (>10s) | Mensagem de espera enviada ao paciente |

---

## 📊 KPIs Monitorados

| Métrica | Baseline Atual | Meta com WELLI |
|---------|---------------|----------------|
| Taxa de Conversão | 30% | >70% |
| Taxa de Abandono | 70% | <25% |
| Tempo Médio de Resposta | >5 min | <3 segundos |
| Taxa de No-Show | ~50% | Redução via lembretes automáticos |
| Satisfação do Paciente | — | >4,5 / 5,0 |

---

## 👥 Equipe

| Nome | RM |
|------|----|
| André da Silva Santos | RM559527 |
| João Vitor Souza Gonçalves de Oliveira | RM559692 |
| João Vitor Vieira Marcarini | RM560819 |
| Laura Aparecida Mariano Da Silva | RM560004 |

**Instituição:** FIAP — São Paulo  
**Turma:** 2TSCOR | 2025  
**Parceiro:** Rede Hospitalar MaterDei  

---

## 📄 Licença

Este projeto foi desenvolvido para fins acadêmicos no contexto do Challenge FIAP 2025/2026.
