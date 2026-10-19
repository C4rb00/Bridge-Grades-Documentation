# Preprocessing_D_V1.0.1_2026_06_02.ipynb

> **Pipeline stage:** Preprocessing — builds **Source D** (percentage of communications that are bipartisan, per legislator) from American Political Pulse communications data.
> **Path:** `V1_Bridge_Grades/Source_Files/Column_D/V1/Preprocessing_D_V1.0.1_2026_06_02.ipynb`

## 1. Purpose

This notebook produces **Source D**, one of the four communication-based inputs to the Bridge Grades methodology. Source D measures, for each member of the 119th U.S. Congress, **what share (percentage) of their public communications were classified as bipartisan** — that is, supportive of the opposite party.

Where Source C counts the raw *number* of bipartisan messages, Source D measures the *rate*: of everything a legislator said publicly, what fraction reached across the aisle. This rewards consistency rather than sheer volume — a legislator who sends fewer messages overall but bipartisan ones a large share of the time scores well here. A higher percentage is considered better, and it is normalized and rewarded in the main Bridge Grades calculation notebook.

This is one of four "communication" notebooks (C, D, E, F) that share the same overall structure — load the raw communications file plus a Congress roster, build a name lookup, and aggregate per legislator. They differ only in the metric each computes. Source D focuses on the **bipartisan percentage**.

## 2. Inputs (what it consumes)

- **Raw communications file** (`rhetoric-all.csv`): the full set of public communications by U.S. House and Senate members, downloaded from [Americas Political Pulse](https://americaspoliticalpulse.com/data/). It includes all forms of public communication (floor speeches, newsletters, press releases, tweets/X). Each row is one message, pre-classified with flags such as `outcome_bipartisanship` (supportive of the opposite party) and `attack_personal` (personal attack). Located under `Input_Files/Snapshot_Raw_2026_06_02/Raw_C_2026_06_02/`.
  - *Download instructions (from the source portal):* open the **"Congressional Rhetoric"** section and download the **"All Classified Statements"** card.
- **119th Congress roster** (`119th_Congress_20260602.csv`): legislators with their unique `bioguide_id` plus `first_name` and `last_name`. Located under `Input_Files/Snapshot_Raw_2026_06_02/Raw_Others_2026_06_02/`.

The notebook runs in Google Colab and reads these files from a shared Google Drive ("Data Archive").

## 3. Process (transformations & rationale)

1. **Mount storage and load data.** Connects to Google Drive and loads the raw communications file and the 119th Congress roster.

2. **Build a bioguide → name lookup (shared helper).** A clean reference table (`df_bioguide`) joins `first_name` and `last_name` into a single `full_name` keyed by `bioguide_id`. (Shared across all four communication notebooks.)

3. **Restrict to 2025 communications.** The data is filtered to messages dated on or after **2025-01-01**, so the score reflects current-term behavior only. This 2025 start date is a **parameter control currently under review by political-domain subject-matter experts** — it defines the time window of communications that count toward the score.

4. **Aggregate per legislator (`clean_communications_app`).** The core transformation:
   - Keeps only the columns needed for the calculation.
   - Computes `communication_count` — the total number of messages per legislator — by counting all rows sharing the same `bioguide_id`.
   - Builds `full_name` from the trimmed first and last names.
   - Groups by legislator and **sums the `outcome_bipartisanship` flag** to get the count of bipartisan messages.
   - Computes `outcome_bipartisanship_pct` = (bipartisan count ÷ total communication count) × 100. This is the **Source D metric**: the percentage of a legislator's communications that are bipartisan.
   - Results are sorted by `bioguide_id` for a stable output order.

5. **Drop the intermediate count column.** After the percentage is computed, the raw `outcome_bipartisanship` count is dropped so the output carries only the rate (the percentage), keeping Source D distinct from Source C.

   *Why a percentage and not a sum:* Source D deliberately measures the *rate* of bipartisan communication, complementing Source C's absolute count. Together they capture both how much and how consistently a legislator communicates bipartisanship.

## 4. Outputs

- **Temp CSV:** `Output_Files/Temp_Files/Temp_D/Temp_D_v1.0.1_2026_06_02.csv`
  - One row per legislator, with columns: `bioguide_id`, `full_name`, `communication_count`, and `outcome_bipartisanship_pct` (the bipartisan percentage = Source D value).
- The notebook's final step prints the row count as a sanity check that all expected legislators are present.
- This Temp_D file is consumed by the **main Bridge Grades calculation notebook**, where Source D is normalized to a 0–100 scale (Normal CDF), weighted (weight 1), and combined with the other sources to produce each legislator's Bridge Score and letter grade. A higher bipartisan percentage contributes to a higher score.
