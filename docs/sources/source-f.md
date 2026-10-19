# Preprocessing_F_V1.0.1_2026_06_02.ipynb

> **Pipeline stage:** Preprocessing — builds **Source F** (percentage of communications that are personal attacks, per legislator) from American Political Pulse communications data.
> **Path:** `V1_Bridge_Grades/Source_Files/Column_F/V1/Preprocessing_F_V1.0.1_2026_06_02.ipynb`

## 1. Purpose

This notebook produces **Source F**, one of the four communication-based inputs to the Bridge Grades methodology. Source F measures, for each member of the 119th U.S. Congress, **what share (percentage) of their public communications were classified as personal attacks**.

Where Source E counts the raw *number* of personal attacks, Source F measures the *rate*: of everything a legislator said publicly, what fraction was a personal attack. This captures how characteristically combative a legislator is, regardless of how much they communicate overall. Like Source E, Source F is scored **inversely** in the main Bridge Grades calculation: a higher percentage of attacks lowers the resulting score. This notebook only produces the percentage; the inversion happens downstream.

This is one of four "communication" notebooks (C, D, E, F) that share the same overall structure — load the raw communications file plus a Congress roster, build a name lookup, and aggregate per legislator. They differ only in the metric each computes. Source F focuses on the **personal-attack percentage**.

## 2. Inputs (what it consumes)

- **Raw communications file** (`rhetoric-all.csv`): the full set of public communications by U.S. House and Senate members, downloaded from [Americas Political Pulse](https://americaspoliticalpulse.com/data/). It includes all forms of public communication (floor speeches, newsletters, press releases, tweets/X). Each row is one message, pre-classified with flags such as `attack_personal` (personal attack) and `outcome_bipartisanship` (supportive of the opposite party). Located under `Input_Files/Snapshot_Raw_2026_06_02/Raw_C_2026_06_02/`.
  - *Download instructions (from the source portal):* on the left side open the **US-OFFICIALS** section and click **download**.
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
   - Groups by legislator and **sums the `attack_personal` flag** to get the count of personal attacks.
   - Computes `attack_personal_pct` = (attack count ÷ total communication count) × 100. This is the **Source F metric**: the percentage of a legislator's communications that are personal attacks.
   - Results are sorted by `bioguide_id` for a stable output order.

5. **Drop the intermediate count column.** After the percentage is computed, the raw `attack_personal` count is dropped so the output carries only the rate (the percentage), keeping Source F distinct from Source E.

   *Why a percentage and not a sum:* Source F deliberately measures the *rate* of attacking communication, complementing Source E's absolute count. Together they capture both how much and how consistently a legislator engages in personal attacks.

## 4. Outputs

- **Temp CSV:** `Output_Files/Temp_Files/Temp_F/Temp_F_v1.0.1_2026_06_02.csv`
  - One row per legislator, with columns: `bioguide_id`, `full_name`, `communication_count`, and `attack_personal_pct` (the personal-attack percentage = Source F value).
- The notebook's final step prints the row count as a sanity check that all expected legislators are present.
- This Temp_F file is consumed by the **main Bridge Grades calculation notebook**, where Source F is normalized to a 0–100 scale (Normal CDF), weighted (weight 1), and combined with the other sources. Because a higher attack rate indicates less bridge-building, Source F is scored **inversely** — a higher attack percentage lowers the legislator's Bridge Score.
