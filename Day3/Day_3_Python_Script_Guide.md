# Day 3 — From Notebook to Script

## Why move out of the Notebook?

In the notebook you could run one groupby, look at the result, adjust, and try the merge again.
That's how you found the right aggregation and caught the inner-merge row loss.

Now that the analysis is settled, a script lets us:

- reproduce the exact same business answer every time, from the same raw source,
- keep `Online Retail.xlsx` untouched,
- always produce the same summary file,
- prove — with printed counts — that grouping and merging did what you expected.

`day3_guided_script.py` reproduces the same steps as `Day_3_Guided.md`. No new Pandas concepts —
only how the process is organized.

---

## The shape of the script

```text
Load → Validate Input → Prepare → Transform → Summarize → Combine → Save
```

```python
def main():
    df = load_data()

    if df is None:
        return

    if not validate_columns(df):
        return

    df = prepare_data(df)
    df = transform_data(df)

    summary = summarize_by_country(df)
    combined = combine_with_region(summary)

    save_output(combined)
```

Read `main()` and you have the whole story: load it, confirm it's the right file, redo Day 2's
cleaning, add the columns this analysis needs, group, enrich with the region table, save.

---

## Functions that change the data vs. functions that don't

`prepare_data`, `transform_data`, `summarize_by_country`, and `combine_with_region` each hand
back something new — a cleaned `df`, a `df` with extra columns, a summary table, an enriched
table — so each one is called with `df = ...` or `summary = ...` / `combined = ...`. That's the
same rule from Day 1 and Day 2: a function that produces new data returns it and gets reassigned.

---

### `load_data()`

**Purpose:** Load the raw Excel file.

**Input:** None. **Returns:** The DataFrame, or `None` if the file can't be found.

```python
def load_data():
    try:
        df = pd.read_excel("../Online Retail.xlsx")
        return df
    except FileNotFoundError:
        print("Error: Online Retail.xlsx was not found.")
        return None
```

Same pattern as Day 1 and Day 2 — the one place a missing file can stop everything is the one
place we use `try/except`.

---

### `validate_columns(df)`

**Purpose:** Confirm the file has the columns the rest of the script depends on.

Identical idea to Day 1 and Day 2's version. `main()` stops if it returns `False`.

---

### `prepare_data(df)`

**Purpose:** Reapply exactly the two changes Day 2 approved — nothing more.

**Input:** The raw DataFrame. **Returns:** The deduplicated, whitespace-stripped DataFrame.

```python
def prepare_data(df):
    rows_before = df.shape[0]

    df = df.drop_duplicates().copy()
    df["Description"] = df["Description"].str.strip()

    print("Rows before:", rows_before, "| Rows after:", df.shape[0])
    return df
```

**Why is this here?** Day 3 doesn't repeat Day 2's investigation — it trusts Day 2's *decision*
and reapplies it. The before/after row count is still printed, because "prepared data" only
means something if you can prove it.

---

### `transform_data(df)`

**Purpose:** Add the derived information this analysis actually needs.

**Input:** The prepared DataFrame. **Returns:** The DataFrame with `Revenue` and
`Transaction_Type` added.

```python
def transform_data(df):
    df["Revenue"] = df["Quantity"] * df["UnitPrice"]

    def classify_transaction(revenue):
        if revenue < 0:
            return "Return"
        elif revenue == 0:
            return "No Charge"
        else:
            return "Sale"

    df["Transaction_Type"] = df["Revenue"].apply(classify_transaction)
    return df
```

**Why a function defined inside a function?** `classify_transaction` is only ever used right
here, for this one column. Keeping it local keeps it next to the only place it matters — you
don't need to go hunting for it elsewhere in the file.

---

### `summarize_by_country(df)`

**Purpose:** Turn row-level transactions into one summary row per country.

**Input:** The transformed DataFrame. **Returns:** A summary DataFrame — sum, mean, and count of
`Revenue` for real sales, per country.

```python
def summarize_by_country(df):
    sales_only = df[df["Transaction_Type"] == "Sale"]
    summary = (
        sales_only.groupby("Country", as_index=False)["Revenue"]
        .agg(["sum", "mean", "count"])
        .sort_values("sum", ascending=False)
    )
    return summary
```

**Why filter before grouping?** The business question is about real sales — including returns
and no-charge rows in a revenue total would misrepresent it. Filter first, then group, the same
order you'd reason about it out loud.

**Why `as_index=False`?** `groupby()` normally turns `Country` into the index. The next step
merges on `Country` as a normal column, so we tell `groupby()` to leave it as one from the
start.

---

### `combine_with_region(summary)`

**Purpose:** Enrich the country summary with a region, and prove the merge didn't quietly lose
data.

**Input:** The country summary. **Returns:** The summary with a `Region` column added.

```python
def combine_with_region(summary):
    rows_before = summary.shape[0]

    combined = pd.merge(summary, REGION_LOOKUP, on="Country", how="left")

    rows_after = combined.shape[0]
    unmapped = combined["Region"].isna().sum()

    print("Rows before merge:", rows_before, "| Rows after merge:", rows_after)
    print("Countries with no region match:", unmapped)
    return combined
```

**Why `how="left"` here, specifically?** We want every country's revenue represented in the
final output, even the ones missing from `REGION_LOOKUP` — dropping them (what `how="inner"`
would do) would silently understate total revenue. `left` keeps everything and makes the gap
visible instead of hiding it.

---

### `save_output(combined)`

**Purpose:** Save the result as a new file.

```python
def save_output(combined):
    combined.to_csv("country_region_revenue_summary.csv", index=False)
    print("Saved summary to country_region_revenue_summary.csv")
```

`Online Retail.xlsx` is only ever read. This script's entire output is one small summary file —
not a modified copy of the original transactions.

---

## Running the script

From inside the `Day_3` folder:

```text
python day3_guided_script.py
```

You should see the row counts settle at `536,641` after `prepare_data`, a `Transaction_Type`
breakdown, the top 5 countries by revenue, then the merge check — `38` rows before and after the
`left` merge, with `23` countries showing no region match. `country_region_revenue_summary.csv`
appears next to the script; `Online Retail.xlsx` is untouched.

---

## What we left out on purpose

Day 3 Guided's final synthesis question (Section 9) isn't hardcoded here as its own function —
this script already answers one specific version of it (`combine_with_region`'s printed region
totals). A script is for reproducing an answer you've already settled on, not for exploring new
questions — that's still the notebook's job.

---

## The pattern to reuse later

```text
load → validate input → prepare (reapply prior decisions) → transform → summarize → combine → save
```

Functions that produce new data return it and get reassigned. Every transformation that could
silently go wrong — cleaning, grouping, merging — prints something that proves it didn't.
