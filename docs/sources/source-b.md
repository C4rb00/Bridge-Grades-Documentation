# Preprocessing_B_V1.7.0_2026_06_02.ipynb

> **Pipeline stage:** Source B preprocessing — builds the "cross-party bills cosponsored" measure (weight 2 in the main Bridge Grade calculation).
> **Path:** `V1_Bridge_Grades/Source_Files/Column_B/V1/Preprocessing_B_V1.7.0_2026_06_02.ipynb`

## 1. Purpose

This notebook produces **Source B** of the Bridge Grade: for each member of the 119th U.S. Congress, a credit-weighted **count of the bills they cosponsored that were authored by a member of the opposite party**.

Where Source A measures *authoring* bipartisan legislation, Source B measures the other side of the same handshake — *signing onto* the other party's bills. It is a slightly weaker signal of bridge-building than authorship (you are supporting someone else's bill rather than initiating it), so it carries weight 2 versus Source A's weight 3.

The notebook downloads the same raw bill data, parses sponsors and cosponsors, identifies cross-party cosponsorships, applies a timing-based credit and a bill-type weight, and aggregates one number per legislator into a "Temp" CSV that the main calculation notebook later reads, normalizes, and weights.

## 2. Inputs (what it consumes)

- **govinfo.gov BILLSTATUS bulk data** for the 119th Congress — the same eight bill types (`hconres`, `hr`, `hres`, `hjres`, `s`, `sconres`, `sres`, `sjres`), one ZIP per type, downloaded live from:
  `https://www.govinfo.gov/bulkdata/BILLSTATUS/119/<type>/BILLSTATUS-119-<type>.zip`
  Each ZIP holds one XML file per bill. No local input files are needed.
- **A small manual list of party corrections** (see Process step 3) for a handful of legislators with missing or ambiguous party labels.

## 3. Process (transformations & rationale)

**Steps 1–5 are identical to Source A** (the two notebooks share the same download and parsing front end):

1. **Download and extract** each bill-type ZIP in memory and read every XML.
2. **Parse each bill** into one row per (bill, cosponsor) pair, capturing bill metadata, introduced date, the primary sponsor (author) with party, and each cosponsor with party and sign-on date.
3. **Clean and fix parties** — drop rows with no cosponsor party, then manually override party labels for a few specific legislators (by bioguide ID) so independents are mapped to the party they caucus with (`D`/`R`).
4. **Build `bill_code`** (type + number) and compute `days_to_join` — how many days after the bill was introduced each cosponsor signed on (day 0 = original cosponsor at introduction).
5. **Flag `is_opposite_party`** — True when the cosponsor's party differs from the author's party.

From step 6 onward, Source B differs from Source A because the credit is attributed to the **cosponsor** (the person signing on), not the author, and because this version splits the work into two complementary pipelines that are recombined at the end.

**Step 6 — Split bills into two timing groups.** The notebook recognizes that a cross-party cosponsor can join either at introduction (day 0) or later, and treats those two situations with separate credit schemes:
- **Group 1 — bills with a day-0 cross-party cosponsor** (`df_final_b`): bills where at least one opposite-party member signed on the day the bill was introduced.
- **Group 2 — bills where the opposite party joined, but never on day 0** (`df_first_opp_all`): bills that did attract cross-party cosponsors, but only after introduction. This is computed as the set difference: bills with any cross-party cosponsor, minus bills with a day-0 cross-party cosponsor.

Splitting this way lets each group be credited on its own scale, so that "joined from the start" support and "joined later" support are not scored identically.

**Step 7 — Per-row credit (timing).** Each row gets a `credit` reflecting how early and whether the cosponsorship crossed party lines:
- In **Group 2**, the earliest opposite-party cosponsor(s) on a bill (those matching the minimum `days_to_join` among the cross-party signers) receive a higher credit, and later cross-party signers a lower one — rewarding being first to reach across the aisle on a bill.
- In **Group 1**, credit is assigned by tiers based on `days_to_join` (day 0, day 1–5, and so on) and on whether the row is cross-party or same-party, with cross-party day-0 rewarded most.

  > **Implementation note:** in the Group 1 credit cell the `.loc` assignments are applied in sequence and partly overlap (a later `days_to_join < 5` line overwrites part of what the earlier day-0 and day-1-5 lines set), so the effective credit values depend on the order the lines run. The exact numbers and these overlapping rules are *parameter controls currently under review by political-domain subject-matter experts* and should be confirmed with them before relying on the precise tier boundaries.

  The intent of the credit parameter throughout is to reward cosponsorships that cross party lines and that happen early.

**Step 8 — Bill-type weighting.** Both groups apply the same `weight_map` as Source A: regular bills and joint resolutions get the full weight, concurrent resolutions a reduced weight, and simple resolutions the smallest weight. *These bill-type weights are parameter controls currently under review by subject-matter experts.* The rationale is that cosponsoring substantive legislation across party lines is a stronger bipartisan act than cosponsoring a ceremonial resolution.

**Step 9 — Compute each row's contribution.** In each group, `total = weight_type * credit` — the bill-type weight scaled by the timing credit.

**Step 10 — Recombine the two pipelines.** The two groups (`df_final_b` and `df_first_opp_all`) are concatenated back into one table, so every cross-party cosponsorship — whether it joined on day 0 or later — is represented with its appropriate credit and bill-type weight.

**Step 11 — Final per-legislator aggregation.** The combined table is grouped by **cosponsor** (`cosponsor_bioguide`) and the per-row `total` values are summed, giving each legislator one credit-weighted Source B score (a row count is also kept for reference). The result is renamed and sorted descending by the score.

## 4. Outputs

- **A single Temp CSV** written to the shared Data Archive:
  `Output_Files/Temp_Files/Temp_B/Temp_B_v1.7.0_2026_06_02.csv`
  (full Google Drive path in the notebook: `/content/drive/Shareddrives/Data Archive/Bridge_Grades/Output_Files/Temp_Files/Temp_B/Temp_B_v1.7.0_2026_06_02.csv`).
- The file has these columns:
  - `bioguide_id` — the legislator's bioguide ID (the cosponsor).
  - `num_cross_party_cosponsored_bills` — the credit-weighted Source B score for that legislator.
  - `num_rows` — a supporting count of contributing rows.
- This file is consumed by the **main Bridge Grade calculation notebook**, which locates it through its `paths` dictionary, normalizes the values to a 0–100 scale (Normal CDF), applies the Source B weight (2), and folds it into the final Bridge Score and letter grade.
