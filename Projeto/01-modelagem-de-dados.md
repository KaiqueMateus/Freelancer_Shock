# 🗂️ Modelagem de Dados — Painel Iplacas e Shock Visual

## 📐 Visão Geral

O modelo foi construído seguindo o padrão **Star Schema (Esquema Estrela)**, com tabelas fato no topo e dimensões orbitando em torno delas, conectadas por relacionamentos do tipo **N:1** (muitos-para-um).

Essa abordagem foi escolhida por garantir:

- ⚡ **Performance superior** em agregações e cálculos contextuais (DAX);
- 🧩 **Manutenibilidade**, com responsabilidades claras entre fato e dimensão;
- 🔄 **Escalabilidade** para integração futura via API;
- 🎯 **Clareza analítica** ao separar métricas (fatos) de atributos (dimensões).

---

## 🖼️ Diagrama do Modelo

assets/modelo-relacional.png

---

## 📊 Tabelas Fato

O modelo possui **6 tabelas fato**, cada uma representando um processo de negócio distinto:

### `Base Consolidada_Vendas`
Tabela fato principal — registra todas as vendas multicanal (Shopee, Mercado Livre Iplacas, Mercado Livre Shock, Shock e Iplacas).

| Coluna | Descrição |
|---|---|
| Ano | Ano da venda |
| Aprovação | Data de aprovação do pedido |
| Canal | Canal de venda |
| Comprador/Empresa | Identificação do cliente |
| Contato | Contato do cliente |
| Data da Venda | Data da transação |
| Data Financeira | Data de reconhecimento financeiro |
| Entrega | Data de entrega |
| Faturamento | Valor faturado |

### `Orçamentos Feitos`
Registra todos os orçamentos enviados pela empresa, com status atual e indicação de retrabalho.

### `Produtos Mais Vendidos`
Detalha a performance de produtos por canal, modelo e acabamento.

### `Metas`
Tabela de metas mensais por canal.

### `Retrabalhos e Motivos`
Histórico de retrabalhos identificando custo, motivo e OS associada.

### `CAC`
Investimento em mídia paga (ADS) por mês e canal, base para cálculo de Custo de Aquisição de Cliente.

---

## 🧩 Tabelas Dimensão

### `dCalendario` ⭐
Tabela de calendário central do modelo, marcada como **Tabela de Datas**, conectada a todas as fatos pela granularidade diária. Permite uso de funções de **Time Intelligence** (DATESBETWEEN, DATEADD, EOMONTH, etc.).

| Coluna | Descrição |
|---|---|
| Ano | Ano calendário |
| AnoMês | Concatenação Ano + Mês para ordenação |
| AnoMês (numérico) | Versão numérica para classificação |
| Date | Data (chave primária) |
| Mês | Nome ou número do mês |

### `Dim_Canais`
Dimensão única para padronizar canais entre todas as tabelas fato (Vendas, Orçamentos, Produtos, Metas, Retrabalhos, CAC).

| Coluna | Descrição |
|---|---|
| Canal | Nome do canal |
| URL Logo | URL do logo do canal (usado em slicers visuais) |

### `Dim_Empresa`
Dimensão de clientes/compradores, unificando o cadastro entre Vendas e Orçamentos.

### `Dim_OS`
Dimensão de ordens de serviço, usada como ponte entre Vendas e Retrabalhos.

---

## 🔗 Relacionamentos

Todos os relacionamentos seguem o padrão **N:1 (Fato → Dimensão)** com **filtro unidirecional**, da dimensão para a fato.

| Origem (N) | Destino (1) | Tipo | Direção |
|---|---|---|---|
| `Base Consolidada_Vendas[Data da Venda]` | `dCalendario[Date]` | N:1 | Única |
| `Base Consolidada_Vendas[Canal]` | `Dim_Canais[Canal]` | N:1 | Única |
| `Base Consolidada_Vendas[Comprador/Empresa]` | `Dim_Empresa[Comprador/Empresa]` | N:1 | Única |
| `Orçamentos Feitos[Mês]` | `dCalendario[Date]` | N:1 | Única |
| `Orçamentos Feitos[Canal]` | `Dim_Canais[Canal]` | N:1 | Única |
| `Produtos Mais Vendidos[Mês]` | `dCalendario[Date]` | N:1 | Única |
| `Produtos Mais Vendidos[Canal]` | `Dim_Canais[Canal]` | N:1 | Única |
| `Metas[Mês]` | `dCalendario[Date]` | N:1 | Única |
| `Metas[Canal]` | `Dim_Canais[Canal]` | N:1 | Única |
| `Retrabalhos e Motivos[Mês]` | `dCalendario[Date]` | N:1 | Única |
| `Retrabalhos e Motivos[OS]` | `Dim_OS[N.º de venda/OS]` | N:1 | Única |
| `CAC[Mês]` | `dCalendario[Date]` | N:1 | Única |
| `CAC[Mês/Canal]` | `Dim_Canais[Canal]` | N:1 | Única |

---

## 🧠 Decisões de Modelagem

### Por que Star Schema?
Modelo flat (tudo numa tabela só) prejudica performance e dificulta evolução. O Star Schema separa **o que aconteceu (fato)** do **como descrever (dimensão)**, facilitando filtros cruzados.

### Por que `dCalendario` separada e marcada como tabela de datas?
Habilita o uso completo de **Time Intelligence** do DAX (`DATESBETWEEN`, `DATEADD`, `EOMONTH`, `SAMEPERIODLASTYEAR`), essencial para todos os comparativos mensais e cards de variação do dashboard.

### Por que `Dim_Canais` se já tinha o campo Canal em cada fato?
Padronização e consistência: qualquer alteração no nome do canal (ex: renomear "Shopee Iplacas" → "Shopee BR") acontece em **um único lugar**, propagando pra todo o modelo.

### Por que `Dim_Empresa` separada?
Permite enriquecer o cadastro do cliente (segmento, região, tipo) no futuro sem mexer na tabela de vendas. Também viabiliza análises cruzadas entre Vendas e Orçamentos para o mesmo cliente.

### Por que filtros unidirecionais?
Filtros bidirecionais foram evitados para **prevenir relacionamentos ambíguos** e perda de performance. Cada fato é filtrada exclusivamente pelas suas dimensões, sem propagação reversa.

---

## ⚠️ Limitações Conhecidas

- **Granularidade da Meta** é mensal — comparativos diários exigem cálculo via DAX (medida `Meta Dia`);
- **Ingestão atual via Excel** — próximo passo é integração via API com Shopee, Mercado Livre e ERP interno;
- **Tabela `Retrabalhos e Motivos`** não tem relacionamento direto com Vendas — análises cruzadas exigem o uso de `Dim_OS` como ponte.

---

## 🚀 Próximos Passos

- [ ] Migrar ingestão para **dataflows do Power BI Service**;
- [ ] Criar **dimensão de tempo de entrega** (faixas de SLA);
- [ ] Implementar **Row-Level Security (RLS)** para visões por canal;
- [ ] Conectar API direta de Shopee e Mercado Livre para ingestão automática.
