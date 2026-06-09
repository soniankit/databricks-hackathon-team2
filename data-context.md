# Data Context — Flo Marketing Campaign Builder

## Workspace

| Setting | Value |
|---|---|
| Profile | `hackathons` |
| Host | `https://emerging-emea-hackathons.cloud.databricks.com` |
| Warehouse | `vibe_coding_hackthon_london` — ID `56adb3367ffc45e8` (2X-Large, RUNNING) |
| Catalog / Schema | `flo_heatlh_hackathon.uc2_marketing_campaigns` |

## Accessing Data

**Python SDK (preferred in code)**
```python
from databricks.sdk import WorkspaceClient

w = WorkspaceClient(profile="hackathons")
result = w.statement_execution.execute_statement(
    warehouse_id="56adb3367ffc45e8",
    statement="SELECT * FROM flo_heatlh_hackathon.uc2_marketing_campaigns.campaign_history LIMIT 10",
    wait_timeout="30s"
)
# rows are in result.result.data_array (list of lists, matching result.manifest.schema.columns order)
```

**Spark / Databricks Connect**
```python
from databricks.connect import DatabricksSession
spark = DatabricksSession.builder.profile("hackathons").getOrCreate()
df = spark.table("flo_heatlh_hackathon.uc2_marketing_campaigns.campaign_history")
```

**Interactive SQL shell**
```bash
databricks --profile hackathons psql --warehouse-id 56adb3367ffc45e8
```

**REST API**
```bash
databricks --profile hackathons api post /api/2.0/sql/statements \
  --json '{"statement":"<SQL>","warehouse_id":"56adb3367ffc45e8","wait_timeout":"30s"}'
```

---

## Tables

### `campaign_history` — 500 rows

Historical campaign performance data.

| Column | Type | Description |
|---|---|---|
| `campaign_id` | string | PK — e.g. `CMP-0001` |
| `campaign_name` | string | Human-readable name |
| `channel` | string | `email` `push_notification` `in_app` `social` |
| `target_segment_id` | string | FK → `user_segments.segment_id` |
| `subject_line` | string | Campaign subject / title |
| `body_text` | string | Campaign body content |
| `cta_text` | string | CTA button text (20 unique variations) |
| `sent_date` | date | Send date (no time-of-day) |
| `open_rate` | double | Fraction opened (0.10–0.60, avg 0.36) |
| `click_rate` | double | Fraction clicked (0.01–0.25, avg 0.11) |
| `conversion_rate` | double | Fraction converted (0.002–0.10, avg 0.04) |
| `language` | string | `en` `es` `pt` `it` `de` `fr` `ja` |
| `_rescued_data` | string | Databricks internal — ignore |

Date range: 2024-01-12 → 2026-06-01.

---

### `user_segments` — 50 rows

Flo user audience segments. Total: 13.7M users.

| Column | Type | Description |
|---|---|---|
| `segment_id` | string | PK — e.g. `SEG-001` |
| `segment_name` | string | e.g. `"Pregnancy Week 7-33 LATAM"` |
| `life_stage` | string | `cycle_tracking` `pregnancy` `ttc` `perimenopause` `fertility` `postpartum` |
| `age_range` | string | `"18-23"` `"25-34"` `"35-44"` `"45-55"` |
| `geo_region` | string | `US` `UK` `EU` `LATAM` `APAC` `MEA` |
| `language` | string | 13 languages incl. `ar` `zh` `pl` `ru` `hi` `tr` + the 7 above |
| `user_count` | int | 149K–490K per segment |
| `avg_engagement_score` | double | 0.37–0.93 |
| `top_interests` | string | Comma-separated tags e.g. `"symptom_tracking,health_insights"` |
| `_rescued_data` | string | Databricks internal — ignore |

---

### `content_themes` — 100 rows

Marketing content theme library.

| Column | Type | Description |
|---|---|---|
| `theme_id` | string | PK — e.g. `THM-001` |
| `theme_name` | string | e.g. `"Period Pain Relief"` |
| `category` | string | `health` `wellness` `nutrition` `mental_health` `fertility` `fitness` |
| `keywords` | string | Comma-separated keywords |
| `seasonal_relevance` | string | `year-round` or specific season/month (`;`-separated) |
| `target_life_stages` | string | `;`-separated life stages e.g. `"pregnancy;ttc"` |
| `_rescued_data` | string | Databricks internal — ignore |

---

## Relationships

```
campaign_history.target_segment_id  →  user_segments.segment_id
campaign_history.language           ⊆  user_segments.language
content_themes.target_life_stages   ⊆  user_segments.life_stage
```

---

## Useful Queries

```sql
-- Explore available tables
SHOW TABLES IN flo_heatlh_hackathon.uc2_marketing_campaigns;

-- Segments for a given life stage + language
SELECT segment_id, segment_name, user_count, avg_engagement_score
FROM flo_heatlh_hackathon.uc2_marketing_campaigns.user_segments
WHERE life_stage = 'pregnancy' AND language = 'en'
ORDER BY avg_engagement_score DESC;

-- Historical performance benchmark for a channel + segment
SELECT AVG(open_rate) AS avg_open, AVG(click_rate) AS avg_click, AVG(conversion_rate) AS avg_cvr
FROM flo_heatlh_hackathon.uc2_marketing_campaigns.campaign_history
WHERE channel = 'push_notification' AND target_segment_id = 'SEG-039';

-- Themes relevant to a life stage this season
SELECT theme_name, category, keywords
FROM flo_heatlh_hackathon.uc2_marketing_campaigns.content_themes
WHERE target_life_stages LIKE '%pregnancy%'
  AND seasonal_relevance IN ('year-round', 'spring');

-- Top CTAs by click rate for a channel
SELECT cta_text, AVG(click_rate) AS avg_click, COUNT(*) AS n
FROM flo_heatlh_hackathon.uc2_marketing_campaigns.campaign_history
WHERE channel = 'push_notification'
GROUP BY cta_text HAVING COUNT(*) >= 3
ORDER BY avg_click DESC LIMIT 5;
```
