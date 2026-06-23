# 📊 Painel de Vendas — Iplacas e Shock Visual

> Dashboard de Business Intelligence desenvolvido em **Power BI** para consolidar e analisar as vendas multicanal (Shopee, Mercado Livre e Vendas Internas) das empresas **Iplacas** e **Shock Visual**, com foco em acompanhamento de **orçamento x realizado**, **CAC** e **retrabalhos operacionais**.

![License](https://img.shields.io/badge/License-MIT-blue)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?logo=powerbi&logoColor=black)
![Excel](https://img.shields.io/badge/Excel-217346?logo=microsoftexcel&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-FFCA28)
![Power Query](https://img.shields.io/badge/Power_Query_M-1F6FEB)

🔗 **Projeto entregue via:** [99freelas — Criação de gráfico e relatório via Power BI](https://www.99freelas.com.br/project/criacao-de-grafico-e-relatorio-via-power-bi-759364)

---

## 🖼️ Preview do Painel

<img width="898" height="668" alt="Preview do Painel de Vendas - Iplacas e Shock Visual" src="https://github.com/user-attachments/assets/ff62df68-af99-4a5f-8aff-07a97e4bef52" />

---

## 📌 Sumário

- [🎯 Visão Geral](#-visão-geral)
- [💼 Problema de Negócio](#-problema-de-negócio)
- [✅ Solução Desenvolvida](#-solução-desenvolvida)
- [🛠️ Stack Utilizada](#️-stack-utilizada)
- [🏗️ Arquitetura de Dados](#️-arquitetura-de-dados)
- [📈 KPIs Principais](#-kpis-principais)
- [💡 Insights e Impacto](#-insights-e-impacto)

---

## 🎯 Visão Geral

Este projeto foi desenvolvido como uma solução completa de **Business Intelligence** para apoiar a tomada de decisão comercial das empresas **Iplacas** e **Shock Visual**, que operam em múltiplos canais de venda (marketplaces e vendas internas).

O painel consolida em um único ambiente todas as operações comerciais segmentadas por canal — **Shopee**, **Mercado Livre** e **Vendas Internas** — permitindo ao cliente acompanhar em tempo real o desempenho frente às metas mensais, mensurar o retorno sobre o investimento em mídia paga (ADS) e identificar gargalos operacionais como retrabalhos.

A solução foi desenhada para ser **escalável**, com estrutura preparada para evoluir de uma ingestão manual via Excel para uma integração direta via API com os sistemas do cliente, sem necessidade de reconstrução do modelo de dados.

---

## 💼 Problema de Negócio

Antes da implementação do painel, o cliente enfrentava dificuldades em ter visibilidade unificada da operação comercial:

- **Falta de comparação centralizada** entre orçamento previsto x realizado para acompanhamento do atingimento de meta;
- **Ausência de visibilidade do CAC** (Custo de Aquisição de Cliente) frente aos investimentos em ADS e o retorno real obtido por canal;
- **Comparação manual e ineficiente** entre canais, metas, meses e principalmente do volume de **retrabalhos** operacionais;
- **Bases de vendas fragmentadas** entre Shopee, Mercado Livre e vendas internas, sem unificação para análise integrada e futura extração via API.

> **Pergunta de negócio:** *Como dar visibilidade unificada das vendas multicanal, mensurando atingimento de meta, eficiência do investimento em ADS e custos operacionais ocultos como retrabalho?*

---

## ✅ Solução Desenvolvida

Foi construído um **dashboard executivo em Power BI** que consolida todas as bases de vendas em um modelo único, segmentado por canal, com navegação intuitiva e cards de KPIs estratégicos.

A solução entrega:

- **Consolidação multicanal**: unificação das vendas de Shopee, Mercado Livre e Vendas Internas em uma base única, permitindo análises cruzadas e comparativas;
- **Acompanhamento de meta em tempo real**: cards e gráficos comparando orçamento vs. realizado, com indicadores visuais de progresso e desvio;
- **Análise de CAC e retorno de ADS**: medição direta do custo de aquisição de cliente por canal, cruzando investimento em mídia com receita gerada;
- **Visão de retrabalho operacional**: identificação dos volumes de retrabalho por canal, evidenciando perdas ocultas no processo;
- **Modelagem preparada para escalar**: estrutura de dados pronta para receber integração futura via API, sem reconstrução do modelo.

---

## 🛠️ Stack Utilizada

| Categoria | Ferramentas |
|---|---|
| **Visualização** | Power BI |
| **ETL / Transformação** | Power Query (M) |
| **Bases de Dados** | Excel (Shopee, Mercado Livre, Vendas Internas) |
| **Linguagens** | M, DAX |
| **Modelagem** | Star Schema |

---

## 🏗️ Arquitetura de Dados

```
Extração dos Relatórios (Mercado Livre, Shopee, Vendas Internas)
                          ↓
            Transformação no Power Query (M)
                          ↓
               Modelagem em Star Schema
                          ↓
            Cálculo de Medidas (DAX)
                          ↓
          Visualização (Power BI Dashboard)
                          ↓
                    Usuário Final
```

---

## 📈 KPIs Principais

- **Acumulado de Vendas** por canal e total
- **Orçamento x Realizado** (atingimento de meta %)
- **CAC** — Custo de Aquisição de Cliente por canal
- **Investimento em ADS** vs. retorno gerado
- **Retrabalhos** por canal e período
- **Comparativo mensal** entre canais
- **Ticket Médio** por cliente e por canal

---

## 💡 Insights e Impacto

A entrega do painel proporcionou ao cliente:

- ✅ **Visão única e centralizada** de toda a operação comercial multicanal, eliminando o trabalho manual de consolidação em planilhas;
- ✅ **Tomada de decisão data-driven** sobre alocação de investimento em ADS, identificando os canais mais rentáveis;
- ✅ **Identificação de gargalos operacionais** por meio da visibilidade dos retrabalhos, permitindo planos de ação corretivos;
- ✅ **Base escalável** para integração futura via API, garantindo perenidade da solução sem retrabalho técnico;
- ✅ **Acompanhamento ágil do atingimento de meta**, com a gestão deixando de depender de relatórios manuais mensais.

---

## 📚 Documentação Técnica

Para entender em profundidade as decisões técnicas, modelagem e fórmulas DAX:

- 🗂️ **01-modelagem-de-dados.md** — Star Schema, tabelas, relacionamentos e justificativas
- 📐 **02-medidas-dax.md** — 8 medidas DAX comentadas e organizadas por categoria
  
---
## 📌 Observação

> Os dados originais são confidenciais e pertencem ao cliente. Este repositório contém **dados anonimizados e sintéticos** para fins de portfólio, mantendo a estrutura, a modelagem e a lógica analítica do projeto original.

---

## 👤 Autor

**Kaique Souza** — Analista de Dados e BI
📧 kaique8mateus@gmail.com
🔗 [LinkedIn](https://www.linkedin.com/in/kaique-mateus-de-souza-0baa83238/) • [Portfólio](https://sites.google.com/view/portfolio-kaique-mateus/projetos) • [99Freelas — Link do Projeto](https://www.99freelas.com.br/project/criacao-de-grafico-e-relatorio-via-power-bi-759364)
