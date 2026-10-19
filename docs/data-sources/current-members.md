# Get_Current_Members.ipynb

> **Pipeline stage:** Supporting / utility notebook. It builds the roster of current voting members of the 119th Congress, which feeds the source preprocessors and the master metadata table used by the main Bridge Grades calculation.
> **Path:** `V1_Bridge_Grades/Source_Files/Others/Get Current Members/Get_Current_Members.ipynb`

## 1. Purpose

This notebook produces a clean, date-stamped list of every **current voting member** of the 119th U.S. Congress. It pulls the official membership from the Congress.gov API, flattens the data, removes non-voting territorial delegates, tidies up the names and chamber labels, and saves the result as a CSV.

The reason this matters for Bridge Grades: the whole project scores members of Congress, so it first needs an accurate, current "who is in Congress" roster to anchor everything else. This notebook is that roster source. The `bioguide_id` it carries is the key that lets every other dataset (votes, bill sponsorships, legislator details) line up to the right person.

## 2. Inputs (what it consumes)

- **Congress.gov API** — the official U.S. government data service. The notebook calls the endpoint for 119th-Congress members and filters to current members only (`currentMember=true`).
  - A **Congress.gov API key is required.** The key is passed in the request header. (The notebook contains a placeholder key and a note to replace it with your own; you can register for a free key at the Congress.gov sign-up page. Do not commit a real key.)
  - The page size and the Congress number (119) are **parameter controls currently under review by political-domain subject-matter experts.**
- No other input files are required; the data is fetched live from the API.

The notebook is written for Google Colab and mounts Google Drive only at the end, to save the output.

## 3. Process (transformations & rationale)

1. **Fetch all members with pagination.** The API returns results in pages, so the notebook loops, requesting one page at a time, appending the members to a running list, and stopping when a page comes back empty or without a `members` section. A short pause between requests avoids hitting the API's rate limit. The *why*: a single request cannot return the full membership, so paging guarantees the complete roster is collected.

2. **Flatten the nested JSON into a table.** Each member record contains a nested history of terms. The notebook normalizes this so every term becomes its own row, while carrying along the member-level fields (bioguide id, name, state, district, party, update date, and the profile-image block).

3. **Extract the profile image.** The image lives inside a nested field; the notebook pulls out the image URL, fills in "No Image Available" where missing, and drops the now-redundant nested column.

4. **Keep only the active term.** Because every member's full term history is included, the notebook keeps only the row where the term has no end year (`endYear` is null) — i.e., the current, ongoing term.

5. **Remove non-voting territorial delegates.** Delegates from American Samoa, Guam, Northern Mariana Islands, Puerto Rico, the Virgin Islands, and the District of Columbia do not have full voting rights, so they are filtered out. This territory list is a **parameter control currently under review by political-domain subject-matter experts.** The *why*: Bridge Grades scores voting behavior, so non-voting members would distort the analysis.

6. **Parse and standardize names.** The API supplies names in "Last, First M." format. The notebook splits each into `first_name`, `middle_name`, `last_name`, a `nickname` (anything in quotes), and a clean, properly ordered full `Name`. This makes the roster easy to match against other datasets and to read.

7. **Standardize the chamber label.** "House of Representatives" is shortened to "House"; "Senate" is left as-is, for consistency with prior versions of the file.

8. **Select and rename the final columns.** A focused set of columns is kept and renamed to a stable, consistent schema (e.g., `bioguideId` -> `bioguide_id`, `partyName` -> `Party`, `startYear` -> `start_year`). This keeps the output compatible with earlier "119th Congress" files used downstream.

## 4. Outputs

A single date-stamped CSV:

- **Filename:** `119th_Congress_YYYYMMDD.csv` (the date is the run date).
- **Location:** a dated input-snapshot folder on Google Drive, `.../Bridge_Grades/Input_Files/Snapshot_Raw_2026_06_02/Raw_Others_2026_06_02/`. The notebook checks that this folder already exists and raises an error rather than creating it, to avoid writing to the wrong place. The specific snapshot folder is a **parameter control currently under review by political-domain subject-matter experts.**

Columns include `bioguide_id`, `Name` (plus split name parts and nickname), `Chamber`, `State`, `District`, `Party`, `start_year`, `end_year`, `update_date`, and `imageUrl`. The notebook prints the total count of voting members as a final confirmation.

This roster feeds the source preprocessors and the master metadata table used by the main Bridge Grades calculation.

### Relationship to `get_current_members.ipynb`

A separate, earlier/alternate notebook (`Source_Files/Others/get_current_members.ipynb`) performs the same core API fetch. This notebook (`Get_Current_Members.ipynb`) is the cleaner, more polished version: it uses a dedicated, more robust name-parsing function and a tidied save step. Notably, the optional **snapshot-comparison** logic (comparing today's roster against an older one to list new and departed members) lives in the earlier `get_current_members.ipynb`, not in this notebook.
