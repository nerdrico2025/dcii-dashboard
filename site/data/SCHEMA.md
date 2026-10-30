# Contrato dos dados da página (schema_version 1.0)

A página `site/index.html` lê só arquivos desta pasta:

| Arquivo | O que é |
|---|---|
| `index.json` | Lista das semanas disponíveis, **mais recente primeiro**. |
| `AAAA-MM-DD.json` | Uma semana. O nome é o `week_start` (segunda-feira). |
| `latest.json` | Cópia idêntica do arquivo da semana mais recente. A página só usa se `index.json` falhar. |

O **workflow C** (`n8n/workflow_C_publish_site.json`) monta e publica cada arquivo de semana assim:
- **1 linha** de `weekly_snapshots`. Se houver mais de uma linha para a mesma semana, use a de `snapshot_date` mais recente.
- A aba `ui_labels` inteira.
- A versão mais recente de cada tópico de `strategy`.
- `config.default_language`.

A página **não calcula status nem reescreve texto**. Ela só exibe, formata números e datas (Intl) e desenha as setas de variação.

## 1. `index.json`

```json
{
  "schema_version": "1.0",
  "weeks": [
    { "week_start": "2026-10-26", "snapshot_date": "2026-11-02", "file": "2026-10-26.json",
      "how_its_going_status": "Yellow", "sample": true }
  ]
}
```

| Campo | Tipo | Origem | Obrigatório |
|---|---|---|---|
| `weeks[]` | array | uma entrada por semana publicada, mais recente primeiro | sim |
| `weeks[].week_start` | string `AAAA-MM-DD` | `weekly_snapshots.week_start` | sim |
| `weeks[].file` | string, só `[A-Za-z0-9_.-]` + `.json` | `<week_start>.json` | sim |
| `weeks[].snapshot_date` | string `AAAA-MM-DD` | `weekly_snapshots.snapshot_date` | não |
| `weeks[].how_its_going_status` | `Green` \| `Yellow` \| `Red` \| `Gray` | `weekly_snapshots.how_its_going_status` | não |
| `weeks[].sample` | boolean | `true` só nos dados de exemplo | não |

## 2. Arquivo da semana (`AAAA-MM-DD.json` e `latest.json`)

```json
{
  "schema_version": "1.0",
  "sample": false,
  "default_language": "en-CA",
  "labels":   { "<key>": { "en_CA": "…", "pt_BR": "…" } },
  "strategy": [ { "topic": "Positioning", "text_en_CA": "…", "text_pt_BR": "…", "valid_from": "2026-10-26", "version": "v1" } ],
  "snapshot": { "snapshot_date": "…", "week_start": "…", "executing_text_en_CA": "…", "…": "…" },
  "kpi":      { "week_start": "…", "funnel": [ … ], "…": "…" }
}
```

| Campo | Tipo | Origem |
|---|---|---|
| `schema_version` | string | fixo `"1.0"` |
| `sample` | boolean | `false` nos dados reais. Com `true`, a página mostra o aviso "Sample data / Dados de exemplo". |
| `default_language` | `"en-CA"` \| `"pt-BR"` | `config.default_language` |
| `labels` | objeto `key → { en_CA, pt_BR }` | aba **ui_labels**, uma entrada por linha (`key`, `en_CA`, `pt_BR`) |
| `strategy` | array, na ordem Positioning, Target, Engine | aba **strategy**: a linha de maior `valid_from` de cada `topic` (`valid_from` como `AAAA-MM-DD`) |
| `snapshot` | objeto | **uma linha de weekly_snapshots**, com os nomes de coluna exatos, **sem** `kpi_json` |
| `kpi` | objeto | `JSON.parse(weekly_snapshots.kpi_json)`, exatamente como o workflow B grava |

### 2.1 `snapshot` (colunas de weekly_snapshots)

| Campo | Tipo | Uso na página |
|---|---|---|
| `snapshot_date` | `AAAA-MM-DD` | cabeçalho ("Updated / Atualizado em") |
| `week_start` | `AAAA-MM-DD` | cabeçalho (semana) |
| `executing_text_en_CA` / `_pt_BR` | string, 1 item por linha (`\n`) | caixa Executing |
| `how_its_going_status` | `Green`\|`Yellow`\|`Red`\|`Gray` | selo da caixa How it's going (ícone + texto + cor) |
| `how_its_going_text_en_CA` / `_pt_BR` | string, 1 item por linha | caixa How it's going (a 1ª linha é o motivo) |
| `what_to_change_text_en_CA` / `_pt_BR` | string, 1 item por linha | caixa What to change |
| `needs_from_team_en_CA` / `_pt_BR` | string (pode ser vazia) | faixa "What we need from the team", que só aparece se houver texto |
| `report_url` | string (pode ser vazia) | link da semana: `SITE_BASE_URL?week=<week_start>`, preenchido pelo workflow B. A página não exibe; o link vai no e-mail do Scott. |

### 2.2 `kpi` (formato atual do workflow B, nó *Compute snapshot data*)

| Campo | Tipo | Uso na página |
|---|---|---|
| `week_start`, `week_end` | `AAAA-MM-DD` | intervalo da semana no cabeçalho |
| `currency` | string (ISO 4217, ex. `EUR`) | formatação de dinheiro |
| `how_its_going_status` | status | reserva, se `snapshot.how_its_going_status` faltar |
| `funnel[]` | array, 1 item por produto × etapa | tabela do funil (a ordem dos produtos e etapas segue a do array) |
| `funnel[].product` / `.stage` | chaves técnicas (`ISRA`, `Demo/Trial request`…) | rótulo via `product.<slug>` / `stage.<slug>` |
| `funnel[].unit` | `count` ou código de moeda | número ou moeda |
| `funnel[].provisional` | boolean | `true` mostra o selo "Provisional / Provisório". A regra fica em `PROVISIONAL_RULE`, nas SETTINGS do workflow B; hoje vale só para AI Transformation × Demo/Trial request. |
| `funnel[].value` | number \| null | valor grande. `null` = "No data yet", nunca 0. |
| `funnel[].change_abs` | number \| null | seta (▲ ▼ ▶) + número com sinal |
| `funnel[].change_pct` | number \| null (em %, ex. `15.7`) | entre parênteses |
| `funnel[].status` | status | selo (ícone + texto) |
| `funnel[].target_per_week`, `.pct_of_plan`, `.previous_week_value`, `.history[]` | number \| null | **não exibidos** (ficam disponíveis) |
| `status_rules` | objeto | não exibido |
| `activities_executed[]` | array | não exibido (o texto da caixa Executing já resume) |
| `in_review` | `{ count, max_days_waiting, items[] }` | rodapé "Waiting in review (days)" (+ itens em revisão) |
| `automation_coverage` | `{ automated, manual, not_yet, total, previous_week, changed_vs_previous_week }` | rodapé "Automation coverage" ("1 de 8 itens automatizados") |
| `pillars_all[]` | `{ group, item, kpi_name, kpi_value, tracking, unit }`, os **8 itens** da aba pillars | cartões dos pilares. `kpi_value` = number, ou `null` quando não há valor ("No data yet"). `tracking` = `Automated`\|`Manual`\|`Not yet`. `unit` = `count`, ou `EUR` para "Paid media". |
| `pillars_with_value[]` | `{ group, item, kpi_name, kpi_value, tracking, last_update }`, só os itens com valor | reserva: a página só usa se `pillars_all` faltar (snapshots antigos) |
| `stages_without_data[]`, `collection` | array / objeto | não exibidos |

**Regra das chaves de rótulo:** prefixo + valor técnico em minúsculas, trocando tudo que não é letra ou número por `_`. Exemplos: `stage.demo_trial_request`, `product.ai_transformation`, `status.gray`, `tracking.not_yet`, `kpi.posts_published`.

## 3. Rótulos da página que foram criados para ela (já estão em ui_labels)

Estas 25 chaves foram acrescentadas à aba ui_labels junto com o workflow C. Se alguma faltar, a página mostra "—" e avisa no console, e o workflow C manda um aviso no e-mail (sem bloquear a publicação).

| key | en_CA | pt_BR |
|---|---|---|
| label.strategy | Strategy | Estratégia |
| strategy.positioning | Positioning | Posicionamento |
| strategy.target | Target | Público-alvo |
| strategy.engine | Engine | Motor de crescimento |
| label.week | Week | Semana |
| label.snapshot_date | Updated | Atualizado em |
| label.funnel | Funnel by product | Funil por produto |
| label.funnel_caption | Value this week, change vs. last week and status against target. | Valor da semana, variação vs. semana anterior e status em relação à meta. |
| label.product | Product | Produto |
| label.vs_last_week | vs. last week | vs. semana anterior |
| label.no_data_yet | No data yet | Sem dados ainda |
| badge.provisional | Provisional | Provisório |
| label.pillars | Pillars | Pilares |
| label.process_health | Process health | Saúde do processo |
| label.coverage_value | {automated} of {total} items automated | {automated} de {total} itens automatizados |
| label.items_in_review | Items in review | Itens em revisão |
| label.sample_data | Sample data | Dados de exemplo |
| pillar_item.linkedin_posts_and_articles | LinkedIn (posts and articles) | LinkedIn (posts e artigos) |
| pillar_item.website_articles | Website articles | Artigos no site |
| pillar_item.newsletter | Newsletter | Newsletter |
| pillar_item.prestigious_outlets_hbr_etc_articles_and_interviews | Prestigious outlets (HBR, etc.) – articles and interviews | Veículos de prestígio (HBR etc.) – artigos e entrevistas |
| pillar_item.white_paper_and_entry_materials | White paper and entry materials | White paper e materiais de entrada |
| pillar_item.paid_media | Paid media | Mídia paga |
| pillar_item.active_prospecting_and_site | Active prospecting and site | Prospecção ativa e site |
| pillar_item.monthly_community_report_internal_feeds_content_ideas | Monthly community report (internal, feeds content ideas) | Relatório mensal da comunidade (interno, alimenta ideias de conteúdo) |

Os `{automated}` e `{total}` de `label.coverage_value` são preenchidos pela página. Sem as chaves `pillar_item.*`, a página mostra o nome técnico do item, em inglês.

## 4. Lacunas

Todas as lacunas do prompt 3 foram fechadas:
- os rótulos entraram em ui_labels;
- `pillars_all` e `funnel[].provisional` passaram a vir no kpi_json;
- a unidade dos pilares vem em `unit`;
- `report_url` passou a ser preenchido pelo workflow B.

A página não tem mais regras de negócio fixas no código. Snapshots gravados antes desta mudança não têm `pillars_all` nem `provisional`: a página mostra só os pilares com valor e nenhum selo Provisional.

## 4.1 O que o workflow C valida antes de publicar (se falhar, não publica)

| Campo | Regra |
|---|---|
| JSON | serializa e volta sem erro |
| `sample` | `false` |
| `default_language` | `en-CA` ou `pt-BR` |
| `labels` | objeto não vazio (rótulos da página ausentes viram **aviso**, não erro) |
| `snapshot.week_start` | `AAAA-MM-DD` e segunda-feira |
| `snapshot.snapshot_date` | `AAAA-MM-DD` |
| `snapshot.how_its_going_status` | `Green`, `Yellow`, `Red` ou `Gray` |
| `snapshot.executing_text_*`, `how_its_going_text_*`, `what_to_change_text_*` | não vazios, nos 2 idiomas |
| `kpi` | objeto; `kpi.week_start` = `snapshot.week_start`; `kpi.funnel` não vazio; `kpi.currency` presente |
| arquivo | `<week_start>.json`, só `[A-Za-z0-9_.-]` |

## 4.2 Como o workflow C atualiza `index.json` e `latest.json`
- **Ordem obrigatória:**
  1. `<week_start>.json`;
  2. `index.json`;
  3. `latest.json`.
  Se uma etapa falhar, as seguintes não rodam, então `index.json` e `latest.json` nunca apontam para um arquivo que não existe.
- **index.json:** acrescenta a semana ou substitui a entrada da mesma semana (sem duplicar), ordena da mais recente para a mais antiga e remove entradas com `sample: true` ou inválidas.
- **latest.json:** só é reescrito se a semana publicada for a mais recente do índice. Republicar uma semana antiga não muda o latest.

## 5. Robustez esperada
- Qualquer campo pode faltar: a página mostra "—" ou "No data yet / Sem dados ainda" e não quebra.
- Status fora da lista vira Gray.
- Todo texto é inserido com `textContent`, então HTML vindo do JSON aparece como texto e não é executado.
- Datas sempre como `AAAA-MM-DD`. Números como número JSON, nunca como texto formatado.
