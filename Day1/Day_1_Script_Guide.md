# Day 1 — From Notebook to Script

## Why move out of the Notebook?

The Notebook was useful because you could run one step at a time, look at each result, and
decide what it meant. That's exploration.

Now that Day 1's process is settled — load, inspect, validate, report — we can put it into a
Python script so that:

- the same process runs again, exactly the same way, every time,
- every run follows the same order, start to finish,
- the raw file stays untouched,
- the output (the report) can be reproduced by anyone who runs the file.

`day1_guided_script.py` does exactly what you already did by hand in `Day_1_Guided.md` — the
same Pandas commands, the same numbers. Nothing new to learn about Pandas here. What's new is
how the code is *organized*.

---

## The shape of the script

```text
Load
  -> Validate Input
  -> Inspect
  -> Validate Data
  -> Report
```

Ten small functions, each with one clear job, called in that order from `main()`:

```python
def main():
    df = load_data()

    if df is None:
        return

    if not validate_columns(df):
        return

    inspect_data(df)
    check_missing_data(df)
    check_duplicates(df)
    explore_categories(df)
    check_suspicious_records(df)

    facts = build_report(df)
    save_report(facts)
```

You can understand the whole program just from reading `main()` top to bottom — that's the
point. Each function name tells you what phase of the process it belongs to.

---

## Error handling vs. data validation

These sound similar but are two different concerns, and the script keeps them separate:

- **Error handling** — something prevented an operation from running at all. The only realistic
  case here is the file not existing. We use `try/except` for this, because there's a real
  decision to make ("stop, and say why") that plain code can't express.
- **Data validation** — the operation *ran fine*, but we still need to check the data is what we
  expect (the right columns, the right counts). This is just `if` statements and Pandas checks —
  no exceptions needed, because nothing actually failed.

You'll see one of each below.

---

### `load_data()`

**Purpose:** Load the raw Excel file.

**Input:** None.

**Returns:** The DataFrame, or `None` if the file couldn't be found.

```python
def load_data():
    try:
        df = pd.read_excel("../Online Retail.xlsx")
        return df
    except FileNotFoundError:
        print("Error: Online Retail.xlsx was not found.")
        return None
```

**Why `try/except` here specifically?** Loading a file is the one place where something outside
our control (a missing or moved file) can realistically stop the whole program. We catch exactly
that one error — `FileNotFoundError` — and nothing else. `main()` checks for `None` right after
calling it:

```python
df = load_data()
if df is None:
    return
```

That's the whole pattern: if loading failed, stop cleanly instead of crashing further down with
a confusing error.

---

### `validate_columns(df)`

**Purpose:** Confirm the file actually has the columns the rest of the script depends on.

**Input:** The loaded DataFrame.

**Returns:** `True` or `False`.

It checks against one list, defined once near the top of the script, right after the imports:

```python
REQUIRED_COLUMNS = [
    "InvoiceNo",
    "StockCode",
    "Description",
    "Quantity",
    "InvoiceDate",
    "UnitPrice",
    "CustomerID",
    "Country",
]
```

```python
def validate_columns(df):
    missing_columns = []
    for column in REQUIRED_COLUMNS:
        if column not in df.columns:
            missing_columns.append(column)

    if missing_columns:
        print("Missing required columns:", missing_columns)
        return False

    print("All required columns are present.")
    return True
```

**Why is this here?** The file loaded successfully, but that doesn't guarantee it's the *right*
file — someone could point the script at a different spreadsheet entirely. This is a data
validation check, not error handling: nothing crashed, we're just confirming the structure is
what the rest of the script expects, the same way you'd glance at `df.columns` yourself before
trusting a new file. `main()` stops if this returns `False`, the same way it stops on a failed
load.

---

### `inspect_data(df)`

**Purpose:** Look at the overall shape, structure, and numeric ranges.

**Input:** The DataFrame. **Returns:** Nothing — it only prints.

This covers `shape`, `dtypes`, and `describe()` — the same commands from Sections 2–4 of the
Notebook. Since it doesn't change `df`, `main()` calls it as a plain line: `inspect_data(df)`,
not `df = inspect_data(df)`.

---

### `check_missing_data(df)`, `check_duplicates(df)`, `explore_categories(df)`, `check_suspicious_records(df)`

**Purpose:** Each one checks exactly one thing you already investigated in the Notebook —
missing values, duplicate rows, country distribution, and negative/zero-or-less numeric values,
respectively.

**Input:** The DataFrame. **Returns:** Nothing — all four only measure and print.

These are four separate functions instead of one big one because each answers a different
question about the data, and each is small enough to explain in a single sentence:

> This function checks how many values are missing in each column.

If you only wanted to know about duplicates, you could call `check_duplicates(df)` on its own
and get exactly that — that's the benefit of keeping them apart.

---

### `build_report(df)`

**Purpose:** Collect the key numbers into one report.

**Input:** The DataFrame. **Returns:** A dictionary of the same measurements you assembled by
hand in Section 9 of the Notebook.

This function recalculates the numbers directly from `df` rather than reusing values from the
`check_*` functions above — a little repetition, but it means `build_report(df)` can be read and
trusted on its own, without having to trace values through four other functions first.

---

### `save_report(facts)`

**Purpose:** Print the report and save it as its own file.

**Input:** The `facts` dictionary from `build_report()`. **Returns:** Nothing.

```python
report_df = pd.DataFrame(facts.items(), columns=["Metric", "Value"])
report_df.to_csv("day1_data_quality_report.csv", index=False)
```

This is a small, separate file — clearly not the raw dataset, and clearly not something you'd
mistake for cleaned data. `Online Retail.xlsx` itself is never touched.

---

## Running the script

From inside the `Day_1` folder:

```text
python day1_guided_script.py
```

You should see the same numbers you found in the Notebook — rows: 541,909; missing `CustomerID`:
135,080 (24.93%); duplicated rows: 5,268; negative `Quantity` rows: 10,624 — followed by
`day1_data_quality_report.csv` being written next to the script.

---

## What we left out on purpose

- **`sample()`** — useful for spotting things by eye, but it returns *random* rows. A script is
  supposed to produce the same result every run, so a random preview doesn't belong here. Use
  the Notebook when you want to look around by hand.
- **`unique()` on `Country`** and **the duplicate-pair preview with `sort_values()`** — both are
  for a human to read and think about, not numbers the report needs. The script keeps
  `nunique()` and `value_counts()`, since those numbers actually feed the report.

If you want to explore the data by eye again, `Day_1_Guided.md` is still the place for that.
This script's job is narrower: run the same checks, the same way, every time.
