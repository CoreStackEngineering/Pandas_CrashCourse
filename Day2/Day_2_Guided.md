# Day 2 — Deciding What To Do About It

# Commands We Will Use Today

### Loading the Data

`pd.read_excel()`

### Checking Duplicates

`duplicated()` · `sum()` · `drop_duplicates()`

### Checking Missing Data

`isna()` · `dropna()` · `fillna()`

### Filtering the Data

Boolean filtering

### Cleaning Text

`notna()` · `.str.strip()`

### Converting Data Types

`pd.to_numeric()` · `errors="coerce"`

### Working with Dates

`pd.to_datetime()` · `.dt.year` · `.dt.month` · `.dt.day`

### Reviewing the Result

`shape` · `dtypes` · `describe()` · `value_counts()`

### Saving the Cleaned File

`to_csv()`

> We will learn what these commands do and when to use them as we work through today's exercise.

## Where We Left Off

Yesterday you investigated the raw file and wrote a Data Quality Report. You did not fix
anything — you measured. Here's what you found:

- `Country` and `InvoiceDate` look clean.
- `CustomerID` is missing on about 24.93% of rows (135,080 rows).
- `Description` is missing on 1,454 rows.
- 5,268 rows are fully duplicated.
- 10,624 rows have a negative `Quantity`.
- 2,517 rows have `UnitPrice` at or below zero.

None of that was fixed. Today it's the team's job to decide what to actually do about each of
these findings — and to prove, with numbers, that whatever you changed did what you intended.

### Today's mindset

```
Measure  →  Decide  →  Clean  →  Verify
```

Every change needs a reason before you make it, and a check after you make it. The goal is
**not** to make the dataset look clean. The goal is to make justified changes and prove what
changed.

### Not every finding gets the same treatment

For each issue from yesterday, we'll sort it into one of three buckets:

| Bucket | Meaning |
|---|---|
| **Clearly actionable** | We have enough evidence to make the change safely. |
| **Needs a business decision** | The data is unusual, but we don't have enough context to change it without guessing. |
| **Leave untouched** | The value is probably valid business data. |

A negative `Quantity` doesn't automatically get deleted. A missing `CustomerID` doesn't
automatically get filled in. Cleaning is a decision, not a command.

### Protecting the raw file

`Online Retail.xlsx` stays exactly as it is — today and always. We reload it fresh, work on a
DataFrame in memory, and at the end save our result as a **new** file:

```text
Online Retail.xlsx  →  Pandas cleaning steps  →  online_retail_cleaned.csv
```

Why bother? If a cleaning decision turns out to be wrong, we need to be able to start over from
the original data — not from an already-modified file. Keeping the raw source untouched is what
makes that possible.

### Your cleaning log

Keep this table open as you work. Fill in a row every time you make (or explicitly decide
against) a change:

| Issue | Before | Action | After | Reason |
|---|---:|---|---:|---|
| Exact duplicate rows | | | | |
| Missing `CustomerID` | | | | |
| Missing `Description` | | | | |
| Negative `Quantity` | | | | |
| `UnitPrice` ≤ 0 | | | | |
| `Description` whitespace | | | | |

By the end of today this table becomes your Cleaning Report.

---

## Section 1 — Reload the Raw Data

**What are we doing?** Starting fresh, exactly like yesterday.

**Run:**

```python
import pandas as pd

df = pd.read_excel("../Online Retail.xlsx")
df.shape
```

**Look at your result:** `df.shape` should be exactly what you found yesterday.

**Expected:**

```text
(541909, 8)
```

If your number is different, stop and check which file you loaded before continuing.

---

## Section 2 — Duplicates: A Decision We Can Actually Make

**What are we trying to fix?** The 5,268 fully duplicated rows found yesterday.

**Why does this qualify as "clearly actionable"?** Look back at the duplicate pair you inspected
on Day 1 — same invoice, same product, same quantity, same price, identical **down to the
minute**. Two separate legitimate purchases don't naturally land on every single field at once,
including the exact timestamp. The far more likely explanation is that the same line got
recorded twice. Removing the extra copy doesn't lose any real transaction — it removes a repeat
of one.

**Before we change anything:**

```python
rows_before = df.shape[0]
duplicates_before = df.duplicated().sum()

rows_before
duplicates_before
```

**Expected:**

```text
rows_before:       541909
duplicates_before:   5268
```

**Run:**

```python
df = df.drop_duplicates()
```

**What does this command do?** `drop_duplicates()` keeps the first occurrence of every row and
removes the rest. We reassign the result back to `df` — this only changes the DataFrame in
memory, not the Excel file.

**Verify:**

```python
rows_after = df.shape[0]
rows_removed = rows_before - rows_after

rows_after
rows_removed
df.duplicated().sum()
```

**Expected:**

```text
rows_after:      536641
rows_removed:      5268
duplicated().sum(): 0
```

`rows_removed` should match `duplicates_before` exactly. If it doesn't, something unexpected
happened and you need to investigate before moving on.

**Update your cleaning log:** Exact duplicate rows — Before: 5,268 — Action: Removed with
`drop_duplicates()` — After: 0 — Reason: full-row matches including exact timestamp are
essentially certain to be repeated entries, not repeated purchases.

---

## Section 3 — Missing Data: Why We're Not Rushing to Fix It

**What are we checking?** Whether yesterday's missing-value findings can be safely resolved
today.

Pandas gives us two obvious tools here: `dropna()` (remove rows with missing values) and
`fillna()` (fill them in). Beginners often reach for `dropna()` first. Let's see why that
instinct is dangerous here — **without actually doing it**:

```python
df.dropna().shape[0]
```

**Look at your result:** Compare this to `df.shape[0]`.

**Expected:**

```text
df.shape[0]:          536641
df.dropna().shape[0]: 401604
```

**What did we learn?** Running `dropna()` with no arguments removes *any* row that's missing
*anything* — in this case, over 135,000 rows, almost all of them because `CustomerID` is empty.
We never assigned the result back to `df`, so nothing was actually removed. That's the point:
we looked before we leaped, and what we saw told us not to jump.

**Why can't we fix this safely?**

- **`CustomerID` missing (135,037 rows, ≈25.16%):** we have no way to know who these
  transactions belong to. Deleting them throws away ~25% of legitimate sales. Filling them in
  would mean inventing a customer identity that doesn't exist. Neither is safe.
- **`Description` missing (1,454 rows):** small in volume, but tied to the write-off-style
  entries Day 1's Bonus round surfaced — filling in a made-up product name would misrepresent
  what those rows actually are.

**`fillna()` — seeing the syntax without misusing it**

`fillna()` is genuinely useful — just not on these two columns today. Here's the pattern on a
tiny example, so you know it when you need it:

```python
sample = pd.DataFrame({"Rating": [4, None, 5, None, 3]})
sample["Rating"].fillna(0)
```

**What does this command do?** It replaces every missing value with whatever you pass in — here,
`0`. That's a real decision about what "missing" should mean, made deliberately for this
example. We are **not** applying this to `CustomerID` or `Description` — there's no value we
could put there that wouldn't be a guess.

**Update your cleaning log:**

- Missing `CustomerID` — Before: 135,037 (≈25.16%) — Action: Kept as-is — After: 135,037 —
  Reason: no safe way to recover or invent a customer identity; flag for a business decision.
- Missing `Description` — Before: 1,454 — Action: Kept as-is — After: 1,454 — Reason: tied to
  non-standard entries found on Day 1; inventing text would hide that.

---

## Section 4 — Numbers That Don't Look Right: Deciding, Not Deleting

**What are we checking?** Whether the negative `Quantity` and `UnitPrice ≤ 0` rows from Day 1 can
be resolved today.

**Recall what Day 1's Bonus round found:** negative quantities were a *mix* — most were real
cancellations (`InvoiceNo` starting with `"C"`), but some were internal stock write-offs
(`Description` values like `"damaged"`, `"check"`, `"thrown away"`). The `UnitPrice ≤ 0` rows
included real product rows at `0.00` and financial entries like `"Adjust bad debt"` at a large
negative price.

**Why this stays a business decision:** we don't have a rule that reliably tells a cancelled sale
apart from a stock write-off apart from a data-entry mistake, just from the columns in front of
us. Guessing which is which — and quietly deleting or "correcting" some of them — would replace
one data quality problem with a worse one: confident-looking numbers that are actually wrong.

**Confirm the counts still hold after removing duplicates:**

```python
(df["Quantity"] < 0).sum()
(df["UnitPrice"] <= 0).sum()
```

**Expected:**

```text
Negative Quantity: 10587
UnitPrice <= 0:      2512
```

(Slightly lower than Day 1's numbers — a few of the rows we deduplicated in Section 2 happened
to fall into these groups too.)

**Update your cleaning log:**

- Negative `Quantity` — Before: 10,587 — Action: Kept as-is — After: 10,587 — Reason: mixes
  legitimate cancellations with internal write-offs; can't safely separate them yet.
- `UnitPrice` ≤ 0 — Before: 2,512 — Action: Kept as-is — After: 2,512 — Reason: includes real
  `"Adjust bad debt"` entries, not only errors.

---

## Section 5 — Fixing Text You Can't See

**What are we trying to fix?** Invisible whitespace hiding inside `Description`.

The plan for this one has an extra step compared to Section 2's cleanup:
**Measure → Preserve Before State → Transform → Compare → Verify.** We're about to overwrite a
column, so before we do, we'll keep a copy of it — that way we can always compare "before" and
"after," even after the change is made.

**Measure — before we change anything:**

```python
desc = df["Description"]
has_extra_space = desc.notna() & (desc != desc.str.strip())
has_extra_space.sum()
```

**What does this command do?** `desc.str.strip()` returns a new Series with leading and
trailing whitespace removed from each value. Comparing it to the original tells us, row by row,
which values actually had extra whitespace. We combine that with `desc.notna()` — using `&`,
the same way you combined conditions on Day 1 — so that missing values don't get miscounted.

**Expected:**

```text
112372
```

That's about 1 in 5 rows — a real, sizeable problem, not a cosmetic one.

**Preserve the before state.** We're about to overwrite `Description`, so save a copy of it
first — not just its missing-value count, but the actual values, in case we need to look back:

```python
description_before = df["Description"].copy()
```

**What does this command do?** `.copy()` takes a snapshot of the Series as it is right now.
Once we change `df["Description"]`, `description_before` stays exactly as it was — our record
of "before."

**Transform:**

```python
df["Description"] = df["Description"].str.strip()
```

**What does this command do?** `str.strip()` removes leading/trailing whitespace from every
text value. We reassign it back into `df["Description"]`. `.str.strip()` is intended to remove
leading and trailing whitespace without changing the meaning of valid text values. However, as
with any transformation, we should verify the result instead of assuming it was harmless.

**Compare before and after:**

```python
missing_before = description_before.isna()
missing_after = df["Description"].isna()

missing_before.sum()
missing_after.sum()
```

**Expected:**

```text
missing_before.sum(): 1454
missing_after.sum():  1455
```

**What changed?** One more missing value than we started with. Stripping whitespace should
never *create* a missing value — so something else happened. Because we kept
`description_before`, we don't have to guess which row it was:

```python
newly_missing = df[missing_after & ~missing_before]
newly_missing
```

This combines two conditions with `&` and `~`, the same way you did on Day 1 — rows that are
missing *now* but were **not** missing *before*.

**Look at what that row used to say, straight from the copy we preserved:**

```python
description_before[newly_missing.index]
```

**Expected:** the original value was the number `20713` — not text at all. `str.strip()` only
works on strings; when it hit a number, Pandas quietly turned it into `NaN`.

**What did we learn?** A cleaning step that looks completely safe — removing invisible spaces —
just created a brand-new missing value, because one row had a number sitting where a product
description should be. Preserving `description_before` *before* making the change is what let us
go back and prove exactly what happened, instead of being left with only the "after" picture and
no way to explain it.

**Verify the whitespace itself is actually gone:**

```python
desc = df["Description"]
(desc.notna() & (desc != desc.str.strip())).sum()
```

**Expected:**

```text
0
```

**Update your cleaning log:** `Description` whitespace — Before: 112,372 rows affected —
Action: `.str.strip()` — After: 0 (plus 1 new missing value, folded into the missing-Description
count) — Reason: invisible whitespace changes nothing about meaning, only representation; the
side effect was caught by preserving a before-copy and comparing, not assumed away.

---

## Section 6 — Numbers Hiding in Text

**What are we checking?** A column can *look* numeric to a human and still be stored as text —
which silently breaks any math you try to do on it.

Good news: check `Quantity` and `UnitPrice` — they're already `int64` and `float64`. Pandas
parsed them correctly when we loaded the file, so there's nothing to fix here today. But this
problem is common enough in real files that it's worth knowing the tool before you need it.

**A small example:**

```python
sample = pd.DataFrame({"Price": ["3.50", "4.00", "N/A", "5.25"]})
pd.to_numeric(sample["Price"])
```

**What does this command do?** `pd.to_numeric()` tries to convert every value to a number. Run
the line above and you'll get an error — `"N/A"` isn't a number, and Pandas refuses to guess.

**Now try:**

```python
pd.to_numeric(sample["Price"], errors="coerce")
```

**Look at your result:** `"N/A"` became `NaN` instead of raising an error.

**What did we learn?** `errors="coerce"` doesn't fix the bad value — it converts it into a
missing value. Before conversion, `sample["Price"].isna().sum()` was `0`; after, it's `1`. This
is the same lesson from Section 5, generalized: **any cleaning step that touches every row can
quietly create new missing values.** Always check `isna().sum()` before and after, on real
columns too.

No log entry needed here — nothing on the real dataset changed.

---

## Section 7 — Dates: Getting Ready for Time-Based Questions

**What are we checking?** Whether `InvoiceDate` is really usable as a date, not just text that
looks like one.

```python
df["InvoiceDate"].dtype
```

**Expected:**

```text
datetime64[ns]
```

**What did we learn?** Pandas already parsed this column correctly when we loaded the file — if
it hadn't, we'd convert it with `pd.to_datetime(df["InvoiceDate"])`, and we'd want to check
afterward whether any dates failed to parse (the same `errors="coerce"` idea from Section 6
applies here too). Since it's already correct, there's nothing to convert today — but it's worth
knowing why this matters: a text column that merely *looks* like a date can't be sorted by time,
compared, or broken into year/month/day. A real datetime column can:

```python
df["InvoiceDate"].dt.year.unique()
df["InvoiceDate"].dt.month.head()
df["InvoiceDate"].dt.day.head()
```

`.dt` unlocks date-specific fields once a column is truly a datetime. We're only looking at what
becomes possible here — grouping sales by month is a Day 3 problem.

**Expected:**

```text
years: [2010, 2011]
```

No log entry needed — verified, no action required.

---

## Bonus Investigation — For Early Finishers

Not required for your Day 2 deliverable. This looks at a real but low-volume issue: casing.

**What are we checking?** Whether the same word appears more than once in `Description`, just
with different capitalization.

```python
df[df["Description"].str.lower() == "damaged"]["Description"].value_counts()
```

**Expected:**

```text
damaged    43
Damaged    14
DAMAGED     1
```

Three different capitalizations of the exact same word, counted as three separate categories.
Try the same thing for `"check"`.

**Why isn't this in the core exercise?** Together these casing variants affect a couple hundred
rows out of 536,641 — real, but small. Normalizing casing across the *entire* `Description`
column (with `.str.upper()`, for example) would fix this, but it would also flatten any
meaningful casing elsewhere in the file that we haven't checked. Try it yourself:

```python
df["Description"].str.upper()
```

Then decide: would you apply this to the whole column? What would you check first before doing
that for real?

### One Conversion We're Deciding Not to Make

**What are we checking?** Should `CustomerID` become a whole-number type, since customer IDs are
logically integers?

**Try it:**

```python
df["CustomerID"].astype(int)
```

**Look at your result:** This raises an error —
`Cannot convert non-finite values (NA or inf) to integer`.

**What did we learn?** You can't put a missing value into a plain integer column — Pandas has no
way to represent "no value" as an `int`. This isn't a bug to work around; it's a direct
consequence of the ~25% missing `CustomerID` values we already decided not to touch in Section 3.

**The decision:** we know *how* to force this conversion (Pandas has an `Int64` type built for
exactly this), but we're intentionally not doing it today — it's tangled up with the same open
question from Section 3. That's a legitimate Data Engineering decision, not a gap in your
skills. This one isn't added to your cleaning log — it's here to round out your understanding of
how missing values and dtypes interact.

---

## Section 8 — Final Validation

Before saving anything, run one last check using the same tools from Day 1 — now used as
**verification** tools instead of discovery tools.

```python
df.shape
df.isna().sum()
df.duplicated().sum()
df.dtypes
df.describe()
df["Country"].value_counts().head(5)
```

Go through each result and sort what you see into three groups:

**Fixed**

- Exact duplicate rows: 5,268 → 0.
- `Description` whitespace: 112,372 rows affected → 0.

**Still present (left alone, on purpose)**

- Missing `CustomerID`: 135,037 rows.
- Missing `Description`: 1,455 rows.
- Negative `Quantity`: 10,587 rows.
- `UnitPrice` ≤ 0: 2,512 rows.

**Needs a business decision**

- Whether negative `Quantity` should be split into "cancellation" vs. "write-off."
- Whether `CustomerID` should ever be converted to an integer type once its missing values are
  resolved.
- Whether `Description` casing should be normalized dataset-wide.

A finished cleaning pass does **not** mean zero nulls and zero unusual values. It means you can
list, with numbers, exactly what changed, what didn't, and why.

---

## Section 9 — Save the Cleaned Output

```python
df.to_csv("online_retail_cleaned.csv", index=False)
```

**Why a new file?** The original `Online Retail.xlsx` is still exactly as it was this morning —
we never touched it. This new CSV is our cleaned *result*, not a replacement for the source. If
tomorrow we decide one of today's choices was wrong, we can reload the raw Excel file and start
over.

**Why `index=False`?** Without it, Pandas would write its own row-number index as an extra
column in the CSV. We don't need that saved — the data speaks for itself.

---

## Day 2 Cleaning Report

Your finished cleaning log from today:

| Issue | Before | Action | After | Reason |
|---|---:|---|---:|---|
| Exact duplicate rows | 5,268 | Removed (`drop_duplicates()`) | 0 | Full-row matches incl. exact timestamp are essentially certain to be repeated entries |
| Missing `CustomerID` | 135,037 (≈25.16%) | Kept as-is | 135,037 | No safe way to recover or invent identity — needs a business decision |
| Missing `Description` | 1,454 | Kept as-is | 1,455* | Tied to non-standard entries from Day 1 — inventing text would hide that |
| Negative `Quantity` | 10,587 | Kept as-is | 10,587 | Mixes cancellations and write-offs — can't safely separate yet |
| `UnitPrice` ≤ 0 | 2,512 | Kept as-is | 2,512 | Includes real "Adjust bad debt" entries, not only errors |
| `Description` whitespace | 112,372 rows affected | Stripped (`.str.strip()`) | 0 | Invisible whitespace changes nothing about meaning |

\* One row's `Description` held a number, not text, and became missing as a side effect of
stripping. Caught by re-measuring, not assumed away — folded into the same "needs a decision"
bucket as the rest of the missing `Description` rows.

**What we found:** the same issues Day 1 flagged, now measured precisely on the deduplicated
dataset.

**What we changed:** removed 5,268 exact duplicate rows; stripped whitespace from 112,372
`Description` values.

**Why we changed it:** both changes are meaning-preserving — they remove noise without altering
what any row represents.

**How we verified it:** `duplicated().sum()` returned to `0`; the whitespace check returned to
`0`; row counts were compared before and after every change.

**What we intentionally did not change:** missing `CustomerID`, missing `Description`, negative
`Quantity`, `UnitPrice` ≤ 0, and `Description` casing.

**What still needs a business decision:** whether to split negative `Quantity` into
cancellations vs. write-offs, whether `CustomerID` should later become an integer type, and
whether `Description` casing should be normalized dataset-wide.

Deliverables from today: this notebook, `online_retail_cleaned.csv`, and the report above.
