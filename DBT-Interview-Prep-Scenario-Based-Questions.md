# dbt (Data Build Tool) Interview Prep — Scenario-Based Questions & Answers (Beginner → Ultra-Advanced)

> Goal: Make you **confidently answer any dbt interview question** — from "what is dbt?" to designing a multi-project dbt Mesh at scale — explained in **very simple words first**, then deep technical detail.
>
> For **every question** we cover four things, exactly as you asked:
> - **What** — what the question is really asking (decode it).
> - **Why** — why interviewers ask it, what skill they are secretly testing.
> - **How / Answer** — a ready-to-speak answer: simple version first, then the deep technical version.
> - **Impact** — what happens in the real world if you apply the solution vs. ignore it.
>
> Plus **🧪 Examples** (real models, failing tests, broken builds) and **📝 Cheat-sheet notes** to memorize.

---

## How To Read These Notes

- 🟢 **BASIC** — builds your foundation. Read first, in order.
- 🟡 **INTERMEDIATE** — how dbt really works day-to-day.
- 🔵 **ADVANCED** — the engineering patterns teams use in production.
- 🔴 **ULTRA-ADVANCED** — scale, CI/CD, contracts, mesh, and the deep internals.
- 💬 **Say-this-in-the-interview** = the actual words you can speak out loud.
- 📝 **Notes to Remember** = your flashcards.
- 🧪 **Example** = a concrete real-world snippet to make it stick.
- ⚡ **Impact** = consequence of getting it right vs. wrong.

You do **not** need to memorize. You need to *understand the story* behind each answer, so you can rebuild it in your own words under pressure.

---

## First: What Is dbt in One Breath?

**dbt = the "T" in ELT.** It is a command-line tool (and cloud product) that lets you transform data **already inside your warehouse** by writing **SELECT statements**. dbt turns those SELECTs into tables and views, in the right order, with **testing, documentation, and version control** built in.

Think of it this way: you write the *business logic* as plain SQL `SELECT` queries; dbt handles the *engineering* — dependencies, `CREATE TABLE` boilerplate, running things in the correct order, testing the output, and documenting it. It brings **software engineering best practices (modularity, testing, CI/CD, version control) to analytics SQL.**

> 🔑 The one-line definition to memorize: *"dbt is a transformation framework that lets analysts and engineers build, test, document, and deploy data models in the warehouse using just SQL and Jinja, following software-engineering best practices."*

---

## The Golden Framework — How To Answer ANY dbt Scenario Question

Most candidates fail not because they lack knowledge, but because they **ramble**. Use this 5-step skeleton. Interviewers love structure.

**Remember the word: C-A-T-C-H**

1. **C — Clarify**: Restate the problem, ask 1–2 sharp questions. *("Is this dbt Core or dbt Cloud? Which warehouse — Snowflake, BigQuery, Redshift? How big is the table?")*
2. **A — Approach**: State your high-level plan in one sentence *before* details. *("I'd model this in layers — staging, intermediate, marts — and make the big fact table incremental.")*
3. **T — Trade-offs**: Name an alternative and why you didn't pick it. *("I could fully rebuild it as a table, but at 2 billion rows that's too slow and costly, so I'd use an incremental model.")*
4. **C — Concrete**: Give a specific config, macro, or snippet. Naming `ref()`, `is_incremental()`, `unique_key`, `merge` builds credibility.
5. **H — Handle failure**: End with "what if it breaks?" — tests, `--full-refresh`, schema changes, idempotency, CI checks. **This makes you sound senior.**

> 🔑 **The pattern interviewers reward:** *Correctness → Maintainability → Performance → Cost.* Touch these four and you sound like someone who has run dbt in production.

---

## Quick Glossary (every scary word, in one line)

| Term | Plain-English meaning |
|---|---|
| **Model** | A single `.sql` file containing one `SELECT`; dbt turns it into a table or view. |
| **Materialization** | *How* dbt builds a model: `view`, `table`, `incremental`, or `ephemeral`. |
| **`ref()`** | "Reference another model" — builds dependencies and the run order automatically. |
| **`source()`** | A pointer to a raw table that landed in the warehouse (not built by dbt). |
| **Staging** | The first clean-up layer — one model per source table, light renaming/casting. |
| **Marts** | The final business-ready layer — facts and dimensions analysts query. |
| **Jinja** | The templating language (`{{ }}`) that adds programming power to SQL. |
| **Macro** | A reusable function written in Jinja/SQL — "DRY" for SQL. |
| **Seed** | A small CSV file dbt loads into the warehouse as a table. |
| **Snapshot** | dbt's way to capture history of changing rows (Slowly Changing Dimension Type 2). |
| **Test** | An assertion about data (e.g., "this column is unique and not null"). |
| **DAG** | Directed Acyclic Graph — the dependency map of all models, built from `ref()`. |
| **Incremental model** | A model that only processes *new/changed* rows, not the whole history. |
| **`dbt build`** | Runs models, tests, seeds, and snapshots together in DAG order. |
| **Manifest** | `manifest.json` — a file describing your whole project (used for docs, state, CI). |
| **Profile** | The connection config (warehouse, credentials) stored in `profiles.yml`. |
| **Package** | A shareable bundle of dbt models/macros (like a library), e.g., `dbt_utils`. |
| **Contract** | An enforced guarantee of a model's columns and types (schema as a promise). |

---

## Table of Contents

**🟢 PART 1 — BASIC**
1. [dbt Fundamentals (what, why, how it fits ELT)](#-part-1--basic)
2. [Models, `ref()`, `source()` & Project Structure](#2-models-ref-source--project-structure)
3. [Materializations (view, table, incremental, ephemeral)](#3-materializations--view-table-incremental-ephemeral)
4. [Commands, Seeds & the Basics of Running dbt](#4-commands-seeds--running-dbt)

**🟡 PART 2 — INTERMEDIATE**
5. [Tests (generic, singular, custom) & Data Quality](#5-tests--data-quality)
6. [Documentation, Sources & Source Freshness](#6-documentation-sources--freshness)
7. [Incremental Models (deep dive)](#7-incremental-models-deep-dive)
8. [Jinja, Macros & DRY SQL](#8-jinja-macros--dry-sql)
9. [Snapshots (SCD Type 2 history)](#9-snapshots--scd-type-2-history)

**🔵 PART 3 — ADVANCED**
10. [Incremental Strategies (merge, append, delete+insert, insert_overwrite)](#10-incremental-strategies)
11. [Packages, Hooks & Project Configuration](#11-packages-hooks--configuration)
12. [Environments, Variables, Targets & Custom Schemas](#12-environments-variables-targets--custom-schemas)
13. [State, Slim CI & dbt Artifacts](#13-state-slim-ci--artifacts)
14. [Exposures, Analyses & Advanced Selection](#14-exposures-analyses--node-selection)

**🔴 PART 4 — ULTRA-ADVANCED**
15. [CI/CD & Deployment Patterns](#15-cicd--deployment-patterns)
16. [Performance & Cost Optimization at Scale](#16-performance--cost-optimization-at-scale)
17. [Model Contracts, Versions & Governance](#17-model-contracts-versions--governance)
18. [Unit Tests & Write-Audit-Publish](#18-unit-tests--write-audit-publish)
19. [dbt Mesh, Semantic Layer & Multi-Project](#19-dbt-mesh-semantic-layer--multi-project)

**🧠 PART 5 — BEHAVIORAL & DESIGN**
20. [Behavioral + Architecture Scenarios](#20-behavioral--architecture-scenarios)

**📌 PART 6 — REFERENCE**
21. [Master Cheat Sheet & Rapid-Fire Q&A](#21-master-cheat-sheet--rapid-fire-qa)

---
---

# 🟢 PART 1 — BASIC

> Running example: an **online shop** warehouse. Raw tables land from the app database — `raw.jaffle_shop.customers`, `raw.jaffle_shop.orders`, `raw.stripe.payments`. We build clean models on top of them. Same world throughout, so it connects brick by brick.

## 1. dbt Fundamentals (what, why, how it fits ELT)

---

### Q1. "What is dbt and what problem does it solve?"

**🎯 What's being asked:** Can you explain dbt simply and say *why it exists* — not just recite a definition.

**🤔 Why they ask it:** It's the opening question of almost every dbt interview. They want to know you understand dbt's *purpose* (transformation + engineering discipline), not just that you've clicked "run."

**💬 Say-this-in-the-interview (simple first):**
> "dbt is a tool that lets me transform data that's already in my warehouse by writing plain `SELECT` statements. I write the business logic as SQL, and dbt handles the engineering around it — it figures out the order to run things, wraps my SELECTs in `CREATE TABLE`/`CREATE VIEW`, tests the results, and documents everything. Before dbt, analysts wrote giant unversioned SQL scripts with no testing and manual run order. dbt brings software-engineering best practices — modularity, version control, testing, and CI/CD — to analytics."

**Then go deeper:**
> "Technically, dbt is the **T in ELT**. Data is first **E**xtracted and **L**oaded raw into the warehouse by tools like Fivetran or Airbyte; dbt does the **T**ransform *inside* the warehouse, pushing compute down to Snowflake/BigQuery/Redshift/Databricks. Each transformation is a **model** — a `.sql` file with one `SELECT`. Models reference each other with the `ref()` function, and from those references dbt builds a **DAG** (dependency graph) so it always runs models in the correct order. It compiles Jinja + SQL into raw SQL and executes it against the warehouse."

**🧪 Example:**
> "Instead of one 800-line script that someone runs by hand, I split logic into small models: `stg_orders` cleans raw orders, `fct_orders` joins in payments. `fct_orders` says `from {{ ref('stg_orders') }}`, so dbt knows to build `stg_orders` first. One command — `dbt build` — runs them in order and tests them."

**📝 Notes to Remember:**
- dbt = **transformation framework**, the **T in ELT**, runs **in the warehouse** (push-down compute).
- You write **`SELECT`s**; dbt handles **run order, DDL, testing, docs, version control**.
- Core building block = **model** (`.sql` file = one SELECT). Dependencies via **`ref()`** → **DAG**.
- It brings **software engineering** (modularity, tests, CI/CD, Git) to analytics SQL.

**⚡ Impact:** Using dbt → reproducible, tested, documented, version-controlled pipelines a whole team can maintain. Without it → fragile, undocumented "SQL spaghetti" only one person understands.

---

### Q2. "dbt Core vs dbt Cloud — what's the difference and which would you choose?"

**🎯 What's being asked:** Do you know the two ways to run dbt and their trade-offs.

**🤔 Why they ask it:** Practical question — they want to know if you can pick the right setup for a team's size and budget.

**💬 Say-this-in-the-interview:**
> "**dbt Core** is the free, open-source command-line tool — I run it myself, anywhere (locally, in Airflow, in a container). **dbt Cloud** is the paid, managed platform on top of Core — it adds a browser IDE, a built-in scheduler, CI/CD integration, hosted docs, logging, and the Semantic Layer. Core gives full control but I have to handle orchestration, scheduling, and CI myself. Cloud reduces ops overhead and is faster for teams to adopt. I'd choose Core if we already have strong infra (Airflow, CI runners) and want to avoid per-seat cost; Cloud if the team wants speed, less infra work, and built-in scheduling and collaboration."

**📝 Notes to Remember:**
- **Core** = free, open-source CLI; you handle orchestration/scheduling/CI.
- **Cloud** = managed: IDE, scheduler, CI/CD, hosted docs, Semantic Layer; per-seat cost.
- Choose by **infra maturity + budget + team size**. Both run the *same* dbt project.

**⚡ Impact:** Right choice = less ops pain or lower cost depending on the team. The underlying project code is identical, so you're never locked in.

---

### Q3. "How does dbt fit into a modern data stack? Walk me through the flow."

**🎯 What's being asked:** Do you see where dbt sits among ingestion, warehouse, orchestration, and BI tools.

**🤔 Why they ask it:** They want context awareness — dbt isn't an island; it's one piece of a pipeline.

**💬 Say-this-in-the-interview:**
> "The modern data stack flows in stages. **Ingestion** tools (Fivetran, Airbyte, Stitch, or custom) extract from sources — APIs, app databases, files — and **load raw data** into a cloud **warehouse** (Snowflake, BigQuery, Redshift, Databricks). That's the EL. Then **dbt does the T** — it transforms that raw data into clean, tested, business-ready models inside the warehouse. An **orchestrator** (Airflow, dbt Cloud scheduler, Dagster) triggers dbt on a schedule. Finally **BI tools** (Looker, Tableau, Power BI) and data scientists consume dbt's final mart models. So dbt is the transformation brain in the middle, turning raw loaded data into trusted datasets."

**🧪 Example:**
> "Fivetran syncs Stripe and the app's Postgres into Snowflake every hour → Airflow triggers `dbt build` → dbt cleans and joins them into `fct_orders` and `dim_customers` → Looker dashboards read those marts. dbt is the step that makes raw data usable."

**📝 Notes to Remember:**
- Flow: **Sources → Ingestion (EL) → Warehouse → dbt (T) → BI / ML**.
- Ingestion = Fivetran/Airbyte; Warehouse = Snowflake/BigQuery/Redshift/Databricks; Orchestration = Airflow/dbt Cloud/Dagster; BI = Looker/Tableau/Power BI.
- dbt **only transforms** — it does not extract or load raw data.

**⚡ Impact:** Knowing dbt's place lets you design clean handoffs. Misplacing it (e.g., expecting dbt to ingest) leads to broken architecture.

---

## 2. Models, `ref()`, `source()` & Project Structure

---

### Q4. "What is a dbt model, and how do you structure a project into layers?"

**🎯 What's being asked:** Do you know what a model is *and* the standard layered architecture (staging → intermediate → marts).

**🤔 Why they ask it:** Project structure is the #1 sign of an experienced dbt developer. Beginners dump everything in one folder; pros layer it.

**💬 Say-this-in-the-interview (simple):**
> "A model is just a `.sql` file with a single `SELECT` statement. dbt takes that SELECT and builds it into a table or view in the warehouse. The file name becomes the model name."

**Then the layered structure (this impresses):**
> "I structure projects in three layers, which is dbt's recommended best practice:
> 1. **Staging** (`stg_`) — one model per *source table*. Light cleanup only: rename columns to a standard, cast types, basic formatting. These are the clean 'atoms' — a one-to-one tidy version of each raw table. Usually materialized as views.
> 2. **Intermediate** (`int_`) — optional middle models that hold complex logic, joins, or reusable pieces between staging and marts. They keep marts readable.
> 3. **Marts** (`fct_`/`dim_`) — the final business-ready layer: fact and dimension tables that analysts and BI tools query. Organized by business area (finance, marketing).
> This layering makes logic modular, reusable, and easy to debug — each layer has one clear job."

**🧪 Example folder layout:**
```text
models/
  staging/
    jaffle_shop/
      stg_customers.sql
      stg_orders.sql
      _jaffle_shop__sources.yml
  intermediate/
    int_orders_joined_payments.sql
  marts/
    finance/
      fct_orders.sql
      dim_customers.sql
```

**📝 Notes to Remember:**
- Model = **one `.sql` file = one `SELECT`** → becomes a table/view.
- Layers: **staging (`stg_`) → intermediate (`int_`) → marts (`fct_`/`dim_`)**.
- Staging = 1:1 clean per source (views). Marts = business facts/dims (tables).
- Layering = modular, reusable, debuggable. Name consistently.

**⚡ Impact:** A layered project is maintainable and onboards new people fast. A flat pile of models with duplicated logic becomes unmaintainable spaghetti.

---

### Q5. "What does `ref()` do, and why is it the heart of dbt?"

**🎯 What's being asked:** Do you understand that `ref()` builds dependencies and the run order automatically.

**🤔 Why they ask it:** `ref()` is dbt's single most important function. If you get this, you get dbt.

**💬 Say-this-in-the-interview:**
> "`ref()` is how one model references another. Instead of hard-coding a table name like `analytics.fct_orders`, I write `{{ ref('fct_orders') }}`. This does two powerful things. First, dbt uses every `ref()` to build the **DAG** — the dependency graph — so it automatically knows `fct_orders` must be built before anything that refs it, and runs models in the correct order with no manual sequencing. Second, `ref()` makes code **environment-aware** — it compiles to the right database/schema for dev, staging, or prod automatically, so the same code runs everywhere without changes."

**🧪 Example:**
```sql
-- models/marts/fct_orders.sql
select
    o.order_id,
    o.customer_id,
    p.amount
from {{ ref('stg_orders') }}   o
left join {{ ref('stg_payments') }} p on o.order_id = p.order_id
```
> "Because I used `ref()`, dbt builds `stg_orders` and `stg_payments` first, then `fct_orders` — automatically. In dev it compiles to my dev schema; in prod, the prod schema."

**📝 Notes to Remember:**
- `ref('model_name')` = reference another model → builds the **DAG** + run order automatically.
- Makes SQL **environment-aware** (dev/prod schemas) — never hard-code table names.
- `ref()` between models; **`source()`** for raw tables dbt didn't build.
- The DAG from `ref()` powers run order, lineage docs, and selective runs.

**⚡ Impact:** `ref()` gives automatic ordering, lineage, and portability across environments. Hard-coding table names breaks all three and causes wrong-environment disasters.

---

### Q6. "What is a `source()` and why use it instead of querying raw tables directly?"

**🎯 What's being asked:** Do you know sources declare raw inputs and unlock freshness checks and lineage.

**🤔 Why they ask it:** It separates people who just write SELECTs from those who model raw inputs properly.

**💬 Say-this-in-the-interview:**
> "A **source** is dbt's way of declaring the raw tables that landed in the warehouse from ingestion — tables dbt did *not* build. I define them once in a YAML file, then reference them with `{{ source('jaffle_shop', 'orders') }}` instead of hard-coding `raw.jaffle_shop.orders`. Three benefits: (1) **lineage** — sources show up at the start of the DAG and in docs, so I can trace data back to its origin; (2) **source freshness checks** — dbt can warn or error if raw data is stale; (3) **one place to change** — if the raw schema moves, I update the YAML once, not every model."

**🧪 Example:**
```yaml
# models/staging/jaffle_shop/_jaffle_shop__sources.yml
sources:
  - name: jaffle_shop
    database: raw
    schema: jaffle_shop
    tables:
      - name: orders
        freshness:
          warn_after: {count: 12, period: hour}
          error_after: {count: 24, period: hour}
        loaded_at_field: _loaded_at
```
```sql
-- stg_orders.sql
select * from {{ source('jaffle_shop', 'orders') }}
```

**📝 Notes to Remember:**
- `source()` = reference **raw, ingested** tables (declared in YAML); `ref()` = reference **dbt-built** models.
- Benefits: **lineage**, **freshness monitoring**, **single point of change** for raw schema.
- Staging models read from `source()`; everything downstream uses `ref()`.

**⚡ Impact:** Sources give end-to-end lineage and stale-data alerts. Hard-coding raw tables loses lineage and breaks silently when raw schemas change.

---

## 3. Materializations — view, table, incremental, ephemeral

---

### Q7. "What are dbt materializations? Explain all four and when to use each."

**🎯 What's being asked:** Do you know *how* dbt physically builds a model and how to pick the right strategy.

**🤔 Why they ask it:** This is the most common dbt concept question. Choosing materializations correctly is the core of performance and cost tuning.

**💬 Say-this-in-the-interview (simple):**
> "A materialization is *how* dbt builds my model in the warehouse — the same `SELECT` can become a view, a full table, an incrementally-updated table, or no object at all. There are four built-in ones."

**The four, with when-to-use:**

| Materialization | What dbt does | When to use |
|---|---|---|
| **view** (default) | Creates a `VIEW` — query runs fresh each time it's read | Light transforms, staging models, data that must always be live, small data |
| **table** | Creates a physical `TABLE`, fully rebuilt each run | Models queried often where read speed matters; complex logic you don't want recomputed |
| **incremental** | Builds the table once, then only **inserts/updates new rows** on later runs | Large fact/event tables where full rebuild is too slow/expensive |
| **ephemeral** | Builds **nothing** — inlined as a CTE into models that ref it | Small reusable logic you don't want as its own object; reduces clutter |

**Then the trade-off logic:**
> "The trade-off is **freshness/storage/compute vs. read speed**. Views cost no storage and are always fresh but recompute on every read — slow for heavy logic. Tables are fast to read but cost storage and full recompute time. Incremental is for big tables where rebuilding everything every run is wasteful. Ephemeral keeps the DAG clean for small helper logic. My default: **views for staging, tables for marts, incremental for large facts.**"

**🧪 Example:**
```sql
-- Set in the model file...
{{ config(materialized='table') }}
select ...

-- ...or in dbt_project.yml for whole folders:
-- models:
--   my_project:
--     staging:
--       +materialized: view
--     marts:
--       +materialized: table
```

**📝 Notes to Remember:**
- 4 types: **view** (default, always fresh, recomputes), **table** (fast read, full rebuild), **incremental** (only new/changed rows, big tables), **ephemeral** (CTE, no object).
- Default rule of thumb: **staging = view, marts = table, large facts = incremental**.
- Set via `{{ config(materialized=...) }}` in the model or `+materialized:` in `dbt_project.yml`.
- Trade-off = **read speed vs. storage + compute + freshness**.

**⚡ Impact:** Right materialization = fast, cheap, fresh-enough pipelines. Wrong choice (e.g., table for everything, or view on a huge heavy model) = slow dashboards and runaway warehouse bills.

---

### Q8. "What is an ephemeral model and what's the catch?"

**🎯 What's being asked:** Do you understand ephemeral models are inlined CTEs with specific limitations.

**🤔 Why they ask it:** It's a nuanced one that reveals depth — many people forget the gotchas.

**💬 Say-this-in-the-interview:**
> "An ephemeral model doesn't create any table or view in the warehouse. Instead, dbt **injects its SQL as a CTE** into any model that `ref()`s it. It's useful for small, reusable bits of logic I want to keep DRY without cluttering the warehouse with tiny objects. The catches: you **can't query an ephemeral model directly** in the warehouse (it doesn't exist there), it can make **debugging harder** (the SQL is buried inside other models), and you **can't run tests on it** the usual way. So I use ephemeral only for simple, lightweight intermediate logic — not for anything I'd want to inspect or test independently."

**📝 Notes to Remember:**
- Ephemeral = **inlined as a CTE**, creates **no warehouse object**.
- Pros: keeps logic DRY, no clutter. Cons: **can't query directly, harder to debug, limited testing**.
- Use for small, lightweight, reusable intermediate logic only.

**⚡ Impact:** Used right, ephemeral keeps the warehouse tidy. Overused, it hides logic and makes debugging painful.

---

## 4. Commands, Seeds & Running dbt

---

### Q9. "Explain `dbt run` vs `dbt test` vs `dbt build`. When do you use each?"

**🎯 What's being asked:** Do you know the core commands and especially why `dbt build` is the modern default.

**🤔 Why they ask it:** Daily-driver knowledge. Confusing these signals you haven't actually used dbt.

**💬 Say-this-in-the-interview:**
> "**`dbt run`** executes my models — it builds the tables and views, in DAG order. **`dbt test`** runs the data tests (assertions like unique, not-null) against already-built models. **`dbt build`** is the modern all-in-one: for each node in DAG order it runs models, **then their tests**, plus seeds and snapshots — and crucially it **stops downstream models if an upstream test fails**. So `build` interleaves run + test node-by-node, which catches bad data before it flows downstream. I use `dbt build` as my default in production; `dbt run` when I just want to materialize quickly during development."

**Other commands to mention:**
> "`dbt seed` loads CSVs, `dbt snapshot` captures history, `dbt compile` generates the raw SQL without running it, `dbt docs generate` builds documentation, and `dbt deps` installs packages."

**🧪 Example:**
```bash
dbt build --select stg_orders+      # build stg_orders and everything downstream, with tests
dbt run   --select marts.finance    # just materialize the finance marts
dbt test  --select fct_orders        # run only tests on fct_orders
```

**📝 Notes to Remember:**
- `dbt run` = build models only. `dbt test` = run tests only. `dbt build` = **run + test + seed + snapshot**, node by node, **halting downstream on test failure**.
- `build` is the **production default** — it prevents bad data from flowing on.
- Others: `seed`, `snapshot`, `compile`, `docs generate`, `deps`.

**⚡ Impact:** `dbt build` catches failures early and protects downstream models. Using only `dbt run` skips tests, letting bad data reach dashboards.

---

### Q10. "What is node selection? How do you run just part of your project?"

**🎯 What's being asked:** Do you know the `--select` / `--exclude` syntax and graph operators to run subsets efficiently.

**🤔 Why they ask it:** On real projects you never run everything every time. Selection syntax is a daily productivity and CI skill.

**💬 Say-this-in-the-interview:**
> "Node selection lets me run a subset of the DAG instead of the whole project — essential for big projects and CI. The key tool is the **graph operators**: `+` means 'and everything upstream/downstream.' So `dbt build --select stg_orders+` runs `stg_orders` and everything *downstream* of it; `+fct_orders` runs `fct_orders` and everything *upstream* it depends on. I can also select by **folder** (`marts.finance`), by **tag** (`tag:nightly`), by **source** (`source:stripe+`), and exclude with `--exclude`. The most powerful for CI is **state-based selection** — `state:modified+` runs only models that changed versus production, plus their downstream."

**🧪 Example:**
```bash
dbt build --select +fct_orders          # fct_orders and all its parents
dbt build --select stg_orders+          # stg_orders and all its children
dbt build --select marts.finance        # everything in the finance folder
dbt build --select tag:daily            # everything tagged 'daily'
dbt build --select state:modified+      # changed models + downstream (Slim CI)
```

**📝 Notes to Remember:**
- `--select` / `--exclude` choose nodes; `+` = include upstream (left) or downstream (right).
- Select by **name, folder, tag, source, path**, and **`state:modified`** (changed vs a baseline).
- `state:modified+` powers **Slim CI** — only build what changed + downstream.

**⚡ Impact:** Selection makes dev fast and CI cheap (build only what changed). Running the whole project every time wastes time and warehouse money.

---

### Q11. "What are seeds, and when should (and shouldn't) you use them?"

**🎯 What's being asked:** Do you know seeds load CSVs and their appropriate (limited) use.

**🤔 Why they ask it:** Seeds are simple but commonly misused for data that belongs in ingestion.

**💬 Say-this-in-the-interview:**
> "A **seed** is a small CSV file in my dbt project that `dbt seed` loads into the warehouse as a table. Seeds are version-controlled with my code, so they're perfect for **small, static reference data** that rarely changes — things like country-code mappings, a list of test accounts to exclude, category labels, or status-code lookups. The key rule: seeds are for **small and static** data only. They are **not** for loading large datasets or anything that changes frequently — that belongs in a proper ingestion tool. A good limit is a few thousand rows; CSVs aren't a loading pipeline."

**🧪 Example:**
```text
seeds/country_codes.csv  →  dbt seed  →  table `country_codes`
Then: select * from {{ ref('country_codes') }}
```

**📝 Notes to Remember:**
- Seed = **CSV in the repo** loaded by `dbt seed`; reference it with `ref()` like a model.
- Use for **small, static, version-controlled reference data** (lookups, mappings, exclusion lists).
- **Don't** use for large or frequently-changing data — that's ingestion's job.

**⚡ Impact:** Seeds keep reference data versioned with logic. Misusing them for big/changing data bloats the repo and slows runs.

---
---

# 🟡 PART 2 — INTERMEDIATE

## 5. Tests & Data Quality

---

### Q12. "What kinds of tests does dbt have? Explain generic vs singular tests."

**🎯 What's being asked:** Do you know dbt's two test types and the four built-in generic tests.

**🤔 Why they ask it:** Testing is dbt's killer feature for trust. Knowing the types is fundamental.

**💬 Say-this-in-the-interview (simple):**
> "A dbt test is an assertion about my data — a query that looks for rows that *break* a rule. If it finds any bad rows, the test fails. There are two kinds."

**The two types:**
> "**Generic tests** are reusable, parameterized tests applied in YAML. dbt ships four out of the box: **`unique`** (no duplicate values), **`not_null`** (no NULLs), **`accepted_values`** (value is in an allowed list), and **`relationships`** (every value exists in a parent table — referential integrity). I just attach them to columns in a YAML file.
> **Singular tests** are one-off tests written as a `.sql` file containing a `SELECT` that returns the rows that *should not exist*. If the query returns zero rows, it passes. I use these for custom business rules that the generic tests don't cover."

**🧪 Example — generic (YAML):**
```yaml
models:
  - name: fct_orders
    columns:
      - name: order_id
        tests: [unique, not_null]
      - name: status
        tests:
          - accepted_values:
              values: ['placed', 'shipped', 'delivered', 'cancelled']
      - name: customer_id
        tests:
          - relationships:
              to: ref('dim_customers')
              field: customer_id
```
**🧪 Example — singular (SQL):**
```sql
-- tests/assert_no_negative_revenue.sql
-- Passes if this returns ZERO rows
select order_id, amount
from {{ ref('fct_orders') }}
where amount < 0
```

**📝 Notes to Remember:**
- Test = a query that finds **rows that break a rule**; **0 bad rows = pass**.
- **Generic** (reusable, in YAML): the big four = **`unique`, `not_null`, `accepted_values`, `relationships`**.
- **Singular** (one-off, in `.sql`): custom business rules; `SELECT` the *bad* rows.
- `dbt_utils` and `dbt_expectations` packages add many more generic tests.

**⚡ Impact:** Tests turn "I think the data's fine" into proof, and catch issues before dashboards. No tests = silent data corruption discovered by angry stakeholders.

---

### Q13. "Your unique test on a primary key fails. How do you debug it?"

**🎯 What's being asked:** Do you have a systematic debugging method, not panic.

**🤔 Why they ask it:** Real work is mostly debugging. They want to see your process.

**💬 Say-this-in-the-interview:**
> "First, I look at the **compiled test SQL** — dbt prints the path to it, or I check the failure in `target/`. dbt can store failing rows if I set **`--store-failures`**, so I can query exactly which keys are duplicated. Then I trace *why*: the most common cause is a **fan-out join** — joining to a table that has multiple rows per key multiplied my rows. I check each join in the model for a one-to-many relationship I didn't expect. Other causes: the source genuinely has duplicates (then I dedupe in staging), or my grain is wrong (the model is at a finer grain than I assumed). I fix it at the right layer — usually dedupe in staging or correct the join — and the test passes."

**🧪 Example:**
> "A `unique` test on `order_id` in `fct_orders` failed. `--store-failures` showed each `order_id` appearing 3 times. The cause: I joined `payments`, and some orders had 3 payment rows. Fix: aggregate payments to one row per order *before* joining, restoring the one-row-per-order grain."

**📝 Notes to Remember:**
- Use **`dbt test --store-failures`** to see the actual bad rows.
- Inspect **compiled SQL** in `target/compiled/`.
- #1 cause of duplicate-key failures = **fan-out join** (one-to-many you didn't expect).
- Fix at the right **grain** — dedupe in staging or aggregate before joining.

**⚡ Impact:** A methodical approach finds the root cause fast (usually a join grain issue). Guessing or just adding `distinct` hides the real bug and can drop valid data.

---

### Q14. "How do you configure test severity and thresholds?"

**🎯 What's being asked:** Do you know tests can warn vs error, and tolerate a threshold of bad rows.

**🤔 Why they ask it:** Production needs nuance — not every issue should halt the pipeline.

**💬 Say-this-in-the-interview:**
> "Tests don't have to be all-or-nothing. I can set **`severity: warn`** so a test logs a warning but doesn't fail the build, versus **`severity: error`** which stops it. I can also set **thresholds**: `error_if` and `warn_if` (e.g., 'warn if more than 10 bad rows, error if more than 100'). This is useful when a tiny number of imperfect rows is acceptable but a spike signals a real problem. I make critical tests (primary keys, financial integrity) hard errors, and softer quality checks warnings, so the build doesn't break over trivial issues — which also prevents alert fatigue."

**🧪 Example:**
```yaml
columns:
  - name: email
    tests:
      - not_null:
          config:
            severity: warn
            warn_if: ">10"
            error_if: ">500"
```

**📝 Notes to Remember:**
- `severity: warn` (log only) vs `error` (fail build).
- Thresholds: `warn_if` / `error_if` to tolerate a small number of bad rows.
- Critical rules (PK, money) = **error**; soft quality = **warn**. Avoids unnecessary build breaks + alert fatigue.

**⚡ Impact:** Tuned severity keeps builds running through trivial issues while still catching real ones. All-error tests break builds over nothing; all-warn tests let real problems slip.

---

## 6. Documentation, Sources & Freshness

---

### Q15. "How does documentation work in dbt? What is the docs site and lineage graph?"

**🎯 What's being asked:** Do you know dbt auto-generates docs and a visual lineage DAG from your project.

**🤔 Why they ask it:** Self-documenting pipelines are a major dbt selling point. Teams live or die by docs.

**💬 Say-this-in-the-interview:**
> "In dbt, documentation lives **next to the code** in YAML files — I add `description:` fields to models and columns. Then `dbt docs generate` builds a website from my project, and `dbt docs serve` hosts it. The docs site shows every model, its columns, descriptions, tests, and the actual compiled SQL. Best of all, it renders an **interactive lineage graph** — the DAG — built automatically from my `ref()` and `source()` calls, so anyone can visually trace how data flows from raw sources to final marts. For longer descriptions I use **doc blocks** (`{% docs %}`) so I can reuse and richly format text. Because docs are version-controlled with the code, they never drift out of date the way a separate wiki does."

**📝 Notes to Remember:**
- Docs = `description:` in YAML + `dbt docs generate` / `serve` → a **website**.
- Auto-generates an **interactive lineage graph (DAG)** from `ref()`/`source()`.
- **Doc blocks** (`{% docs name %}`) for reusable, rich descriptions.
- Docs are **version-controlled with code** → always current, unlike a separate wiki.

**⚡ Impact:** Auto-docs + lineage make the project self-explanatory and onboard people fast. No docs = tribal knowledge and slow, error-prone changes.

---

### Q16. "What is source freshness and how do you use it?"

**🎯 What's being asked:** Do you know dbt can monitor whether raw data is stale.

**🤔 Why they ask it:** Detecting stale upstream data is a key reliability practice; this shows operational maturity.

**💬 Say-this-in-the-interview:**
> "Source freshness checks whether the raw data loaded by ingestion is recent enough. In my source YAML I set a **`loaded_at_field`** (a timestamp column) and thresholds — **`warn_after`** and **`error_after`**. Then `dbt source freshness` checks how old the newest row is: if data hasn't loaded within my window, dbt warns or errors. This catches a broken or delayed ingestion *before* I run transformations on stale data. I typically run the freshness check at the start of my pipeline, so if Fivetran failed overnight, I find out immediately instead of building dashboards on yesterday's data."

**🧪 Example:**
```yaml
sources:
  - name: stripe
    tables:
      - name: payments
        loaded_at_field: _synced_at
        freshness:
          warn_after:  {count: 6,  period: hour}
          error_after: {count: 12, period: hour}
```
```bash
dbt source freshness
```

**📝 Notes to Remember:**
- Freshness = is raw data recent? Set `loaded_at_field` + `warn_after`/`error_after`.
- Run `dbt source freshness` (often **first** in the pipeline) to catch broken ingestion early.
- Prevents transforming **stale** data into fresh-looking dashboards.

**⚡ Impact:** Freshness checks catch upstream failures before they poison reports. Without them, a silently failed ingestion serves stale data that looks current.

---

## 7. Incremental Models (Deep Dive)

---

### Q17. "Explain incremental models. How do they actually work?"

**🎯 What's being asked:** Do you deeply understand the most important performance feature in dbt.

**🤔 Why they ask it:** Incremental models are *the* intermediate/advanced topic. Getting this right shows you can handle big data in dbt.

**💬 Say-this-in-the-interview (simple):**
> "An incremental model avoids rebuilding a huge table from scratch every run. The first run builds the full table; every run after that, dbt only processes the **new or changed rows** and adds them to the existing table. This is essential for large fact/event tables where a full rebuild would take hours and cost a fortune."

**Then how it works technically:**
> "The magic is the **`is_incremental()`** macro. dbt wraps my model in logic: on the *first* run (or with `--full-refresh`), it builds everything. On *later* runs, the `is_incremental()` block adds a filter that pulls only rows newer than what's already in the table. I define a **`unique_key`** so dbt knows how to update existing rows versus insert new ones. The flow is: build once → then filter to new rows → merge them in."

**🧪 Example:**
```sql
{{ config(materialized='incremental', unique_key='order_id') }}

select *
from {{ source('jaffle_shop', 'orders') }}

{% if is_incremental() %}
  -- only on incremental runs: grab rows newer than what we already have
  where _loaded_at > (select max(_loaded_at) from {{ this }})
{% endif %}
```
> "`{{ this }}` refers to the existing version of this very table. So the filter says: 'only rows loaded after the latest one I already stored.'"

**📝 Notes to Remember:**
- Incremental = build full **once**, then only process **new/changed** rows.
- Powered by **`is_incremental()`** (true only on non-first, non-full-refresh runs) + **`{{ this }}`** (the existing table).
- Define a **`unique_key`** so dbt can update vs insert.
- Rebuild from scratch anytime with **`dbt run --full-refresh`**.

**⚡ Impact:** Incremental models cut runtime and cost dramatically on big tables (hours → minutes). Full-rebuilding huge tables every run is the classic dbt cost blow-up.

---

### Q18. "What are the dangers of incremental models, and how do you handle late-arriving or changed data?"

**🎯 What's being asked:** Do you know incremental models can miss data and how to make them robust.

**🤔 Why they ask it:** Incremental models are powerful but easy to get subtly wrong — a senior signal.

**💬 Say-this-in-the-interview:**
> "Incremental models trade safety for speed, so they have real pitfalls. The big one is **missing late-arriving or updated rows**: if my filter is `_loaded_at > max(_loaded_at)` and a record arrives late with an older timestamp, I'll skip it. To handle this I use a **lookback window** — reprocess, say, the last 3 days (`where event_date >= dateadd(day, -3, current_date)`) — so late and updated rows get re-captured, combined with a `unique_key` and a `merge` strategy so re-processed rows update rather than duplicate. The other danger is **schema drift** — if I add a column, incremental runs won't pick it up until a `--full-refresh`. And I must always set a `unique_key` or risk duplicates. The safety valve for any of these is `dbt run --full-refresh` to rebuild cleanly."

**🧪 Example:**
> "An events table filtered strictly on `max(loaded_at)`. Mobile events from offline phones arrived late and were silently dropped — counts were wrong. Fix: a 3-day lookback plus `unique_key=event_id` with `merge`, so late events get inserted and corrected events get updated. We accept reprocessing a few extra days for correctness."

**📝 Notes to Remember:**
- Pitfalls: **missed late/updated rows**, **schema drift** (need `--full-refresh`), **duplicates** without `unique_key`.
- Fix late data with a **lookback window** + `unique_key` + `merge`.
- New columns require **`--full-refresh`** (or `on_schema_change`).
- Always keep `--full-refresh` as the safety valve.

**⚡ Impact:** Robust incremental logic stays both fast *and* correct. Naive incremental filters silently lose late data — wrong numbers that are very hard to trace.

---

### Q19. "What is `on_schema_change` and why does it matter for incremental models?"

**🎯 What's being asked:** Do you know how dbt handles a changed model schema during incremental runs.

**🤔 Why they ask it:** A subtle but real production issue — adding a column shouldn't silently break things.

**💬 Say-this-in-the-interview:**
> "By default, if I add or remove a column in an incremental model, the existing table's schema doesn't automatically match, which can cause errors or silently ignored columns. The **`on_schema_change`** config controls this. Options: **`ignore`** (default — don't change the table, new columns are dropped from the insert), **`append_new_columns`** (add new columns to the existing table), **`sync_all_columns`** (add new *and* remove deleted columns), and **`fail`** (stop with an error so I notice). I usually set `append_new_columns` so adding a column doesn't require a full refresh, or `fail` in strict environments where I want to be forced to decide. For big structural changes, a `--full-refresh` is still the clean path."

**📝 Notes to Remember:**
- `on_schema_change`: **`ignore`** (default), **`append_new_columns`**, **`sync_all_columns`**, **`fail`**.
- Controls how incremental tables react to **added/removed columns**.
- `append_new_columns` avoids a full refresh for simple additions; `fail` forces awareness.

**⚡ Impact:** Setting `on_schema_change` prevents silently dropped columns and surprise errors. The default `ignore` can quietly lose new fields if you're unaware.

---

## 8. Jinja, Macros & DRY SQL

---

### Q20. "What is Jinja in dbt and why is it powerful?"

**🎯 What's being asked:** Do you understand dbt SQL is actually templated, and what Jinja unlocks.

**🤔 Why they ask it:** Jinja is what turns dbt from "SQL files" into a programmable framework. Intermediate+ roles need it.

**💬 Say-this-in-the-interview:**
> "Jinja is a templating language that lets me add **programming logic to my SQL**. Anything in `{{ }}` gets evaluated and `{% %}` is for statements like loops and conditionals. dbt **compiles** my Jinja+SQL into plain SQL before running it. This is what powers everything: `ref()` and `source()` are Jinja functions, I can write **`if`/`else`** logic (like `is_incremental()`), **`for` loops** to generate repetitive SQL, use **variables**, and call **macros**. So instead of copy-pasting 20 nearly-identical CASE statements, I loop over a list. It makes SQL dynamic, reusable, and DRY (Don't Repeat Yourself)."

**🧪 Example — a loop generating SQL:**
```sql
select
    order_id,
    {% for status in ['placed', 'shipped', 'delivered'] %}
    sum(case when status = '{{ status }}' then 1 else 0 end) as {{ status }}_count
    {%- if not loop.last %},{% endif %}
    {% endfor %}
from {{ ref('stg_orders') }}
group by order_id
```
> "That loop writes three near-identical lines for me. I can run `dbt compile` to see the final plain SQL it produces."

**📝 Notes to Remember:**
- Jinja = templating: `{{ }}` expressions, `{% %}` statements; dbt **compiles** it to plain SQL.
- Powers `ref()`, `source()`, `is_incremental()`, **variables, loops, conditionals, macros**.
- Use it to keep SQL **DRY and dynamic**; `dbt compile` shows the rendered SQL.

**⚡ Impact:** Jinja makes SQL reusable and dynamic, eliminating copy-paste errors. Overusing it, though, can make models unreadable — balance power with clarity.

---

### Q21. "What is a macro? Give an example of when you'd write one."

**🎯 What's being asked:** Do you know macros are reusable functions and when they add value.

**🤔 Why they ask it:** Macros are how teams avoid repeating logic. Writing good ones is an advanced-intermediate skill.

**💬 Say-this-in-the-interview (simple):**
> "A macro is a reusable function written in Jinja and SQL — basically 'DRY for SQL.' I define it once and call it in many models, so common logic lives in one place. If the logic needs to change, I change the macro, not 50 models."

**Then a concrete use case:**
> "A classic example is converting currency, or generating a standard set of date columns, or cleaning phone numbers consistently. Say every model needs the same 'cents to dollars' conversion — I write a `cents_to_dollars` macro and call it everywhere. dbt itself is built on macros, and packages like **`dbt_utils`** give me battle-tested ones like `dbt_utils.generate_surrogate_key()` so I don't reinvent them."

**🧪 Example:**
```sql
-- macros/cents_to_dollars.sql
{% macro cents_to_dollars(column_name, decimals=2) %}
    round({{ column_name }} / 100.0, {{ decimals }})
{% endmacro %}
```
```sql
-- used in a model:
select
    order_id,
    {{ cents_to_dollars('amount_cents') }} as amount_usd
from {{ ref('stg_payments') }}
```

**📝 Notes to Remember:**
- Macro = **reusable Jinja/SQL function** → define once, call everywhere (**DRY**).
- Great for repeated logic: conversions, surrogate keys, standardized cleaning.
- Use **`dbt_utils`** for ready-made macros (e.g., `generate_surrogate_key`, `pivot`, `date_spine`).
- Live in the `macros/` folder.

**⚡ Impact:** Macros centralize logic so a change happens in one place, consistently. Copy-pasted SQL drifts — fix it in one model, forget the other 12, and now numbers disagree.

---

## 9. Snapshots — SCD Type 2 History

---

### Q22. "What are dbt snapshots and what problem do they solve?"

**🎯 What's being asked:** Do you know snapshots capture history of changing source data (Slowly Changing Dimension Type 2).

**🤔 Why they ask it:** Source systems overwrite data; snapshots are dbt's answer to "what did this look like last month?" — a core warehousing concept.

**💬 Say-this-in-the-interview (simple):**
> "Many source systems only store the *current* state — when a customer changes their address or an order changes status, the old value is overwritten and lost. A **snapshot** is dbt's feature for capturing that history. Every time it runs, it checks for changes and records them, so I can see what a record looked like at any point in time. It's how dbt implements a **Slowly Changing Dimension Type 2**."

**Then how it works:**
> "I define a snapshot that selects from a source and tell dbt how to detect changes — either **`timestamp`** strategy (watch an `updated_at` column) or **`check`** strategy (compare a list of columns for any change). When `dbt snapshot` runs and detects a change, it **closes off the old row** by setting `dbt_valid_to` to now, and **inserts a new row** with `dbt_valid_to = null` (the current version). dbt manages these `dbt_valid_from`/`dbt_valid_to` columns for me. The result is a full history table I can query 'as of' any date."

**🧪 Example:**
```sql
{% snapshot orders_snapshot %}
{{ config(
    target_schema='snapshots',
    unique_key='order_id',
    strategy='timestamp',
    updated_at='updated_at'
) }}
select * from {{ source('jaffle_shop', 'orders') }}
{% endsnapshot %}
```
> "If order 101 goes from 'shipped' to 'delivered,' the snapshot closes the 'shipped' row and adds a 'delivered' row — both preserved with their valid date ranges."

**📝 Notes to Remember:**
- Snapshot = capture **history** of changing source rows → **SCD Type 2**.
- Strategies: **`timestamp`** (watch `updated_at`) or **`check`** (compare columns).
- dbt manages **`dbt_valid_from` / `dbt_valid_to`** (and `dbt_scd_id`); current row has `valid_to = null`.
- Run with **`dbt snapshot`**; store in a dedicated schema. Run it **before** transformations so you don't miss changes.

**⚡ Impact:** Snapshots preserve history that the source destroys, enabling point-in-time analysis. Without them, "what was the price last quarter?" is unanswerable — the data is gone forever.

---

### Q23. "Snapshot `timestamp` strategy vs `check` strategy — when do you use each?"

**🎯 What's being asked:** Do you know the two change-detection strategies and their trade-offs.

**🤔 Why they ask it:** Picking the wrong one either misses changes or wastes resources — a practical nuance.

**💬 Say-this-in-the-interview:**
> "The **`timestamp`** strategy relies on a reliable `updated_at` column — dbt compares it to detect changes. It's efficient and the preferred choice *when the source has a trustworthy, always-updated timestamp*. The **`check`** strategy compares the actual values of a specified list of columns (or all columns) between runs to detect any change — I use it when there's **no reliable `updated_at`** column. The trade-off: `timestamp` is cleaner and cheaper but depends on the source updating that column correctly; `check` works without a timestamp but is more resource-intensive and only catches changes in the columns I list. So: use `timestamp` if a dependable `updated_at` exists, otherwise fall back to `check`."

**📝 Notes to Remember:**
- **`timestamp`** = watch a reliable `updated_at`; efficient, preferred when available.
- **`check`** = compare listed columns' values; use when **no reliable timestamp**; heavier.
- Beware: `timestamp` misses changes if the source doesn't update the column; `check` only catches columns you list.

**⚡ Impact:** Right strategy reliably captures history at reasonable cost. Wrong one silently misses changes (bad timestamp) or wastes compute (over-broad check).

---
---

# 🔵 PART 3 — ADVANCED

## 10. Incremental Strategies

---

### Q24. "Explain the incremental strategies: merge, append, delete+insert, insert_overwrite."

**🎯 What's being asked:** Do you know *how* dbt actually inserts incremental rows, and the right strategy per warehouse and use case.

**🤔 Why they ask it:** This is the deepest incremental question. It separates people who've tuned big tables from those who just copied a config.

**💬 Say-this-in-the-interview:**
> "The `incremental_strategy` config controls *how* dbt merges new rows into the existing table. The main ones:
> - **`merge`** (default on Snowflake, BigQuery, Databricks) — uses a SQL `MERGE` keyed on `unique_key`: updates matching rows, inserts new ones. Best for upserts where rows can change. Needs a `unique_key`.
> - **`append`** — just inserts new rows, no update or dedup. Fastest, but creates duplicates if the same row comes twice. Good for **immutable** event logs where rows never change and never repeat.
> - **`delete+insert`** — deletes rows matching the new batch's keys, then inserts. Used on warehouses without efficient `MERGE` (e.g., older Redshift), or to replace whole partitions/keys.
> - **`insert_overwrite`** — replaces entire **partitions** with new data (great on BigQuery/Spark/Databricks partitioned tables). Ideal for 'rebuild the last N days' patterns — overwrite just those date partitions idempotently.
> I choose based on the warehouse and whether rows are mutable: `merge` for upserts, `append` for pure immutable logs, `insert_overwrite` for partition-based reprocessing."

**🧪 Example:**
```sql
{{ config(
    materialized='incremental',
    incremental_strategy='insert_overwrite',
    partition_by={'field': 'order_date', 'data_type': 'date'}
) }}
-- On each run, fully overwrites the partitions present in the new data
-- → idempotent 'reprocess last 3 days' pattern
```

**📝 Notes to Remember:**
- **`merge`** = upsert by `unique_key` (default Snowflake/BQ/Databricks); handles changing rows.
- **`append`** = insert only; fastest; **immutable** logs only (dup risk otherwise).
- **`delete+insert`** = delete matching keys then insert; for no-efficient-MERGE warehouses / key replacement.
- **`insert_overwrite`** = replace whole **partitions**; best for "reprocess last N days" idempotently (BQ/Spark/Databricks).
- Choose by **warehouse + mutability + partition strategy**.

**⚡ Impact:** Right strategy = correct, fast, idempotent incremental loads. Wrong one (e.g., `append` on mutable data) silently creates duplicates or misses updates.

---

### Q25. "How do incremental models stay idempotent? Why does idempotency matter in dbt?"

**🎯 What's being asked:** Do you connect dbt patterns to the broader engineering principle of safe re-runs.

**🤔 Why they ask it:** Pipelines get retried; idempotency is the difference between safe and corrupt.

**💬 Say-this-in-the-interview:**
> "Idempotent means running a model twice gives the same result as running it once — no duplicates, no double-counting. It matters because jobs fail and get retried, and someone might re-run a model manually. In dbt I get idempotency by using **`merge` with a `unique_key`** (re-running just updates the same rows) or **`insert_overwrite`** on partitions (re-running overwrites the same partition cleanly). The strategy to *avoid* for mutable data is **`append`**, because re-running inserts the rows again and duplicates them. So I design incremental models so a retry is always safe — overwrite or upsert, never blind append."

**🧪 Example:**
> "A daily incremental used `append`. A retry after a failure inserted the day's rows twice — revenue looked doubled. We switched to `insert_overwrite` partitioned by date, so re-running the day just overwrites that date's partition. Retries became harmless."

**📝 Notes to Remember:**
- Idempotent = **safe to re-run**; achieved via **`merge` + `unique_key`** or **`insert_overwrite`** by partition.
- **`append` is not idempotent** for repeatable/mutable data → duplicates on retry.
- Always design so a **retry overwrites/upserts**, never blindly appends.

**⚡ Impact:** Idempotent models make retries no-ops. Non-idempotent ones turn every failed-and-retried run into duplicate data and a manual cleanup.

---

## 11. Packages, Hooks & Configuration

---

### Q26. "What are dbt packages? Name ones you'd use and why."

**🎯 What's being asked:** Do you know packages are reusable dbt code (like libraries) and the ecosystem staples.

**🤔 Why they ask it:** Reusing community code shows maturity — you don't reinvent wheels.

**💬 Say-this-in-the-interview:**
> "A package is a bundle of dbt models and macros someone else wrote that I can import into my project — like a library. I list them in `packages.yml` and run `dbt deps` to install. The most important is **`dbt_utils`** — a Swiss-army knife of macros and generic tests (surrogate keys, `date_spine`, `pivot`, `union_relations`, extra tests). **`dbt_expectations`** adds Great-Expectations-style data quality tests. **`codegen`** auto-generates boilerplate YAML and staging models. **`dbt_audit_helper`** compares two models for validating refactors. And there are source-specific packages (like for Stripe or Salesforce) that model common SaaS data for you. Using packages saves huge time and gives me battle-tested, community-maintained code."

**🧪 Example:**
```yaml
# packages.yml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1
  - package: calogica/dbt_expectations
    version: 0.10.1
```
```sql
-- then use a macro:
select {{ dbt_utils.generate_surrogate_key(['order_id', 'product_id']) }} as id, ...
```

**📝 Notes to Remember:**
- Package = reusable dbt code (library); declare in **`packages.yml`**, install with **`dbt deps`**.
- Staples: **`dbt_utils`** (macros + tests), **`dbt_expectations`** (DQ tests), **`codegen`** (scaffolding), **`audit_helper`** (refactor validation).
- Source packages model common SaaS data (Stripe, Salesforce, etc.).

**⚡ Impact:** Packages save time and bring tested code. Reinventing surrogate keys or date spines by hand wastes time and adds bugs.

---

### Q27. "What are hooks (pre-hook, post-hook, on-run-start/end)? Give real uses."

**🎯 What's being asked:** Do you know how to run SQL around model builds for operational tasks.

**🤔 Why they ask it:** Hooks handle the operational glue — grants, logging, cleanup — that real deployments need.

**💬 Say-this-in-the-interview:**
> "Hooks let me run SQL **around** my dbt runs. **`pre-hook`** and **`post-hook`** run *before/after a specific model* builds. **`on-run-start`** and **`on-run-end`** run *once at the start/end of the whole `dbt run`*. Common real uses: **granting permissions** on a freshly built table with a post-hook (so BI tools can read it), **vacuum/analyze** or clustering maintenance after a build, **logging** run metadata to an audit table with `on-run-end`, or setting warehouse/session parameters with `on-run-start`. They're the operational glue that wraps transformation with the housekeeping production needs."

**🧪 Example:**
```sql
-- grant access automatically after each model builds
{{ config(
    post_hook="grant select on {{ this }} to role reporter"
) }}
```
```yaml
# dbt_project.yml — log at the end of every run
on-run-end:
  - "{{ log_run_results() }}"
```

**📝 Notes to Remember:**
- **`pre-hook`/`post-hook`** = SQL around **each model**; **`on-run-start`/`on-run-end`** = around the **whole run**.
- Real uses: **grants**, table maintenance (vacuum/analyze/cluster), **audit logging**, session settings.
- Set per-model in `config()` or globally in `dbt_project.yml`.

**⚡ Impact:** Hooks automate operational tasks (like grants) so they never get forgotten. Without them, freshly built tables may be inaccessible or unmaintained.

---

### Q28. "How is a dbt project configured? Walk through `dbt_project.yml` and config hierarchy."

**🎯 What's being asked:** Do you understand project-level config and how configs cascade and override.

**🤔 Why they ask it:** Knowing where to set config (and which wins) is essential for managing a real project.

**💬 Say-this-in-the-interview:**
> "**`dbt_project.yml`** is the heart of the project — it names the project, points to model/seed/macro paths, and sets **default configurations** for groups of models (materializations, tags, schemas, etc.). Configs follow a **hierarchy of specificity**: project-level defaults in `dbt_project.yml` are the most general, then **YAML config files**, then the **`{{ config() }}` block inside a model file** is the most specific and wins. So I set broad defaults centrally — e.g., 'everything in `staging/` is a view, everything in `marts/` is a table' — and override per-model only where needed. The connection details (warehouse, credentials) live separately in **`profiles.yml`**, kept out of the repo for security."

**🧪 Example:**
```yaml
# dbt_project.yml
models:
  my_project:
    staging:
      +materialized: view
      +tags: ['staging']
    marts:
      +materialized: table
```
> "A specific model can override with `{{ config(materialized='incremental') }}` — the in-file config beats the project default."

**📝 Notes to Remember:**
- **`dbt_project.yml`** = project name, paths, and **default configs** for folders.
- Precedence (specific wins): **in-model `config()`** > property YAML > `dbt_project.yml`.
- **`profiles.yml`** holds **connection/credentials** — kept out of version control.
- Set broad defaults centrally; override per-model only when needed.

**⚡ Impact:** A clean config hierarchy means consistent defaults with surgical overrides. Scattering config everywhere makes behavior unpredictable and hard to audit.

---

## 12. Environments, Variables, Targets & Custom Schemas

---

### Q29. "How do you manage dev vs prod environments in dbt?"

**🎯 What's being asked:** Do you know targets/profiles isolate environments so dev never clobbers prod.

**🤔 Why they ask it:** Environment separation is critical — a dev run hitting prod tables is a disaster.

**💬 Say-this-in-the-interview:**
> "Environments are managed through **targets** in `profiles.yml`. Each target (e.g., `dev`, `prod`) points to a different database/schema and credentials. In **dev**, dbt builds into my personal dev schema, so my work is fully isolated from production. In **prod**, the scheduled job runs with `--target prod` and builds into the production schema. Because all my models use **`ref()`** — which resolves to the current target's schema automatically — the *same code* builds safely in either place with zero changes. I never hard-code schema names. In dbt Cloud this maps to environments; in Core, I pass `--target` or set it via env vars in CI/CD."

**📝 Notes to Remember:**
- **Targets** in `profiles.yml` define each environment (dev/prod) — different schema + credentials.
- **`ref()` auto-resolves** to the active target's location → same code, any environment.
- Dev builds to a **personal/isolated schema**; prod via `--target prod` in the scheduler.
- **Never hard-code** database/schema names.

**⚡ Impact:** Proper targets keep dev experiments away from prod data. Hard-coded schemas or a single environment risk a dev run overwriting production tables.

---

### Q30. "What are dbt variables (`var`) and environment variables (`env_var`)? When use each?"

**🎯 What's being asked:** Do you know how to parameterize a project for flexibility and secrets.

**🤔 Why they ask it:** Variables enable reusable, configurable, secure pipelines — an engineering concern.

**💬 Say-this-in-the-interview:**
> "Both inject values into my project so I'm not hard-coding. **`var()`** reads a variable defined in `dbt_project.yml` or passed on the command line with `--vars` — I use it for things like a configurable start date, a list of countries, or feature flags. **`env_var()`** reads an **environment variable** from the system — I use it for anything environment-specific or **secret**, like credentials, account IDs, or the target schema in CI/CD, so secrets never live in the repo. Rule of thumb: `var` for project logic/configuration committed to the repo; `env_var` for secrets and per-environment values that must stay outside the code."

**🧪 Example:**
```sql
-- var: configurable lookback, with a default
where order_date >= '{{ var("start_date", "2024-01-01") }}'
```
```sql
-- env_var: never commit secrets
{{ config(snowflake_warehouse=env_var('DBT_WAREHOUSE')) }}
```
```bash
dbt run --vars '{start_date: "2025-01-01"}'
```

**📝 Notes to Remember:**
- **`var()`** = project variables (in `dbt_project.yml` or `--vars`); for logic/config in the repo.
- **`env_var()`** = system environment variables; for **secrets** and **per-environment** values (kept out of repo).
- Both support defaults: `var('x', default)`, `env_var('X', default)`.

**⚡ Impact:** Variables make projects flexible and secure. Hard-coding values or committing secrets is rigid and a security risk.

---

### Q31. "How does custom schema/database generation work, and why customize it?"

**🎯 What's being asked:** Do you understand `generate_schema_name` and controlling where models land.

**🤔 Why they ask it:** Default schema naming surprises people; controlling it is needed for real warehouse organization.

**💬 Say-this-in-the-interview:**
> "By default, when I set a custom schema on a model, dbt **concatenates** it onto the target schema — so a model with `+schema: marts` in the `analytics` target lands in `analytics_marts`, not just `marts`. That confuses people. dbt does this via a built-in macro called **`generate_schema_name`**. I can **override** that macro in my project to control naming — for example, to use the custom schema directly in prod but keep everything in my personal schema in dev. Similarly `generate_database_name` controls database placement. I customize this to organize models into logical schemas (staging, marts, finance) cleanly while keeping dev isolated."

**📝 Notes to Remember:**
- Default custom-schema behavior = **`<target_schema>_<custom_schema>`** (concatenated), via **`generate_schema_name`** macro.
- **Override** `generate_schema_name` (and `generate_database_name`) to control placement.
- Common pattern: use custom schemas directly in **prod**, but isolate everything in **dev**.

**⚡ Impact:** Controlling schema generation keeps the warehouse organized and dev isolated. The default concatenation confuses teams and can scatter models unexpectedly.

---

## 13. State, Slim CI & Artifacts

---

### Q32. "What are dbt's artifacts (manifest.json, run_results.json) and why do they matter?"

**🎯 What's being asked:** Do you know dbt produces metadata files that power state, docs, and CI.

**🤔 Why they ask it:** Artifacts are the backbone of advanced workflows (Slim CI, observability). Knowing them signals depth.

**💬 Say-this-in-the-interview:**
> "Every dbt invocation writes JSON **artifacts** to the `target/` folder. The big ones: **`manifest.json`** is a complete description of my project — every model, test, source, their configs, and the full DAG. **`run_results.json`** records what happened in the last run — each node's status, timing, and errors. **`catalog.json`** holds column-level metadata for the docs site, and **`sources.json`** holds freshness results. These matter because they enable powerful workflows: **state comparison** (compare the current `manifest.json` to production's to find what changed → Slim CI), **observability** (parse `run_results.json` for timing/failures to monitor performance), and **docs generation**. They're how dbt knows about itself programmatically."

**📝 Notes to Remember:**
- Artifacts (in `target/`): **`manifest.json`** (whole project + DAG), **`run_results.json`** (last run status/timing), **`catalog.json`** (column docs), **`sources.json`** (freshness).
- Power **state comparison (Slim CI)**, **observability/monitoring**, and **docs**.
- `manifest.json` = the project's self-description; the foundation of `state:` selection.

**⚡ Impact:** Artifacts enable Slim CI, monitoring, and docs. Ignoring them means rebuilding everything in CI and flying blind on run performance.

---

### Q33. "What is Slim CI and how does `state:modified` work?"

**🎯 What's being asked:** Do you understand running only changed models in CI by comparing state.

**🤔 Why they ask it:** Slim CI is the gold-standard cost-saving CI pattern. It's a strong senior signal.

**💬 Say-this-in-the-interview:**
> "Slim CI means: in a pull request, only build and test the models that **actually changed**, plus their downstream dependents — instead of the whole project. It works by **state comparison**. I keep the `manifest.json` from the last successful **production** run as a baseline. In CI, I run `dbt build --select state:modified+ --defer --state path/to/prod/manifest`. dbt compares the PR's project to the production manifest, finds modified nodes (`state:modified`), and the `+` includes everything downstream. **`--defer`** lets unchanged upstream models resolve to the existing production tables instead of rebuilding them. So a one-model change tests just that model and its children — CI runs in seconds and costs almost nothing."

**🧪 Example:**
```bash
dbt build --select state:modified+ \
          --defer --state ./prod-artifacts
# Only changed models + downstream are built; the rest defer to prod
```

**📝 Notes to Remember:**
- **Slim CI** = build/test only **changed models + downstream** in PRs.
- Mechanism: **`state:modified+`** (compare to a baseline `manifest.json`) + **`--defer`** (use prod for unchanged upstream).
- Needs the **production `manifest.json`** stored as the `--state` baseline.
- Massively cuts CI time and warehouse cost.

**⚡ Impact:** Slim CI makes CI fast and cheap, so testing every PR is painless. Rebuilding the whole project per PR is slow and expensive, discouraging good CI habits.

---

### Q34. "How do you write a custom generic test?"

**🎯 What's being asked:** Do you know you can build your own reusable parameterized tests with Jinja.

**🤔 Why they ask it:** Custom tests show you can extend dbt's quality framework to business-specific rules.

**💬 Say-this-in-the-interview:**
> "When the built-in tests aren't enough, I write a **custom generic test** — a reusable, parameterized test as a macro-like block in a `tests/generic/` file. It's a `test` block that takes `model` and `column_name` (and any extra args) and returns a `SELECT` of the **failing rows**. Then I can apply it in YAML to any column, just like `unique` or `not_null`. For example, a test that a numeric column is always positive, or that a value matches a regex, or a business rule like 'discount can't exceed price.' This lets me encode company-specific quality rules once and reuse them everywhere."

**🧪 Example:**
```sql
-- tests/generic/test_is_positive.sql
{% test is_positive(model, column_name) %}
select {{ column_name }}
from {{ model }}
where {{ column_name }} <= 0
{% endtest %}
```
```yaml
# apply it like any generic test
columns:
  - name: amount
    tests: [is_positive]
```

**📝 Notes to Remember:**
- Custom generic test = a **`{% test %}`** block taking `model` + `column_name`, returning **failing rows**.
- Lives in `tests/generic/`; applied in YAML like built-in tests.
- Encodes **reusable business-specific quality rules** (positivity, regex, cross-column rules).

**⚡ Impact:** Custom tests extend quality coverage to your exact business rules, reused everywhere. Without them, business rules go unchecked or get copy-pasted as fragile singular tests.

---

## 14. Exposures, Analyses & Node Selection

---

### Q35. "What are exposures and why are they useful?"

**🎯 What's being asked:** Do you know exposures document downstream consumers (dashboards, ML) in the DAG.

**🤔 Why they ask it:** Exposures complete end-to-end lineage — into the BI layer — a governance plus.

**💬 Say-this-in-the-interview:**
> "An **exposure** is a way to declare a downstream use of my dbt models — like a specific dashboard, a report, or an ML model — directly in the dbt project as YAML. It extends my lineage graph *past* the warehouse to show 'this Looker dashboard depends on `fct_orders`.' Two big benefits: (1) **full lineage and impact analysis** — before I change or deprecate a model, I can see exactly which dashboards it feeds; and (2) I can **test and run by exposure** — `dbt build --select +exposure:weekly_finance_dashboard` builds everything that dashboard depends on. It also surfaces ownership and a link in the docs site, so people know who owns each downstream asset."

**🧪 Example:**
```yaml
exposures:
  - name: weekly_finance_dashboard
    type: dashboard
    maturity: high
    url: https://looker.company.com/dashboards/42
    depends_on:
      - ref('fct_orders')
      - ref('dim_customers')
    owner:
      name: Finance Team
      email: finance@company.com
```

**📝 Notes to Remember:**
- Exposure = declare a **downstream consumer** (dashboard/report/ML) in YAML → extends lineage past the warehouse.
- Benefits: **impact analysis** (what breaks if I change this model) + **selection** (`+exposure:name`) + ownership in docs.

**⚡ Impact:** Exposures give true end-to-end lineage and safe change management. Without them, you can't see which dashboards a model change will break.

---

### Q36. "What's the difference between an analysis and a model?"

**🎯 What's being asked:** Do you know analyses are compiled-but-not-materialized SQL.

**🤔 Why they ask it:** A smaller nuance that shows you know the full toolset.

**💬 Say-this-in-the-interview:**
> "An **analysis** is SQL that lives in the `analyses/` folder. dbt will **compile** it — resolving `ref()`, Jinja, and macros into runnable SQL — but it will **not materialize** it into a table or view. It's for ad-hoc or one-off queries that benefit from dbt's templating and references but that I don't want to persist as warehouse objects: investigative queries, audits, or training/QA SQL. A model, by contrast, is always built into the warehouse. So: analysis = compiled, version-controlled, reusable SQL I run manually; model = a persisted table/view in the DAG."

**📝 Notes to Remember:**
- **Analysis** = in `analyses/`; dbt **compiles but doesn't materialize** (no table/view).
- Use for **ad-hoc/investigative SQL** that still wants `ref()`/Jinja and version control.
- **Model** = always materialized and part of the DAG.

**⚡ Impact:** Analyses keep useful one-off queries versioned and templated without warehouse clutter. Misusing models for throwaway queries pollutes the warehouse.

---
---

# 🔴 PART 4 — ULTRA-ADVANCED

## 15. CI/CD & Deployment Patterns

---

### Q37. "Design a complete CI/CD pipeline for a dbt project."

**🎯 What's being asked:** Can you design the full software-engineering deployment workflow around dbt — PR checks, environments, promotion, production runs.

**🤔 Why they ask it:** This is the flagship senior question. Running dbt in production *well* is what they're hiring for.

**💬 Say-this-in-the-interview (use C-A-T-C-H):**
> "I'd treat dbt like application code with a Git-based workflow.
> 1. **Development:** Each developer works on a branch, building into their **isolated dev schema**. Everything — models, tests, YAML — is in Git.
> 2. **Pull Request → CI:** Opening a PR triggers a CI job that spins up a temporary schema and runs **Slim CI**: `dbt build --select state:modified+ --defer --state <prod-manifest>`. This builds and tests only changed models plus downstream, against real data, in minutes. It also runs `dbt compile`/linting (e.g., SQLFluff). The PR can't merge unless CI is green.
> 3. **Merge → Deploy:** Merging to `main` triggers the **production** job (via dbt Cloud scheduler, Airflow, or GitHub Actions) running `dbt build --target prod`, which materializes and tests into production schemas.
> 4. **Scheduling:** Production runs on a schedule (or is triggered by upstream ingestion completion), with **source freshness** checked first.
> 5. **Artifacts & monitoring:** I store the prod `manifest.json` as the next CI baseline, and parse `run_results.json` for run-time/failure monitoring and alerting.
> The whole loop gives tested, reviewed, reproducible deployments with fast, cheap CI."

**Trade-offs / extras to voice:**
> "I'd add **environment separation** (dev/CI/prod schemas), **blue-green or WAP** for risky changes, and protect `main` with required reviews. For data volume in CI, I `--defer` to prod so I don't rebuild the world."

**📝 Notes to Remember:**
- Flow: **branch + dev schema → PR triggers Slim CI (`state:modified+ --defer`) + lint → merge → prod `dbt build` → scheduled runs → store manifest + monitor**.
- CI builds **only changed + downstream** against real data in a temp schema; gate merges on green.
- Production via scheduler/orchestrator with **freshness first**; persist artifacts for the next baseline.
- Add lint (SQLFluff), required reviews, environment isolation.

**⚡ Impact:** A real CI/CD loop makes every change tested, reviewed, and safe to ship. Without it, dbt deployments are manual, risky, and break production silently.

---

### Q38. "How do you handle secrets, credentials, and environment config securely in dbt?"

**🎯 What's being asked:** Do you follow security best practices for connection and secrets management.

**🤔 Why they ask it:** Security is non-negotiable; leaking warehouse credentials is a serious incident.

**💬 Say-this-in-the-interview:**
> "Credentials and secrets never go in the repo. Connection details live in **`profiles.yml`**, which is kept *outside* version control, and within it I reference secrets via **`env_var()`** so the actual values come from environment variables or a secrets manager (Vault, AWS Secrets Manager, GitHub/dbt Cloud secrets), not hard-coded. In CI/CD, secrets are injected as masked environment variables. I also follow **least privilege** — the dbt service account has only the grants it needs (read sources, write its target schemas), and dev/prod use separate credentials. In dbt Cloud, connections and environment variables are managed in the UI with role-based access. The principle: secrets in a vault, referenced by name, scoped to least privilege."

**📝 Notes to Remember:**
- Keep **`profiles.yml` out of Git**; reference secrets via **`env_var()`** from a vault/secrets manager.
- CI/CD injects secrets as **masked env vars**; never echo them.
- **Least privilege** service accounts; separate **dev/prod** credentials.
- dbt Cloud manages connections + env vars with RBAC.

**⚡ Impact:** Proper secret handling prevents credential leaks and limits blast radius. Hard-coded secrets in the repo are a breach waiting to happen.

---

## 16. Performance & Cost Optimization at Scale

---

### Q39. "Your dbt project takes hours to run and the warehouse bill is climbing. How do you optimize it?"

**🎯 What's being asked:** Can you systematically diagnose and tune a slow, expensive dbt project.

**🤔 Why they ask it:** Cost/performance at scale is exactly what senior dbt engineers are paid to manage.

**💬 Say-this-in-the-interview (give a method):**
> "I diagnose with data, then fix the biggest offenders. First, I **find the slow models** — parse `run_results.json` or use the dbt Cloud model-timing view to rank models by execution time. Then I apply the right lever per model:
> - **Materialization fixes:** convert expensive, frequently-rebuilt tables to **incremental**; switch rarely-read heavy tables to views; avoid `table` for things only used as intermediate steps.
> - **Incremental tuning:** use `insert_overwrite`/`merge` with partitioning so I process only new data.
> - **Warehouse physics:** add **partitioning/clustering** (BigQuery/Snowflake) and sort keys (Redshift) so queries scan less; prune columns and filter early.
> - **DAG parallelism:** increase **`threads`** so independent models run concurrently; restructure the DAG to reduce long dependency chains and bottleneck models.
> - **Reduce rebuilds:** Slim CI so dev/CI don't rebuild everything; schedule heavy models less often if freshness allows.
> - **Right-size compute:** match warehouse size to the workload; don't run everything on an XL.
> I prioritize the few models consuming most of the time and cost — usually a handful of full-refresh giants."

**🧪 Example:**
> "Model-timing showed one `table` model rebuilding 1.5 billion rows every hour took 70% of runtime. We made it **incremental** with `insert_overwrite` partitioned by date, processing only the last day. Runtime dropped from ~3 hours to ~12 minutes and the bill fell sharply — one model fixed most of the problem."

**📝 Notes to Remember:**
- **Measure first**: `run_results.json` / model-timing to rank slow models.
- Levers: **incremental** big tables, **views** for light/rarely-read, **partition/cluster/sort keys**, more **`threads`**, **Slim CI**, right-size warehouse, **prune + filter early**.
- Fix the **few giant models** that dominate time/cost (Pareto).
- Reduce **full refreshes** and unnecessary `table` materializations.

**⚡ Impact:** Targeted tuning can cut runtime and cost by an order of magnitude. Blindly scaling up the warehouse hides the real problem and burns money.

---

### Q40. "What's the difference between threads and warehouse sizing, and how do they affect performance?"

**🎯 What's being asked:** Do you distinguish dbt's parallelism from warehouse compute, and tune both.

**🤔 Why they ask it:** People conflate the two; getting it right is real performance knowledge.

**💬 Say-this-in-the-interview:**
> "They're different knobs. **`threads`** is a dbt setting that controls how many models dbt runs **in parallel** — with 8 threads, dbt builds up to 8 independent (non-dependent) models at once, respecting the DAG. More threads = more concurrency, but only helps if the DAG has independent branches and the warehouse can handle the concurrent load. **Warehouse sizing** is the compute power of each query — a bigger Snowflake warehouse runs each individual model faster (and more concurrently). So `threads` parallelizes *across* models; warehouse size speeds up *each* model. I tune both together: enough threads to exploit DAG parallelism, and a warehouse sized to handle that concurrency without queuing — but not so big I'm overpaying for idle capacity."

**📝 Notes to Remember:**
- **`threads`** = how many models dbt runs **concurrently** (DAG-aware parallelism).
- **Warehouse size** = compute power per query (faster individual models + concurrency).
- `threads` helps only with **independent DAG branches** + enough warehouse capacity.
- Tune **together**: threads to exploit parallelism, warehouse sized to avoid queuing without waste.

**⚡ Impact:** Balancing threads and warehouse size maximizes throughput per dollar. Too few threads underuses compute; too many on a small warehouse causes queuing and timeouts.

---

## 17. Model Contracts, Versions & Governance

---

### Q41. "What are model contracts and why use them?"

**🎯 What's being asked:** Do you know dbt can enforce a model's schema (columns/types) as a guarantee.

**🤔 Why they ask it:** Contracts (dbt 1.5+) are a modern governance feature for reliability at scale — a senior topic.

**💬 Say-this-in-the-interview:**
> "A **model contract** is an enforced promise about a model's output schema — its exact columns, data types, and constraints. I declare it in YAML with `contract: {enforced: true}` and list the columns and types. Then dbt **checks at build time** that the model produces exactly that shape — if my SQL returns a different type or is missing a column, the build **fails before** publishing bad structure. This protects downstream consumers: a contracted model is a stable interface they can rely on. It's especially valuable for 'public' models other teams or dashboards depend on, where an accidental column rename or type change would break them. It turns schema from an accident into a guaranteed contract."

**🧪 Example:**
```yaml
models:
  - name: fct_orders
    config:
      contract: {enforced: true}
    columns:
      - name: order_id
        data_type: integer
        constraints: [{type: not_null}, {type: primary_key}]
      - name: amount
        data_type: numeric
```

**📝 Notes to Remember:**
- Contract = **enforced schema** (columns, **data types**, constraints) checked **at build time**.
- Declared with `contract: {enforced: true}` + column `data_type`s in YAML.
- Build **fails** if output doesn't match → protects downstream consumers.
- Use for **public/shared models** that others depend on.

**⚡ Impact:** Contracts guarantee a stable interface, catching breaking schema changes before they ship. Without them, a silent column/type change breaks downstream dashboards and models.

---

### Q42. "What is model versioning and when would you use it?"

**🎯 What's being asked:** Do you know dbt supports multiple versions of a model for safe, gradual migration.

**🤔 Why they ask it:** Versioning (paired with contracts) is how mature teams evolve shared models without breaking consumers.

**💬 Say-this-in-the-interview:**
> "Model versioning lets me maintain **multiple versions of the same logical model** simultaneously — like `fct_orders` v1 and v2 — so I can roll out a breaking change gradually. When I need to change a contracted, widely-used model in a breaking way (drop a column, change grain), I create **v2** with the new shape while **v1 keeps running**. Consumers migrate from v1 to v2 on their own timeline, and once everyone's moved, I deprecate v1. dbt tracks versions and can mark a `latest_version`. This is essential for 'public' models with many downstream dependents — it turns a scary big-bang breaking change into a smooth, backward-compatible migration."

**📝 Notes to Remember:**
- Versioning = **multiple versions of one model** (v1, v2) coexisting for **gradual migration**.
- Pairs with **contracts**; use for **breaking changes** to shared/public models.
- Keep old version alive, migrate consumers, then **deprecate**.

**⚡ Impact:** Versioning lets you evolve shared models without breaking anyone — migrate on their schedule. Without it, a breaking change to a popular model breaks many dashboards at once.

---

### Q43. "How do you implement governance: groups, access, and model ownership?"

**🎯 What's being asked:** Do you know dbt's governance features for access control and ownership in large projects.

**🤔 Why they ask it:** At scale, controlling who can reference what and who owns what is essential — a maturity signal.

**💬 Say-this-in-the-interview:**
> "dbt has governance features for large, multi-team projects. **Groups** let me assign models to a team with an owner. **Access modifiers** control visibility: a model can be **`private`** (only referenceable within its group), **`protected`** (referenceable within the same project — the default), or **`public`** (referenceable by other projects/teams). So I mark internal staging/intermediate models as `private` and only expose curated marts as `public` interfaces. Combined with **contracts** (stable schema) and **versions** (safe evolution), this lets me publish trustworthy 'data products' other teams consume while keeping internal models hidden and changeable. Ownership metadata also routes responsibility and shows up in docs."

**📝 Notes to Remember:**
- **Groups** = assign models to a team + owner.
- **Access**: **`private`** (within group), **`protected`** (within project, default), **`public`** (cross-project).
- Expose curated marts as **`public` + contracted + versioned** data products; keep internals **`private`**.
- Foundation for **dbt Mesh** / multi-team governance.

**⚡ Impact:** Governance lets teams safely share curated data products while protecting internal models. Without it, everything is referenceable and any change risks breaking unknown consumers.

---

## 18. Unit Tests & Write-Audit-Publish

---

### Q44. "What are dbt unit tests and how do they differ from data tests?"

**🎯 What's being asked:** Do you know unit tests (dbt 1.8+) validate model *logic* on fixed inputs, vs data tests that check *actual data*.

**🤔 Why they ask it:** Unit testing is a newer, advanced practice that shows you're current and rigorous.

**💬 Say-this-in-the-interview:**
> "They test different things. A **data test** checks the *actual data* in a built model — 'is `order_id` unique in production?' A **unit test** checks my *transformation logic* against small, **fixed mock inputs and expected outputs** — like unit tests in software. I provide a handful of sample input rows and the exact output I expect, and dbt verifies my SQL logic produces it. This catches logic bugs — a wrong CASE statement, a bad join condition, an edge case like nulls or a leap-year date — **before** running on real data, and without needing the real data. So data tests validate the *data*; unit tests validate the *code*. I use unit tests especially for complex transformations with tricky logic where I want to lock in correctness."

**🧪 Example:**
```yaml
unit_tests:
  - name: test_discount_logic
    model: fct_orders
    given:
      - input: ref('stg_orders')
        rows:
          - {order_id: 1, price: 100, discount_pct: 10}
    expect:
      rows:
        - {order_id: 1, final_price: 90}
```

**📝 Notes to Remember:**
- **Unit test** = validate **logic** on **fixed mock inputs → expected outputs** (dbt 1.8+); catches code bugs before real data.
- **Data test** = validate **actual data** in built models (unique, not_null, etc.).
- Use unit tests for **complex/edge-case logic** (CASE, joins, date math, nulls).

**⚡ Impact:** Unit tests catch logic errors early and document intended behavior, independent of data. Relying only on data tests means logic bugs surface late, on real data, in production.

---

### Q45. "Explain the Write-Audit-Publish (WAP) pattern in dbt."

**🎯 What's being asked:** Do you know this advanced pattern for validating data *before* it goes live.

**🤔 Why they ask it:** WAP is how mature teams guarantee bad data never reaches consumers — a strong senior concept.

**💬 Say-this-in-the-interview:**
> "Write-Audit-Publish is a pattern that prevents bad data from ever being published to consumers. **Write:** build the new data into a **staging/temporary** location, not the production table. **Audit:** run all my tests against that staged data. **Publish:** only if every audit passes do I **swap** the staged data into the production table (a quick atomic rename/swap or partition swap). If the audit fails, production is untouched — consumers keep seeing the last good data while I investigate. In dbt I implement this with environment separation and a blue-green swap, or on Snowflake with zero-copy clones, or by testing in a temp schema and promoting on success. It's the data equivalent of not deploying code that fails CI."

**🧪 Example:**
> "We built marts into a `staging` schema, ran `dbt test` against it, and only on all-green swapped it to `prod` via a clone/rename. One day a source glitch caused a test failure — the swap was skipped, dashboards kept yesterday's correct data, and we fixed the source before publishing. Consumers never saw bad data."

**📝 Notes to Remember:**
- **WAP = Write (to staging) → Audit (run tests) → Publish (swap to prod only if green)**.
- Failed audit = **production untouched**; consumers keep last-good data.
- Implement via **blue-green swap, zero-copy clones (Snowflake), or temp-schema promotion**.
- Data equivalent of "don't ship code that fails CI."

**⚡ Impact:** WAP guarantees consumers only ever see validated data. Without it, a bad run publishes wrong data directly to dashboards before anyone notices.

---

## 19. dbt Mesh, Semantic Layer & Multi-Project

---

### Q46. "What is dbt Mesh and when would a company need it?"

**🎯 What's being asked:** Do you understand splitting one giant dbt project into governed, interconnected multiple projects.

**🤔 Why they ask it:** Mesh is the cutting-edge pattern for large orgs — knowing it signals you think at scale.

**💬 Say-this-in-the-interview:**
> "**dbt Mesh** is an architecture where instead of one massive monolithic dbt project, a company splits into **multiple smaller dbt projects** owned by different teams, that can **reference each other's models** in a governed way. Each team owns its domain (finance, marketing, product), publishes curated **`public`, contracted** models as data products, and other projects consume them via **cross-project `ref()`**. A company needs Mesh when a single project grows too big — hundreds of models, many teams stepping on each other, slow CI, unclear ownership. Mesh enables **domain ownership** (like data mesh principles), **independent deployment**, and **clear interfaces** (contracts + versions) between teams, while preserving end-to-end lineage across projects. It relies on contracts, groups/access, versions, and cross-project references working together."

**📝 Notes to Remember:**
- **dbt Mesh** = multiple governed dbt projects with **cross-project `ref()`**, vs one monolith.
- Each team owns a domain, publishes **`public` + contracted + versioned** models as **data products**.
- Needed when a project is **too big / multi-team** (slow CI, unclear ownership, collisions).
- Built on **contracts + groups/access + versions + cross-project refs**; keeps lineage across projects.

**⚡ Impact:** Mesh lets large orgs scale dbt with clear ownership and stable interfaces. A single giant monolith becomes slow, fragile, and politically tangled as teams grow.

---

### Q47. "What is the dbt Semantic Layer and what problem does it solve?"

**🎯 What's being asked:** Do you know the Semantic Layer centralizes metric definitions for consistency.

**🤔 Why they ask it:** It addresses the classic "every dashboard defines revenue differently" problem — a governance and trust topic.

**💬 Say-this-in-the-interview:**
> "The **Semantic Layer** lets me define **metrics** — like 'revenue,' 'active users,' 'churn rate' — **once, centrally**, in my dbt project (using MetricFlow), instead of re-implementing them in every BI tool and dashboard. Consumers then query these consistent metric definitions through a unified API, and the layer dynamically generates the correct SQL with the right aggregations and joins. The problem it solves is **metric inconsistency**: without it, finance's dashboard, the data team's notebook, and a Looker report each calculate 'revenue' slightly differently, and leadership gets three different numbers. The Semantic Layer makes one governed definition the single source of truth, so 'revenue' means the same thing everywhere, regardless of which tool asks."

**📝 Notes to Remember:**
- Semantic Layer = define **metrics once** (MetricFlow) → query via a unified API; SQL generated dynamically.
- Solves **metric inconsistency** ("revenue" defined differently in every tool).
- One **governed definition** = single source of truth across BI tools.

**⚡ Impact:** The Semantic Layer guarantees consistent metrics everywhere, building trust. Without it, conflicting metric definitions cause arguments over "whose number is right."

---

### Q48. "How would you migrate a large legacy SQL/stored-procedure codebase to dbt?"

**🎯 What's being asked:** Can you plan a real-world migration — strategy, layering, validation, incremental rollout.

**🤔 Why they ask it:** Many companies hire dbt engineers specifically to lead this migration. It tests strategy and pragmatism.

**💬 Say-this-in-the-interview (C-A-T-C-H):**
> "I'd do it incrementally, not big-bang. **Clarify** the scope, the warehouse, and which outputs are critical. Then:
> 1. **Set up foundations:** dbt project, connection, CI/CD, and define **sources** for the raw tables.
> 2. **Map the lineage** of the legacy procedures — inputs, outputs, dependencies — so I understand the DAG I'm recreating.
> 3. **Rebuild in layers, prioritized:** start with **staging** models for raw sources, then port logic into intermediate/marts, picking the highest-value or most-broken pipelines first. Use **`codegen`** to scaffold staging quickly.
> 4. **Validate with parallel runs:** run dbt models alongside the legacy ones and **compare outputs** (dbt's `audit_helper` package compares row-by-row) until they match exactly — this builds trust.
> 5. **Cut over gradually:** once a dbt model is validated, repoint consumers to it and retire the legacy procedure. Repeat pipeline by pipeline.
> 6. **Add tests and docs** as I go, so the new system is better than the old, not just a copy.
> The principles: incremental migration, continuous validation against the old system, and value-first prioritization — never a risky overnight switch."

**📝 Notes to Remember:**
- **Incremental, not big-bang**; foundations (project + CI + sources) first.
- **Map legacy lineage**, rebuild in **layers** (staging → marts), prioritize high-value/broken pipelines.
- **Validate with parallel runs** (`audit_helper` row-level compare) before cutover.
- **Cut over gradually**, adding **tests + docs** to leave it better than before.

**⚡ Impact:** A phased, validated migration de-risks the move and builds trust. A big-bang rewrite risks wrong numbers, broken dashboards, and a loss of confidence in the whole project.

---
---

# 🧠 PART 5 — BEHAVIORAL & ARCHITECTURE SCENARIOS

## 20. Behavioral + Architecture Scenarios

> These blend technical judgment with communication. Answer behavioral ones with **STAR** (Situation, Task, Action, Result) and design ones with **C-A-T-C-H**.

---

### Q49. "Your dbt run failed in production at 2 AM. Walk me through what you do."

**🎯 What's being asked:** Do you have a calm incident playbook for dbt failures.

**💬 Say-this-in-the-interview:**
> "I work in phases: **Triage → Diagnose → Recover → Prevent.**
> **Triage:** Check the run logs (dbt Cloud or the orchestrator) to see *which node* failed and whether it's a model error or a test failure. If it's a **test failure** in a `dbt build`, dbt already halted downstream models — so bad data hasn't propagated, which is good. I assess whether stale data on dashboards is acceptable until fixed.
> **Diagnose:** Read the error. Classify it: a **SQL/model error** (warehouse error, syntax, permissions), a **test failure** (data quality issue upstream), a **source freshness** failure (ingestion broke), or **infrastructure** (warehouse down, timeout). I check the compiled SQL and recent changes/deploys.
> **Recover:** For a transient warehouse error, re-run the failed node and its children with `dbt build --select <node>+`. For a data issue, I trace it upstream — often a source problem — fix or quarantine, then re-run. Because models are idempotent (merge/insert_overwrite), re-running is safe.
> **Prevent:** Add a test or source-freshness check that would have caught it earlier, and document the incident."

**📝 Notes to Remember:**
- Playbook: **Triage → Diagnose → Recover → Prevent.**
- `dbt build` **halts downstream on test failure** → bad data contained.
- Classify: **model error / test failure / freshness / infra.**
- Re-run safely via idempotent models + `--select <node>+`; finish with a **new guardrail**.

**⚡ Impact:** A structured response restores service fast and prevents recurrence. Panic and ad-hoc fixes spread bad data and invite repeat failures.

---

### Q50. "A stakeholder says a dbt-built dashboard number is wrong. How do you investigate?"

**🎯 What's being asked:** Can you debug using dbt's lineage and tests, and communicate well.

**💬 Say-this-in-the-interview:**
> "First I **clarify** — which metric, which number did they expect vs. see? Often it's a definition mismatch, not a bug. Then I **trace lineage** in the dbt docs DAG: dashboard → mart → intermediate → staging → source, checking each step. I look at **test results** along that path — a failing or missing test often points right at the issue. I check recent **code changes** (git blame on the model) and **source freshness** (is the raw data stale or did ingestion break?). I classify the cause: a **logic bug** (fix the model, add a unit test), a **data quality issue** (fix source, add a data test), a **definition mismatch** (align and document the metric — ideally via the Semantic Layer), or **correct-but-surprising** (a real business change). Throughout, I communicate status clearly and close by adding a test so it's caught automatically next time."

**📝 Notes to Remember:**
- **Clarify → trace lineage (docs DAG) → check tests + git blame + freshness → classify → fix + add test.**
- Many "wrong number" reports are **definition mismatches** → align + document (Semantic Layer helps).
- Lineage + tests make dbt debugging fast and systematic.

**⚡ Impact:** Systematic lineage-based debugging finds root cause fast and builds trust. Guessing erodes confidence in the data team.

---

### Q51. "How do you structure a dbt project for a large team to keep it maintainable?"

**🎯 What's being asked:** Can you design conventions and structure that scale across many contributors.

**💬 Say-this-in-the-interview:**
> "Maintainability at scale comes from **conventions enforced consistently**. I'd establish: (1) the **layered structure** — staging → intermediate → marts — with clear folder organization by source and business domain; (2) **naming conventions** (`stg_`, `int_`, `fct_`, `dim_`) so any model's role is obvious; (3) **one staging model per source table**, with all light cleanup centralized there so downstream logic is consistent; (4) **DRY logic** via macros and `dbt_utils` instead of copy-paste; (5) **tests and descriptions required** on key models (enforced in CI); (6) **`dbt_project.yml` defaults** for materializations per layer; (7) for very large multi-team setups, **groups, access modifiers, contracts**, and possibly **dbt Mesh**. I'd document these in a style guide and enforce with CI checks (SQLFluff linting, required tests). Consistency is what keeps a 500-model project navigable."

**📝 Notes to Remember:**
- **Layered structure + naming conventions + 1 staging model per source.**
- **DRY** via macros/`dbt_utils`; **tests + docs required** (enforced in CI/lint).
- **`dbt_project.yml`** defaults per layer; **groups/access/contracts/Mesh** for multi-team.
- Write a **style guide**; enforce with **CI + SQLFluff**.

**⚡ Impact:** Strong conventions keep large projects navigable and consistent. Without them, a big project becomes inconsistent spaghetti nobody can safely change.

---

### Q52. "How do you decide between doing a transformation in dbt vs upstream (ingestion) vs in the BI tool?"

**🎯 What's being asked:** Do you understand where logic belongs in the stack ("transformation placement").

**💬 Say-this-in-the-interview:**
> "My principle is: **transform in dbt (the warehouse) as the single source of truth**, and keep the other layers thin. Ingestion (EL) should stay 'dumb' — just land raw data faithfully, no business logic — so I keep full raw history and can reprocess. The **BI tool should not hold heavy business logic** either, because logic trapped in Looker or Tableau isn't version-controlled, tested, reusable, or visible to other tools — that's how 'revenue' ends up defined five different ways. So the vast majority of transformation belongs in **dbt**: it's tested, documented, version-controlled, and reusable across all consumers. The exceptions: very lightweight, presentation-only formatting can live in BI, and truly source-specific parsing might happen at ingestion. But business definitions and joins belong in dbt — ideally exposed through the Semantic Layer for consistency."

**📝 Notes to Remember:**
- **Default: transform in dbt** (tested, versioned, reusable, single source of truth).
- Keep **ingestion thin** (land raw faithfully, keep history); keep **BI light** (presentation only).
- Logic in BI = untested, unversioned, tool-siloed → inconsistent metrics.
- Business definitions → dbt + **Semantic Layer**.

**⚡ Impact:** Centralizing logic in dbt gives consistent, testable, reusable metrics. Scattering it into BI tools creates conflicting, untestable definitions.

---

### Q53. "Tell me about a time you improved a dbt project (performance, quality, or structure)." (STAR)

**🎯 What's being asked:** Can you tell a structured story showing real impact and ownership.

**💬 Say-this-in-the-interview (template to adapt):**
> - **Situation:** "Our dbt project's nightly run had crept to 3+ hours and was missing the 6 AM SLA, and CI rebuilt everything so PRs took 40 minutes."
> - **Task:** "I owned making it fast and reliable again without cutting data quality."
> - **Action:** "I parsed `run_results.json` to rank models by run time and found two full-refresh `table` models on huge fact tables dominating runtime. I converted them to **incremental** with `insert_overwrite` partitioned by date, added a 3-day lookback for late data with a `unique_key`. I also set up **Slim CI** (`state:modified+ --defer`) so PRs only built changed models, and tuned `threads`."
> - **Result:** "Nightly runtime dropped from ~3 hours to ~25 minutes, we comfortably met the SLA, CI fell from 40 minutes to ~3, and the warehouse bill dropped noticeably. I documented the incremental patterns so the team reused them."
> - **Lesson:** "Measure first, fix the few models that dominate cost, and make CI cheap so good testing habits stick."

**📝 Notes to Remember:**
- Use **STAR**; lead with a measurable problem (runtime, SLA, cost, CI time).
- Show: **measure → target the biggest offenders → incremental + Slim CI → quantified result → documented for the team.**
- Have 2–3 stories ready (a perf win, a data-quality save, a structure/migration).

**⚡ Impact:** A specific, quantified story proves you deliver real improvements. Vague answers signal you've only used dbt superficially.

---
---

# 📌 PART 6 — REFERENCE

## 21. Master Cheat Sheet & Rapid-Fire Q&A

### 🎯 The 7 phrases that make you sound senior (sprinkle these in)
1. *"Let me clarify — is this dbt Core or Cloud, and which warehouse?"* (never answer blindly)
2. *"I'd use `ref()` everywhere so the DAG and environments resolve automatically."*
3. *"For a table this big, I'd make it **incremental** with a `unique_key`."*
4. *"I'd make it **idempotent** — `merge` or `insert_overwrite` — so re-runs are safe."*
5. *"I'd add a **test** so this can't silently break again."*
6. *"In CI I'd run **Slim CI** — `state:modified+ --defer` — to only build what changed."*
7. *"There's a trade-off between **freshness, cost, and read speed** in this materialization."*

---

### 🧩 Core mental models (one line each)
- **dbt = the T in ELT** — transform in the warehouse with SELECTs + engineering discipline.
- **`ref()` builds the DAG** — automatic run order, lineage, and environment-awareness.
- **staging → intermediate → marts** — layer everything; one staging model per source.
- **Materialization = read speed vs storage/compute/freshness** — staging=view, marts=table, big facts=incremental.
- **`dbt build`** = run + test + halt downstream on failure (production default).
- **Idempotency** = `merge`/`insert_overwrite`, never blind `append`.
- **Slim CI** = `state:modified+ --defer` = only build what changed.
- **Tests + contracts + WAP** = bad data never reaches consumers.
- **Correctness → Maintainability → Performance → Cost** — touch these four in every design.

---

### ⚡ Rapid-fire one-liners (common quick questions)

| Question | Crisp answer |
|---|---|
| What is dbt? | Transformation framework (T in ELT) — build/test/doc models in the warehouse with SQL + Jinja. |
| dbt Core vs Cloud? | Free CLI you orchestrate vs managed platform (IDE, scheduler, CI, Semantic Layer). |
| What is a model? | A `.sql` file with one `SELECT` → materialized as a table/view. |
| `ref()` vs `source()`? | Reference a dbt model vs a raw ingested table; both build lineage. |
| Why `ref()`? | Auto run-order (DAG) + environment-aware schema resolution. |
| Materializations? | view (default), table, incremental, ephemeral. |
| When incremental? | Large fact/event tables where full rebuild is too slow/costly. |
| `is_incremental()`? | True on non-first/non-full-refresh runs; gates the "new rows only" filter. |
| `{{ this }}`? | The existing version of the current model (used in incremental filters). |
| Incremental strategies? | merge (upsert), append (insert only), delete+insert, insert_overwrite (partition swap). |
| `--full-refresh`? | Rebuild an incremental model from scratch. |
| `dbt run` vs `build`? | Models only vs run + test + seed + snapshot, halting downstream on test failure. |
| Generic tests? | unique, not_null, accepted_values, relationships. |
| Singular test? | One-off `.sql` test selecting the bad rows (0 rows = pass). |
| Test severity? | warn vs error, with `warn_if`/`error_if` thresholds. |
| Snapshot? | Captures history of changing source rows → SCD Type 2. |
| Snapshot strategies? | timestamp (watch updated_at) or check (compare columns). |
| Seed? | Small static CSV loaded into the warehouse; reference with `ref()`. |
| Macro? | Reusable Jinja/SQL function — DRY for SQL. |
| Jinja? | Templating (`{{ }}`/`{% %}`) compiled to plain SQL; powers ref, loops, macros. |
| Package? | Reusable dbt code (dbt_utils, dbt_expectations); `packages.yml` + `dbt deps`. |
| Hooks? | SQL run around models (pre/post-hook) or runs (on-run-start/end) — grants, logging. |
| Sources freshness? | Warn/error if raw data is stale (`loaded_at_field` + thresholds). |
| Exposure? | Declares a downstream consumer (dashboard/ML) → extends lineage. |
| `var` vs `env_var`? | Project config (in repo) vs system/secret values (out of repo). |
| Slim CI? | Build only `state:modified+` with `--defer` to prod — fast, cheap CI. |
| manifest.json? | Full project + DAG description; powers state comparison, docs, observability. |
| Model contract? | Enforced schema (columns/types) checked at build time. |
| Model versions? | Multiple versions (v1/v2) coexisting for safe breaking-change migration. |
| Unit test? | Validate logic on fixed mock inputs → expected output (dbt 1.8+). |
| WAP? | Write to staging → Audit (test) → Publish (swap) only if green. |
| dbt Mesh? | Multiple governed projects with cross-project `ref()` for large orgs. |
| Semantic Layer? | Define metrics once centrally → consistent numbers across all tools. |
| threads? | How many models dbt builds in parallel (DAG-aware). |

---

### 📋 Pre-interview confidence checklist
- [ ] I can explain **what dbt is** and that it's the **T in ELT** in one breath.
- [ ] I can explain **`ref()`** and why it builds the DAG and environment-awareness.
- [ ] I can describe the **staging → intermediate → marts** layering.
- [ ] I can choose the right **materialization** and justify it.
- [ ] I can explain **incremental models**, `is_incremental()`, `{{ this }}`, and late-data handling.
- [ ] I can list the **incremental strategies** and when each applies.
- [ ] I can describe **tests** (generic vs singular) and `dbt build` halting on failure.
- [ ] I can explain **snapshots** (SCD2) and the two strategies.
- [ ] I can explain **Jinja and macros** with an example.
- [ ] I can design a **CI/CD pipeline** with **Slim CI** (`state:modified+ --defer`).
- [ ] I can explain **contracts, versions, governance, and dbt Mesh** at a high level.
- [ ] I can explain **unit tests** and **Write-Audit-Publish**.
- [ ] I have **2–3 STAR stories** ready (perf win, data-quality save, migration).
- [ ] I always close answers with **"and I'd add a test / make it idempotent so it can't break silently."**

---

### 🗣️ How to deliver in the room (the meta-skill)
1. **Pause and clarify** before answering — "Core or Cloud? Which warehouse? How big?"
2. **State your structure first**: *"I'll cover three things — layering, materialization, and testing."*
3. **Think out loud** — they're scoring your *reasoning*, not just the final answer.
4. **Use concrete dbt terms** — `ref()`, `is_incremental()`, `unique_key`, `state:modified`, `merge` — specificity = credibility.
5. **Name trade-offs** — *"view vs table vs incremental trades freshness, cost, and read speed."*
6. **Always close with reliability** — tests, idempotency, CI, contracts.
7. If unsure, **reason from fundamentals** out loud ("dbt compiles Jinja to SQL, so…") — thinking beats bluffing.

---

> **Final mindset:** Interviewers aren't testing whether you memorized every flag. They want someone who **understands dbt's philosophy** — bringing software engineering (modularity, testing, version control, CI/CD) to analytics SQL — and can **reason through trade-offs**: which materialization, how to make it idempotent, how to test it, how to deploy it safely. Master the *frameworks* in this guide (C-A-T-C-H for design, STAR for stories, Triage→Diagnose→Recover→Prevent for incidents) and the **layering → materialization → testing → deployment** mental flow, and you can rebuild any dbt answer from first principles — calm, structured, and senior. You've got this. 💪

---





