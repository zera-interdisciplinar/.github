# Zera

Armazenamento inteligente para rastrear cada item eletrônico dentro de uma empresa, do cadastro ao
descarte. Projeto interdisciplinar do Instituto J&F, desenvolvido inicialmente para o Grupo JBS.

## O projeto

O Zera é um site web e um aplicativo mobile. O funcionário responsável pelo controle dos itens
cadastra as informações de cada equipamento manualmente, uma única vez, e depois só supervisiona o
status dele.

- **Consulta por código de barras** — depois do cadastro, basta escanear o código de barras
  aderido ao eletrônico para ver as informações do item.
- **Prioridade de descarte** — itens que ficam muito tempo parados no estoque, sem uso, são
  marcados como prioritários para descarte.
- **Relatório de descarte** — quando o usuário decide descartar, o aplicativo gera um relatório
  dos itens selecionados, o que facilita a coleta pela empresa terceirizada.
- **Mapa de coleta** — o aplicativo ajuda a encontrar a empresa certa, com um mapa de pontos de
  coleta de resíduos eletrônicos.
- **Zack, o chatbot** — qualquer dúvida pode ser enviada ao Zack, que responde de forma a ajudar o
  usuário a se localizar no aplicativo.

O **gestor** acompanha, aprova e destina os itens (pelo painel web ou pelo aplicativo); o
**operário** faz o cadastro e o dia a dia do inventário (pelo aplicativo).

### Planos

| Plano | O que oferece |
|---|---|
| **Free** | Gratuito, com cadastro de poucos itens |
| **Profissional** | Limite maior de cadastros, alertas de itens que devem ir para o descarte e relatórios para a empresa terceirizada |
| **Empresarial** | Tudo do Profissional, mais suporte dedicado ao cliente com o chatbot |

## Repositórios

### Aplicações

| Repositório | O que é |
|---|---|
| [`zera-android-app`](https://github.com/zera-interdisciplinar/zera-android-app) | Aplicativo Android (Kotlin, Jetpack Compose) — o ponto de entrada operacional |
| [`zera-web-app`](https://github.com/zera-interdisciplinar/zera-web-app) | Painel web do gestor (React, TypeScript) — [gestao.zeratech.online](https://gestao.zeratech.online) |
| [`Projeto-Zera`](https://github.com/zera-interdisciplinar/Projeto-Zera) | Landing page e cadastro por servlets — [www.zeratech.online](https://www.zeratech.online) |

### Serviços

| Repositório | O que é |
|---|---|
| [`ms-administrative-core`](https://github.com/zera-interdisciplinar/ms-administrative-core) | Organizações, unidades, usuários, autenticação e alertas (Java, PostgreSQL) |
| [`ms-inventory`](https://github.com/zera-interdisciplinar/ms-inventory) | Inventário: itens, ciclo de vida, manutenção, descarte, regras e dashboard (Java, Neo4j) |
| [`ms-analytics-bff`](https://github.com/zera-interdisciplinar/ms-analytics-bff) | Indicadores do dashboard, calculados por um pipeline medallion e entregues por SSE (Python) |
| [`ms-artificial-intelligence-core`](https://github.com/zera-interdisciplinar/ms-artificial-intelligence-core) | O assistente Zack e a geração de relatórios (Python, LangGraph) |
| [`sdk-ml-failure-predictor`](https://github.com/zera-interdisciplinar/sdk-ml-failure-predictor) | Modelo que prevê em quantos meses um equipamento quebra (Python, PyTorch) |
| [`scrapy`](https://github.com/zera-interdisciplinar/scrapy) | Configuração ao vivo do aplicativo: envs de boot, feature flags e kill-switches (Go) |

### Infraestrutura e modelos

| Repositório | O que é |
|---|---|
| [`infra-gtw-kong`](https://github.com/zera-interdisciplinar/infra-gtw-kong) | Gateway Kong no GKE: rotas, autenticação e proteção de borda |
| [`ms-template`](https://github.com/zera-interdisciplinar/ms-template) | Ponto de partida para novos microsserviços |

## Arquitetura

```
 App Android          Painel web
      │                    │
      └─────────┬──────────┘
                ▼
        Kong (gateway, GKE)  ── autenticação, rotas, rate limit
                │
   ┌────────────┼──────────────┬──────────────────┬───────────┐
   ▼            ▼              ▼                  ▼           ▼
 admin-core  inventory   analytics-bff     ai-core (Zack)   scrapy
 PostgreSQL    Neo4j      lê PostgreSQL       │               (só o app)
   ▲            │ ▲        e Neo4j            │ MCP
   │ alertas    │ └── previsão de quebra      └──► inventory
   └────────────┘      (failure-predictor)         e failure-predictor
```

O `ms-administrative-core` emite os tokens JWT (RS256) que os demais serviços validam; o
`ms-inventory` envia a ele os alertas das regras de cada unidade e consulta o preditivo para
estimar a data de quebra de cada item.

## Padrões comuns

- Os serviços rodam no GKE. O CI roda em toda PR para `main` ou `qa`; o deploy de QA acontece no
  push na branch `qa`, e o de produção por tag ou release.
- Cada repositório tem um README no mesmo formato — Domínio, Arquitetura, Stack, Executando
  localmente, Configuração, API, Testes e Deploy. Comece por ele.
- Uma branch por card do Jira (`feature/ZERA-XXX-sufixo`), commits no formato
  `tipo(escopo): resumo` e PR no [template da organização](https://github.com/zera-interdisciplinar/.github/blob/main/.github/PULL_REQUEST_TEMPLATE.md).
- Segredos nunca entram no repositório: vêm de variável de ambiente ou de Secret do Kubernetes.
