# Day 2 — From Notebook to Script

## Why move out of the Notebook?

In the Notebook you could run one cell, look at the result, and decide what to do next. That's
how you figured out which cleaning steps were safe and which weren't.

Now that those decisions are made, a script lets us:

- repeat the exact same cleaning steps, in the exact same order, every time,
- keep the raw file untouched,
- always produce the same `online_retail_cleaned.csv` from the same input,
- prove — with printed before/after numbers — that each step did what it was supposed to.

`day2_guided_script.py` makes the **same decisions** the guided Notebook made. It does not
resolve anything the Notebook left open. If a column was "kept as-is, needs a business
decision" on Day 2, it's still exactly that here.

---

## The shape of the script

```text
Load
  -> Validate Input
  -> Measure/Decide
  -> Clean
  -> Verify
  -> Save
```

```python
def main():
    df = load_data()

    if df is None:
        return

    if not validate_columns(df):
        return

    df = remove_duplicates(df)

    review_missing_and_unusual_values(df)

    df = clean_description(df)

    validate_data(df)

    save_data(df)
```

One ordering detail worth noticing: duplicates are removed *before* the missing-data and
unusual-value review, even though "Clean" comes after "Measure/Decide" in the phase list above.
That's not an accident — Day 2's Notebook measured missing `CustomerID` and negative `Quantity`
*after* deduplication, so the script removes duplicates first to get the same numbers you saw
there (25.16% missing, not 24.93%). Real pipelines are rarely perfectly linear — what matters is
that every step still happens in a deliberate, explainable order.

---

## Error handling vs. data validation

- **Error handling** (`try/except`) — for something that can stop an operation outright. The
  only case here is the source file not existing, handled once, in `load_data()`.
- **Data validation** (`if` checks on the data itself) — for confirming an operation that *did*
  run produced what we expected. This covers the required-columns check up front, and the
  duplicate/whitespace checks after each cleaning step.

Neither one is used more than it needs to be — one `try/except`, and validation only where a
transformation has a clear expected outcome.

---

## Functions that change `df` vs. functions that don't

Only two functions actually change the data: `remove_duplicates` and `clean_description`. Both
**return** the result, and `main()` reassigns it: `df = remove_duplicates(df)`.

```python
def remove_duplicates(df):
    cleaned_df = df.drop_duplicates()
    return cleaned_df
```

`review_missing_and_unusual_values` and `validate_data` only *look* at `df` and print what they
find — they never change it, so `main()` calls them as plain lines, no `df = ...` in front. If a
function doesn't return a new DataFrame, `main()` doesn't reassign anything after calling it —
that's your signal the data wasn't touched.

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

Same reasoning as Day 1: this is the one place a missing file can realistically stop everything,
so it's the one place we use `try/except`.

---

### `validate_columns(df)`

**Purpose:** Confirm the loaded file has the columns the pipeline depends on.

**Input:** The DataFrame. **Returns:** `True` or `False`.

Identical idea to Day 1 — the file loaded, but we still confirm it's structured the way the rest
of the script expects before doing anything else. `main()` stops if this returns `False`.

---

### `remove_duplicates(df)`

**Purpose:** Remove exact duplicate rows discovered on Day 1.

**Input:** The DataFrame. **Returns:** The DataFrame after duplicate removal.

```python
def remove_duplicates(df):
    rows_before = df.shape[0]
    duplicates_before = df.duplicated().sum()

    cleaned_df = df.drop_duplicates()

    rows_after = cleaned_df.shape[0]
    duplicates_after = cleaned_df.duplicated().sum()

    print("Rows removed:", rows_before - rows_after)

    if duplicates_after == 0:
        print("Duplicate check: PASS")
    else:
        print("Duplicate check: WARNING")

    return cleaned_df
```

This keeps the Notebook's "measure before, change, measure after" pattern intact — a script
doesn't get to skip verification just because no one's watching interactively. The
`PASS`/`WARNING` line is a simple, one-line proof that the transformation did what it was
supposed to; it isn't a testing framework, just an explicit statement of the expected outcome.

---

### `review_missing_and_unusual_values(df)`

**Purpose:** Measure missing `CustomerID`/`Description` and unusual `Quantity`/`UnitPrice`
values, and state why we're deciding not to fix them today.

**Input:** The DataFrame. **Returns:** Nothing — read-only.

This includes the real `df.dropna().shape[0]` check — run on the actual data, never assigned
back to `df` — showing exactly what a careless `dropna()` would cost, without ever doing it.

**A demonstration that didn't make the cut:** the Notebook also showed `fillna()` on a tiny,
made-up example, since there was no real column we could safely fill. That demonstration stayed
in the Notebook and isn't in this script — a script's job is to run the process we actually
decided on, not to re-teach a tool we chose not to use. If you want to see `fillna()` in action
again, `Day_2_Guided.md` still has it.

---

### `clean_description(df)`

**Purpose:** Strip stray whitespace from `Description`, and confirm the fix didn't quietly
create a new problem.

**Input:** The DataFrame. **Returns:** The DataFrame after cleaning.

```python
def clean_description(df):
    description_before = df["Description"].copy()

    df["Description"] = df["Description"].str.strip()

    missing_before = description_before.isna()
    missing_after = df["Description"].isna()

    newly_missing = df[missing_after & ~missing_before]
    print("New missing values created:", len(newly_missing))
    ...
    return df
```

Same sequence as the Notebook: **Measure → Preserve Before State → Transform → Compare →
Verify**. `.copy()` is taken *before* the overwrite specifically so the function can still prove
what changed afterward — not just notice that something looks different. Run it, and you'll see
the exact row (`420391`, original value `20713`) that this catches, live.

---

### `validate_data(df)`

**Purpose:** Run the final checklist and sort today's findings into Fixed, Still Present, and
Needs a Business Decision.

**Input:** The DataFrame. **Returns:** Nothing — read-only.

This reuses the same Day 1 tools — `shape`, `isna().sum()`, `duplicated().sum()`, `dtypes`,
`describe()`, `value_counts()` — now as verification tools instead of discovery tools. It also
includes one direct check that `InvoiceDate` is really a `datetime64[ns]` column:

```python
print("InvoiceDate is a real datetime column:", df["InvoiceDate"].dtype == "datetime64[ns]")
```

**A demonstration that didn't make the cut:** the Notebook also ran a tiny `pd.to_numeric()`
example on made-up text values, since `Quantity` and `UnitPrice` are already numeric in the real
file and never needed converting. That stayed in the Notebook too. This function only validates
what's actually true about our real data — it doesn't reconvert a column that was never broken,
and it doesn't re-run a conversion demo that has nothing to convert.

---

### `save_data(df)`

**Purpose:** Save the cleaned result as a new file.

**Input:** The DataFrame. **Returns:** Nothing.

```python
def save_data(df):
    df.to_csv("online_retail_cleaned.csv", index=False)
```

Last step, on purpose — nothing gets saved until every prior step has already printed its own
proof that it worked.

---

## Running the script

From inside the `Day_2` folder:

```text
python day2_guided_script.py
```

Read the output top to bottom — input validation, duplicate cleaning (with a PASS check),
the missing-data/anomaly decisions, description cleaning (with its own PASS check and the
"new missing value" discovery), final validation, then save.

---

## What we left out on purpose

- **The `fillna()` and `pd.to_numeric()` demonstrations** — both ran on made-up data in the
  Notebook because the real dataset never needed them. A script's job is to reproduce the
  process we actually decided to use, not everything we experimented with along the way.
- **The Bonus Investigation** — `Description` casing and the `CustomerID.astype(int)` attempt —
  isn't part of this script either, for the same reason it's optional in the Notebook: neither
  is required to reach a safe, verified `online_retail_cleaned.csv`.

If you want to revisit any of these, `Day_2_Guided.md` is still the right place — this script's
job is to reliably repeat the decisions that were actually made.

---

## The pattern to reuse later

Strip away the retail-specific details and this script follows a pattern you can reuse on any
dataset:

```text
load
  -> validate input
  -> measure / decide (read-only)
  -> clean (only the changes you actually decided on)
  -> verify
  -> save
```

Functions that change the data return it and get reassigned. Functions that only look at the
data don't. Errors that stop an operation outright get a `try/except`; everything else gets a
plain validation check. That's the whole pattern — the same one you'll reach for on your own
dataset later.
