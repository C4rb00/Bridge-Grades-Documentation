# Preprocessing_A_V1.4.0_2026_06_02.ipynb

> **Pipeline stage:** Source A preprocessing — builds the "bills authored that attracted a cross-party cosponsor" measure (the most heavily weighted input, weight 3, in the main Bridge Grade calculation).
> **Path:** `V1_Bridge_Grades/Source_Files/Column_A/V1/Preprocessing_A_V1.4.0_2026_06_02.ipynb`

## 1. Purpose

This notebook produces **Source A** of the Bridge Grade: for each member of the 119th U.S. Congress, a credit-weighted **count of the bills they personally authored (were the primary sponsor of) that drew at least one cosponsor from the opposite party**.

The idea behind Source A is that authoring legislation and getting a member of the other party to sign onto it is a strong, deliberate act of bipartisanship. Because authorship is the clearest signal of bridge-building, it carries the highest weight (3) of all sources in the final score.

The notebook downloads raw bill data straight from the U.S. government's bulk data service, parses every bill's sponsor and cosponsors, figures out who crossed party lines, and rewards each authored bill more or less depending on (a) how early the cross-party cosponsor signed on and (b) what kind of bill it is. The result is one number per legislator, written to a "Temp" CSV that the main calculation notebook later reads, normalizes, and weights.

## 2. Inputs (what it consumes)

- **govinfo.gov BILLSTATUS bulk data** for the 119th Congress. The notebook loops over the eight bill types — `hconres`, `hr`, `hres`, `hjres`, `s`, `sconres`, `sres`, `sjres` — and downloads one ZIP archive per type from:
  `https://www.govinfo.gov/bulkdata/BILLSTATUS/119/<type>/BILLSTATUS-119-<type>.zip`
  Each ZIP contains one XML file per bill. No local input files are required; the data is pulled live over HTTP.
- **A small manual list of party corrections** (see Process step 3) for a handful of legislators whose party label in the raw data is missing or ambiguous.

## 3. Process (transformations & rationale)

**Step 1 — Download and extract.** For each of the eight bill types the notebook requests the ZIP, opens it in memory, and reads every `.xml` file inside. There is no need to save anything to disk.

**Step 2 — Parse each bill's XML.** A parsing function walks each bill's XML and pulls out:
- General bill data: congress number, bill number, type, originating chamber, **introduced date**, and the latest action's code/type/text.
- The **primary sponsor** (the author): full name, bioguide ID, and party.
- Every **cosponsor**: full name, bioguide ID, party, and the date that cosponsor signed on.

The data is "exploded" so that each row represents one (bill, cosponsor) pair. A bill with ten cosponsors becomes ten rows, every row repeating the same sponsor/bill information. This flat shape makes the later cross-party and timing comparisons straightforward.

**Step 3 — Clean and fix parties.** Rows with no cosponsor party are dropped (a bill with no cosponsors cannot demonstrate cross-party support, so it contributes nothing to Source A). A short manual override then corrects party labels for a few specific legislators (by bioguide ID) who are recorded as independents or have missing/inconsistent party data, mapping them to the party they effectively caucus with (`D` or `R`). This keeps the cross-party test from being thrown off by independents.

**Step 4 — Build a bill key and measure sponsorship timing.** A `bill_code` is built by joining the bill type and number (e.g. `HR-1234`) so all rows of the same bill can be grouped. The cosponsorship date and the bill's introduced date are converted to real dates, and a **`days_to_join`** value is computed for every cosponsor: how many days after the bill was introduced that cosponsor signed on. Day 0 means they were an original cosponsor present at introduction.

**Step 5 — Flag cross-party relationships.** `is_opposite_party` is set to True whenever the cosponsor's party differs from the sponsor's party. This is the core bipartisanship signal.

**Step 6 — Keep only bills that earned a day-0 cross-party cosponsor.** The notebook identifies the set of bills that had at least one opposite-party cosponsor who signed on **on day 0**, and restricts the working data to those bills. In this version of Source A, a bill only counts if the cross-party support was present from the very start — a deliberate, "in from the beginning" signal rather than a later add-on.

**Step 7 — Per-row credit weighting (timing).** Each row is assigned a `credit` value that rewards earlier cross-party sign-ons more than later ones, and treats same-party sign-ons differently from cross-party ones (cross-party day-0 gets the largest credit; cross-party day 1–5 and same-party rows get smaller credits; anything else gets a small floor value). *The exact credit values (e.g. day-0 vs day 1–5, cross-party vs same-party) are parameter controls currently under review by political-domain subject-matter experts.* The intent of the parameter is to reward bipartisan commitment that is both genuine (crosses party lines) and early (joined at or near introduction).

**Step 8 — Bill-type weighting.** A `weight_map` assigns each bill type a multiplier so that substantive legislation counts for more than symbolic measures: regular bills and joint resolutions get the full weight, concurrent resolutions a reduced weight, and simple resolutions the smallest weight. *These bill-type weights are parameter controls currently under review by subject-matter experts.* The rationale is that authoring a real law-making bill that attracts cross-party support is a stronger act of bridge-building than doing so on a ceremonial resolution.

**Step 9 — Final per-legislator aggregation (the version actually exported).** The exported figure is computed with a cleaner, bill-level method that avoids double-counting a bill once per cosponsor:
- For each authored bill that had cross-party cosponsors, the notebook determines a **quality flag**: did *any* opposite-party member cosponsor it on day 0?
- That flag sets a **quality weight** — a full value if there was a day-0 cross-party cosponsor, and a slightly reduced value otherwise. *The two quality-weight values are parameter controls currently under review by subject-matter experts;* the parameter rewards bills where the cross-party support was there from the start over bills where it came later.
- The quality weight is multiplied by the bill-type weight to give each authored bill a `final_score`.
- Scores are summed per **primary sponsor** (the author), producing one total per legislator.

The result is renamed to `primary_bioguide_id` and `num_bills_with_cross_party_cosponsors` and sorted descending.

## 4. Outputs

- **A single Temp CSV** written to the shared Data Archive:
  `Output_Files/Temp_Files/Temp_A/Temp_A_v1.4.0_2026_06_02.csv`
  (full Google Drive path in the notebook: `/content/drive/Shareddrives/Data Archive/Bridge_Grades/Output_Files/Temp_Files/Temp_A/Temp_A_v1.4.0_2026_06_02.csv`).
- The file has two columns:
  - `primary_bioguide_id` — the legislator's bioguide ID (the author).
  - `num_bills_with_cross_party_cosponsors` — the credit-weighted Source A count for that legislator.
- This file is consumed by the **main Bridge Grade calculation notebook**, which locates it through its `paths` dictionary, normalizes the values to a 0–100 scale (Normal CDF), applies the Source A weight (3), and folds it into the final Bridge Score and letter grade.
