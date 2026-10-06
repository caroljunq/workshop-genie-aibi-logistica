# Workshop Databricks — Genie + AI/BI (Logística)

Guia de **1h30** para um time de logística multimodal: criar tabelas no Unity
Catalog, definir KPIs governados em uma **metric view**, montar um dashboard
AI/BI com mapas (choropleth, pontos e rotas) e explorar os dados no
**Genie Agent** e no **Genie One**.

Este roteiro **não** cobre Custom Viz Vega-Lite nem importação de BI.

Todos os nomes e dados são sintéticos. Nenhuma empresa real é mencionada.
A operadora fictícia do caso é a **Atlântica Cargo**, com parceiros
**VerdeMar Shipping**, **RioNorte Express**, **SerraSul Cargas** e
**CostaBrava Log**.

---

## Estrutura

```text
.
├── README.md
├── notebook_guia_workshop.ipynb
├── prompts/
│   ├── 01_metric_view.md
│   ├── 02_aibi_dashboard.md
│   └── 03_genie_agent.md
└── dados/
    ├── hubs_geo.csv
    ├── entregas.csv
    └── rotas_entrega.csv
```

Cópia de trabalho no FEVM:

https://fevm-workspace-carolina-ferreira.cloud.databricks.com/editor/notebooks/1706070174784181?o=7474646657426677

Pasta workspace: `/Users/carolina.ferreira@databricks.com/workshop-october`

O `notebook_guia_workshop.ipynb` contém o passo a passo. A pasta `dados/`
já está no workspace; o Módulo 1 cria as tabelas a partir desses CSVs.
Os arquivos em `prompts/` são os textos para colar no Genie Code / Assistant.

---

## Agenda (1h30)

| Min | Módulo | Resultado |
|---:|---|---|
| 0–10 | Contexto + tabelas SQL | Três tabelas no Unity Catalog |
| 10–30 | Metric view (Genie Code) | Camada semântica de custo, demanda e eficiência |
| 30–65 | AI/BI com Genie Code | Dashboard com KPIs e três mapas |
| 65–80 | Genie Agent | Sala curada, instruções e exemplos |
| 80–90 | Genie One + fechamento | Mesmas perguntas, comparação de experiência |

---

## Módulos

| Módulo | Tema | Resultado |
|---:|---|---|
| 1 | Criar tabelas com SQL | Três tabelas no Unity Catalog |
| 2 | Metric view | `{prefixo}_mv_logistica` com KPIs de custo e serviço |
| 3 | AI/BI com Genie Code | Dashboard com gráficos e mapas, usando a metric view |
| 4 | Genie Agent | Agent pequeno, instruções em português e SQL de exemplo |
| 5 | Genie One | Exploração assistida dos mesmos dados |

O Módulo 2 cria a camada semântica reutilizada pelo dashboard e pelo Genie.
Regras como “ocupação nunca pode ser somada” e “custo por tonelada =
soma do custo / soma do volume” ficam definidas uma única vez.

Versão atual da metric view `{prefixo}_mv_logistica`:

- Dimensões: `periodo`, `ano`, `mes`, `modo_transporte`, `operadora`,
  `status_entrega`, `cliente_marca`, `rota_id`, hubs de origem/destino,
  `modal_alternativo`, `faixa_distancia`, `faixa_ocupacao`.
- Medidas: `entregas`, `entregas_no_prazo`, `otif`, `entregas_atrasadas`,
  `taxa_atraso`, `volume_total`, `demanda_total`, `custo_total`,
  `custo_combustivel`, `participacao_combustivel`, `custo_por_tonelada`,
  `custo_por_km`, `ocupacao_media`, `tempo_medio_transito`,
  `percentual_volume_modal_alternativo`, `mediana_custo_entrega`.

O prompt do Genie Code está em `prompts/01_metric_view.md` (igual à célula
do notebook). O SQL de fallback no notebook replica essa definição.

---

## Modelo de dados

O workshop usa exatamente **três tabelas**. Entregas e rotas se ligam aos
hubs por `hub_origem_id` / `hub_destino_id`.

| CSV | Tabela | Conteúdo |
|---|---|---|
| `hubs_geo.csv` | `{prefixo}_hubs_geo` | CDs, portos e bases, com capacidade e coordenadas |
| `entregas.csv` | `{prefixo}_entregas` | Demanda, custo, modo, prazo e ocupação |
| `rotas_entrega.csv` | `{prefixo}_rotas_entrega` | Pontos ordenados para path map |

```text
hubs_geo (hub_id)
   ├──< entregas (hub_origem_id, hub_destino_id)
   └──< rotas_entrega (hub_origem_id, hub_destino_id)
```

Use um prefixo individual nos widgets (`seu_prefixo`) para não sobrescrever
a tabela de outro participante.

---

## Caso de negócio (falar no opening)

A **Atlântica Cargo** opera caminhão, cabotagem, barco fluvial, balsa e
ferrovia. O maior custo da empresa é logística. O objetivo da sessão é
**atender a demanda no menor custo possível**, sem perder OTIF, usando:

1. uma **metric view** como fonte oficial de KPI;
2. um **dashboard AI/BI** para ver custo, demanda e geografia;
3. **Genie Agent** e **Genie One** para perguntas de trade-off
   (modo × custo × prazo × ocupação).

---

## Como usar

1. Abra `notebook_guia_workshop` no Databricks.
2. Preencha os widgets: catálogo, schema, prefixo e caminho da pasta `dados`.
3. Rode a célula dos widgets e, em seguida, as células SQL do Módulo 1.
4. Se o `read_files` falhar, use o upload manual descrito no mesmo módulo.
5. No Módulo 2, rode a célula do prompt da metric view, cole no Genie Code /
   Assistant e, se preferir, execute o SQL de fallback. Valide com `MEASURE()`.
6. No Módulo 3, rode a célula do prompt do dashboard (mapas + metric view)
   e cole no Genie Code.
7. No Módulo 4, crie o Genie Agent com a metric view + tabelas geo, cole as
   instruções e os exemplos SQL. Faça o laboratório de perguntas.
8. No Módulo 5, repita perguntas no Genie One.

Células Python imprimem textos já preenchidos (prompts, pergunta, SQL).
Rode a célula **antes** de copiar: os `${widgets}` só viram nomes reais na
execução.

---

## Pré-requisitos

- Unity Catalog
- SQL Warehouse Pro ou Serverless
- permissão para criar schema e tabelas
- Genie Agents, Genie One e Genie Code habilitados
- AI/BI Dashboards
- Metric views (Databricks Runtime **17.3+** recomendado)

---

## Referências

- [Genie best practices](https://docs.databricks.com/aws/en/genie/best-practices)
- [Create and manage a Genie Agent](https://docs.databricks.com/aws/en/genie/set-up)
- [Genie Code for dashboards](https://docs.databricks.com/aws/en/dashboards/manage/dashboard-agent)
- [AI/BI visualization types](https://docs.databricks.com/aws/en/dashboards/manage/visualizations/types)
- [Unity Catalog metric views](https://docs.databricks.com/aws/en/uc-semantics/metric-views/)
- [Create a metric view](https://docs.databricks.com/aws/en/uc-semantics/metric-views/create)
