# Snapshot_Scorecard_2026_06_02.ipynb

> **Pipeline stage:** SCORECARD — compares Bridge Grades snapshots produced on different dates/versions and reports how scores and grades changed.
> **Path:** `Scorecards/V1/Snapshot_Scorecard_2026_06_02/Snapshot_Scorecard_2026_06_02.ipynb`

## 1. Purpose

The main calculation notebook produces one Excel "snapshot" of every legislator's Bridge Score and Bridge Grade each time it is run. This scorecard notebook is the tool for looking **across** those snapshots: it merges several dated snapshot files into one wide table (one row per legislator, one pair of score/grade columns per snapshot date), then analyzes how things moved between versions.

It answers questions like: Did a legislator's score go up or down between runs? Did anyone change letter grade? Who moved the most? Are any legislators missing from some snapshots (for example because they retired, passed away, or left office)? It also draws distribution charts so the overall shift between two versions can be seen at a glance.

This notebook is exploratory/analytical rather than a fixed production pipeline — several cells point at different snapshot sets (the `2026_06_02` batch, the `2026_03_23` comparison batch, and an older `2026_02_04` pair) and are run as needed.

## 2. Inputs (what it consumes)

The inputs are the **single-sheet Excel snapshots** produced by the main calculation notebook (`congress_scores_119_single_sheet_{date}.xlsx`), stored in the scorecard's `Data/` subfolders on the shared Google Drive. Each file has one row per legislator and includes the shared metadata columns plus a `Bridge_Score` and `Bridge_Grade` column.

The notebook reads several sets:

- **Primary snapshot batch** (this scorecard's `Data/` folder): four dated files — `2026-03-23`, `2026-03-31`, `2026-05-01`, and `2026-06-02`.
- **Version-comparison batch** (from the `Snapshot_Scorecard_2026_03_23` folder): two files labeled `119.1` (`...2026-03-23_V1.0.2.xlsx`) and `119.2` (`...2026-03-23_V1.5.2.xlsx`) — same date, different methodology versions. These are used to compare the underlying Source A and Source B metrics between versions.
- **Older pair** (from the `Snapshot_Scorecard_2026_02_04` folder): a pre-merged file (`Bridge_Grades_Snapshot_Scorecard_2026_02_04_V.xlsx`) containing two score columns (`Bridge_Score_2026_02_04` and `Bridge_Score_2026_02_04_V`) used for the difference/visualization analysis.

Key columns read: `bioguide_id` (the join key), the metadata columns (`Chamber`, `Name`, `Party`, `State_Abbr`, `last_name`, etc.), and per-snapshot `Bridge_Score` / `Bridge_Grade`. The version-comparison cells additionally read `A_norm`, `A: num_bills_with_cross_party_cosponsors`, `B_norm`, and `B: num_cross_party_cosponsored_bills`.

## 3. Process (transformations & rationale)

### Step 1 — Merge snapshots into one wide table

For each snapshot the notebook:

1. Reads the Excel file and strips stray whitespace from column names.
2. Locates the score and grade columns by keyword search (`find_column`), so it is tolerant of small naming differences.
3. Renames them to date-stamped names, e.g. `Bridge_Score_2026_03_23` and `Bridge_Grade_2026_03_23`, so columns from different snapshots never collide.
4. Keeps the full metadata **only from the first snapshot**; for every later snapshot it keeps just `bioguide_id` plus the new date-stamped score/grade columns.
5. Outer-merges everything on `bioguide_id`.

**Why an outer join and metadata-once:** the outer join keeps a legislator even if they appear in some snapshots but not others (important for detecting departures). Carrying metadata only once avoids duplicate `Name`/`Party` columns. A small backfill step repairs any metadata gaps from the first-snapshot row.

The merged table is sorted by state then last name and written to `Bridge_Grades_Snapshot_Scorecard_2026_06_02.xlsx`.

### Step 2 — Presence / missing-legislator check

The notebook builds a small presence matrix marking, for each `bioguide_id`, whether it appeared in each snapshot (1/0), and flags anyone present in fewer than all snapshots. It then prints those legislators with a per-date "Present / Missing" status. A markdown note records the human explanation for the current gaps (e.g., one retired, one passed away, one retired). **Why:** this distinguishes a real data problem from an expected roster change.

### Step 3 — Version A/B metric comparison (119.1 vs 119.2)

Two additional merge blocks compare the *inputs* behind the score, not just the score itself, between methodology versions 119.1 and 119.2:

- **Comparison A:** pulls `A_norm` and the raw `A` bill count from each version, renamed to `119.1_Anorm/Anum` and `119.2_Anorm/Anum`, and exports `..._ComparisonA.xlsx`.
- **Comparison B:** the same for Source B, exported as `..._ComparisonB.xlsx`.

Follow-up cells filter to the House (and separately the Senate) and sum the 119.2 raw counts **by party**, to see which party accounts for more bipartisan bills/cosponsorships. **Why:** this lets analysts trace a change in scores back to the specific source metric and see partisan patterns in the underlying behavior.

### Step 4 — Difference analysis and visualization

Using the older pre-merged pair (`2026_02_04`), the notebook:

- Computes `Diferencia = newer_score − older_score` per legislator and prints the **mean difference** between versions (the average drift in scores).
- Finds the **biggest loser** (largest drop, via `idxmin`) and the **most drastic absolute change** (via the largest `|difference|`), printing the before/after scores and the affected `bioguide_id`.
- Flags grade changes: `difieren = grade_new ≠ grade_old`, then counts how many legislators changed letter grade.

Two charts are produced and saved as PNGs:

- **KDE distribution** (`score_kde_distribution.png`) — smoothed density curves of both versions' scores overlaid, showing how the whole distribution shifted.
- **Grouped bar chart** (`grade_distribution.png`) — count of each letter grade (A/B/C/F) side by side per version, showing how the grade mix changed.

**Why:** the single-number mean difference can hide movement that cancels out; the distribution and grade-count views reveal whether the change was a uniform shift, a reshuffling, or concentrated in a few members.

## 4. Outputs

- **`Bridge_Grades_Snapshot_Scorecard_2026_06_02.xlsx`** — the master wide table: one row per legislator, metadata once, and a date-stamped `Bridge_Score_*` / `Bridge_Grade_*` pair for each of the four snapshots. This is the main deliverable of the notebook.
- **`Bridge_Grades_Snapshot_Scorecard_2026_03_23_ComparisonA.xlsx`** and **`..._ComparisonB.xlsx`** — side-by-side Source A and Source B metrics for versions 119.1 vs 119.2.
- **`score_kde_distribution.png`** and **`grade_distribution.png`** — the distribution and grade-count charts.
- Printed console output: the mean score difference, the biggest-loser / biggest-change records, the count of legislators who changed grade, and the missing-legislator report.

These outputs are reporting artifacts — they summarize how Bridge Grades evolved across runs and feed manual review and communication rather than being consumed by a further automated stage.
