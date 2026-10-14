# SPEC.md

> **DRAFT v1 — Klára writes this.**
>
> The four headings below are quoted verbatim from the assignment
> (Exercise 5, part "5. ML study"). The questions under them are the
> assignment's own hints, which it explicitly says are *"hints, not a checklist
> to copy"*. So: answer them in your own words, and add constraints the hints
> do not mention.
>
> `Context` is pre-filled with facts we measured and verified in step 1 — that
> part you can trust. `Goal`, `Constraints`, and `Done when` have `> ANSWER:`
> blocks. Replace those with your own words and delete the quote blocks.
>
> Facts the assignment pins down for us (not open for debate, see below):
> the task is **classification of `Tree type`**; a **most-frequent-class
> baseline is mandatory** and your model must beat it; the test set is
> **frozen before tuning**; deliverables are an **HTML report** and a
> **five-slide Quarto presentation** generated from **saved results**.

**Assignment source:** <https://tomasvicar.github.io/MLR-public/exercises/ex05.html>,
part "5. ML study". Project is worth 5 of the 30 semester points.

---

# Goal

*What question should the result answer?*

> ANSWER: One or two sentences, in your own words.
>
> The assignment's stated goal is: *"find out how well the tree type can be
> predicted from leaf height and width — with a number you can trust."* Say
> that in your own words, and make it precise enough to be checkable.
>
> Prompts:
> - The three classes are unbalanced (41.7 / 33.4 / 24.9 %). "How well" is
>   meaningless without saying *relative to what*. What is the number you want
>   to be able to defend?
> - Why "a number you can trust" and not "the best possible number"? The
>   assignment is asking about honesty of the estimate, not peak accuracy. What
>   would make your number untrustworthy?
> - Is the deliverable a *decision* ("this classifier is good enough to use"),
>   or a *measurement* ("here is how separable these classes are")? These want
>   different framing.

---

# Context

*The data: file, columns, rows, where it comes from, what you found in step 1.*

**File:** `data/leaves.csv` — 10,024 bytes, 519 lines (1 header + 518 data rows).
The assignment says to save it as `data/leaves.csv` and **record where it came
from** — see the open question below.

**Columns:**

| Column | Type | Range | Role |
|---|---|---|---|
| `Leaf height` | numeric | 3.835 – 25.5 | feature |
| `Leaf width` | numeric | 2.500 – 29.7 | feature |
| `Tree type` | string | `maple`, `cherry`, `walnut` | **target** |

**Class balance:** maple 216 (41.7%), cherry 173 (33.4%), walnut 129 (24.9%).
No missing values, no non-positive dimensions, no whitespace or encoding
problems.

**How the features relate** — per-type Pearson / Spearman:

| Group | n | Pearson r | Spearman ρ |
|---|---:|---:|---:|
| all leaves | 518 | 0.5894 | 0.6352 |
| maple | 216 | 0.8016 | 0.7606 |
| cherry | 173 | 0.6834 | 0.7445 |
| walnut | 129 | 0.6672 | 0.7780 |

| Type | mean height | mean width |
|---|---:|---:|
| cherry | 9.334 | 5.328 |
| maple | 13.482 | 15.245 |
| walnut | 15.322 | 7.567 |

Every per-type correlation exceeds the pooled one — the signature of three
groups with different centres pooled together. Note walnut and maple have
similar *mean heights* (15.3 vs 13.5) but very different *mean widths*
(7.6 vs 15.2), so width does more class-separating work than height.

## Findings from step 1

Step 3 of the assignment says to *"first list anything you would have to
guess"* — these findings are what removes most of the guessing. Three of them
directly answer the assignment's own questions ("Are there identical rows? What
would that mean for a train/test split?").

1. **The file is three datasets stacked, split by row order.** Counting decimal
   places per row partitions the file into exactly three *contiguous* runs.
   Verified two independent ways (raw CSV text, and pandas after float parsing):

   | Rows | n | Precision | distinct h | distinct w | exact-dup rows |
   |---|---:|---|---:|---:|---:|
   | 0 – 99 | 100 | ≤ 2 dp | 67.0% | 76.0% | 6 |
   | 100 – 301 | 202 | up to 6 dp | 100.0% | 100.0% | 0 |
   | 302 – 517 | 216 | ≤ 2 dp | 39.4% | 45.4% | 17 |

   Overall decimal-place distribution: `{1dp: 283, 2dp: 33, 5dp: 76, 6dp: 126}`.

2. **Block C differs in distribution, not just precision.** Rows 302–517 have
   larger means (height 14.03 vs 11.48/11.51, width 11.13 vs 9.25/9.22) and a
   different class mix (walnut 30.1% vs 22.0%/20.8%). Blocks A and B are
   near-identical to each other; C is not.

3. **26 exact duplicate rows; 29 rows duplicate another row's `(height, width)`.**
   Coordinate `(7.5, 3.8)` occurs 4 times.

   Duplicates are overwhelmingly **within** a block — only **3 coordinate pairs
   are shared between blocks**, all between A and C. Block B shares nothing
   with either. That is the leakage exposure, and it is small but not zero.

4. **Three rows are labelled inconsistently** — identical dimensions, two
   different types:

   | height | width | types assigned |
   |---|---|---|
   | 11.0 | 5.0 | walnut, cherry |
   | 16.0 | 7.0 | cherry, walnut |
   | 16.0 | 7.5 | cherry, walnut |

   All three lie in the low-precision region. These are contradictory labels,
   and they cap the accuracy any model can reach — at most 515 of 518 rows can
   be correct if they cannot all be satisfied.

5. **Class clusters overlap substantially.** Maple and cherry spans overlap on
   59.0% of the height range and 66.7% of the width range — separated by
   *density*, not a clean boundary. So a high accuracy is not expected, and the
   baseline number matters for reading whatever accuracy you get.

6. **Deduplication does not move the correlations.** Dropping the 29
   coordinate-duplicates (518 → 489) changes pooled Pearson 0.5894 → 0.5923.
   So duplicates are present but are *not* what drives the headline numbers.

**Provenance — OPEN, and the assignment requires it.** I could not establish
where `leaves.csv` came from: not in git history, no other copy on disk, and
nothing in this repo writes to `data/` (every reference is a read). It appeared
2026-10-14 08:39. **Klára — the assignment says to record where the data came
from.** Where did you get it? (Moodle? handed out in the lab? If you do not
know, say "provided by the course, source unrecorded" — but do not leave the
sentence out, it is an explicit requirement.)

---

# Constraints

*Which models, and which hyperparameters over which ranges? How and when is the
test set held out? What about the repeated rows? Which metric chooses the
model, and on which data? How is tuning done, and with what budget?*

**Pinned by the assignment — state these, do not re-decide them:**

- Task is **classification** of `Tree type` from the two measurements.
- "Try several classical classifiers **from the lectures**", tune their
  hyperparameters, "compare them fairly". The lecture menu (L1 p.2, Lab 6) is
  SVM, decision trees, random forest; k-NN and linear models appear earlier.
- A **baseline that always predicts the most common class is mandatory**, and
  "...yours must beat it" (step 3).
- "**Freeze the test data before tuning; choose the model without it**"
  (Tips: "Check the result"). Tested **once** (step 5).
- "All trial results saved to files" (step 4).

> ANSWER: Now the decisions that are genuinely yours. The assignment says
> outright: *"The evaluation design is yours... you decide."*
>
> Prompts:
> - **Which classifiers, and how many?** "Several" is vague on purpose. Pick a
>   set you can justify — at least one of each family, and say why. Which need
>   feature scaling (k-NN, SVM) and how do you keep that inside the pipeline so
>   it never sees the test fold?
> - **Exact grids.** Not "tune k-NN" but e.g. "k ∈ {1,3,5,…,25}, weights ∈
>   {uniform, distance}". Same precision for every model.
> - **Test set.** What fraction, held out *when* (before any tuning, per the
>   quotation above), stratified? What `random_state`? The value is not fixed
>   by the assignment — so *you* fix it and write it down, or the number is not
>   reproducible.
> - **The repeated rows.** This is asked directly by the assignment. Duplicates
>   are mostly within-block, and only 3 coordinate pairs cross blocks — so
>   exact leakage is small. But the low-precision blocks are grid data with
>   heavy repetition, so *near*-duplicates are far more common than exact ones.
>   Do you deduplicate before splitting, split by block, or keep everything and
>   report both? Give the reasoning, and note that whatever you choose, the
>   choice must be identical for every model or the comparison is unfair.
> - **The 3 conflicting labels.** Drop, keep, or resolve? Any choice changes
>   the achievable ceiling, so state it.
> - **Metric.** Accuracy is the obvious pick but is arguably wrong: classes are
>   41.7/33.4/24.9 and overlap heavily. If you use accuracy, say why it is
>   adequate here. If balanced accuracy or macro-F1, say what it buys. The
>   assignment names no metric — *"The evaluation design is yours."* Whichever
>   you pick, it must be the same for the baseline and every model.
> - **Metric on which data.** The baseline's number and the model-selection
>   number come from CV folds; the final number comes from the frozen test set.
>   Be explicit about which is which, because conflating them is the classic
>   mistake the assignment's step 5 exists to prevent.
> - **Tuning budget.** Grid or random search? Total candidates, CV folds
>   (stratified k-fold, k = ?), and a cap — wall-clock or candidate count. Say
>   the cap even if you do not hit it.
> - **Class imbalance.** Do you correct it (class weights, resampling), and does
>   your metric choice already handle it? Note the majority class is only 41.7%,
>   so the baseline is weak — make sure you report what it actually scores,
>   since "beat the baseline" is only meaningful next to its real number.
> - **Environment.** There is still **no `pyproject.toml` / `uv.lock`**. Per
>   `AGENTS.md` §2 these are needed for a reproducible run, and `uv.lock` must
>   be committed. `scikit-learn` currently resolves to 1.9.1 — pin it. Decide
>   whether to do `uv init` + `uv add` before the modelling code or as part of
>   it. Quarto also has to be installed for step 6.

---

# Done when

*Which files must exist (results, report, slides) and which numbers and checks
must be in them, so that a classmate can reproduce and check it?*

**Pinned by the assignment (step 6, closing "Done when"):**

- `SPEC.md`, the code, the saved results, the report, and the slides are all in
  the repository, and every item of *this* checklist is met.
- Report is **HTML**; presentation is **five slides**, built with **Quarto**.
- Both are **generated from the saved results, not retyped numbers**.
- The chosen model is tested **once** on the held-out data, and the
  **predictions are saved**.

> ANSWER: Turn that into a checklist a classmate can execute against. "The
> report looks good" is not checkable; "`results.csv` has one row per model with
> columns model, params, cv_mean, cv_std, test_acc" is.
>
> Prompts:
> - **Files.** Name each one and its exact path. Which are required, which are
>   nice-to-have? (Candidates: the experiment script, a results table, a
>   baseline number, saved test predictions, the `.qmd` sources, the rendered
>   report and slides.)
> - **Numbers that must appear.** The baseline's score; each model's CV score
>   as mean ± spread; the chosen model and its hyperparameters; its score on the
>   frozen test set; the class counts; the test-set size and seed. Exact names
>   and how many significant figures.
> - **Checks a reader can run.** At minimum: one command to reproduce, from a
>   clean checkout, with no global installs. The seed. And a statement of what
>   was done about duplicates, so a reader can re-run without them and compare.
> - **Does the report have to state the three-block structure?** It is the most
>   interesting thing in step 1 and it plausibly affects the result. Discuss it,
>   or scope it out — but decide, do not leave it ambiguous.
> - **What is explicitly out of scope?** So the work does not drift. (The
>   assignment offers PCA-in-CV and Git-worktree experiments as optional extra;
>   are those in or out?)
>
> `AGENTS.md` §3 applies here: the final claim must be backed by a check that
> actually ran, with the command and its real output in the report. Not "should
> reproduce".

---

## Where this sits in the assignment

Steps 1–2 of 7. Step 1 (look at the data) is done — the counts, scatter plot and
anomalies were produced and are summarised in `Context` above.

| Step | Status |
|---|---|
| 1. Look at the data | **done** (counts, scatter, anomalies) |
| 2. Write `SPEC.md` | **this file** — draft scaffold, you write the answers |
| 3. Plan, baseline, trial run | next — `/plan Read SPEC.md and propose a plan` |
| 4. Models and tuning | pending — all trial results saved to files |
| 5. The final number | pending — test once, save predictions |
| 6. Report and slides | pending — Quarto, HTML + 5 slides |
| 7. Review in a fresh session | pending — `/clear`, then re-read against the spec |

Step 3's first move is **"Read SPEC.md. First list anything you would have to
guess."** So the value of this file is precisely in how few guesses are left.

## Open questions for the interview

Park anything unresolved here rather than guessing:

- [ ] Provenance of `data/leaves.csv` — **assignment requires this**, and it is
      the one Context item I could not determine.
- [ ] Deduplicate before splitting, split by block, or keep and report both?
- [ ] Are the three *blocks* treated as three sources in the write-up, or pooled?
- [ ] `uv init` / `uv add` / pin `scikit-learn` — before or during modelling?
- [ ] Step-1 scratch scripts (`check_leaves*.py`, `verify_*.py`) — keep, fold
      into one script, or delete? All currently untracked.
- [ ] Are you working alone or in pairs? The assignment allows pairs and says
      each student keeps their own repo and records who did what.
