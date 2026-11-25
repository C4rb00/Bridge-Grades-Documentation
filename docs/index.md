# Bridge Grades — Technical Documentation

**Bridge Grades** is a data pipeline that measures how much each Member of the 119th U.S. Congress works *across party lines*. For every legislator it produces a **Bridge Score** (a 0–100 number) and a **Bridge Grade** (a letter: **A**, **B**, **C**, or **F**), computed separately for the House and the Senate so members are only ever compared against their own chamber's peers.

This site documents the full methodology: every data source, every preprocessing step, the composite scoring formula, and the final outputs.

---

## Flow

*The order in which the notebooks run, and the file each one produces. The stream flows top‑to‑bottom; inside every card the elbow connector (└▸) points from a notebook to the file it writes out.*

<style>
#bg-flow{--a:var(--md-primary-fg-color,#3f51b5);--fg:var(--md-default-fg-color,#1b1b1f);--mut:var(--md-default-fg-color--light,#73737c);--sf:var(--md-default-bg-color,#fff);--ok:#2e9e5b;font-size:.82rem;line-height:1.4;margin:1.1rem 0 1.7rem}
#bg-flow *{box-sizing:border-box}
#bg-flow .lg{display:flex;flex-wrap:wrap;gap:.85rem;margin:.1rem 0 1rem;font-size:.7rem;color:var(--mut)}
#bg-flow .lg b{display:inline-flex;align-items:center;gap:.35rem;font-weight:600}
#bg-flow .lg i{width:.85rem;height:.85rem;border-radius:3px;flex:0 0 auto;font-style:normal}
#bg-flow .st{border:1px solid color-mix(in srgb,var(--a) 26%,transparent);border-left:4px solid var(--a);border-radius:12px;padding:.75rem .9rem .85rem;background:color-mix(in srgb,var(--a) 5%,var(--sf))}
#bg-flow .hd{display:flex;align-items:center;gap:.55rem;margin:0 0 .5rem;font-weight:700;color:var(--fg)}
#bg-flow .hd .n{flex:0 0 auto;width:1.55rem;height:1.55rem;border-radius:50%;background:var(--a);color:#fff;display:flex;align-items:center;justify-content:center;font-size:.8rem;font-weight:700}
#bg-flow .hd .s{font-weight:400;color:var(--mut);font-size:.72rem}
#bg-flow .nd{margin:.5rem 0;padding:.5rem .6rem .55rem;border:1px solid color-mix(in srgb,var(--a) 20%,transparent);border-radius:10px;background:color-mix(in srgb,var(--a) 3%,var(--sf))}
#bg-flow .nd:first-of-type{margin-top:.15rem}
#bg-flow .nb{display:inline-flex;align-items:center;gap:.45rem;max-width:100%;padding:.4rem .6rem;border-radius:8px;background:color-mix(in srgb,var(--a) 16%,var(--sf));border:1px solid color-mix(in srgb,var(--a) 50%,transparent);border-left:3px solid var(--a)}
#bg-flow .nb .run{flex:0 0 auto;width:1.1rem;height:1.1rem;border-radius:4px;background:var(--a);color:#fff;display:inline-flex;align-items:center;justify-content:center;font-size:.55rem}
#bg-flow .tag{font-size:.57rem;letter-spacing:.07em;text-transform:uppercase;color:var(--mut);font-weight:700;flex:0 0 auto}
#bg-flow .nm{font-family:var(--md-code-font,ui-monospace,monospace);font-size:.72rem;color:var(--fg);word-break:break-word;line-height:1.3}
#bg-flow .ol{margin:.4rem 0 0 .3rem;font-size:.57rem;letter-spacing:.07em;text-transform:uppercase;color:var(--mut);font-weight:700}
#bg-flow .outs{position:relative;margin:.05rem 0 0 1.05rem;padding-left:1rem}
#bg-flow .outs::before{content:"";position:absolute;left:0;top:-.1rem;bottom:.95rem;width:2px;border-radius:2px;background:color-mix(in srgb,var(--a) 55%,transparent)}
#bg-flow .o{position:relative;display:flex;align-items:center;padding:.28rem 0}
#bg-flow .o::before{content:"";position:absolute;left:-1rem;top:50%;width:.72rem;height:2px;border-radius:2px;background:color-mix(in srgb,var(--a) 55%,transparent)}
#bg-flow .o::after{content:"\25B8";position:absolute;left:-.3rem;top:50%;transform:translateY(-55%);color:var(--a);font-size:.74rem;line-height:1}
#bg-flow .f,#bg-flow .fin,#bg-flow .ins{display:inline-flex;align-items:center;gap:.4rem;max-width:100%;padding:.3rem .55rem;border-radius:7px}
#bg-flow .f{background:var(--sf);border:1px dashed color-mix(in srgb,var(--fg) 40%,transparent)}
#bg-flow .fin{background:color-mix(in srgb,var(--ok) 13%,var(--sf));border:1px solid color-mix(in srgb,var(--ok) 55%,transparent)}
#bg-flow .ins{background:var(--sf);border:1px dashed color-mix(in srgb,var(--fg) 28%,transparent);color:var(--mut)}
#bg-flow .feed{margin:.05rem 0 .05rem .55rem;color:var(--mut);font-size:.85rem;line-height:1}
#bg-flow .badge{font-size:.55rem;letter-spacing:.03em;text-transform:uppercase;font-weight:700;color:#fff;background:var(--a);border-radius:10px;padding:.08rem .42rem;white-space:nowrap}
#bg-flow .note{font-size:.68rem;color:var(--mut);margin:.55rem 0 0}
#bg-flow .cn{position:relative;width:6px;height:34px;margin:4px auto;border-radius:6px;overflow:hidden;background:color-mix(in srgb,var(--a) 22%,transparent)}
#bg-flow .cn::after{content:"";position:absolute;left:0;right:0;top:-24px;bottom:-24px;background:repeating-linear-gradient(180deg,transparent 0 7px,color-mix(in srgb,var(--a) 85%,#fff) 7px 15px);animation:bgfl 1.15s linear infinite}
#bg-flow .cn::before{content:"\25BC";position:absolute;left:50%;bottom:-15px;transform:translateX(-50%);color:var(--a);font-size:.72rem;z-index:1}
@keyframes bgfl{to{transform:translateY(15px)}}
@media (prefers-reduced-motion:reduce){#bg-flow .cn::after{animation:none}}
</style>
<div id="bg-flow">
<div class="lg"><b><i style="background:color-mix(in srgb,var(--a) 50%,var(--sf));border:1px solid var(--a)"></i>Notebook — runs &amp; writes a file</b><b><i style="background:var(--sf);border:1px dashed currentColor"></i>Temp / intermediate file</b><b><i style="background:var(--ok)"></i>Final output</b></div>
<div class="st">
<div class="hd"><span class="n">1</span>Rosters &amp; data sources <span class="s">— build the member list and metadata</span></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Get_Current_Members.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">roster</span><span class="nm">119th_Congress_YYYYMMDD.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Get_Current_Legislators_Information.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">metadata</span><span class="nm">plural_current_legislators_with_bioguide.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Get_Retired_Legislators_Information.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">metadata</span><span class="nm">plural_retired_legislators_with_bioguide.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Get_Classifications_Bills.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">bills</span><span class="nm">US_119_bills_processed_YYYY-MM-DD.csv</span></span></div></div></div>
<p class="note">Plus the attendance file <span class="nm">profiles.csv</span>, used later to filter low‑attendance members.</p>
</div>
<div class="cn"></div>
<div class="st">
<div class="hd"><span class="n">2</span>Per‑source preprocessing <span class="s">— one notebook per signal, run in any order</span></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Preprocessing_A_V1.4.0_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_A_v1.4.0_2026_06_02.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Preprocessing_B_V1.7.0_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_B_v1.7.0_2026_06_02.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Preprocessing_C_V1.0.1_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_C_v1.0.1_2026_06_02.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Preprocessing_D_V1.0.1_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_D_v1.0.1_2026_06_02.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Preprocessing_E_V1.0.1_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_E_v1.0.1_2026_06_02.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Preprocessing_F_V1.0.1_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_F_v1.0.1_2026_06_02.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Preprocessing_M_V1.0.0_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_M_House_v1.0.0_2026_06_02.csv</span></span></div><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_M_Senate_v1.0.0_2026_06_02.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Preprocessing_N_V1.0.0_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_N_House_v1.0.0_2026_06_02.csv</span></span></div><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_N_Senate_v1.0.0_2026_06_02.csv</span></span></div></div></div>
<div class="nd"><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Preprocessing_P_V1.0.0_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="f"><span class="tag">temp</span><span class="nm">Temp_P_v1.0.0_2026_06_02.csv</span></span></div></div></div>
</div>
<div class="cn"></div>
<div class="st">
<div class="hd"><span class="n">3</span>Main calculation <span class="s">— merges every Temp file into the score &amp; grade</span></div>
<div class="nd"><div class="ins"><span class="tag">merges</span><span class="nm">Temp_A … Temp_P · 119th_Congress.csv · profiles.csv</span></div><div class="feed">&#9662;</div><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Bridge_Grades_Calculation_V1.5.2_119.2.ipynb</span></div><div class="ol">produces — 3 Excel layouts</div><div class="outs"><div class="o"><span class="fin"><span class="tag">output</span><span class="nm">house_scores_119_YYYY-MM-DD.xlsx</span></span></div><div class="o"><span class="fin"><span class="tag">output</span><span class="nm">senate_scores_119_YYYY-MM-DD.xlsx</span></span></div><div class="o"><span class="fin"><span class="tag">output</span><span class="nm">congress_scores_119_separate_sheets_YYYY-MM-DD.xlsx</span></span></div><div class="o"><span class="fin"><span class="tag">output</span><span class="nm">congress_scores_119_single_sheet_YYYY-MM-DD.xlsx</span><span class="badge">feeds scorecards</span></span></div></div></div>
</div>
<div class="cn"></div>
<div class="st">
<div class="hd"><span class="n">4</span>Scorecards <span class="s">— compares single‑sheet snapshots across dates</span></div>
<div class="nd"><div class="ins"><span class="tag">reads</span><span class="nm">congress_scores_119_single_sheet_*.xlsx  (several dates)</span></div><div class="feed">&#9662;</div><div class="nb"><span class="run">&#9654;</span><span class="tag">notebook</span><span class="nm">Snapshot_Scorecard_2026_06_02.ipynb</span></div><div class="ol">produces</div><div class="outs"><div class="o"><span class="fin"><span class="tag">final</span><span class="nm">Bridge_Grades_Snapshot_Scorecard_2026_06_02.xlsx</span></span></div></div></div>
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
