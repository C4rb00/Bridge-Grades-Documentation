# Preprocessing_E_V1.0.1_2026_06_02.ipynb

> **Pipeline stage:** Preprocessing — builds **Source E** (count of personal-attack communications, per legislator) from American Political Pulse communications data.
> **Path:** `V1_Bridge_Grades/Source_Files/Column_E/V1/Preprocessing_E_V1.0.1_2026_06_02.ipynb`

## 1. Purpose

This notebook produces **Source E**, one of the four communication-based inputs to the Bridge Grades methodology. Source E measures, for each member of the 119th U.S. Congress, **how many of their public communications were classified as personal attacks**.

Personal attacks are the opposite of bridge-building behavior, so this metric is the negative counterpart to Source C. It captures the **raw volume** of personal-attack messages. Importantly, Source E is scored **inversely** in the main Bridge Grades calculation: the more personal attacks a legislator makes, the lower their resulting score. This notebook only produces the raw count; the inversion happens downstream.

This is one of four "communication" notebooks (C, D, E, F) that share the same overall structure — load the raw communications file plus a Congress roster, build a name lookup, and aggregate per legislator. They differ only in the metric each computes. Source E focuses on the **sum (count) of personal attacks**.

## 2. Inputs (what it consumes)

- **Raw communications file** (`rhetoric-all.csv`): the full set of public communications by U.S. House and Senate members, downloaded from [Americas Political Pulse](https://americaspoliticalpulse.com/data/). It includes all forms of public communication (floor speeches, newsletters, press releases, tweets/X). Each row is one message, pre-classified with flags such as `attack_personal` (the message is a personal attack) and `outcome_bipartisanship` (supportive of the opposite party). Located under `Input_Files/Snapshot_Raw_2026_06_02/Raw_C_2026_06_02/`.
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
   - Groups by legislator and **sums the `attack_personal` flag**. Because each message's flag is 1 when it is a personal attack and 0 otherwise, this sum is exactly the **count of personal attacks** — the Source E metric.
   - Results are sorted by `bioguide_id` for a stable output order.

   *Why a sum and not a percentage:* Source E deliberately measures the absolute number of personal attacks. (Its sibling, Source F, measures the percentage instead — together they capture both the volume and the rate of attacking behavior.)

## 4. Outputs

- **Temp CSV:** `Output_Files/Temp_Files/Temp_E/Temp_E_v1.0.1_2026_06_02.csv`
  - One row per legislator, with columns: `bioguide_id`, `full_name`, `communication_count`, and `attack_personal` (the personal-attack count = Source E value).
- The notebook's final step prints the row count as a sanity check that all expected legislators are present.
- This Temp_E file is consumed by the **main Bridge Grades calculation notebook**, where Source E is normalized to a 0–100 scale (Normal CDF), weighted (weight 1), and combined with the other sources. Because more attacks indicate less bridge-building, Source E is scored **inversely** — a higher attack count lowers the legislator's Bridge Score.
