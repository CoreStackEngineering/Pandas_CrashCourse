# Day 3 Independent — Answering Business Questions

## Before You Start

Day 1 you investigated. Day 2 you cleaned and validated. Today you analyze — using the same
dataset, carried forward through the same pipeline.

```text
Independent Raw Dataset → Day 1: Investigate → Day 2: Clean + Validate → Day 3: Analyze
```

**The dataset:** the file you saved at the end of Day 2 Independent (your cleaned version of
`ecommerce_fulfillment_dirty.csv`). If you didn't save one, use `ecommerce_fulfillment_dirty.csv`
directly and re-apply the duplicate removal from yesterday first — you'll need a duplicate-free
starting point for today's counts to mean anything.

**The rules are the same as every day:**

- Treat your input file as raw. Never overwrite it.
- Every transformation gets verified — a merge or a groupby that "ran" is not the same as one
  that did what you expected.
- Some data-quality issues from yesterday (missing values, the negative `Delivery_Days` rows)
  are still in this data, on purpose. Today's tasks work *around* them, the same way Day 3
  Guided worked around what Day 2 Guided didn't fix.

### Commands available to you today

Everything below was taught in Day 3 Guided, or earlier. Nothing else is required.

```text
Sorting:       sort_values()
Grouping:      groupby() · sum() · mean() · min() · max() · count() · size() · agg()
Derived data:  vectorized arithmetic · Series.apply()
Combining:     pd.merge() · on= · how="inner" · how="left"

Still available from Day 1 & Day 2:
  head() · shape · columns · dtypes · info() · describe()
  isna() · sum() · duplicated() · drop_duplicates() · sort_values()
  unique() · nunique() · value_counts()
  Boolean filtering · .loc[] · .iloc[]
  notna() · .str.strip() · .str.replace()
  pd.to_numeric() · errors="coerce" · pd.to_datetime() · .dt.year/.month/.day/.days
```

---

## Task 1 — Load and Re-verify

Load your Day 2 Independent output. Before doing anything else, confirm it's actually the
prepared data you think it is:

- Row count and column names match what you expect.
- `duplicated().sum()` is `0`.
- Quick dtype check on the columns you'll use today — are they what you need them to be?

If anything looks wrong, that's worth fixing before you build on top of it — the same lesson as
"prepared data" from this morning.

---

## Task 2 — Sort to Find What Matters

Find the orders that sit at the extremes of shipping cost or delivery time.

- Identify the highest and lowest values in a numeric column of your choice.
- Look at the actual rows behind them, not just the number.

---

## Task 3 — Classify With `apply()`

`Delivery_Days` describes how long an order took — but a raw number isn't always the most useful
way to communicate that to the business.

- Write a small function that turns `Delivery_Days` into a category (for example: fast, normal,
  slow).
- Decide what your function should do with a value that's negative. You investigated exactly
  this problem on Day 2 Independent — use that finding to justify your decision, whatever you
  choose.
- Apply your function and look at how many rows fall into each category.

---

## Task 4 — Group and Aggregate

Pick a business question that compares a numeric column *by category* — for example, how
delivery time or shipping cost differs across product categories or shipping methods.

- Decide which column you're grouping by and which aggregation actually answers your question
  (revisit Day 3 Guided's reasoning: what would `sum()` vs. `mean()` vs. `count()` each tell
  you here?).
- Run it, and sanity-check the result: does the number of groups match how many categories you
  actually expect? If not, that's worth a second look before you trust the numbers.

### Hint

> If a column you're grouping by still has more distinct values than it should, that's not a
> bug in `groupby()` — it's telling you something about how far yesterday's cleaning got.

---

## Task 5 — Combine With a Reference Table

You want to report on delivery performance by **department**, not just by individual product
category. That grouping doesn't exist in your data yet — it lives in a small reference table.

```python
department_lookup = pd.DataFrame({
    "Product_Category": ["Electronics", "Home", "Fashion", "Beauty", "Sports"],
    "Department": ["Tech & Appliances", "Tech & Appliances", "Apparel & Style",
                   "Apparel & Style", "Leisure"]
})
```

- Decide what you're merging (hint: an aggregated summary, not the full row-level data, will be
  easier to reason about here — the same shape as Day 3 Guided's country/region merge).
- Try it with `how="inner"` first, and check the row count before and after.
- Try it again with `how="left"`. What changed?
- Decide which one actually answers your business question, and explain why in one sentence.

---

## Task 6 — Answer a Business Question End to End

Choose one and answer it completely, showing your steps:

> Which department has the best (lowest) average delivery time?

or

> Ignoring rows with a known date/duration data-quality issue, which shipping method delivers
> most consistently?

Plan it yourself using this shape:

1. Decide if you need to filter anything out first.
2. Group by whatever answers "by what."
3. Choose the aggregation that answers the actual question.
4. Merge in the reference table if your question needs it.
5. Sort so the answer is visible at the top.
6. State the answer in one sentence, with the number.

---

## Task 7 — Save and Report

Save your final result (the merged/aggregated table, not the full row-level dataset) as a new
file — for example `ecommerce_department_summary.csv`. Never overwrite your input file.

Finish with a short report:

- The business question you answered, and the answer.
- Which merge type you used, and why.
- Any groupby result that came out with more categories than you expected, and what that tells
  you about the data going into today.
- Anything you deliberately left unresolved, and why.
