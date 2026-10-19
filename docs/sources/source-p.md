# Preprocessing_P_V1.0.0_2026_06_02.ipynb

> **Pipeline stage:** Preprocessing for Source P (Problem Solvers Caucus membership) — produces the Temp_P file consumed by the main Bridge Grades calculation notebook.
> **Path:** `V1_Bridge_Grades/Source_Files/Column_P/V1/Preprocessing_P_V1.0.0_2026_06_02.ipynb`

## 1. Purpose

This notebook prepares **Source P**, a flag identifying members of the **Problem Solvers Caucus (PSC)** — a bipartisan group in the U.S. House of Representatives focused on cross-party cooperation. Because membership in this caucus is itself an act of bridge-building, the Bridge Grades methodology gives PSC members a fixed bonus ("bump").

This source applies to the **House only** (the Problem Solvers Caucus is a House caucus). The notebook's job is narrow: maintain the authoritative list of PSC members' unique identifiers and write it out so the main notebook can flag the corresponding members and apply the bonus.

## 2. Inputs (what it consumes)

- **A maintained list of PSC members**, hard-coded directly inside the notebook as two parallel lists of `Name` and `bioguide_id` (45 members in this version). This is the working copy of `problem_solvers.csv`, maintained in the project's data repo. Source: the official caucus membership roster at https://problemsolverscaucus.house.gov/caucus-members. Last checked March 31, 2026.

Because the list is embedded in the notebook itself, there is no external file read at run time — the notebook *is* the maintained source of truth for who is in the caucus. When the caucus roster changes, this list is updated.

## 3. Process (transformations & rationale)

**Step 1 — Define the membership list.** The notebook holds a hard-coded dictionary pairing each PSC member's name with their `bioguide_id` (the unique identifier used to join every Bridge Grades source together). Names are kept alongside the IDs for human readability and auditing, but only the IDs matter downstream.

**Step 2 — Build a table and export the IDs.** The list is turned into a table, and **only the `bioguide_id` column** is written out. The names are intentionally dropped from the output: the main notebook does not need them, since it matches members by `bioguide_id`. Any member whose ID appears in this file is flagged as a PSC member; everyone else is not.

The bonus itself — a fixed points boost applied to flagged members — is calculated in the **main** notebook. The size of that bonus is a **parameter control currently under review by political-domain subject-matter experts**; this notebook only supplies the membership flag.

## 4. Outputs

One CSV file, written to the shared archive under `Output_Files/Temp_Files/Temp_P/`:

- **`Temp_P_v1.0.0_2026_06_02`** — a single column, `bioguide_id`, listing every Problem Solvers Caucus member.

This file is read by the **main Bridge Grades calculation notebook**, which checks each House member's `bioguide_id` against this list to set the PSC flag and apply the caucus bonus on top of the weighted base score.

> Note: The output filename as written in the notebook has no `.csv` extension. The content is comma-separated values regardless; this is worth flagging to the maintainers in case a `.csv` extension is expected by the main notebook's read step.
