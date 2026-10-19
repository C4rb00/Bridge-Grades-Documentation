# Preprocessing_N_V1.0.0_2026_06_02.ipynb

> **Pipeline stage:** Preprocessing for Source N (VoteView member ideology) — produces the Temp_N files consumed by the main Bridge Grades calculation notebook.
> **Path:** `V1_Bridge_Grades/Source_Files/Column_N/V1/Preprocessing_N_V1.0.0_2026_06_02.ipynb`

## 1. Purpose

This notebook prepares **Source N**, a measure of each member's **ideological position**, for the 119th U.S. Congress. The underlying data comes from VoteView, an academic project that scores legislators on an ideological scale based on their voting records. The specific measure used here is `nominate_dim1`, the standard left–right (liberal–conservative) dimension, where strongly negative values are very liberal and strongly positive values are very conservative.

The Bridge Grades methodology rewards members who sit closer to the ideological **center**. To express that, this notebook converts each member's signed ideology score into an **absolute distance from the center** (how far from the middle they are, regardless of direction). A member with a small distance is near the center; a large distance is at one of the extremes. The main notebook later turns this distance into a bonus that favors centrist members.

This notebook only cleans and attaches the ideology distance to each member and splits the result by chamber. The bonus math and its scaling factor live in the main notebook.

## 2. Inputs (what it consumes)

- **VoteView member ideology** — `HS119_members.csv`, downloaded from VoteView (https://voteview.com/data; Data Type: Member Ideology; Chamber: Both; Congress: 119th). Downloaded February 16, 2026. Only two columns are read: `bioguide_id` and `nominate_dim1`.
- **Congressional roster** — `119th_Congress_20260602.csv` (the shared "Raw_Others" snapshot). Supplies `Name`, `bioguide_id`, and `Chamber`, used to attach each member's chamber and split the data into House and Senate.

All paths point to the shared `Bridge_Grades` Google Drive data archive under the `Snapshot_Raw_2026_06_02` snapshot folder.

## 3. Process (transformations & rationale)

**Step 1 — Take the absolute value.** The raw `nominate_dim1` score is signed (negative for liberal, positive for conservative). The notebook computes its absolute value into a new column, capturing **distance from the ideological center** rather than direction. This is the key transformation: Bridge Grades cares only about how moderate or extreme a member is, not which side they are on. The column is then renamed `ideology_dist` for clarity.

**Step 2 — Diagnose and remove non-voting members.** The notebook counts how many rows are missing a `bioguide_id` or a score. The two members found without a `bioguide_id` are non-voting members (delegates), and they are dropped, since they are not part of the scored Congress and cannot be joined to the other sources.

**Step 3 — Attach chamber and split.** The ideology data is merged with the roster on `bioguide_id` to bring in each member's chamber, then split into a House subset and a Senate subset. They are handled separately because the main notebook compares members within their own chamber.

**Step 4 — Deduplicate and select output columns.** Each chamber subset is deduplicated on `bioguide_id` (one row per member) and trimmed to the final output columns.

> Note: The markdown narrative in the notebook mentions computing percentile scores per chamber, and the original VoteView score is carried through. The actual conversion to a percentile and the bonus scaling factor are applied in the **main** notebook. The scaling factor is a **parameter control currently under review by political-domain subject-matter experts**; this notebook simply provides the clean per-member ideology distance.

## 4. Outputs

Two CSV files, written to the shared archive under `Output_Files/Temp_Files/Temp_N/`:

- **`Temp_N_House_v1.0.0_2026_06_02.csv`** — House members, columns: `Name`, `bioguide_id`, `nominate_dim1`, `ideology_dist`.
- **`Temp_N_Senate_v1.0.0_2026_06_02.csv`** — Senators, columns: `Name`, `bioguide_id`, `nominate_dim1`, `ideology_dist`.

Both files are read by the **main Bridge Grades calculation notebook**, which uses `bioguide_id` to merge Source N with the other sources, converts `ideology_dist` into a percentile, and computes the centrist bonus on top of each member's weighted base score.
