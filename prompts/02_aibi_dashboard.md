# Prompt 2 — Dashboard AI/BI com mapas (Genie Code)

Cole no Genie Code de um dashboard novo (Dashboards > Create dashboard).
O notebook imprime o texto com catálogo, schema e prefixo preenchidos.

Use `@` para referenciar a metric view e as tabelas geo antes de enviar.

A metric view oficial é `{prefixo}_mv_logistica`. Use os nomes das **medidas
atuais** (`custo_total`, `otif`, `ocupacao_media`, `custo_por_tonelada`, …).

Este prompt já inclui o choropleth com ISO `BRA`, o point map e o path map
com `ST_MAKELINE` + `ST_ASGEOJSON` no widget (não no dataset).

---

Crie um dashboard chamado `{prefixo} - Atlântica Cargo — Custo e Rota` usando a metric view de KPIs e as tabelas geográficas.

Fontes (obrigatório):
- Metric view `{catalogo}.{schema}.{prefixo}_mv_logistica` para TODOS os KPIs, barras, linhas, heatmap e tabelas de custo/demanda/OTIF/ocupação. No AI/BI, MEASURE() é aplicado automaticamente. Use os nomes das medidas da metric view: custo_total, volume_total, demanda_total, custo_por_tonelada, otif, ocupacao_media, taxa_atraso, entregas, entregas_atrasadas, participacao_combustivel, mediana_custo_entrega.
- Dimensões úteis da metric view: periodo, modo_transporte, operadora, status_entrega, modal_alternativo, faixa_distancia, faixa_ocupacao, estado_origem, hub_origem, hub_destino, regiao_origem.
- Tabela `{catalogo}.{schema}.{prefixo}_hubs_geo` apenas para o point map (latitude, longitude, capacidade, tipo de hub).
- Tabela `{catalogo}.{schema}.{prefixo}_rotas_entrega` apenas para o path map e para o gráfico de linha de distância acumulada (rota_id, ordem_ponto, latitude, longitude, modo_transporte, operadora, distancia_acumulada_km).

Não calcule custo, OTIF, ocupação ou custo por tonelada direto da tabela entregas. Se o gráfico for de KPI, a origem é a metric view.

Crie duas páginas. Evite gráfico de pizza. Títulos em português, unidades nos rótulos, moeda em R$ com 2 casas, percentuais com 2 casas.
Nos gráficos de ocupacao_media e de otif, comece o eixo Y em 50 (não em 0).

Página 1 — Visão executiva de custo e demanda
Faixa de KPIs no topo, depois uma grade:
- KPI: custo_total (R$)
- KPI: volume_total (t)
- KPI: custo_por_tonelada (R$/t)
- KPI: otif (%)
- KPI: ocupacao_media (%)
- Gráfico de linha: custo_total por periodo
- Gráfico de área: volume_total e demanda_total por periodo
- Gráfico de barras verticais: custo_total por modo_transporte
- Gráfico de barras horizontais: custo_por_tonelada por modo_transporte
- Gráfico combinado: barras de volume_total e linha de ocupacao_media por modo_transporte
- Gráfico de barras agrupadas: entregas vs entregas_atrasadas por operadora
- Tabela: ranking de corredores (hub_origem → hub_destino) com volume_total, custo_total, custo_por_tonelada, otif e ocupacao_media

Página 2 — Geografia da malha (mapas em destaque)
Três mapas, nesta ordem:

1. Choropleth por estado de ORIGEM: use regionType mapbox-v4-admin, admin0 com geographicRole admin0-iso-3166-1-alpha-3 e valor fixo "BRA" (não use admin0-name nem o valor "Brazil"), admin1 com geographicRole admin1-name e campo estado_origem. Cor = custo_total em R$, tooltip com volume_total (t), custo_por_tonelada e otif. Escala sequencial laranja/vermelho. Título: Custo logístico por estado de origem (R$).

2. Point map dos hubs: latitude e longitude de hubs_geo, tamanho por capacidade_ton_dia, cor por tipo_hub, tooltip com hub_nome, cidade, operadora e capacidade_ton_dia.

3. Path map chamado Rotas multimodais: para o dataset de geometria, use SQL que agrupa por rota_id e constrói a linha com ST_MAKELINE. Como ARRAY_AGG com ORDER BY não é suportado neste warehouse, use a seguinte lógica de ordenação: COLLECT_LIST de NAMED_STRUCT com campos ('seq', ordem_ponto, 'lng', longitude, 'lat', latitude), ordene com ARRAY_SORT passando um comparador lambda (l, r) -> CASE WHEN l.seq < r.seq THEN -1 WHEN l.seq > r.seq THEN 1 ELSE 0 END, depois aplique TRANSFORM para extrair ST_POINT(s.lng, s.lat), e envolva tudo em ST_MAKELINE. Armazene o resultado como coluna GEOMETRY (não aplique ST_ASGEOJSON no dataset). No widget, o campo da query deve usar a expressão ST_ASGEOJSON(`route_geom`) — o path-map exige que a expressão comece com ST_ASGEOJSON sobre uma coluna GEOMETRY; uma coluna STRING pré-computada é silenciosamente descartada. Cor por modo_transporte. Tooltip com rota_id, hub_origem_id, hub_destino_id, distancia_total_km.

Em seguida:
- Barras: custo_total por estado_origem
- Barras empilhadas (não 100%): volume_total por modo_transporte em cada regiao_origem
- Dispersão: custo_por_tonelada por faixa_distancia, cor por modo_transporte
- Heatmap: otif por modo_transporte e estado_origem
- Heatmap: custo_por_tonelada por faixa_distancia e faixa_ocupacao
- Linha: AVG(distancia_acumulada_km) ao longo de ordem_ponto, uma série por modo_transporte (dataset da tabela `{prefixo}_rotas_entrega`, agregado)
- Tabela: hubs com hub_nome, cidade, tipo_hub, capacidade_ton_dia e custo_fixo_diario_brl

Regras dos mapas:
- rota_id separa os trajetos; nunca misture pontos de rotas diferentes
- ordem_ponto define a sequência do path
- não inverter latitude e longitude (longitude primeiro em ST_POINT)
- filtros globais não devem ser aplicados ao dataset do path map (usar dataset separado para rotas)
- choropleth usa nome completo do estado brasileiro (São Paulo, Amazonas), não a sigla

Filtros globais: modo_transporte, operadora, estado_origem, status_entrega, modal_alternativo.

Tema: fonte Poppins, cores consistentes por modo_transporte em todos os gráficos (Caminhão, Cabotagem, Barco, Balsa, Ferrovia).

Narrativa: atender a demanda no menor custo, mostrando onde o caminhão é caro, onde modal_alternativo = Sim dilui R$/t, e quais estados concentram custo e atraso.
