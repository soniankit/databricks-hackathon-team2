# Tech Instructions — Flo Marketing Campaign Builder

## Project Type
Internal Databricks Web App (hackathon project). Bias to the simplest thing that demos well. Don't over-engineer.

## Tech Stack & Infra
- **Backend:** Python + FastAPI
- **Frontend:** React + Vite
- **Database:** Lakebase (autoscaling) for Postgres — use `psycopg` for all DB queries
- **Auth:** Databricks OAuth via SDK
- **AI Generation:** Databricks Foundation Model endpoint via the SDK (all campaign copy, variants, and suggestions route through this)
- **Infrastructure-as-code:** DABs (Databricks Asset Bundles)
- **Deployment:** `app.yaml`

## How I Want to Work
- Use skills from the `.claude/skills` folder — especially **databricks-apps-python**, **databricks-config**, and **lakebase** skills. Don't reinvent what's already documented there.
- Start with a quick spec/plan — endpoints, DB schema, component tree, data flow. **Get my sign-off before writing code.**
- Scaffold the full project — backend, frontend, configs, everything.
- Seed mock data as part of setup so the app works on deploy (see **data-context.md**).

## Data
See **data-context.md** for all data context (mock data requirements, segments, content themes, and seeding).

## Ground Rules
- Hackathon — bias to the simplest thing that demos well. Don't over-engineer.
- No feature flags, no auth roles, no speculative abstractions.
- Keep it clean enough to reuse as a template for future apps.
- If something breaks, the logs should tell me exactly what went wrong.