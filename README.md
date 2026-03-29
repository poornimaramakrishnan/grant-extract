# Government Grant API Extraction Scripts — Docker Image

> **Pre-built Docker image to extract grant data from four major government research-funding APIs.**
> **Includes a DuckDB analytical warehouse and a browser-based SQL Playground.**

| Source | API | Coverage |
|--------|-----|----------|
| 🇺🇸 **NIH RePORTER** | `api.reporter.nih.gov/v2/projects/search` | 2015–present |
| 🇺🇸 **NSF Awards** | `api.nsf.gov/services/v1/awards.json` | 2015–present |
| 🇪🇺 **CORDIS (EU)** | `cordis.europa.eu/search?format=json` | 2015–present |
| 🇬🇧 **UKRI Gateway** | `gtr.ukri.org/gtr/api/projects` | 2015–present |

---

## 🔬 Live SQL Playground

> **[▶ Launch SQL Playground](https://poornimaramakrishnan.github.io/grant-extract/)**

Explore 200K+ government research grants in your browser — no server, no sign-up, 100% client-side.

- **DuckDB-WASM** — full analytical SQL engine running in your browser
- **45+ pre-built queries** across 15 categories: cross-source analytics, geographic analysis, keyword/topic mining, funding trends, strategic insights, and more
- **4 data sources** unified: NIH, NSF, CORDIS, UKRI
- **Interactive** — edit any query, run custom SQL, export results

### What can you explore?

| Category | Examples |
|----------|----------|
| 🌍 Cross-Source | Global funding comparison, top institutions worldwide, cross-border collaborators |
| 🏛️ NIH Deep Dive | Funding by institute, R01 vs R21 mechanisms, top PIs, funding by US state |
| 🔬 NSF Deep Dive | Funding by directorate, active vs expired awards, top institutions |
| 🇪🇺 CORDIS Deep Dive | EU funding by country, programme distribution, top organisations |
| 🇬🇧 UKRI Deep Dive | Funding by research council, grant status, top UK organisations |
| 📈 Trends | Year-over-year, quarterly velocity, monthly cadence, grant size distribution |
| 🔑 Keywords | AI/ML, climate, COVID-19, cancer, neuroscience, quantum computing |
| 🧠 Strategic | Emerging themes (2023 vs 2015), innovation pulse, US vs EU vs UK macro |
| 📊 Data Quality | Field completeness, duplicate detection |

---

## 🐳 Quick Start

### Pull the image

```bash
docker pull ghcr.io/poornimaramakrishnan/grant-extract:latest
```

### Run an extraction

Each extractor produces a standardised `.xlsx` file in `/app/output/` inside the container.  
Mount a local directory to retrieve the output.

```bash
# NIH RePORTER — defaults to yesterday
docker run --rm -v ./output:/app/output \
  ghcr.io/poornimaramakrishnan/grant-extract:latest \
  python -m grant_extract.scripts.nih_reporter

# NSF Awards — specific date
docker run --rm -v ./output:/app/output \
  ghcr.io/poornimaramakrishnan/grant-extract:latest \
  python -m grant_extract.scripts.nsf_awards --date 2024-06-01

# CORDIS (EU) — date range
docker run --rm -v ./output:/app/output \
  ghcr.io/poornimaramakrishnan/grant-extract:latest \
  python -m grant_extract.scripts.cordis_eu --date-from 2024-06-01 --date-to 2024-06-07

# UKRI Gateway to Research
docker run --rm -v ./output:/app/output \
  ghcr.io/poornimaramakrishnan/grant-extract:latest \
  python -m grant_extract.scripts.ukri_gateway --date 2024-06-01
```

### Using docker compose

Create a `docker-compose.yml`:

```yaml
services:
  nih:
    image: ghcr.io/poornimaramakrishnan/grant-extract:latest
    command: ["python", "-m", "grant_extract.scripts.nih_reporter"]
    volumes:
      - ./output:/app/output
      - ./logs:/app/logs

  nsf:
    image: ghcr.io/poornimaramakrishnan/grant-extract:latest
    command: ["python", "-m", "grant_extract.scripts.nsf_awards"]
    volumes:
      - ./output:/app/output
      - ./logs:/app/logs

  cordis:
    image: ghcr.io/poornimaramakrishnan/grant-extract:latest
    command: ["python", "-m", "grant_extract.scripts.cordis_eu"]
    volumes:
      - ./output:/app/output
      - ./logs:/app/logs

  ukri:
    image: ghcr.io/poornimaramakrishnan/grant-extract:latest
    command: ["python", "-m", "grant_extract.scripts.ukri_gateway"]
    volumes:
      - ./output:/app/output
      - ./logs:/app/logs
```

Then run:

```bash
docker compose run --rm nih --date 2024-06-01
docker compose run --rm nsf --date 2024-06-01
```

---

## 📊 Output Format

Every `.xlsx` file follows this fixed column template:

| Column | Description |
|--------|-------------|
| `FIRST` | PI / contact first name |
| `LAST` | PI / contact last name |
| `INSTITUTION` | Lead organisation or awardee |
| `TITLE` | Grant / project title |
| `FUNDING_AMT` | Funding amount (numeric) |
| `CURRENCY` | Currency code (`USD`, `EUR`, `GBP`) |
| `AWARD_DATE` | Award / start date (YYYY-MM-DD) |
| `SOURCE` | Data source (`NIH`, `NSF`, `CORDIS`, `UKRI`) |
| `LINK` | Direct URL to the grant record |

Files are named `<SOURCE>_YYYYMMDD.xlsx` and written to `/app/output/`.

---

## ⚙️ CLI Options

| Flag | Default | Description |
|------|---------|-------------|
| `--date YYYY-MM-DD` | yesterday | Single date to query |
| `--date-from YYYY-MM-DD` | yesterday | Start of date range |
| `--date-to YYYY-MM-DD` | same as from | End of date range |
| `--output-dir` | `output/` | Directory for `.xlsx` output |
| `--log-dir` | `logs/` | Directory for `.log` files |

---

## �️ DuckDB Analytical Warehouse

Beyond Excel extraction, this project includes a **DuckDB-powered analytical warehouse** that:

- **Ingests all 4 APIs** into normalised DuckDB tables with full schema coverage
- **Producer/consumer parallel engine** — multi-threaded fetchers with single-writer for data integrity
- **ETL tracking** — chunk-level progress with `--resume` support for fault-tolerant backfills
- **Unified view** — `grant_unified` merges all 4 sources into a single queryable view
- **Smart deduplication** — upsert-on-conflict prevents duplicate records
- **Parquet export** — for browser-based analytics via DuckDB-WASM

### Architecture

```
┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐
│ NIH API    │  │ NSF API    │  │ CORDIS API │  │ UKRI API   │
└─────┬──────┘  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘
      │               │               │               │
      └───────┬───────┴───────┬───────┘               │
              │               │                       │
        ┌─────▼─────┐  ┌─────▼─────┐           ┌─────▼─────┐
        │ Fetcher   │  │ Fetcher   │  ...       │ Fetcher   │
        │ Thread    │  │ Thread    │            │ Thread    │
        └─────┬─────┘  └─────┬─────┘           └─────┬─────┘
              │               │                       │
              └───────┬───────┴───────────────────────┘
                      │  queue.Queue (bounded)
                ┌─────▼─────┐
                │  Writer   │
                │  Thread   │──► DuckDB (single file)
                └───────────┘
                      │
                ┌─────▼─────┐
                │  Parquet  │──► docs/ (for DuckDB-WASM)
                │  Export   │
                └───────────┘
```

---

## �📄 License

MIT
