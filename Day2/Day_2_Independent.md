# Day 2 Independent — Cleaning What You Found

## Before You Start

This morning you moved from finding data-quality problems to handling them carefully. You
learned that cleaning is not just running a command:

```text
Measure  →  Decide  →  Transform  →  Verify
```

Yesterday you investigated `ecommerce_fulfillment_dirty.csv` and documented what you found.
Today you decide what to actually do about it — using the same reasoning from this morning: only
change what you can justify, and prove every change worked.

**The rules are the same as this morning:**

- `ecommerce_fulfillment_dirty.csv` is your raw input. Never overwrite it.
- Every transformation needs a reason before you make it, and a check after.
- Not everything gets fixed today. Some things get documented as open questions instead — that's
  a legitimate outcome, not a shortcut.

### Commands available to you today

Everything below was taught yesterday or this morning. Nothing else is required.

```text
From today:
  Duplicates:    duplicated() · sum() · drop_duplicates()
  Missing data:  isna() · dropna() · fillna()
  Cleaning text: notna() · .str.strip() · .str.replace()
  Converting:    pd.to_numeric() · errors="coerce"
  Dates:         pd.to_datetime() · .dt.year · .dt.month · .dt.day · .dt.days
  Reviewing:     shape · dtypes · describe() · value_counts()
  Saving:        to_csv()

From Day 1 (still available):
  head() · sample() · columns · info() · sort_values() · unique() · nunique()
  Boolean filtering, e.g. df[df["column"] < value]
  .loc[] · .iloc[]
```

---

## Task 1 — Reload and Confirm

Start fresh, exactly like this morning: load the raw file and confirm its shape matches what you
found yesterday. If it doesn't, stop and check which file you loaded.

---

## Task 2 — Duplicates: A Decision You Can Actually Make

Yesterday you found fully duplicated rows.

- Record the row count and duplicate count *before* changing anything.
- Decide: is removing them justified? Look again at one duplicate pair — does anything about the
  order identifier support treating this as a repeated ingestion event rather than a coincidence?
- Apply the transformation you've decided is safe.
- Verify: row count after, duplicate count after, and confirm the number of rows removed matches
  what you expected.

---

## Task 3 — Missing Data: Revisit Yesterday's Decision

You already measured missing values in `Delivery_Date` and `Shipping_Cost`. Today, decide what
to actually do about them — on the deduplicated data.

- Before deciding anything, check what would happen if you ran `dropna()` with no arguments.
  How many rows would that remove? Don't apply it — just look.
- For each affected column, decide: keep as-is, or is there a change you can make that isn't
  just a guess? Write down your reasoning for each one.
- Update your findings with the exact counts on the deduplicated data (they may differ slightly
  from yesterday).

---

## Task 4 — Category Consistency: How Far Can You Get?

Yesterday you found that a categorical column has more distinct values than real categories.

- Apply the one text-cleaning tool you know that's safe here — it doesn't change what any value
  *means*, only how it's written.
- Measure the number of distinct values before and after.
- Did the problem fully disappear? If some inconsistency remains, look at what kind it is. Is it
  something you can safely fix with what you've learned so far, or is it a decision to document
  and leave for later?

Do the same check for any other categorical column you flagged yesterday.

---

## Task 5 — Shipping Cost: Getting to a Real Number

Yesterday you noticed a numeric-looking column wasn't behaving like one.

- Measure how many missing values this column currently has, before touching anything.
- Inspect it more closely: why would a column that's supposed to hold prices end up stored as
  text instead of a number? Check its dtype, then look at a sample of its actual values — not
  just the ones that look normal.

### Hint

> A column can contain numbers and still be stored as text because of how some values are
> formatted. Inspect several values before deciding what needs to be removed.

- Once you can see what's actually inside the unusual values, decide: is there a safe, simple way
  to remove that formatting using a tool from this morning's Guided exercise? Apply it only to
  the formatting you can clearly identify — don't guess at what else might be wrong.
- Only now convert the column to a proper numeric type, using the approach that turns anything it
  still can't understand into a missing value rather than crashing.
- Verify: measure missing values again and compare that to your very first count. Did your
  cleanup recover the values you expected? If the conversion still produced new missing values
  you didn't account for, investigate those directly rather than deleting or replacing them
  automatically.

**Once the column is numeric where possible:**

- Look at its minimum and maximum. Do any values look implausible for a shipping cost (for
  example, at or below zero, or far above the typical range)?
- Do not delete these. Measure how many there are and decide whether this is something you can
  resolve today or something that needs a business decision, the same way you handled unusual
  numeric values this morning.

---

## Task 6 — Dates: Make Them Usable

You noticed yesterday that the date columns aren't stored the way `InvoiceDate` was in this
morning's dataset.

- Convert the order, ship, and delivery date columns to real datetime values.
- Verify: check the dtype after conversion, and check whether the conversion produced any new
  missing values. What would a new missing value here actually mean?

---

## Task 7 — Does the Timeline Make Sense?

An order should be placed, then shipped, then delivered — in that order. Now that your dates are
real datetime values, investigate whether that's actually true for every row.

- Decide what "the timeline makes sense" means in terms of a comparison between two columns.
- Measure how many rows break that rule, for each pair of columns you'd expect to be in order.
- Look at a couple of the rows that break it. Are the individual dates themselves invalid, or is
  it just their relationship to each other that doesn't make sense?
- Decide: can you safely correct these, or does this need to be documented as an open question?

### Hint

> You already know how to compare a column to a fixed value (`df["column"] < 5`). Comparing two
> date columns to each other uses the exact same operator — just with a column on both sides
> instead of a column and a number.

---

## Task 8 — Does the Reported Duration Agree With the Dates?

`Delivery_Days` is supposed to describe how long an order took to arrive. Investigate whether
the reported value actually agrees with what the ship and delivery dates say — for the rows
where the timeline from Task 7 is valid.

Measure how many rows disagree. Do not assume it's an error in every case — document what you
find.

### Hint

> You calculated the number of days between two datetime columns in this morning's Guided
> exercise. The same idea applies here — compare that result to `Delivery_Days` on the rows
> where the dates are in valid order.

---

## Task 9 — Final Validation

Before saving anything, run the same kind of checklist from this morning, using these tools as
verification instead of discovery:

```text
shape · isna().sum() · duplicated().sum() · dtypes · describe() · value_counts()
```

Sort what you see into three groups:

- **Fixed** — problems you measured, changed, and confirmed are gone.
- **Still present, on purpose** — things you deliberately left alone, and why.
- **Needs a business decision** — things you don't have enough information to resolve safely.

A completed cleaning pass does not mean zero missing values and zero unusual numbers. It means
you can explain exactly what changed, what didn't, and why.

---

## Task 10 — Save the Result

Save your processed DataFrame as a **new** file — for example `ecommerce_fulfillment_cleaned.csv`.
Never overwrite `ecommerce_fulfillment_dirty.csv`.

---

## Day 2 Cleaning Report

Finish with a short report, the same shape as this morning's:

| Issue | Before | Action | After | Reason |
|---|---:|---|---:|---|
| Exact duplicate rows | | | | |
| Missing `Delivery_Date` | | | | |
| Missing `Shipping_Cost` | | | | |
| Category inconsistency | | | | |
| `Shipping_Cost` stored as text | | | | |
| Unusual `Shipping_Cost` values | | | | |
| Date columns not usable as dates | | | | |
| Invalid timeline order | | | | |
| `Delivery_Days` disagreement | | | | |

Add a few sentences: what you found, what you changed, why, how you verified it, what you
intentionally left alone, and what still needs a business decision.

Deliverables: your cleaned CSV and this report.
