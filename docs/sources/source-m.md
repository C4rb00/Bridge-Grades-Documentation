# Preprocessing_M_V1.0.0_2026_06_02.ipynb

> **Pipeline stage:** Preprocessing for Source M (Cook Political Partisan Voting Index / PVI) — produces the Temp_M files consumed by the main Bridge Grades calculation notebook.
> **Path:** `V1_Bridge_Grades/Source_Files/Column_M/V1/Preprocessing_M_V1.0.0_2026_06_02.ipynb`

## 1. Purpose

This notebook prepares **Source M**, the Cook Political **Partisan Voting Index (PVI)**, for every member of the 119th U.S. Congress. The PVI is a measure of how strongly a district (for House members) or a state (for Senators) leans toward one of the two major parties relative to the national average. A larger PVI magnitude means a more partisan, "safe" seat; a smaller magnitude means a more competitive, centrist seat.

In the Bridge Grades methodology, Source M is later turned into a **bonus** that rewards members who hold a more partisan seat yet still behave in a bridge-building way. That bonus calculation (and its scaling parameters) lives in the main notebook; this notebook's only job is to attach the correct, cleaned PVI value to each member and write it out as a Temp file.

The raw PVI strings (for example `"R+12"` or `"D+5"`) come in a single text field that combines a **party letter** and a **number**. This notebook's core work is to split that string into its two parts, line it up against the official congressional roster, and handle the edge case where a member belongs to a different party than their seat's partisan lean.

## 2. Inputs (what it consumes)

- **Cook PVI for the House** — `Cook PVI 1997-2025.xlsx`, sheet `119 (25-26)`. District-level PVI values in the column `2025 Cook PVI`. Obtained via paid subscription from Cook Political (https://www.cookpolitical.com/cook-pvi). Downloaded April 03, 2025; data updates periodically.
- **Cook PVI for the Senate** — `2025 PVI States.csv`. State-level PVI values in the column `2025 PVI`.
- **Congressional roster** — `119th_Congress_20260602.csv` (the shared "Raw_Others" snapshot). Supplies the authoritative list of members with `Name`, `bioguide_id`, `Party`, `Chamber`, `State`, and `District`. The `bioguide_id` is the unique key that lets the main notebook join every source together.

All paths point to the shared `Bridge_Grades` Google Drive data archive under the `Snapshot_Raw_2026_06_02` snapshot folder.

## 3. Process (transformations & rationale)

The notebook processes the House and the Senate separately, because PVI is reported at the district level for the House and at the state level for the Senate.

**Step 1 — Parse the PVI strings.** The raw value (e.g. `"R+12"`) is split on the `+` character into two new columns: `PVI_Party` (the party letter) and `PVI_Number` (the magnitude). The numeric part is converted to a whole number; anything that cannot be parsed is treated as `0`. This is done for both the House Excel data and the Senate CSV. The split is the whole reason the notebook exists: downstream math needs a clean number, and the bonus logic needs to know which party the lean favors.

**Step 2 — Build matching keys.** To join PVI data to the roster, the notebook standardizes identifiers:
- A one-letter party abbreviation is derived from each member's full party name (`Republican` to `R`, `Democratic` to `D`, `Independent` to `I`).
- State names are mapped to their two-letter USPS abbreviations (e.g. `California` to `CA`) in every table so they can be compared.
- For the House, a combined `State_District` key is built in the form `XX-DD` (e.g. `CA-12`), with the district number zero-padded to two digits. "At-Large" districts (states with a single representative) are treated as district `01`.

**Step 3 — Merge onto the roster.** The PVI values are left-joined onto the official member list — for the House on the `State_District` key, for the Senate on the state. A left join guarantees every member from the roster is kept even if a PVI value is missing.

**Step 4 — Handle party mismatches.** The notebook compares each member's own party letter against the party that their seat's PVI leans toward, recording the result in a `PVI_Party_Diff` flag. **When the member's party does NOT match the seat's lean, the PVI number is zeroed out.** The rationale: the PVI bonus is meant to credit a member for representing a seat that leans toward their own party (a partisan environment they nonetheless bridge across). If the seat leans the other way, crediting them on that scale would not be meaningful, so the magnitude is reset to zero while the flag preserves a record of why.

> Note: The PVI magnitude is capped (at 15) and combined with scaling factors when it is turned into a bonus. Those values — the cap and the scaling factor — are **parameter controls currently under review by political-domain subject-matter experts** and are applied in the main notebook, not here.

## 4. Outputs

Two CSV files, written to the shared archive under `Output_Files/Temp_Files/Temp_M/`:

- **`Temp_M_House_v1.0.0_2026_06_02.csv`** — House members, columns: `Name`, `Chamber`, `bioguide_id`, `Party`, `Cook_PVI_Party`, `Cook_PVI_Number`, `PVI_Party_Diff`.
- **`Temp_M_Senate_v1.0.0_2026_06_02.csv`** — Senators, columns: `Name`, `Chamber`, `bioguide_id`, `Party`, `PVI_Party`, `PVI_Number`, `PVI_Party_Diff`.

Both files are read by the **main Bridge Grades calculation notebook**, which uses `bioguide_id` to merge Source M with the other sources and to compute the PVI bonus on top of each member's weighted base score.
