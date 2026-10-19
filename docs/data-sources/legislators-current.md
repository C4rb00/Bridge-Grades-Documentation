# Get_Current_Legislators_Information.ipynb

> **Pipeline stage:** Supporting / utility notebook. It builds a structured identity-and-identifier table for *current* legislators from the OpenStates ("Plural") dataset, which feeds the source preprocessors and the master metadata table used by the main Bridge Grades calculation.
> **Path:** `V1_Bridge_Grades/Source_Files/Others/Get Legislator Information (Plural)/Get_Current_Legislators_Information.ipynb`

## 1. Purpose

This notebook reads the per-person YAML files published by the **OpenStates ("Plural") "people" dataset** for **current** legislators and turns them into a single flat table (CSV), with one row per legislator and a column for every identity field and every external identifier scheme.

The reason this matters for Bridge Grades: different data sources identify the same politician in different ways (Bioguide ID, Wikidata ID, VoteSmart ID, etc.). To join all of those sources together reliably, the pipeline needs a "crosswalk" that lists, for each legislator, all of their IDs side by side. This notebook produces that crosswalk for current legislators. The Bioguide ID in particular is the key used to line members up with the Congress.gov roster and vote data.

## 2. Inputs (what it consumes)

- The **OpenStates "people" GitHub repository** (`https://github.com/openstates/people.git`). The notebook clones it and reads the YAML files under `data/us/legislature` — the folder for **current** state/federal legislators. Each `.yml` file describes one legislator.
- A commented-out "Local Version" cell is included as an alternative for running the same logic against a local copy of the repository instead of cloning it in Colab.

No API key is required for this notebook. It is written for Google Colab and mounts Google Drive only to save the output.

Each YAML file may contain: basic fields (`id`, `name`, `given_name`, `family_name`, `birth_date`, `gender`, `email`, `image`), a `party` list, a `roles` list (type, district, start/end dates), a social-media `ids` block, and an `other_identifiers` block listing external ID schemes and their values.

## 3. Process (transformations & rationale)

1. **List all YAML files** in the current-legislators folder and report how many were found (a quick load check).

2. **First pass — gather every identifier scheme.** The notebook loops once through all files just to collect the complete set of distinct `scheme` names found in any legislator's `other_identifiers` (e.g., bioguide, wikidata, votesmart). The *why*: not every legislator has every ID type, so the code must first learn the full universe of possible columns before building the table — otherwise the columns would depend on whichever file happened to be read first.

3. **Second pass — build one record per legislator.** Looping through the files again, the notebook extracts each legislator into a dictionary:
   - Basic demographics (name, given/family name, birth date, gender, email, image).
   - Party (the first party entry's name).
   - Role info from the **last** role entry (role type, district, start/end dates), i.e., the most recent role.
   - Social-media handles, each stored as a `social_<platform>` column.
   - Every external identifier, stored as a `<scheme>_id` column (e.g., `bioguide_id`, `wikidata_id`).

   A "default" dictionary is used so any missing field simply comes through empty rather than causing an error.

4. **Assemble the table.** All records are combined into a single pandas DataFrame, one row per legislator.

5. **Guarantee a column for every scheme.** Using the full scheme set from the first pass, the notebook adds any `<scheme>_id` column that did not naturally appear, filling it with empty values. This ensures the output schema is complete and consistent regardless of which legislators happened to have which IDs.

6. **Data-completeness check.** The notebook displays any legislators that are missing a `bioguide_id`, so a reviewer can spot gaps in the all-important federal identifier.

## 4. Outputs

A single CSV:

- **Filename:** `plural_current_legislators_with_bioguide.csv`
- **Location:** a dated input-snapshot folder on Google Drive, `.../Bridge_Grades/Input_Files/Snapshot_Raw_2026_06_02/Raw_Others_2026_06_02/`. The notebook verifies the folder exists and raises an error rather than creating it. The specific snapshot folder is a **parameter control currently under review by political-domain subject-matter experts.**

The file contains one row per current legislator with all identity fields, social-media handles, and a complete set of `<scheme>_id` identifier columns. This identifier crosswalk feeds the source preprocessors and the master metadata table used by the main Bridge Grades calculation.

> **Companion notebook:** `Get_Retired_Legislators_Information.ipynb` runs the same process for *retired* legislators (reading the `data/us/retired` folder and writing `plural_retired_legislators_with_bioguide.csv`).
