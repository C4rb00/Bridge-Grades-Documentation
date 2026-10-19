# Get_Classifications_Bills.ipynb

> **Pipeline stage:** Supporting / utility notebook. It is an upstream helper that classifies 119th-Congress legislative measures and flags those that became law. Its output feeds the source preprocessors and the master metadata table used by the main Bridge Grades calculation.
> **Path:** `V1_Bridge_Grades/Source_Files/Others/Get Clasifications Bills/Get_Classifications_Bills.ipynb`

## 1. Purpose

This notebook takes the raw list of every bill and resolution introduced in the **119th U.S. Congress** and adds simple, human-readable labels describing *what kind of measure each one is* (a Bill vs. a Resolution, and the specific sub-type), plus a flag for whether the measure **became law**.

The reason this matters for Bridge Grades: not every item Congress votes on is the same. A binding "Bill" that can become law is treated very differently from a non-binding "Simple Resolution." Downstream scoring logic needs to know each measure's type so it can weight or filter votes appropriately. This notebook produces that classification once, so the rest of the pipeline does not have to re-derive it.

## 2. Inputs (what it consumes)

The data originates from the public **Plural Policy** data portal (americaspoliticalpulse.com / data.openstates.org). The download instructions embedded in the notebook are:

1. On the data portal, find **United States 119th Congress** (or download the packaged ZIP directly).
2. Inside the package, navigate to `US` -> `119` -> `US_119_bill.csv`.

The notebook reads two CSVs from the shared Google Drive project archive:

- `US_119_bills.csv` — one row per legislative measure. Key columns used: `identifier` (e.g., "HR 1234", "S 56", "HRES 22"), `id` (the measure's unique key), and `classification` (a list-style text field such as `['bill']` or `['resolution']`).
- `US_119_bill_actions.csv` — one row per recorded action on a measure. Key columns used: `bill_id` and `classification` (a list-style text field describing the action, including the value `became-law`).

Both files live under an input "snapshot" folder dated in the path (the dated snapshot folder is a **parameter control currently under review by political-domain subject-matter experts**). The notebook is written for Google Colab and mounts Google Drive to reach these files.

## 3. Process (transformations & rationale)

1. **Classify each measure by its prefix.** Every congressional measure has an identifier that starts with a standard prefix. The notebook uses a lookup of regular-expression patterns to read that prefix and assign two new columns:
   - `prefix_category` — the raw prefix (HR, S, HJRES, SJRES, HCONRES, SCONRES, HRES, SRES).
   - `measure_type` — the plain-English category that prefix maps to:
     - `HR`, `S` -> **Bill**
     - `HJRES`, `SJRES` -> **Joint Resolution**
     - `HCONRES`, `SCONRES` -> **Concurrent Resolution**
     - `HRES`, `SRES` -> **Simple Resolution**

   Anything that does not match a known prefix (or is blank) is labelled `Unknown`. This explicit prefix-to-type mapping is a **parameter control currently under review by political-domain subject-matter experts.** The *why*: the prefix is the official, reliable signal for a measure's legal nature, so deriving the type from it is more trustworthy than free-text guessing.

2. **Clean the `classification` column into readable labels.** The source `classification` field arrives as list-style text (e.g., `['bill']`). The notebook safely parses that text and collapses it to a clean single word — `Bill` or `Resolution` — and `Unknown` for anything else. This makes the column directly usable by people and by later code without further parsing.

3. **Identify measures that became law.** From the separate bill-actions file, the notebook parses each action's `classification` list and keeps only the rows whose actions include `became-law`. This produces a short list of enacted measures.

4. **Merge the "became law" flag back onto the bills.** The enacted-measures list is joined onto the main bills table (matching the bills' `id` to the actions' `bill_id`) with a left join, so every bill keeps its row and only enacted bills receive a matching `bill_id` / `classification_clean` value. The notebook then checks how many rows did not match (i.e., did not become law) as a sanity check.

## 4. Outputs

A single date-stamped CSV written to the project's preprocessing-temp output folder on Google Drive:

- **Filename:** `US_119_bills_processed_YYYY-MM-DD.csv` (the date is the run date).
- **Location:** `.../Bridge_Grades/Output_Files/Temps_Files_For_Preprocessing/`

The file is the original 119th-Congress bills table plus the added columns `prefix_category`, `measure_type`, the cleaned `classification`, and the merged `bill_id` / `classification_clean` (present only for measures that became law). At the end, the notebook prints the distribution of `measure_type` and `classification` values as a final check.

This processed bill table is consumed downstream by the source preprocessors / master metadata that feed the main Bridge Grades calculation.
