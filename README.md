# Bridge Grades Documentation

Technical documentation for the **Bridge Grades** pipeline — a Python project that scores how much each Member of the 119th U.S. Congress works across party lines, producing a **Bridge Score** (0–100) and a **Bridge Grade** (A/B/C/F) for the House and Senate.

📖 **Published site:** https://C4rb00.github.io/Bridge-Grades-Documentation/

## What's here

This repository contains the documentation site and the source notebooks. The site is built from the Markdown in [`docs/`](docs/) with [MkDocs](https://www.mkdocs.org/) and the [Material](https://squidfunk.github.io/mkdocs-material/) theme, and deployed to GitHub Pages automatically on every push to `main`. The Python notebooks that each page documents live in [`docs/notebooks/`](docs/notebooks/) and are rendered into the site's **Notebooks** section via [mkdocs-jupyter](https://github.com/danielfrg/mkdocs-jupyter). Large data files (`.xlsx`, `.csv`) are intentionally kept out of the repository.

| Section | Contents |
|---------|----------|
| **Overview** | Project summary, technical specs, and data sources |
| **Main Calculation** | How sources are merged, normalized, weighted, and turned into the score and grade |
| **Scored Sources (A–F)** | One page per scored signal |
| **Bonus Sources (M, N, P)** | The contextual adjustments |
| **Data Sources & Rosters** | Supporting notebooks (member roster, legislator metadata, bill classifications) |
| **Scorecards** | Comparing snapshots across dates/versions |

## Source notebooks

The Python (Jupyter) notebooks documented by this site are in [`docs/notebooks/`](docs/notebooks/) and appear in the site's **Notebooks** section:

| Folder | Notebooks |
|--------|-----------|
| `docs/notebooks/pipeline/` | Main Bridge Grades calculation |
| `docs/notebooks/sources/` | Preprocessing for sources A–F, M, N, P |
| `docs/notebooks/data-sources/` | Member roster, legislator metadata, bill classifications |
| `docs/notebooks/scorecards/` | Snapshot comparison |

## Build locally

```bash
pip install -r requirements.txt
mkdocs serve
```

Then open http://127.0.0.1:8000/.

## Deployment

A GitHub Actions workflow ([`.github/workflows/deploy.yml`](.github/workflows/deploy.yml)) builds the site and publishes it to GitHub Pages. No manual steps are required after the first push — the workflow enables Pages automatically.
