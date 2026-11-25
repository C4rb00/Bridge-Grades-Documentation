# Bridge Grades — Technical Documentation

**Bridge Grades** is a data pipeline that measures how much each Member of the 119th U.S. Congress works *across party lines*. For every legislator it produces a **Bridge Score** (a 0–100 number) and a **Bridge Grade** (a letter: **A**, **B**, **C**, or **F**), computed separately for the House and the Senate so members are only ever compared against their own chamber's peers.

This site documents the full methodology: every data source, every preprocessing step, the composite scoring formula, and the final outputs.

---

## Flow

*The order in which the notebooks run, and the files each one produces. The stream flows top‑to‑bottom — follow the current from the rosters down to the final scorecard.*

<style>
#bg-flow{--bgf-accent:var(--md-primary-fg-color,#3f51b5);--bgf-fg:var(--md-default-fg-color,#1b1b1f);--bgf-muted:var(--md-default-fg-color--light,#73737c);--bgf-surface:var(--md-default-bg-color,#fff);--bgf-out:#2e9e5b;font-size:.82rem;margin:1.1rem 0 1.6rem}
#bg-flow *{box-sizing:border-box}
#bg-flow .bgf-legend{display:flex;flex-wrap:wrap;gap:.75rem;margin:.1rem 0 1rem;font-size:.7rem;color:var(--bgf-muted)}
#bg-flow .bgf-legend b{display:inline-flex;align-items:center;gap:.35rem;font-weight:600}
#bg-flow .bgf-dot{width:.8rem;height:.8rem;border-radius:3px;display:inline-block;flex:0 0 auto}
#bg-flow .bgf-stage{background:var(--bgf-surface);background:color-mix(in srgb,var(--bgf-accent) 5%,var(--bgf-surface));border:1px solid color-mix(in srgb,var(--bgf-accent) 28%,transparent);border-left:4px solid var(--bgf-accent);border-radius:10px;padding:.7rem .9rem .8rem}
#bg-flow .bgf-head{display:flex;align-items:center;gap:.55rem;margin:0 0 .55rem;font-weight:700;color:var(--bgf-fg)}
#bg-flow .bgf-num{flex:0 0 auto;width:1.5rem;height:1.5rem;border-radius:50%;background:var(--bgf-accent);color:#fff;display:flex;align-items:center;justify-content:center;font-size:.78rem}
#bg-flow .bgf-sub{font-weight:400;color:var(--bgf-muted);font-size:.72rem}
#bg-flow .bgf-lane{display:flex;align-items:center;flex-wrap:wrap;gap:.35rem .45rem;padding:.22rem 0}
#bg-flow .bgf-nb,#bg-flow .bgf-file,#bg-flow .bgf-fin{display:inline-flex;flex-direction:column;gap:.12rem;padding:.32rem .55rem;border-radius:8px;line-height:1.25}
#bg-flow .bgf-nb{background:var(--bgf-surface);background:color-mix(in srgb,var(--bgf-accent) 13%,var(--bgf-surface));border:1px solid color-mix(in srgb,var(--bgf-accent) 45%,transparent)}
#bg-flow .bgf-file{background:var(--bgf-surface);border:1px dashed color-mix(in srgb,var(--bgf-fg) 38%,transparent)}
#bg-flow .bgf-fin{background:var(--bgf-surface);background:color-mix(in srgb,var(--bgf-out) 14%,var(--bgf-surface));border:1px solid color-mix(in srgb,var(--bgf-out) 55%,transparent)}
#bg-flow .bgf-tag{font-size:.58rem;letter-spacing:.07em;text-transform:uppercase;color:var(--bgf-muted);font-weight:700}
#bg-flow .bgf-name{font-family:var(--md-code-font,monospace);font-size:.72rem;color:var(--bgf-fg);white-space:nowrap}
#bg-flow .bgf-arrow{color:var(--bgf-accent);font-weight:700;flex:0 0 auto}
#bg-flow .bgf-note{font-size:.68rem;color:var(--bgf-muted);margin:.35rem 0 0}
#bg-flow .bgf-conn{position:relative;width:5px;height:30px;margin:3px auto;border-radius:5px;overflow:hidden;background:color-mix(in srgb,var(--bgf-accent) 22%,transparent)}
#bg-flow .bgf-conn::after{content:"";position:absolute;left:0;right:0;top:-24px;bottom:-24px;background:repeating-linear-gradient(180deg,transparent 0 7px,color-mix(in srgb,var(--bgf-accent) 85%,#fff) 7px 15px);animation:bgf-flow 1.15s linear infinite}
#bg-flow .bgf-conn::before{content:"\25BC";position:absolute;left:50%;bottom:-15px;transform:translateX(-50%);color:var(--bgf-accent);font-size:.7rem;z-index:1}
@keyframes bgf-flow{to{transform:translateY(15px)}}
@media (prefers-reduced-motion:reduce){#bg-flow .bgf-conn::after{animation:none}}
</style>
<div id="bg-flow">
<div class="bgf-legend"><b><i class="bgf-dot" style="background:color-mix(in srgb,var(--bgf-accent) 45%,var(--bgf-surface));border:1px solid var(--bgf-accent)"></i>Notebook (.ipynb)</b><b><i class="bgf-dot" style="background:var(--bgf-surface);border:1px dashed currentColor"></i>Temp / intermediate file</b><b><i class="bgf-dot" style="background:var(--bgf-out)"></i>Final output</b></div>
<div class="bgf-stage">
<div class="bgf-head"><span class="bgf-num">1</span>Rosters &amp; data sources <span class="bgf-sub">— build the member list and metadata</span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Get_Current_Members.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">roster</span><span class="bgf-name">119th_Congress_YYYYMMDD.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Get_Current_Legislators_Information.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">metadata</span><span class="bgf-name">plural_current_legislators_with_bioguide.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Get_Retired_Legislators_Information.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">metadata</span><span class="bgf-name">plural_retired_legislators_with_bioguide.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Get_Classifications_Bills.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">bills</span><span class="bgf-name">US_119_bills_processed_YYYY-MM-DD.csv</span></span></div>
<p class="bgf-note">Plus the attendance file <span class="bgf-name">profiles.csv</span>, used later to filter low‑attendance members.</p>
</div>
<div class="bgf-conn"></div>
<div class="bgf-stage">
<div class="bgf-head"><span class="bgf-num">2</span>Per‑source preprocessing <span class="bgf-sub">— one notebook per signal, run in any order</span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Preprocessing_A_V1.4.0_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_A_v1.4.0_2026_06_02.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Preprocessing_B_V1.7.0_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_B_v1.7.0_2026_06_02.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Preprocessing_C_V1.0.1_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_C_v1.0.1_2026_06_02.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Preprocessing_D_V1.0.1_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_D_v1.0.1_2026_06_02.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Preprocessing_E_V1.0.1_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_E_v1.0.1_2026_06_02.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Preprocessing_F_V1.0.1_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_F_v1.0.1_2026_06_02.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Preprocessing_M_V1.0.0_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_M_House_v1.0.0_2026_06_02.csv</span></span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_M_Senate_v1.0.0_2026_06_02.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Preprocessing_N_V1.0.0_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_N_House_v1.0.0_2026_06_02.csv</span></span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_N_Senate_v1.0.0_2026_06_02.csv</span></span></div>
<div class="bgf-lane"><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Preprocessing_P_V1.0.0_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-file"><span class="bgf-tag">temp</span><span class="bgf-name">Temp_P_v1.0.0_2026_06_02.csv</span></span></div>
</div>
<div class="bgf-conn"></div>
<div class="bgf-stage">
<div class="bgf-head"><span class="bgf-num">3</span>Main calculation <span class="bgf-sub">— merges every Temp file into the score &amp; grade</span></div>
<div class="bgf-lane"><span class="bgf-file"><span class="bgf-tag">inputs</span><span class="bgf-name">Temp_A … Temp_P + roster + profiles.csv</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Bridge_Grades_Calculation_V1.5.2_119.2.ipynb</span></span></div>
<div class="bgf-lane"><span class="bgf-arrow">&darr;</span><span class="bgf-fin"><span class="bgf-tag">output</span><span class="bgf-name">house_scores_119_YYYY-MM-DD.xlsx</span></span><span class="bgf-fin"><span class="bgf-tag">output</span><span class="bgf-name">senate_scores_119_YYYY-MM-DD.xlsx</span></span></div>
<div class="bgf-lane"><span class="bgf-fin"><span class="bgf-tag">output</span><span class="bgf-name">congress_scores_119_separate_sheets_YYYY-MM-DD.xlsx</span></span><span class="bgf-fin"><span class="bgf-tag">output &rarr; scorecards</span><span class="bgf-name">congress_scores_119_single_sheet_YYYY-MM-DD.xlsx</span></span></div>
</div>
<div class="bgf-conn"></div>
<div class="bgf-stage">
<div class="bgf-head"><span class="bgf-num">4</span>Scorecards <span class="bgf-sub">— compares single‑sheet snapshots across dates</span></div>
<div class="bgf-lane"><span class="bgf-file"><span class="bgf-tag">snapshots</span><span class="bgf-name">congress_scores_119_single_sheet_*.xlsx</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-nb"><span class="bgf-tag">notebook</span><span class="bgf-name">Snapshot_Scorecard_2026_06_02.ipynb</span></span><span class="bgf-arrow">&rarr;</span><span class="bgf-fin"><span class="bgf-tag">final</span><span class="bgf-name">Bridge_Grades_Snapshot_Scorecard_2026_06_02.xlsx</span></span></div>
</div>
</div>

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
