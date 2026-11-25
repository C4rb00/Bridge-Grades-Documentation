# Tips & Recommendations

!!! note "Handover notes"
    This page gathers practical recommendations for whoever continues developing the Bridge Score — cautions and lessons accumulated while building V1 and running the first snapshots, together with a few additional recommendations from the documentation review. They are **not** part of the formal methodology. Several of the values mentioned here (weights, thresholds, caps, and multipliers) are **parameter controls still under review by domain experts**.

## Data integrity and external sources

- **Party of the independents.** In every notebook that takes party into account, verify that **independent** legislators are assigned a party **manually** (mapping them to the party they effectively caucus with). A mislabeled independent distorts the entire cross-party test.
- **Watch for large jumps in the temp files.** Check the **balance of the scores** and look for **large jumps** in the `Temp_*` files. Because most of the inputs come from sources we do not control, they often receive updates without notice, and we don't find out until we see the big jump in the data.
- **`"No data"` is not the same as `zero`.** Missing source values are currently filled with `0`, which is correct when it genuinely means *"no activity."* Revisit the cases where a member simply has **no data** for a source (e.g., no communications records at all) versus a true zero, since treating them identically can penalize a member unfairly.
- **Download and archive the inputs regularly.** Keep a **steady, periodic download** of the score's inputs — at least of the sources handled today — so that if an **atemporal (historical)** analysis is ever needed, the lack of historical data (especially in the early part of the year, before the first changes are applied) is not a problem.
- **Check each source's licensing and terms of use.** Before redistributing any derived data, confirm redistribution rights — especially for **Cook PVI**, which is a proprietary product — and the usage limits of the APIs (Congress.gov, govinfo, American Political Pulse / Plural, VoteView).

## Parameters and methodology

- **Bonus multipliers.** The multipliers assigned to the bonuses were calibrated **after the first snapshot tests**, so that their impact would be measured. It would be worth reviewing how those fixed multipliers are behaving **over time**.
- **Rethink the bonuses in the next version.** We believe it is worth treating the bonuses as something **more general** rather than granting them to only a few: giving an "extra" to part of the group can end up adding more **noise** than a score with equal conditions for everyone. This matters especially with the new election results: when **ideology and district** line up (Republican and Republican), it becomes much easier for that bonus to "lift" many members at once. That could add a **balancing point** or inject **partisan noise** — it deserves careful analysis.
- **Fix the E/F normalization quirk.** Sources E and F are flipped with `1 - norm.cdf(...) * 100`, an expression that mixes scales (a known quirk already flagged in the main-calculation docs). The next version should correct it and document the resulting **re-baseline** as an expected one-time jump.
- **Move the tunable parameters into a versioned config file.** Keep the weights, thresholds, caps, and multipliers out of the notebooks, in a single configuration file under version control, with a **changelog** and the **sign-off** of the expert who approved each change. This gives traceability for *why* a snapshot changed.

## Reproducibility and engineering

- **Pin the environment versions.** Freeze the versions of `pandas` / `numpy` / `scipy` and of Python in a lockfile. A change in `scipy.stats.norm` or in the merge behavior can move scores **without any change to the data**.
- **Add a "golden snapshot" regression test.** Keep a reference snapshot and compare every run against it **member by member**, flagging any difference above a threshold. This turns the manual "check the jump member by member" recommendation into something systematic.

## Continuity and versioning (this is a sequential score)

- **Work on a copy and measure the jump.** If you want to make new changes to the code, always work on a **copy** and check the jump produced **member by member** after the change. Don't assume that because a change is small **qualitatively**, it will be the same **quantitatively**. Remember this is a score **with sequence**: if it receives updates on the fly, anyone following the series will notice **jumps** in the metric.
- **Temporal comparability.** Because normalization is **relative** (within chamber and within each snapshot), the scores are not absolute; comparisons across dates must be made with care. If longitudinal analysis is a goal, evaluate a **time-stable anchor or normalization** — this reinforces the caution about the score being sequential.

## Political context and bias risk

- **Effect of the recent elections.** Keep in mind the **natural drop** Democrats' scores will have — since, naturally, it will be less necessary for them to collaborate with Republicans to advance bills — and, likewise, the **natural rise** in Republicans' scores. We believe this should be a **critical point to address next year**, so that the score does not become **partisan** and end up reflecting the **convenience of the moment** more than bipartisanship itself.
- **Monitor partisan balance.** Define metrics that track the distribution of grades by party **over time**, with alert thresholds, so the partisan drift noted above can be detected **objectively** rather than by eye.

## Recommended analyses before publishing

- **Elections vs. historical bridgers.** Analyze the **current election results** against the scores of the **historical bridgers**, to see whether a strong relationship or perspective emerges in the cross-referenced data.
- **Study the post-election jump.** Before publishing new scores, analyze the **post-election jump** and how the current ("old") score behaves under the **new partisan dynamic**.
