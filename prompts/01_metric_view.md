# Prompt 1 — Metric view (Genie Code / Assistant)

Cole no Genie Code ou no Assistant do SQL Editor / Catalog Explorer.
O notebook imprime este texto com catálogo, schema e prefixo preenchidos
(célula do Módulo 2). Não altere essa célula no notebook: ela é a versão
oficial do prompt.

---

Crie uma metric view no Unity Catalog chamada `{prefixo}_mv_logistica`, no mesmo schema de `@{catalogo}.{schema}.{prefixo}_entregas`, com base nessa tabela.
Faça dois joins many-to-one com `@{catalogo}.{schema}.{prefixo}_hubs_geo`: um como "origem" (hub_origem_id = hub_id) e outro como "destino" (hub_destino_id = hub_id).

Dimensões:
- periodo (mês de referência, a partir de data_entrega)
- ano e mes (derivados do período)
- modo_transporte
- operadora
- status_entrega
- cliente_marca
- rota_id
- hub_origem (nome do hub de origem) e hub_destino (nome do hub de destino)
- regiao_origem, estado_origem e cidade_origem
- estado_destino e cidade_destino
- modal_alternativo (Sim quando o modo for Cabotagem, Barco, Balsa ou Ferrovia; Não quando for Caminhão)
- faixa_distancia (até 500 km, 500 a 1.000, 1.000 a 2.000, acima de 2.000, com base em distancia_km)
- faixa_ocupacao (abaixo de 60%, 60 a 75%, 75 a 90%, acima de 90%, com base em ocupacao_pct)

Métricas:
- entregas: quantidade de entregas
- entregas_no_prazo: quantidade de entregas com entregue_no_prazo verdadeiro (nulo é tratado como falso)
- otif: entregas_no_prazo dividido por entregas, em percentual
- entregas_atrasadas: quantidade de entregas com status Atrasada
- taxa_atraso: entregas_atrasadas dividido por entregas, em percentual
- volume_total: soma de volume_ton
- demanda_total: soma de demanda_unidades
- custo_total: soma de custo_total_brl, tratando nulo como zero
- custo_combustivel: soma de custo_combustivel_brl, tratando nulo como zero
- participacao_combustivel: custo_combustivel dividido por custo_total, em percentual
- custo_por_tonelada: custo_total dividido por volume_total
- custo_por_km: custo_total dividido pela soma de distancia_km
- ocupacao_media: média de ocupacao_pct, desconsiderando entregas com status Cancelada
- tempo_medio_transito: média de tempo_horas
- percentual_volume_modal_alternativo: percentual do volume_total transportado fora do Caminhão
- mediana_custo_entrega: mediana de custo_total_brl

Regras:
- Use NULLIF no denominador de todas as divisões para evitar divisão por zero.
- Razões (custo_por_tonelada, custo_por_km, otif, taxa_atraso) devem ser razão de somas, nunca média de razões por linha.
- ocupacao_media e otif nunca podem ser somados.

Adicione comentários em português em cada dimensão e métrica, explicando a regra de negócio. Inclua display_name e synonyms em português (ex.: modal, frete, R$/t, nível de serviço).

Após criar, valide executando custo_por_tonelada, otif e custo_total por modo_transporte nos últimos 3 meses e apresente o resultado.

---

## Versão atual (fallback SQL no notebook)

O objeto criado no fallback é `{catalogo}.{schema}.{prefixo}_mv_logistica`.

Source: `{prefixo}_entregas`  
Joins: `origem` e `destino` → `{prefixo}_hubs_geo` (many-to-one, rely).

Validação esperada:

```sql
SELECT
  modo_transporte,
  MEASURE(`custo_total`) AS custo_total,
  MEASURE(`custo_por_tonelada`) AS custo_por_tonelada,
  MEASURE(`otif`) AS otif
FROM {catalogo}.{schema}.{prefixo}_mv_logistica
WHERE periodo >= ADD_MONTHS(DATE_TRUNC('MONTH', CURRENT_DATE()), -3)
GROUP BY ALL
ORDER BY custo_total DESC;
```
