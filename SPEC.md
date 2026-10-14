# SPEC.md

> **DRAFT v2 — decisions made, one round of interview done.**
>
> The four headings are quoted verbatim from the assignment (Exercise 5, part
> "5. ML study"). The questions under them are the assignment's own hints, which
> it says are *"hints, not a checklist to copy"* — they have been answered, and
> the answers go past them where the hints are silent.
>
> `Context` holds facts measured and verified in step 1. Everything else is a
> decision made in the interview below, and every number quoted here was
> produced by a command that ran — the seed, the 104-row test set, the 41.35%
> baseline, and the grid counts were all executed, not estimated.
>
> Facts the assignment pins down (not open for debate): the task is
> **classification of `Tree type`**; a **most-frequent-class baseline is
> mandatory** and the model must beat it; the test set is **frozen before
> tuning**; deliverables are an **HTML report** and a **five-slide Quarto
> presentation** generated from **saved results**.

**Assignment source:** <https://tomasvicar.github.io/MLR-public/exercises/ex05.html>,
part "5. ML study". Project is worth 5 of the 30 semester points.

**Decisions taken** (the assignment says *"The evaluation design is yours...
you decide"*): accuracy as the selecting metric with balanced accuracy,
macro-F1 and the confusion matrix reported beside it · five classifiers, one per
lecture family · 20% stratified hold-out, `random_state=42` · all 518 rows for
the primary result, with a duplicate-free sensitivity run reported alongside ·
repeated-row and contradictory-label handling answered explicitly.

---

# Goal

*What question should the result answer?*

How well can maple, cherry, and walnut be told apart from leaf height and width
alone — and is that measurably better than just guessing the most common type?

The answer is **one number**: accuracy on leaves the model never saw during any
part of the work. It is only meaningful next to the baseline it has to beat, so
the result is always reported as a pair: *this model scored X%, always answering
"maple" scores 41.7%*.

I want "a number I can trust", not the best number I can produce. Concretely
that means:

- The test leaves were set aside **before** any model or hyperparameter was
  chosen, and were used **once**, at the end. Nothing was picked by looking at
  them.
- The split is seeded, so a classmate gets the same 104 leaves and therefore the
  same number.
- The baseline is reported from the *same* test set as the model, not from the
  full 518 — comparing a model's test accuracy to a whole-dataset majority rate
  would flatter the model by about a third of a point.

Reading the finding that the two measurements only partly separate the classes
(finding 5 — cherry and maple spans overlap on ~59% of height and ~67% of
width), a very high accuracy is not expected here. The goal is an honest
measurement of how much signal two measurements actually carry, not a headline.

# Context

*The data: file, columns, rows, where it comes from, what you found in step 1.*

**File:** `data/leaves.csv` — 10,024 bytes, 519 lines (1 header + 518 data rows).
The assignment says to save it as `data/leaves.csv` and **record where it came
from** — done, see Provenance below.

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
   The partition is exact: **0 rows** in 100–301 fall below 5 decimal places, so
   the three runs are clean cuts, not a rough trend. The measure matters —
   *printed* decimal places, taking the maximum across both columns. Stripping
   trailing zeros instead produces a different distribution `{0dp: 39, 1dp: 244,
   …}` because `24.0` then counts as 0 dp. Both measures agree on the three
   boundaries, but only the printed one is used throughout.

2. **Block C differs in distribution, not just precision.** Rows 302–517 have
   larger means (height 14.03 vs 11.48/11.51, width 11.13 vs 9.25/9.22) and a
   different class mix (walnut 30.1% vs 22.0%/20.8%). Blocks A and B are
   near-identical to each other; C is not.

3. **26 exact duplicate rows; 29 rows duplicate another row's `(height, width)`.**
   Coordinate `(7.5, 3.8)` occurs 4 times.

   Duplicates are overwhelmingly **within** a block — only **3 coordinate pairs
   are shared between blocks**, all between A and C. Block B shares nothing
   with either. That is the leakage exposure, and it is small but not zero.

   The per-block exact-dup counts above (6, 0, 17) sum to 23, not 26. The
   3-row difference is exactly those cross-block pairs, which belong to no
   single block:

   ```
   23 within-block  +  3 crossing A↔C  =  26 total
   ```

   The three crossing rows are `(16.5, 17.0, maple)`, `(10.0, 4.5, cherry)`,
   `(11.0, 7.0, cherry)`. All three are A↔C — the same 3 pairs counted in the
   coordinate-duplicate check, so both duplicate counts agree on the leakage.

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

**Provenance — RESOLVED.** The dataset is published by the course itself, at

> <https://tomasvicar.github.io/MLR-public/data/leaves.csv>

which is the `Download leaves.csv` link in the Exercise 5 description at
<https://tomasvicar.github.io/MLR-public/exercises/ex05.html>. The assignment
describes it as *"518 leaves, three tree types; you know it from the earlier
labs"*.

Verified rather than assumed — our copy is **byte-identical** to the published
one:

```
$ shasum -a 256 data/leaves.csv   # and the freshly downloaded copy
b24ad6369c3c29c8cb35084357940aff335d8fa79b3ee4d94559c670f8a4e839  both
10024 bytes, 519 lines, in both
```

So the data is reproduced from a fetchable URL, not a file of unknown origin.
Anyone re-running this can `curl` that link and get the same bytes. (Note the
local file's 2026-10-14 08:39 timestamp is just a download/extract artefact.)

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

**Models — five, one per family from the lectures.**

| Model | Hyperparameters | Grid |
|---|---|---|
| Logistic regression | `C` | `{0.01, 0.1, 1, 10}` |
| k-NN | `n_neighbors`, `weights` | `k ∈ {1,3,5,7,11,15,21,25}`, `{uniform, distance}` |
| SVM (RBF) | `C`, `gamma` | `{0.1,1,10,100}` × `{scale, 0.01, 0.1, 1}` |
| Decision tree | `max_depth`, `min_samples_leaf` | `{2,3,4,5,6,None}` × `{1,3,5,10}` |
| Random forest | `n_estimators`, `max_depth` | `{200,500}` × `{3,5,8,None}` |

One model per family rather than several flavours of the same thing: linear,
distance-based, margin-based, single tree, ensemble. That gives an honest spread
of what "classical" can do here.

**Scaling lives inside the pipeline.** Logistic regression, k-NN, and SVM all
get a `StandardScaler` as the first pipeline step. This is not cosmetic: if the
scaler is fit on the whole dataset before cross-validation, the fold statistics
leak and every CV score comes out too optimistic. Putting it in the pipeline
means it is refit on each training fold only. Trees do not need it — and adding
it would not change them, since they split on order, not distance.

**Test set.** 20% of rows, `train_test_split(..., stratify=y, random_state=42)`.
Measured, not assumed:

```
test = 104 rows   (maple 43, cherry 35, walnut 26)   train = 414 rows
```

Exact counts are 103.6, so it rounds to 104 — and those class counts are why
`stratify` is not optional: without it the rare class can be badly
under-represented in 104 rows and the test score becomes noise.

Held out **before any tuning**, and touched exactly once at the end. The seed is
fixed at 42 and written down, so the same 104 leaves come back every run.

**The repeated rows — keep all 518, then prove the choice does not matter.**
This is the question the assignment asks directly, so it gets an explicit answer
rather than silence:

- *Primary result:* all 518 rows, as published. Nothing is quietly deleted from
  the course's data.
- *Sensitivity check:* the entire experiment re-run with the 29
  coordinate-duplicate rows dropped (518 → 489), reported side by side.
- *Why this is defensible:* duplicates are overwhelmingly **within** a block,
  and only **3 coordinate pairs cross blocks** (all A↔C). A stratified random
  split keeps ~80% of each duplicate group in train, so the exposure is small.
  We already know the features barely move — pooled Pearson goes 0.5894 →
  0.5923. If the accuracy also barely moves, the choice is *shown* not to
  matter, which is a stronger statement than asserting it.
- The dedup decision is applied identically to every model and to the baseline,
  or the comparison is unfair.

**The 3 conflicting labels.** Kept as-is in the primary run. They are
irreconcilable — identical dimensions, two different answers — so **no model can
score 100%**; at most 515 of 518 rows. That is a property of the data, not a
defect to patch, and picking a winner for them would be inventing information.
Within the 104-row test set they are at most a couple of rows, so the effect is
inside the noise; the sensitivity run covers it.

**Metric — accuracy selects; three more are reported.** Accuracy is the selector
precisely because the assignment defines the baseline *as* the majority-class
rate, so both are on the same scale and "beat the baseline" is a like-for-like
comparison. Reported alongside it, because accuracy alone can hide a class being
written off: **balanced accuracy** (macro-average recall), **macro-F1**, and the
**confusion matrix**. Walnut is only 24.9% of the data, so a model that never
predicts walnut could still post a respectable accuracy — the extras make that
visible. This is a real risk here, given the overlap in finding 5.

**Which data each number comes from** — these must not be conflated:

| Number | Source | Used for |
|---|---|---|
| baseline accuracy | the 104 test rows | the floor to beat |
| CV accuracy (mean ± std) | 5-fold stratified CV **on the 414 train rows** | selecting the model |
| final accuracy | the 104 test rows, once | the headline number |

The baseline is computed **on the test rows** (41.35%, i.e. 43/104), not as the
41.7% whole-dataset rate. Same denominator as the model's score, or the
comparison is sloppy.

**Tuning.** Exhaustive `GridSearchCV`, `cv=StratifiedKFold(n_splits=5,
shuffle=True, random_state=42)`, `scoring='accuracy'`, `refit=True`. All grids
above are small enough to search exhaustively — the largest single grid is the
decision tree at 24 combinations, and the whole sweep is 68 candidates:

| Model | Candidates |
|---|---:|
| logistic regression | 4 |
| k-NN | 16 |
| SVM (RBF) | 16 |
| decision tree | 24 |
| random forest | 8 |
| **total** | **68** |

That is 340 fits across 5 folds, plus 68 refits on the full training set = 408
fits, on 414 rows with 2 features. **Budget cap: 20 minutes wall-clock for the
full sweep**, which is a guard against a pathological grid rather than an
expectation — at this size the sweep should finish in well under a minute. If it
does overrun, the sweep reports where it stopped rather than silently dropping
models. `n_jobs=-1`.

**Class imbalance — no correction.** The majority class is only 41.7%, so the
baseline is already weak and the classes are not badly skewed on a 3-class
scale. `class_weight='balanced'` would be worth trying, but it changes what the
model optimises and so muddies the "beat the baseline" comparison. Instead the
imbalance is *reported*: balanced accuracy and per-class recall next to the
accuracy, so a sacrificed class is visible.

**Environment.** `uv init` + `uv add` happen **first**, before any modelling
code — `AGENTS.md` §2 requires it, and a result produced outside a pinned
environment is not reproducible. `scikit-learn`, `pandas`, and `scipy` get
pinned via `uv.lock`, which is committed. Quarto is installed for step 6 only.

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

**Files that must exist in the repository.**

| Path | What it is |
|---|---|
| `SPEC.md` | this file |
| `AGENTS.md`, `CLAUDE.md`, `README.md`, `PLAN.md`, `RUNS.md` | working rules and tracking |
| `data/leaves.csv` | the data, byte-identical to the published copy |
| `pyproject.toml`, `uv.lock` | pinned environment |
| `ml_study.py` | the experiment: split, baseline, grid search, save results |
| `sensitivity.py` | the same experiment with coordinate-duplicates dropped |
| `results.csv` | one row per model per run (see columns below) |
| `test_predictions.csv` | the chosen model's predictions on the 104 test rows |
| `report.qmd` → `report.html` | the HTML report |
| `slides.qmd` → `slides.html` | the five-slide presentation |

**`results.csv` columns** — every trial, not just the winner (step 4 requires
"all trial results saved to files"):

```
run, model, params, cv_accuracy_mean, cv_accuracy_std,
cv_balanced_accuracy, cv_macro_f1, test_accuracy,
test_balanced_accuracy, test_macro_f1, fit_seconds
```

`run` is `primary` or `dedup`, so both appear in one file and the sensitivity
comparison is a filter, not a second document.

**`test_predictions.csv` columns:**

```
Leaf height, Leaf width, Tree type, predicted, correct
```

**Numbers that must appear, with the precision shown:**

| Number | Where | Example form |
|---|---|---|
| test-set size, seed, split fraction | report, first paragraph | "104 leaves, `random_state=42`, 20%" |
| baseline accuracy **on the test set** | report, next to every model score | 41.35% (43/104) |
| per-model CV accuracy as mean ± std | results table, 4 significant figures | 0.7834 ± 0.0201 |
| chosen model and its exact hyperparameters | report, as a literal dict | `{'n_neighbors': 7, 'weights': 'distance'}` |
| chosen model's test accuracy | report headline | 3 significant figures |
| balanced accuracy, macro-F1, confusion matrix | report, beside the accuracy | full 3×3 matrix |
| class counts (whole data and test set) | Context, already there | 216 / 173 / 129 |

The rule for all of them: **generated from `results.csv`, never retyped.** A
number typed by hand into the report is a number that can silently disagree with
the run.

**Checks a reader can run.** All from a clean checkout, no global installs:

```bash
git clone <repo> && cd mlr-agent-lab
shasum -a 256 data/leaves.csv    # expect b24ad636...f8a4e839
uv sync                          # installs pinned deps, nothing global
uv run python ml_study.py        # reproduces results.csv and test_predictions.csv
uv run python sensitivity.py     # reproduces the 'dedup' rows
uvx quarto render                # rebuilds report.html and slides.html
```

Additional checkable claims:

- Re-running `ml_study.py` twice produces **identical** `results.csv`. That is
  what the fixed seed buys, and it is the single easiest check to run.
- The chosen model's test accuracy **exceeds the baseline's 41.35%**. If it does
  not, the honest conclusion is that these two measurements are not predictive
  of tree type — and that gets written down, not hidden.
- The dedup run's numbers are reported next to the primary ones, and the report
  states plainly whether the conclusion changed.
- Every confusion matrix row sums to that class's test count (43 / 35 / 26).

**The three-block structure: discuss, do not silently ignore.** Blocks A and B
are near-identical (means 11.48 vs 11.51 height); block C differs (14.03). If
the chosen model's errors concentrate in block C, that is a real finding about
the data and belongs in the report. The report states the block structure, and
`test_predictions.csv` keeps the original row order so block membership is
recoverable. It is not treated as a third experimental factor — that would be a
different study — but it is not hidden either.

**Out of scope, explicitly**, so the work does not drift:

- PCA or any dimensionality reduction — with 2 features there is nothing to
  reduce, and the lecture's PCA-in-CV point is already covered by pipeline
  scaling.
- Git worktree experiments and the optional extras in the assignment.
- Any tuning of the baseline — it is a fixed reference point, 43/104.
- Deep learning, or anything not a "classical classifier from the lectures".
- Chasing a higher number by re-splitting. The seed is 42; if it changes, it
  changes once, before the work, and the change is recorded.

`AGENTS.md` §3 governs the final claim: the report cites the **command** that
was run and its **real output**. Not "should reproduce" — reproduced.

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

## Resolved in the interview

- [x] Provenance of `data/leaves.csv` — resolved: published by the course,
      byte-identical copy verified by hash.
- [x] Metric — accuracy selects; balanced accuracy, macro-F1 and the confusion
      matrix reported beside it.
- [x] Repeated rows — keep all 518; a duplicate-free run is reported alongside.
- [x] Which classifiers — five, one per family from the lectures.
- [x] Blocks in the write-up — discussed as a finding, not a third factor.
- [x] Order of work — `uv init` and dependency pinning come first.

## Still open — not yet answerable from the repo

- [ ] **Solo or in pairs?** The assignment allows pairs and requires each student
      to keep their own repository and record who did what. If it is a pair,
      `RUNS.md` needs a "who did what" section. Klára, this one is yours.
- [ ] **Fate of the step-1 scratch scripts** (`check_leaves.py`,
      `check_leaves2.py`, `check_leaves3.py`, `analyze_leaves.py`,
      `verify_dups.py`, `verify_dedup.py`, `verify_blocks.py` — all untracked).
      Worth knowing: `check_leaves3.py` contains an assertion that was **wrong**
      (it reported 0 duplicates because it inserted a unique row index before
      `drop_duplicates()`). `verify_dups.py` replaced it. Recommend: delete the
      three `check_leaves*.py` and fold the findings into `ml_study.py`'s
      docstring, keeping the `verify_*.py` files since the report cites their
      numbers. Decision needed before they are committed or deleted.
- [ ] **Does `leaves_scatter.png` get committed?** The report regenerates it, so
      the committed artefact is arguably redundant — but step 1 delivered it.
- [ ] **Commit email.** `git config user.email` is `klarka.selingrova@gmail.com`.
      Fine for a public repo only if she is happy publishing that address; a
      GitHub noreply address avoids it.

## Next action

Step 3: `/plan Read SPEC.md and propose a plan`, whose first move is *"First
list anything you would have to guess."* With this file the list should be
short — the remaining guessing is confined to the open items above.
