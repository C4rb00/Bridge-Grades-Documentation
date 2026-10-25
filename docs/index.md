# Bridge Grades — Technical Documentation

**Bridge Grades** is a data pipeline that measures how much each Member of the 119th U.S. Congress works *across party lines*. For every legislator it produces a **Bridge Score** (a 0–100 number) and a **Bridge Grade** (a letter: **A**, **B**, **C**, or **F**), computed separately for the House and the Senate so members are only ever compared against their own chamber's peers.

This site documents the full methodology: every data source, every preprocessing step, the composite scoring formula, and the final outputs.

---

## What this project is

The pipeline ingests separate, independently-measured signals of bipartisan behavior — bill authorship, cross-party cosponsorship, four communications metrics, district partisan lean, member ideology, and Problem Solvers Caucus membership — normalizes each one, combines them with fixed weights, applies three contextual bonuses, and converts the result into a published score and letter grade.

Each stage is implemented as a **Jupyter notebook** and documented here one-to-one.

!!! note "Parameter controls under review"
    Several tunable values in the methodology — the 20% attendance threshold, the source weights (3 / 2 / 1), the PVI cap (15), the bonus scaling factors (0.38 / 0.05 / 0.01), and the grade thresholds — are **parameter controls currently under review by political-domain subject-matter experts**. This documentation describes *what each control does*, not a justification for its exact value.

---

## Technical specifications

| Item | Detail |
|------|--------|
| **Language** | Python 3 |
| **Environment** | Jupyter notebooks (`.ipynb`), originally run in Google Colab |
| **Core libraries** | `pandas`, `numpy` (data handling); `scipy.stats.norm` (Normal-CDF normalization); `matplotlib` (sanity-check charts); `openpyxl` (Excel I/O); `requests` / `zipfile` / `xml` (live bill-data download and parsing) |
| **Normalization** | Per-source, per-chamber percentile via the Normal cumulative distribution function (CDF) |
| **Storage** | Shared Google Drive ("Data Archive") for intermediate and final files |
| **Final output format** | Excel workbooks (`.xlsx`) in three layouts |
| **Scope** | 119th U.S. Congress — House and Senate scored separately |

---

## Data sources

The score is built from several independent inputs. Sources **A–F** are scored and weighted; sources **M, N, P** act as contextual bonuses.

| Source | Signal | Origin | Role |
|--------|--------|--------|------|
| **A** | Bills authored that drew a cross-party cosponsor | [govinfo.gov](https://www.govinfo.gov/bulkdata/BILLSTATUS/119) BILLSTATUS bulk data | Scored · weight **3** |
| **B** | Cross-party bills the member cosponsored | govinfo.gov BILLSTATUS bulk data | Scored · weight **2** |
| **C** | Communications supportive of the opposite party (count) | American Political Pulse (APP) | Scored · weight **1** |
| **D** | Same as C, as a percentage of the member's communications | American Political Pulse (APP) | Scored · weight **1** |
| **E** | Communications classified as personal attacks (count) | American Political Pulse (APP) | Scored **inversely** · weight **1** |
| **F** | Personal attacks as a percentage | American Political Pulse (APP) | Scored **inversely** · weight **1** |
| **M** | District partisan lean (Cook PVI) | Cook Political Report | Bonus (capped at 15) |
| **N** | Member ideology / distance from center | VoteView (`nominate_dim1`) | Bonus |
| **P** | Problem Solvers Caucus membership (House only) | PSC membership list | Bonus |

Supporting metadata includes the **119th Congress member roster** and **attendance** records (from American Political Pulse `profiles.csv`), used to filter out members with too little of a track record to score fairly.

---

## How the score is built

```
Preprocessing (per source)          Main calculation                 Outputs
─────────────────────────           ────────────────                 ───────
A B C D E F  ──► Temp CSVs ──►  merge ► filter (attendance ≥ 20%)
M N P        ──► Temp CSVs ──►  normalize each source (Normal CDF, 0–100)
                               weighted sum  →  T_score
                               + PVI bonus (M) + ideology bonus (N)
                                 + caucus bonus (P)  →  U_score
                               normalize U_score  →  Bridge_Score (0–100)
                               thresholds        →  Bridge_Grade (A/B/C/F)   ──► Excel scorecards
```

1. **Preprocess each source** into a per-legislator "Temp" CSV.
2. **Filter** out members below the 20% attendance threshold.
3. **Normalize** every scored source (A–F) to a 0–100 percentile within its chamber. Sources E and F (personal attacks) are flipped so more attacks lower the score.
4. **Weighted sum** of the normalized sources → `T_score`.
5. **Bonuses** for partisan district (M), distance from center (N), and caucus membership (P) → `U_score`.
6. **Normalize** `U_score` into the published **Bridge Score**, then apply distribution-based thresholds to assign the **Bridge Grade**.

The grade bands, relative to the chamber's distribution:

- **A** — above `mean + std` (clear top tier)
- **B** — above the median, up to `mean + std`
- **C** — below the median, down to `mean − std`
- **F** — the lowest tier

---

## How to read this documentation

- **[Main Calculation](pipeline/main-calculation.md)** — the heart of the pipeline: how every source is merged, normalized, weighted, bonused, and turned into the final score and grade.
- **[Scored Sources (A–F)](sources/source-a.md)** — one page per scored signal, with its data origin and exact preprocessing logic.
- **[Bonus Sources (M, N, P)](sources/source-m.md)** — the contextual adjustments.
- **[Data Sources & Rosters](data-sources/current-members.md)** — the supporting notebooks that build the member roster, legislator metadata, and bill classifications.
- **[Scorecards](scorecards/snapshot-scorecard.md)** — how snapshots from different dates/versions are compared over time.

Every page mirrors one Jupyter notebook and follows the same four-part structure: **Purpose · Inputs · Process · Outputs**.

The source notebooks themselves are rendered in the **Notebooks** section of this site (with their code and outputs), and live in the [`docs/notebooks/` folder of the repository](https://github.com/C4rb00/Bridge-Grades-Documentation/tree/main/docs/notebooks), organized the same way as these pages.
