# Day 3 — Answering Business Questions

## Where We Left Off

Day 1 answered: *can we trust this data?* Day 2 answered: *what do we do about the problems we
found?* You now have a dataset you've measured, cleaned where justified, and can explain.

Today's question is different:

> **Now that the data is prepared, what can it actually tell us?**

The flow for today:

```text
Prepared Data  →  Transform  →  Group  →  Aggregate  →  Combine  →  Answer  →  Verify
```

Same rule as always: every result gets inspected before you trust it, and every transformation
gets verified after you run it.

---

# Commands We Will Use

### Sorting

`sort_values()`

### Grouping & Aggregating

`groupby()` · `sum()` · `mean()` · `min()` · `max()` · `count()` · `size()` · `agg()`

### Derived Columns

Vectorized arithmetic (already known) · `Series.apply()`

### Combining Data

`pd.merge()` · `on=` · `how="inner"` · `how="left"`

---

## Reload Where Day 2 Left You

Start fresh, then reapply exactly the two changes Day 2 approved — nothing more:

```python
import pandas as pd

df = pd.read_excel("../Online Retail.xlsx")
df = df.drop_duplicates()
df["Description"] = df["Description"].str.strip()

df.shape
```

**Expected:** `(536641, 8)` — the same shape Day 2 ended with. Everything else Day 2 flagged
(missing `CustomerID`, negative `Quantity`, `UnitPrice` ≤ 0) is still exactly as it was. Today
we're not resolving those — we're working *around* them, carefully.

---

## Section 1 — A Derived Column: Revenue

**What are we trying to answer?** None of our columns tell us how much money a line actually
made. `Quantity` and `UnitPrice` each tell half the story.

**Build it:**

```python
df["Revenue"] = df["Quantity"] * df["UnitPrice"]
```

**What does this do?** Same idea as `df["Score"] + 5` back in the Pandas Intro — an arithmetic
operation across two columns, applied to every row at once. No loop, no new syntax.

**Look at your result:**

```python
df["Revenue"].describe()
```

**What did we learn?** `Revenue` can be negative — that's expected, not a bug. A negative
`Quantity` (a cancellation, from Day 1) multiplied by a positive `UnitPrice` gives a negative
`Revenue`. The column is telling the truth about data we already know is mixed.

---

## Section 2 — Sorting to Find What Matters

**What are we checking?** Which single transactions moved the most money — in either direction?

**Run:**

```python
df.sort_values("Revenue", ascending=False).head(5)
```

**Look at your result:** Read the `Description` column for these rows, not just the number.

**What did we learn?** The biggest "sale" isn't necessarily a real product sale — you'll spot
`AMAZON FEE` and `Adjust bad debt` near the top, exactly the kind of non-product rows Day 1's
Bonus round flagged. Sorting doesn't clean anything; it just puts the extremes where you can see
them.

**Small task:** Run the same sort with `ascending=True` (or drop the argument and use
`.head()` on the smallest values). What shows up at the other extreme?

---

## Section 3 — Grouping: Answering "By X" Questions

**What are we checking?** Total revenue, broken down by country.

Every "by category" business question follows the same shape:

```text
What am I grouping by?  →  What am I measuring?  →  How should it be combined?
```

**Run:**

```python
df.groupby("Country")["Revenue"].sum().sort_values(ascending=False).head(5)
```

**What does this do?** `groupby("Country")` splits the DataFrame into one group per country.
`["Revenue"].sum()` adds up `Revenue` inside each group separately. Nothing here is new syntax —
it's the same `.sum()` from Day 1's `describe()`-style thinking, just applied per group instead
of to the whole column.

**Look at your result:** One number per country, sorted. `United Kingdom` dominates — expected,
given what you already know about this dataset's country balance from Day 1.

**Small task:** Group by `Country` again, but measure `Quantity` instead of `Revenue`. Does the
ranking change?

---

## Section 4 — Choosing the Right Aggregation

**What are we checking?** Not every question should be answered with `sum()`.

**Run:**

```python
df.groupby("Country")["UnitPrice"].sum().head(5)
```

**Look at your result:** A large number that means... what, exactly? Adding up prices from
thousands of unrelated transactions doesn't describe anything real.

**Now try:**

```python
df.groupby("Country")["UnitPrice"].mean().head(5)
```

**What did we learn?** The question decides the aggregation, not the other way around:

```text
"How much money, in total?"        → sum()
"What's typical?"                  → mean()
"What's the cheapest / priciest?"  → min() / max()
```

Summing a total makes sense for `Revenue` (money that actually accumulates). Summing `UnitPrice`
doesn't — prices don't add up into a meaningful total. Always ask what the number would *mean*
before running the aggregation.

---

## Section 5 — `count()` vs. `size()`

**What are we checking?** How many transactions came from each country.

**Run both:**

```python
df.groupby("Country")["CustomerID"].count().head(10)
df.groupby("Country").size().head(10)
```

**Look at your result:** For most countries these match. For a few, they don't. Check `Hong Kong`
specifically.

**What did we learn?** `size()` counts rows — every row, no matter what's in them. `count()`
counts non-missing values in the column you pick. `Hong Kong` has rows, but `CustomerID.count()`
shows `0` for it — every single Hong Kong transaction is missing a `CustomerID`, something you
already knew existed in general from Day 1, now visible at the country level. Add up the
difference between `size()` and `count()` across every country and you'll land on the exact
missing-`CustomerID` total from Day 1 — same problem, viewed through a new tool.

**Small task:** Which countries show the *largest* gap between `size()` and `count()`? What does
that suggest about relying on `CustomerID` for a per-country customer report?

---

## Section 6 — Several Statistics at Once: `agg()`

**What are we checking?** Total, average, and transaction count for revenue, by country — without
running three separate lines.

**Run:**

```python
df.groupby("Country")["Revenue"].agg(["sum", "mean", "count"]).sort_values("sum", ascending=False).head(5)
```

**What does this do?** `agg()` takes a list of the same aggregation names you already know and
runs all of them at once, one column per statistic. It's not a new concept — it's `sum()`,
`mean()`, and `count()`, just requested together instead of one call at a time.

**Look at your result:** One table, three numbers per country. Notice how a country can have a
high total but a low average, or the reverse — a single summary number can hide the full
picture.

---

## Section 7 — Custom Logic With `apply()`

**What are we checking?** Simple arithmetic (Section 1) can't answer everything. Sometimes you
need to turn a number into a category using a rule with more than one branch.

**Build a small, plain Python function:**

```python
def classify_transaction(revenue):
    if revenue < 0:
        return "Return"
    elif revenue == 0:
        return "No Charge"
    else:
        return "Sale"
```

**Apply it:**

```python
df["Transaction_Type"] = df["Revenue"].apply(classify_transaction)
df["Transaction_Type"].value_counts()
```

**What does this do?** `.apply()` runs your function once per value in the Series and collects
the results into a new column — same "operation across every value" idea as `df["Revenue"] =
df["Quantity"] * df["UnitPrice"]`, just powered by a function you wrote instead of a built-in
operator.

**Why `apply()` here, specifically?** Because the logic has branches (`if` / `elif` / `else`)
that a single arithmetic expression can't express. Classifying "Sale" vs. "Return" vs.
"No Charge" isn't something `+`, `-`, `*`, or `/` can do directly.

**Why not use `apply()` for everything?** Go back to Section 1: `Revenue` was one multiplication.
Writing a function and calling `.apply()` for that would work, but it's slower and harder to
read than the plain arithmetic you already had. **If a vectorized operation solves the problem
cleanly, prefer it. Reach for `apply()` only when the logic genuinely needs branches or custom
rules that arithmetic can't express.**

---

## Section 8 — Combining Datasets: `merge()`

**What are we checking?** `Country` tells us where an order shipped to. It doesn't tell us which
*region* that country belongs to — and "revenue by region" is a more useful business question
than "revenue by all 38 individual countries."

That information lives in a second, small table — exactly the kind of situation `merge()` is
for: two datasets, related by a shared column, that need to become one.

**Build the small lookup table:**

```python
region_lookup = pd.DataFrame({
    "Country": ["United Kingdom", "Germany", "France", "EIRE", "Netherlands", "Spain",
                "Belgium", "Switzerland", "Portugal", "Italy", "Australia", "USA",
                "Canada", "Japan", "Singapore"],
    "Region": ["Europe", "Europe", "Europe", "Europe", "Europe", "Europe",
               "Europe", "Europe", "Europe", "Europe", "Oceania", "North America",
               "North America", "Asia", "Asia"]
})
```

This table only covers 15 of your 38 countries — that's deliberate, and it's about to matter.

**First, prepare what you're merging:**

```python
revenue_by_country = df.groupby("Country", as_index=False)["Revenue"].sum()
revenue_by_country.shape
```

**Expected:** `(38, 2)` — one row per country. `as_index=False` keeps `Country` as a normal
column instead of turning it into the index — you'll need it as a column in a moment, to merge
on.

**Now merge with `how="inner"`:**

```python
inner_result = pd.merge(revenue_by_country, region_lookup, on="Country", how="inner")
inner_result.shape
```

**Look at your result:** `(15, 3)`. Twenty-three countries just disappeared.

**What does this do?** `on="Country"` tells Pandas which column to match rows by.
`how="inner"` keeps only rows where that value exists in **both** tables. Any country missing
from `region_lookup` gets silently dropped — including all of its revenue.

**Now merge with `how="left"` instead:**

```python
left_result = pd.merge(revenue_by_country, region_lookup, on="Country", how="left")
left_result.shape
```

**Look at your result:** `(38, 3)` — every country kept. Check `left_result["Region"].isna().sum()`
— those are exactly the 23 countries with no match, now visible as missing values instead of
silently vanishing.

**What did we learn?**

```text
how="inner"  →  keep only rows that match in both tables — anything unmatched disappears
how="left"   →  keep every row from the left table — unmatched rows get NaN instead
```

**Why this must be verified, not trusted:** the `inner` merge ran without a single error or
warning — and it still quietly deleted a fifth of your data. Compare row counts before and after
*every* merge, the same way you compared row counts before and after `drop_duplicates()` on
Day 2. A merge that "ran successfully" and a merge that "did what you expected" are not the
same claim.

**Small task:** Using `left_result`, group by `Region` (include `dropna=False` so the unmapped
countries show up as their own group) and sum `Revenue`. How much revenue is sitting in that
unmapped group?

---

## Section 9 — Putting It Together

You now have a `Transaction_Type` column, a `Revenue` column, and a way to map countries to
regions. Answer one business question end to end:

> Which region generates the most revenue from actual sales — excluding returns and no-charge
> rows?

Plan your own steps using only what's in this document:

1. Filter to the rows that count as real sales.
2. Group by whatever column answers "which region."
3. Choose the aggregation that answers "generates the most revenue."
4. Sort so the answer is at the top.
5. State the answer in one sentence, with the number.

This is the same shape every question today has followed:
**Question → Tool → Run → Inspect → Interpret → Verify.** That shape is today's real takeaway —
not any single command.
