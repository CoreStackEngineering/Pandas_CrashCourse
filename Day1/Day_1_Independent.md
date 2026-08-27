# Day 1 Independent — Investigating a New Dataset

## Before You Start

This morning you learned how to approach an unfamiliar dataset systematically: understand its
structure, inspect its columns and types, measure missing data, investigate duplicates, inspect
important values, and document what you discovered — without changing anything.

Now you'll repeat that process on your own, on a different dataset.

**The dataset:** `ecommerce_fulfillment_dirty.csv` — an export from an order fulfillment system.
Each row represents one order: where it was shipped, how, and when.

**The rules are the same as this morning:**

- The file is your raw input. Never overwrite it.
- You are investigating, not cleaning. Nothing gets fixed today.
- Every claim needs a number behind it. "Something looks off" is not a finding —
  "14 rows show X" is.

### Commands available to you today

Everything below was taught this morning. Nothing else is required.

```text
Loading:      pd.read_csv()
Inspecting:   head() · sample() · shape · columns · dtypes · info() · describe()
Missing data: isna() · sum()
Duplicates:   duplicated() · sort_values()
Values:       unique() · nunique() · value_counts()
Filtering:    Boolean filtering, e.g. df[df["column"] < value]
```

(`pd.read_csv()` works exactly like `pd.read_excel()` — same idea, different file type.)

---

## Task 1 — Understand the Dataset

Load the file and figure out what you're actually looking at.

Investigate and determine:

- How large is the dataset?
- What columns does it contain, and what kind of information does each one appear to hold?
- Are the Pandas data types consistent with what each column represents? Look at every column,
  not just the ones that seem obviously numeric or obviously text.

### Hint

> A column can display numbers when you print `head()` and still not be stored as a number.
> Check `dtypes` for every column that *looks* numeric or date-like — don't assume.

---

## Task 2 — Missing Data

Investigate whether the dataset contains missing information.

- Which columns have missing values, and how many?
- What percentage of the dataset does that represent for each one?
- Which of those columns do you think matter enough to flag for a future decision?

---

## Task 3 — Duplicates

Determine whether the dataset contains repeated records.

- Measure how many fully duplicated rows exist.
- Look at one actual duplicate pair, not just the count. Does every column match, including the
  order identifier?
- Explain in one or two sentences why this might have happened, and why it matters for any
  report built on this data.

---

## Task 4 — Category Consistency

Investigate whether the categorical columns are represented consistently.

Pick the columns that describe a business category (not a name or an ID), and check whether
values that should represent the same category are being counted as different ones.

### Hint

> If the number of unique values in a column seems larger than the number of real-world
> categories it should have, inspect how those values are actually written — not just what they
> say.

---

## Task 5 — Suspicious Numeric Values

Investigate the dataset's numeric columns.

- Run a numeric sanity check across the dataset. Does every column you'd expect to be numeric
  actually show up in the result?
- For the column(s) that behave unexpectedly, compare what `describe()` returns to what you'd
  expect from a normal price-like column.
- For the column(s) that *do* behave as proper numbers, look at the minimum and maximum values.
  Do they make business sense for what the column represents? If not, look at a few of those
  rows directly.

Do not delete or change anything — just measure what you find and note it.

### Hint

> A column full of numbers isn't automatically a numeric column to Pandas. If `describe()`
> across the whole DataFrame is missing a column you expected to see, that's a clue worth
> following up with `dtypes`.

---

## Task 6 — Data Quality Findings

Put together a short findings summary — the same kind of report you built this morning, split
into two parts.

**Measured facts** (numbers only — a small table or a printed list is enough):

- Rows and columns
- Data types worth flagging, and why
- Missing values per affected column (count and %)
- Fully duplicated rows
- Categorical columns with inconsistent values, and how many distinct values you found vs. how
  many you'd expect
- Any numeric columns with unexpected minimum/maximum values

**Open questions** — in your own words:

1. Which columns can you trust to be complete, and which can't?
2. Is every unusual value you found necessarily an error? What would you need to know to be
   sure?
3. What would you tell a colleague *not* to do yet with this data, and why?
4. What decisions still need more information before anything can safely be changed?

This report is today's deliverable. Nothing in the dataset should be different from how you
received it — you should be able to explain exactly what you found, without having changed a
single value.
