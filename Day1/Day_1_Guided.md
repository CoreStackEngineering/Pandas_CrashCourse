# Day 1 — Can We Trust This Data?

# Commands We Will Use Today

### Loading the Data

`pd.read_excel()`

### Inspecting the Dataset

`head()` · `sample()` · `shape` · `columns` · `dtypes` · `info()` · `describe()`

### Checking Missing Data

`isna()` · `sum()`

### Checking Duplicates

`duplicated()` · `sort_values()`

### Exploring Values

`unique()` · `nunique()` · `value_counts()`

### Filtering the Data

Boolean filtering

> We will learn what these commands do and when to use them as we work through today's exercise.

## Business Scenario

You just joined the Data Engineering team at an online retail company. Management wants
reliable sales reports: revenue, top products, customer behavior, sales by country.

Before anyone calculates a single number, someone has to answer a more basic question first:

> **What data did we actually receive, and can we trust it enough to continue working with it?**

That's your job today. You are investigating a raw file — `Online Retail.xlsx` — and writing a
**Data Quality Report** that the rest of the team will rely on before they touch this data.

### Ground rules

The raw file is **immutable**. Today you must not: edit the Excel file, remove rows, fill in
missing values, normalize text, convert data types, or delete duplicates. If you find something
strange, **describe it clearly** — don't fix it. Cleaning comes later, once we understand what
we're cleaning and why.

### The mindset for today

```
Understand  →  Inspect  →  Validate  →  Document findings
```

Whenever you're tempted to make a claim about the data, use this loop instead of guessing:

```
Observe  →  Measure  →  Investigate  →  Conclude
```

"There are duplicates" is not a finding. "There are 5,268 duplicated rows" is.

---

## Section 1 — Load the Raw Dataset

**What are we checking?** Can we load the file into Pandas without touching the original?

**Run:**

```python
import pandas as pd

df = pd.read_excel("Online Retail.xlsx")
```

**What does it do?** `pd.read_excel()` reads the Excel file into a **DataFrame** — a table of
rows and columns. We store it in `df`, the standard name for "the DataFrame."

**Look at your result:** Just confirm it ran with no error and `df` looks like a table.

---

## Section 2 — First Look at the Dataset

**What are we checking?** What does one transaction look like, and how big is this dataset?

**Run:**

```python
df.head()
df.sample(5)
df.shape
df.columns
```

**What does it do?** `head()` shows the first 5 rows. `sample(5)` grabs 5 *random* rows instead
— useful because the top of a file is often its cleanest part, and a 500,000-row file can hide
plenty in the middle. `shape` gives you `(rows, columns)`; `columns` lists the column names.

**Look at your result:** Compare `head()` and `sample()` — does anything look different between
them? Read the column names — do they look like what you'd expect for retail transactions?

**Expected:**

```text
Rows: 541,909
Columns: 8
```

Columns: `InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country`.
Each row is one product line on one invoice.

---

## Section 3 — Data Types and Structure

**What are we checking?** Does Pandas understand each column the way we'd expect — numbers as
numbers, dates as dates?

**Run:**

```python
df.dtypes
df.info()
```

**What does it do?** `dtypes` lists the type Pandas assigned to every column. `info()` adds row
counts and a non-null count per column on top of that.

**Look at your result:** Check which columns are numeric (`int64`/`float64`), which are text
(`object`), and which are dates. Then look closer at `CustomerID` — is it `int64` or `float64`?
A customer ID should logically be a whole number. Also compare the **non-null count** `info()`
reports for each column against the total row count (`df.shape[0]`) — do any columns fall short?

**What did we learn?** `CustomerID` is stored as `float64`, not `int64` — Pandas does this
automatically whenever an integer column contains missing values. That's your first hint,
straight from the dtype itself, that `CustomerID` has gaps we'll measure properly in Section 5.

### DataFrame, Series, Index — in practice

You've already been using all three:

```python
df               # the whole table -> a DataFrame
df["Quantity"]   # one column -> a Series
df.index         # the row labels -> an Index
```

Selecting a single column with `df["ColumnName"]` returns a **Series** — a single labeled
column of data, not a DataFrame. Keep an eye out: several tools coming up (`isna()`,
`duplicated()`, Boolean filters) also return a Series.

---

## Section 4 — Numeric Sanity Check

**What are we checking?** Before `Quantity` and `UnitPrice` ever get used to calculate revenue,
what does their overall range look like?

**Why does it matter?** Revenue is basically `Quantity × UnitPrice`. If either column holds
values that don't make sense for a normal sale, we need to know *before* building a report on
top of them.

**Run:**

```python
df.describe()
```

**Look at your result:** Ignore most of the table for now. Find the `min` row for `Quantity`
and for `UnitPrice`. Then look at their `max` row. Ask yourself: can a real sale have a negative
quantity? A negative unit price?

**What did we learn?** Both `Quantity` and `UnitPrice` go negative — and their maximums are
large too. That doesn't automatically mean the data is wrong. It means we've found something
worth a closer look, which we'll do properly with real rows in Section 8.

---

## Section 5 — Missing Data

**What are we checking?** Are important fields missing information, and how much of the dataset
does that affect?

**Why does it matter?** A report built on `CustomerID` — say, "revenue per customer" — is only
as reliable as how complete that column is. We need an exact number, not an impression.

**Build the investigation:**

```python
df.isna()
```

Every cell becomes `True` (missing) or `False` (present). With over half a million rows, you
can't scan this by eye — which is exactly why we count it:

```python
df.isna().sum()
```

`.sum()` on a column of `True`/`False` values counts the `True`s, giving one missing-value count
per column.

**Look at your result:** Which columns show a number greater than 0? How does that number
compare to `df.shape[0]`?

**What did we learn?** Turn the counts into percentages using arithmetic you already know:

```python
df.isna().sum() / len(df) * 100
```

**Expected:**

```text
Description missing:  1,454   (≈0.27%)
CustomerID  missing: 135,080  (≈24.93%)
```

Roughly **1 in 4 rows has no `CustomerID`**. That doesn't mean we delete those rows today — it
means any future customer-level report is automatically based on about 75% of the data, and
that has to be stated, not hidden.

---

## Section 6 — Duplicates

**What are we checking?** Could some records have been entered more than once?

**Why does it matter?** A duplicated row means one invoice line gets counted twice — quietly
inflating every quantity and revenue total built on top of it.

**Build the investigation:**

```python
df.duplicated()
```

This returns a Series of `True`/`False` — `True` means this row is an exact repeat of an
earlier row. Count how many:

```python
df.duplicated().sum()
```

**Expected:**

```text
Duplicated rows: 5,268
```

A count alone doesn't tell a story — look at real rows next. Save the Boolean result, then
filter with it:

```python
duplicate_mask = df.duplicated(keep=False)
df[duplicate_mask].sort_values(by=["InvoiceNo", "StockCode"]).head(6)
```

(`keep=False` here marks *both* copies of a duplicate, not just the extra one — so you can see
the full matching pair side by side.)

**Look at your result:** Pick one pair with the same `InvoiceNo`. Is every column identical,
down to the exact timestamp?

**What did we learn?** Some pairs look like the exact same product line entered twice on the
same invoice, same minute. That's suspicious — but `duplicated()` only tells us rows are
*identical*, not *why*. Two different customers legitimately buying the same product on the same
day would **not** trigger this — only exact, full-row repeats do. We are not deleting anything;
this goes into the report as an open question.

---

## Section 7 — Categorical Exploration

**What are we checking?** What countries actually appear in this data, and how evenly is the
data spread across them?

**Why does it matter?** "Sales by country" is a report management wants. Before trusting it, we
need to know whether one country dominates so heavily that country comparisons could be
misleading.

**Build the investigation:**

```python
df["Country"].nunique()
```

`nunique()` counts how many *distinct* values a column has — a quick check before listing
anything, since some columns (like `Description`) have thousands of distinct values, too many to
print. `Country` is small enough to list directly:

```python
df["Country"].unique()
```

Now look at how the rows are actually distributed across those countries:

```python
df["Country"].value_counts()
```

`value_counts()` counts how many times each value appears, sorted from most common to least.

**Look at your result:** Which country sits at the top? Roughly what share of the total 541,909
rows does it represent?

**What did we learn?**

**Expected:**

```text
Distinct countries: 38
Top: United Kingdom — 495,478 rows (≈91% of all data)
```

Almost all rows come from one country. That's not a data quality *error*, but it's a fact that
changes how a "sales by country" report should be read.

Now reuse `nunique()` the same way, without re-explaining it, on two more columns:

```python
df["Description"].nunique()
df["StockCode"].nunique()
```

**Look at your result:** Both return figures in the thousands. That confirms `unique()` would
dump an unreadably long list for either column — good instinct to check `nunique()` first.

---

## Section 8 — Investigating Suspicious Records

We already spotted two things worth a closer look: negative values in Section 4, and duplicate
rows in Section 6. Now let's look directly at the rows behind the numeric anomalies, using
simple Boolean filtering.

**What are we checking?** What do the rows with negative `Quantity` actually look like?

**Build the investigation:**

```python
df["Quantity"] < 0
```

This condition alone returns a Series of `True`/`False` — one per row. Wrap it in `df[...]` to
pull out only the rows where it's `True`:

```python
df[df["Quantity"] < 0]
```

**Look at your result:** Scan the `InvoiceNo` column. Notice anything about how many of them
start with the same letter?

**What did we learn?** Negative `Quantity` shows up on thousands of rows. We can already see a
pattern forming in `InvoiceNo` — but confirming it, and checking whether it explains *every*
negative row, needs a closer look than we have time for in the core exercise. That's a great
candidate for the Bonus section below. For now, the finding stands as-is: **negative quantities
exist, and we don't yet know if they all mean the same thing.**

**What are we checking?** What about the negative `UnitPrice` values we saw in `describe()`?

**Build the investigation:**

```python
df[df["UnitPrice"] <= 0]
```

**Look at your result:** Check the `Description` column for these rows — does it look like a
normal product line?

**What did we learn?** Some rows have `UnitPrice` at or below zero. At a glance, a few don't
look like ordinary product sales at all. We're flagging this as an open question rather than
deciding anything about it today.

**The key lesson:** a command returning results is not the same as understanding them. Every
finding above raises a question for Day 2, not an answer for today.

---

## Bonus Investigation — For Early Finishers

Everything here is optional and **not required** for your Day 1 report. It goes deeper into
*why* the anomalies from Section 8 happen — genuinely useful, but not necessary to judge whether
the dataset is safe to move forward with.

### Are all negative quantities cancellations?

Look again at the `InvoiceNo` values from Section 8 — many start with `"C"`. We can check that
directly:

```python
cancelled = df["InvoiceNo"].astype(str).str.startswith("C")
cancelled.sum()
```

> `.astype(str).str.startswith("C")` is beyond what Day 1 covers — you're not expected to master
> string methods yet. All you need to know is: this line produces `True` for every row whose
> `InvoiceNo` starts with `"C"`.

Now compare that count to the negative-quantity count from Section 8:

```python
(df["Quantity"] < 0).sum()
cancelled.sum()
```

The two numbers are close but not equal. Look at the negative-quantity rows that do **not**
start with `"C"`:

```python
df[(df["Quantity"] < 0) & (~cancelled)].head(10)
```

Read the `UnitPrice` and `Description` columns for these rows. What kind of entry does this look
like — a customer cancelling an order, or something else happening internally (stock damage,
write-offs, manual corrections)?

### What is `StockCode` actually holding?

```python
df["StockCode"].value_counts().head(15)
```

A few of the most frequent `StockCode` values — `POST`, `M`, `D`, `S`, `AMAZONFEE` — aren't
products at all. Look up what each row containing them actually represents. If a future report
says "top-selling products," should these be counted?

### What does `UnitPrice <= 0` actually contain?

Revisit the rows from Section 8. Some carry a `Description` like `"Adjust bad debt"`. Is that a
product sale, or a financial adjustment that happens to live in the same file?

**The lesson from the bonus round:** none of these stories were visible from a single column in
isolation. They only appeared by comparing `Quantity` against `InvoiceNo`, and `UnitPrice`
against `Description`. **Real data quality issues usually hide in the relationship between
columns, not inside one column alone.**

---

## Section 9 — Day 1 Data Quality Report

No cleaning, no decisions about what "should" happen next — just a clear, measured account of
the data as it stands today, in two parts: what we know for a fact, and what still needs a
decision.

**Build the report — Measured Facts:**

```python
facts = {
    "Total rows": df.shape[0],
    "Total columns": df.shape[1],
    "Missing Description": df["Description"].isna().sum(),
    "Missing CustomerID": df["CustomerID"].isna().sum(),
    "Missing CustomerID (%)": round(df["CustomerID"].isna().sum() / len(df) * 100, 2),
    "Fully duplicated rows": df.duplicated().sum(),
    "Distinct countries": df["Country"].nunique(),
    "Negative Quantity rows": (df["Quantity"] < 0).sum(),
    "UnitPrice <= 0 rows": (df["UnitPrice"] <= 0).sum(),
}

pd.DataFrame(facts.items(), columns=["Metric", "Value"])
```

**Risks / Open Questions — answer in your own words, in a markdown cell:**

1. Can every negative `Quantity` be treated the same way (as a cancelled order)? What would you
   need to check before assuming that?
2. Can every row in this file safely be treated as a normal product sale?
3. About a quarter of rows have no `CustomerID`. What does that mean for any future
   customer-level analysis?
4. The duplicated rows found in Section 6 — are they double-entries, or could there be another
   explanation? What would you need to know to decide?
5. What would you tell the analytics team **not** to do yet with this data, and why?

This report is the Day 1 deliverable. It becomes the starting point for Day 2, where we start
making — carefully, and with reasons — the cleaning decisions this file clearly needs.


