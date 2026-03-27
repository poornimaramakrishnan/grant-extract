# Government Grant API Extraction Scripts — Docker Image

> **Pre-built Docker image to extract grant data from four major government research-funding APIs.**

| Source | API |
|--------|-----|
| 🇺🇸 **NIH RePORTER** | `api.reporter.nih.gov/v2/projects/search` |
| 🇺🇸 **NSF Awards** | `api.nsf.gov/services/v1/awards.json` |
| 🇪🇺 **CORDIS (EU)** | `cordis.europa.eu/search?format=json` |
| 🇬🇧 **UKRI Gateway** | `gtr.ukri.org/gtr/api/projects` |

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

## 📄 License

MIT
