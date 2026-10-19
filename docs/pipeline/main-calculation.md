# Bridge_Grades_Calculation_V1.5.2_119.2.ipynb

> **Pipeline stage:** MAIN calculation — merges every preprocessed "Temp" CSV, filters and normalizes the data, computes the composite score, assigns letter grades, and exports the final Excel scorecards.
> **Path:** `V1_Bridge_Grades/BridgeGrades_2026_06_02/Bridge_Grades_Calculation_V1.5.2_119.2.ipynb`

## 1. Purpose

This is the heart of the Bridge Grades pipeline. Every other preprocessing notebook in the project exists to feed this one. Its job is to take the separate, already-cleaned metrics for each Member of the 119th U.S. Congress — bill authorship, cross-party cosponsorship, the four communications metrics, district partisan lean, member ideology, and Problem Solvers Caucus membership — and combine them into a single **Bridge Score** (a 0–100 number) and a **Bridge Grade** (a letter: A, B, C, or F) that measure how much each legislator works across party lines.

The notebook does this separately for the House and the Senate, because the two chambers have very different sizes and behavior and should be compared only against their own peers. At the end it writes out three Excel files in different layouts so downstream consumers (the website, the scorecards, analysts) can use whichever is most convenient.

## 2. Inputs (what it consumes)

All inputs are CSV files produced by the upstream preprocessing notebooks, stored on a shared Google Drive. The base folders are:

- `Output_Files/Temp_Files/` — the per-source "Temp" CSVs
- `Input_Files/Snapshot_Raw_2026_06_02/Raw_Others_2026_06_02/` — supporting metadata

Specific files read (all dated `2026_06_02`):

- **Master member roster** — `119th_Congress_20260602.csv`. Provides the base list of legislators with `bioguide_id`, `Name`, `Chamber`, `State`, `District`, `Party`, etc. This is the spine that every metric is merged onto.
- **Attendance** — `profiles.csv` (from American Political Pulse), columns `full_name`, `bioguide_id`, `attendance_total`, `attendance_max`.
- **Source A** (`Temp_A_...csv`) — `primary_bioguide_id`, `num_bills_with_cross_party_cosponsors`. Bills a legislator authored that drew at least one opposite-party cosponsor.
- **Source B** (`Temp_B_...csv`) — `bioguide_id`, `num_cross_party_cosponsored_bills`. Cross-party bills the legislator cosponsored.
- **Source C** (`Temp_C_...csv`) — `bioguide_id`, `outcome_bipartisanship`. Count of communications supportive of the opposite party (APP data).
- **Source D** (`Temp_D_...csv`) — `bioguide_id`, `outcome_bipartisanship_pct`. The same as C but as a percentage of all that member's communications.
- **Source E** (`Temp_E_...csv`) — `bioguide_id`, `attack_personal`. Count of communications classified as personal attacks (scored inversely).
- **Source F** (`Temp_F_...csv`) — `bioguide_id`, `attack_personal_pct`. Personal attacks as a percentage (scored inversely).
- **Source M** (`Temp_M_House_...csv`, `Temp_M_Senate_...csv`) — `bioguide_id` plus a Cook Political PVI number column. District partisan lean.
- **Source N** (`Temp_N_House_...csv`, `Temp_N_Senate_...csv`) — `bioguide_id`, `nominate_dim1`, `ideology_dist`. VoteView ideology and a precomputed distance-from-center.
- **Source P** (`Temp_P_...csv`) — `bioguide_id` only: the list of Problem Solvers Caucus members (House only).

Before doing any work the notebook checks that every required file exists (raising an error if not) and validates that the `weights` dictionary keys exactly match the set of scored sources `{A, B, C, D, E, F}`. This fail-fast step prevents a silent miscalculation from a missing or mislabeled input.

## 3. Process (transformations & rationale)

### Step 0 — Global parameters

All tunable numbers are set at the top in one place:

- `att_pct = 0.2` — the minimum attendance proportion (20%) required to be scored.
- `weights = {A:3, B:2, C:1, D:1, E:1, F:1}` — how much each source counts toward the composite.
- `bonuses` — the PVI cap (15) and the three bonus scaling factors (M: 0.38, N: 0.05, P: 0.01).

*Note: these specific values (the 20% threshold, the 3/2/1 weights, the PVI cap of 15, and the 0.38 / 0.05 / 0.01 scaling factors) are parameter controls currently under review by political-domain subject-matter experts. The documentation describes what each control does, not a justification for the exact value.*

### Step 1 — Build base tables and filter on attendance

The master roster is split into `house_final` and `senate_final`. Then attendance is applied:

- Each member's attendance percentage is computed as `attendance_total / attendance_max` (guarded so a zero denominator becomes 0 rather than an error).
- Members below the `att_pct` (20%) threshold are removed from both chambers. In practice these are almost entirely non-voting members from U.S. territories, who also show up as having no attendance record at all.
- A short hard-coded list of recently departed legislators (`recent_legislators`) is also dropped.

**Why:** A legislator who barely shows up has too little of a track record to score fairly, and including them would distort the peer comparisons used for normalization. The 20% cutoff is one of the parameter controls under expert review.

### Step 2 — Merge and normalize each source (A–F)

Each source is processed the same way, once for the House and once for the Senate:

1. **Merge** the source's metric onto the chamber table by `bioguide_id` (a left join, so every member is kept).
2. **Fill missing values with 0.** A missing count genuinely means "no activity" (e.g., the member authored no cross-party bills), not "unknown." Zero is the correct representation.
3. **Normalize to a 0–100 percentile** using the Normal cumulative distribution function (CDF): the column's own mean and standard deviation are computed, and each member's value is converted to `norm.cdf(value, mean, std) * 100`. This turns a raw count (which can be on very different scales for different sources) into a comparable "where does this member sit relative to peers" percentile.
4. **Store** a `{X}_norm` column (the percentile) and a `{X}_weight` column.

**Inverse scoring for E and F.** Sources E and F measure *personal attacks*, which are bad. For these the normalized value is flipped: `1 - norm.cdf(...) * 100`, so that more attacks produce a lower score. (This expression mixes scales and is a known quirk of the current version, but the intent is clear: attacks should hurt, not help.)

**Why normalize per source and per chamber:** raw counts are not comparable across sources (a bill count and a communications percentage live on different scales), and chamber sizes differ. Converting each to a within-chamber percentile via the CDF puts everything on the same 0–100 footing before combining.

### Step 3 — Attach the bonus sources (M, N, P)

These are not scored on their own; they adjust the combined score later.

- **Source M (Cook PVI):** the raw PVI number is read and **capped at 15** (`M_Cook_PVI_Cap`). The cap prevents extremely safe districts from producing an unbounded bonus. The cap value of 15 is a parameter control under expert review.
- **Source N (VoteView ideology):** `nominate_dim1` and the precomputed `ideology_dist` (distance from the partisan center) are merged in. Members closer to the center will earn a larger bonus.
- **Source P (Problem Solvers Caucus):** members on the PSC list get `P_flag = 1`, everyone else `0`. The PSC is House-only, so all Senators get `P_flag = 0`.

### Step 4 — The composite score (`cal_score` function)

This single function produces the final numbers. In order:

1. **Weighted total `score_T`.** Sum each source's percentile times its weight: `score_T = Σ (X_norm × weight_X)` over A–F.

2. **PVI bonus (`bonus_m`).** `bonus_m = score_T × (1 + (PVI_cap/100) × 0.38) − score_T`, which simplifies to `score_T × (PVI_cap/100) × 0.38`. It is a percentage uplift on `score_T` scaled by how partisan the district is — the more lopsided the district, the more credit a member gets for any bipartisanship they show. Clipped to be non-negative.

3. **Ideology bonus (`bonus_n`).** `bonus_n = score_T × 0.05 × N_ideology_dist`. An uplift proportional to how far the member sits from the center. Clipped to be non-negative.

4. **Caucus bonus (`bonus_p`).** `bonus_p = ceil(0.01 × max(score_T) × P_flag)`. A small fixed bump (rounded up to a whole number) for Problem Solvers Caucus members, sized off the highest `score_T` in the chamber so it is comparable across the cohort. Non-members get 0.

5. **Adjusted score `score_U`.** `score_U = score_T + bonus_m + bonus_n + bonus_p`.

6. **Normalize to the Bridge Score (`norm_U`).** `score_U` is converted to a 0–100 percentile via the Normal CDF using the cohort's mean and standard deviation. This is the published **Bridge_Score**.

7. **Assign a letter grade** from `score_U` and its distribution:
   - **A** if `score_U > mean + std`
   - **B** if `median < score_U ≤ mean + std`
   - **C** if `mean − std < score_U ≤ median`
   - **F** otherwise

   In plain terms: A is the clear top tier, B is above the typical member (above the median), C is below typical but not at the bottom, and F is the lowest tier.

*The bonus scaling factors (0.38, 0.05, 0.01), the PVI cap (15), and the grade-threshold rules are all parameter controls currently under review by political-domain subject-matter experts.*

`cal_score` is run once for the House and once for the Senate, producing `house_final_test` and `senate_final_test`.

### Step 5 — Visualization and cleanup

A grouped bar chart shows the A/B/C/F grade distribution by party for each chamber (a sanity check, not an output file). Then the tables are tidied for export:

- Party names → abbreviations (`R`, `D`, `I`).
- State names → two-letter abbreviations; House districts formatted as `CA-01` and `CA01` (with at-large states like `AK`, `WY` handled specially); Senate rows have no district.
- Columns are reordered into a logical layout and the internal names are renamed to the published names: `score_T → T_score`, `bonus_m → M_bonus`, `bonus_n → N_bonus`, `bonus_p → P_bonus`, `score_U → U_score`, `norm_U → Bridge_Score`, `Grade → Bridge_Grade`.

## 4. Outputs

All files are written with the current date in the filename (`congress_scores_119_..._{YYYY-MM-DD}.xlsx`) for version control. Three formats are produced from the same final data:

1. **Separate files per chamber** — `house_scores_119_{date}.xlsx` and `senate_scores_119_{date}.xlsx`, each with one sheet named `119 Grades`.
2. **One workbook, two sheets** — `congress_scores_119_separate_sheets_{date}.xlsx` with a `House` sheet and a `Senate` sheet.
3. **One workbook, one combined sheet** — `congress_scores_119_single_sheet_{date}.xlsx`, House and Senate stacked together in a single `119 Grades` sheet.

Each file carries the full per-member detail: all raw source values, the `{X}_norm` percentiles and weights, the M/N/P inputs, the four bonus columns, `T_score`, `U_score`, and the published `Bridge_Score` and `Bridge_Grade`.

**Downstream use:** the **single-sheet** export is the file the Scorecard notebooks consume. Each calculation run on a given date produces one such snapshot; the Snapshot Scorecard notebook later collects several of these dated single-sheet files and compares how `Bridge_Score` and `Bridge_Grade` changed over time.
