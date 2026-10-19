# Preprocessing_C_V1.0.1_2026_06_02.ipynb

> **Pipeline stage:** Preprocessing — builds **Source C** (count of bipartisan communications per legislator) from American Political Pulse communications data.
> **Path:** `V1_Bridge_Grades/Source_Files/Column_C/V1/Preprocessing_C_V1.0.1_2026_06_02.ipynb`

## 1. Purpose

This notebook produces **Source C**, one of the four communication-based inputs to the Bridge Grades methodology. Source C measures, for each member of the 119th U.S. Congress, **how many of their public communications were classified as bipartisan** — that is, supportive of the opposite party.

The intuition is simple: a legislator who frequently speaks in ways that reach across the aisle is showing bipartisan behavior. Source C captures the **raw volume** of those bipartisan messages. A higher count is considered better, and it is later normalized and rewarded in the main Bridge Grades calculation notebook.

This is the first of the four "communication" notebooks (C, D, E, F). All four share the same overall structure — load the raw communications file plus a Congress roster, build a name lookup, and aggregate per legislator. They differ only in the specific metric each one computes. Source C focuses on the **sum (count) of bipartisan communications**.

## 2. Inputs (what it consumes)

- **Raw communications file** (`rhetoric-all.csv`): the full set of public communications by U.S. House and Senate members, downloaded from [Americas Political Pulse](https://americaspoliticalpulse.com/data/). It includes all forms of public communication (floor speeches, newsletters, press releases, tweets/X). Each row is one message and is pre-classified with flags such as `outcome_bipartisanship` (the message is supportive of the opposite party) and `attack_personal` (the message is a personal attack). Located under the project's `Input_Files/Snapshot_Raw_2026_06_02/Raw_C_2026_06_02/` folder.
  - *Download instructions (from the source portal):* open the **"Congressional Rhetoric"** section and download the **"All Classified Statements"** card.
- **119th Congress roster** (`119th_Congress_20260602.csv`): the official list of legislators with their `bioguide_id` (the unique government identifier used to join every data source in this project) plus `first_name` and `last_name`. Located under `Input_Files/Snapshot_Raw_2026_06_02/Raw_Others_2026_06_02/`.

The notebook runs in Google Colab and reads these files from a shared Google Drive ("Data Archive").

## 3. Process (transformations & rationale)

1. **Mount storage and load data.** The notebook connects to Google Drive and loads the raw communications file and the 119th Congress roster.

2. **Build a bioguide → name lookup (shared helper).** A clean reference table (`df_bioguide`) is built by combining `first_name` and `last_name` into a single `full_name` and keeping only `bioguide_id` and `full_name`. This gives a tidy, human-readable label for each legislator keyed by their unique ID. (This same helper appears in all four communication notebooks.)

3. **Restrict to 2025 communications.** The data is filtered to messages dated on or after **2025-01-01**, so the score reflects current-term behavior only. The 2025 start date is a **parameter control currently under review by political-domain subject-matter experts** — it defines the time window of communications that count toward the score.

4. **Aggregate per legislator (`clean_communications_app`).** The core transformation:
   - Keeps only the columns needed for the calculation (`bioguide_id`, `attack_personal`, `outcome_bipartisanship`, `policy`, and the name fields).
   - Computes `communication_count` — the total number of messages each legislator produced — by counting all rows that share the same `bioguide_id`.
   - Builds `full_name` from the trimmed first and last names.
   - Groups by legislator and **sums the `outcome_bipartisanship` flag**. Because each message's flag is 1 when bipartisan and 0 otherwise, this sum is exactly the **count of bipartisan communications** — the Source C metric.
   - Results are sorted by `bioguide_id` for a stable, predictable output order.

   *Why a sum and not a percentage:* Source C deliberately measures absolute bipartisan output. (Its sibling, Source D, measures the percentage instead — together they capture both volume and rate.)

## 4. Outputs

- **Temp CSV:** `Output_Files/Temp_Files/Temp_C/Temp_C_v1.0.1_2026_06_02.csv`
  - One row per legislator, with columns: `bioguide_id`, `full_name`, `communication_count`, and `outcome_bipartisanship` (the bipartisan-communication count = Source C value).
- The notebook's final step prints the row count as a sanity check that all expected legislators are present.
- This Temp_C file is consumed by the **main Bridge Grades calculation notebook**, where Source C is normalized to a 0–100 scale (Normal CDF), weighted (weight 1), and combined with the other sources to produce each legislator's Bridge Score and letter grade. Higher bipartisan counts contribute to a higher score.
