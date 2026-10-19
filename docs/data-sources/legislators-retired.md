# Get_Retired_Legislators_Information.ipynb

> **Pipeline stage:** Supporting / utility notebook. It builds a structured identity-and-identifier table for *retired* legislators from the OpenStates ("Plural") dataset, which feeds the source preprocessors and the master metadata table used by the main Bridge Grades calculation.
> **Path:** `V1_Bridge_Grades/Source_Files/Others/Get Legislator Information (Plural)/Get_Retired_Legislators_Information.ipynb`

## 1. Purpose

This notebook reads the per-person YAML files published by the **OpenStates ("Plural") "people" dataset** for **retired** legislators and turns them into a single flat table (CSV), with one row per legislator and a column for every identity field and every external identifier scheme.

It is the retired-legislator counterpart of `Get_Current_Legislators_Information.ipynb`. The reason this matters for Bridge Grades: the 119th Congress can include members who served in earlier sessions or whose records moved to the retired set, and historical vote/sponsorship data references people who are no longer serving. Having an identifier crosswalk (Bioguide ID, Wikidata ID, VoteSmart ID, etc.) for retired legislators lets the pipeline match those historical references to the right person.

## 2. Inputs (what it consumes)

- The **OpenStates "people" GitHub repository** (`https://github.com/openstates/people.git`). The notebook clones it and reads the YAML files under `data/us/retired` — the folder for **retired** legislators. Each `.yml` file describes one person.
- A commented-out "Local Version" cell is included as an alternative for running against a local copy of the repository instead of cloning it in Colab.

No API key is required. It is written for Google Colab and mounts Google Drive only to save the output. The YAML fields consumed are the same as in the current-legislators notebook (basic demographics, party, roles, social-media ids, and `other_identifiers`).

## 3. Process (transformations & rationale)

The logic is identical to the current-legislators notebook, pointed at the retired folder:

1. **List all YAML files** in the retired-legislators folder and report the count.

2. **First pass — gather every identifier scheme** across all files, so the full set of possible `<scheme>_id` columns is known before the table is built. The *why*: legislators differ in which IDs they have, so the column set must be learned up front for consistency.

3. **Second pass — build one record per legislator**, extracting basic demographics, party (first party entry), role info from the **last** role entry, social-media handles (`social_<platform>` columns), and every external identifier (`<scheme>_id` columns). A default dictionary lets missing fields pass through empty.

4. **Assemble the table** into a single pandas DataFrame, one row per legislator.

5. **Guarantee a column for every scheme**, adding any missing `<scheme>_id` columns filled with empty values, for a complete and consistent schema.

6. **Data-completeness check and one manual fix.** The notebook displays legislators missing a `bioguide_id`. In this retired set it then **manually fills in the Bioguide ID for Marlin Stutzman** (`S001188`) — a known record that lacked the federal identifier in the source data. The *why*: the Bioguide ID is the primary key for matching across datasets, so patching a known gap keeps that legislator joinable downstream. (This manual override is a **parameter control currently under review by political-domain subject-matter experts.**)

## 4. Outputs

A single CSV:

- **Filename:** `plural_retired_legislators_with_bioguide.csv`
- **Location:** a dated input-snapshot folder on Google Drive, `.../Bridge_Grades/Input_Files/Snapshot_Raw_2026_06_02/Raw_Others_2026_06_02/`. The notebook verifies the folder exists and raises an error rather than creating it. The specific snapshot folder is a **parameter control currently under review by political-domain subject-matter experts.**

The file contains one row per retired legislator with all identity fields, social-media handles, and a complete set of `<scheme>_id` identifier columns. This identifier crosswalk feeds the source preprocessors and the master metadata table used by the main Bridge Grades calculation.

> **Companion notebook:** `Get_Current_Legislators_Information.ipynb` runs the same process for *current* legislators.
