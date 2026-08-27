# Pandas Basics — Before the Real Dataset

You just got comfortable with NumPy arrays. Before we open a real dataset, we need to meet the
two structures Pandas is built around: the **Series** and the **DataFrame**.

This is not a Data Quality exercise yet — we're not investigating anything real today. We're
building tiny examples by hand so you can clearly see what Pandas gives you back every time you
ask it for something. That habit — knowing exactly what you're holding — is what makes the real
dataset in `Day_1_Guided.md` much easier to work with.

Expect this to take **60–120 minutes**. By the end you'll be able to:

- explain what a DataFrame is and what a Series is,
- tell the difference between the Index, column labels, and values,
- know exactly what Pandas hands back when you select data in different ways,
- use `.loc[]` and `.iloc[]` without mixing them up,
- and recognize that a CSV file, a JSON file, an API response, and a plain Python dictionary can
  all become the exact same kind of DataFrame.

---

# Commands We Will Use

### Creating Pandas Objects

`pd.Series()` · `pd.DataFrame()`

### Understanding Structure

`.index` · `.columns` · `.shape` · `.dtypes`

### Selecting Data

`[]` · `.loc[]` · `.iloc[]`

### Filtering Data

comparison operators · `df[condition]`

### Loading Data

`pd.read_csv()` · `pd.read_json()`

---

## Section 1 — Import Pandas

**What are we doing?** Loading the library.

**Run:**

```python
import pandas as pd
```

**Short explanation:** Same idea as `np` for NumPy — `pd` is the convention everyone uses.
Nothing prints yet; this just makes Pandas available.

---

## Section 2 — Build a Series Manually

**What are we doing?** Creating the simplest Pandas structure there is: a single labeled
column of values.

**Run:**

```python
scores = pd.Series([85, 90, 78, 92])
print(scores)
```

**Look at the result:**

```text
0    85
1    90
2    78
3    92
dtype: int64
```

**Short explanation:** Look at the two columns of numbers you're seeing. The left one is the
**Index** — automatically generated labels for each value. The right one is the actual data.

```text
Index     Value
0         85
1         90
2         78
3         92
```

The Index is **not** another data column. It's a label attached to each value, so Pandas (and
you) always know which value is which.

**Run:**

```python
print(type(scores))
print(scores.index)
print(scores.values)
```

**Look at the result:**

```text
<class 'pandas.core.series.Series'>
RangeIndex(start=0, stop=4, step=1)
[85 90 78 92]
```

**Short explanation:** `scores` is a `Series`. `.index` shows the labels (here, a simple
auto-generated range). `.values` shows just the raw numbers — that array should look familiar
from yesterday.

**Small task:** Create your own Series of 4 numbers and print it, `.index`, and `.values`.

---

### A Series with custom labels

**What are we doing?** Replacing the default 0, 1, 2, 3 labels with names that mean something.

**Run:**

```python
scores = pd.Series(
    [85, 90, 78],
    index=["student_a", "student_b", "student_c"]
)
print(scores)
```

**Look at the result:**

```text
student_a    85
student_b    90
student_c    78
dtype: int64
```

**Run:**

```python
print(scores["student_b"])
```

**Look at the result:**

```text
90
```

**Short explanation:** Same Series, same values — only the labels changed. You can now grab a
value by its label instead of its position.

**Small task:** Build a Series of 3 prices with your own custom labels, then print one value by
its label.

---

## Section 3 — Keys, Labels, and Values (Clearing Up the Confusion)

Beginners often mix up four very similar-looking ideas: dictionary **keys**, DataFrame
**column names**, row **Index labels**, and the actual **values**. Let's separate them clearly
before DataFrames show up.

**Run:**

```python
student = {
    "name": "Daniel",
    "age": 24,
    "course": "Data Engineering"
}
print(student["name"])
```

**Look at the result:**

```text
Daniel
```

**Short explanation:**

```text
"name"               → key
"Daniel"             → value
student["name"]      → look up a value using its key
```

This is plain Python — nothing new. Keep this picture in mind, because a DataFrame has its own,
similar-looking set of names:

```text
Python dictionary   →   keys                  and   values

Pandas DataFrame     →   column labels
                     →   Index labels          and   cell values
```

A dictionary **key** and a DataFrame **Index label** are **not** the same concept — they belong
to two different structures. What they have in common is only the *role* they play: each one is
a label used to identify and access something within its own structure. A dictionary key looks
something up in a dictionary; an Index label looks up a row in a DataFrame; a column label looks
up a column. Same idea, three separate things — don't treat them as interchangeable.

A **column label** names a column. A **row Index label** names a row. Neither one is the data
itself — both are just names you use to find the data. We'll see this play out for real in the
next two sections.

**Small task:** Write your own tiny dictionary with 3 keys. Print one value using its key, and
say out loud which part is the key and which part is the value.

---

## Section 4 — Create a DataFrame Manually

**What are we doing?** Building a full table — the structure you'll use constantly from now on.

**Run:**

```python
data = {
    "Name": ["Daniel", "Sara", "David"],
    "Age": [24, 27, 22],
    "Score": [85, 91, 78]
}

df = pd.DataFrame(data)
df
```

**Look at the result:**

```text
     Name  Age  Score
0  Daniel   24     85
1    Sara   27     91
2   David   22     78
```

**Short explanation:** Each dictionary key became a **column label**. Each list became that
column's **values**. The numbers on the left (`0, 1, 2`) are the **Index** — one label per row,
generated automatically, just like in Section 2.

**Run:**

```python
print(type(df))
print(df.columns)
print(df.index)
print(df.shape)
```

**Look at the result:**

```text
<class 'pandas.core.frame.DataFrame'>
Index(['Name', 'Age', 'Score'], dtype='object')
RangeIndex(start=0, stop=3, step=1)
(3, 3)
```

**Short explanation:** Match each line to a piece of the table you printed above:

- `type(df)` → this is a **DataFrame**.
- `df.columns` → the column labels: `Name`, `Age`, `Score`.
- `df.index` → the row labels: `0, 1, 2`.
- `df.shape` → `(3, 3)` → 3 rows, 3 columns.

**Small task:** Build your own DataFrame from a dictionary with 3 columns and 4 rows of made-up
data. Print it, then print `.columns`, `.index`, and `.shape`.

---

## Section 5 — Build a DataFrame Another Way

**What are we doing?** Creating the exact same kind of table from a different Python structure —
a list of dictionaries, one dictionary per row.

**Run:**

```python
data = [
    {"Name": "Daniel", "Age": 24, "Score": 85},
    {"Name": "Sara", "Age": 27, "Score": 91},
    {"Name": "David", "Age": 22, "Score": 78}
]

df2 = pd.DataFrame(data)
df2
```

**Look at the result:**

```text
     Name  Age  Score
0  Daniel   24     85
1    Sara   27     91
2   David   22     78
```

**Predict, then check:** Does this look identical to the DataFrame from Section 4? Run
`df.equals(df2)` and see if you predicted correctly.

**Short explanation:** A dictionary-of-lists (columns first) and a list-of-dictionaries
(rows first) are different Python structures — but Pandas turns both into the same DataFrame.
Once your data is a DataFrame, it doesn't matter anymore where it came from.

---

## Section 6 — DataFrame vs Series (the Most Important Distinction Today)

This is the one idea that will save you the most confusion later. Go slowly.

**Predict first:** What do you think `df["Name"]` gives you back — a Series, or a DataFrame?

**Run:**

```python
df["Name"]
```

**Look at the result:**

```text
0    Daniel
1      Sara
2     David
Name: Name, dtype: object
```

**Run:**

```python
print(type(df["Name"]))
```

**Look at the result:**

```text
<class 'pandas.core.series.Series'>
```

**Short explanation:** Selecting **one column with a single set of brackets** returns a
**Series** — a single labeled column, same shape as what you built by hand in Section 2.

**Predict first:** What about `df[["Name"]]` — with double brackets this time?

**Run:**

```python
df[["Name"]]
print(type(df[["Name"]]))
```

**Look at the result:**

```text
     Name
0  Daniel
1    Sara
2   David
<class 'pandas.core.frame.DataFrame'>
```

**Short explanation:** Double brackets mean "give me a list of columns" — even a list containing
just one column name. That returns a **DataFrame**, not a Series.

**Predict first:** And `df[["Name", "Score"]]`?

**Run:**

```python
df[["Name", "Score"]]
```

**Look at the result:**

```text
     Name  Score
0  Daniel     85
1    Sara     91
2   David     78
```

**Short explanation:** Same rule — a list of column names always gives you a DataFrame back,
whether that list has one column or several.

```text
df                    → DataFrame
df["Name"]            → Series
df[["Name"]]          → DataFrame
df[["Name", "Score"]] → DataFrame
```

**Small task:** Using your own DataFrame from Section 4's task, predict and then check the type
of: one column selected with single brackets, the same column selected with double brackets, and
two columns together.

---

## Section 7 — Understand the Index

**What are we doing?** Replacing the automatic row numbers with labels that mean something —
just like you did for a Series in Section 2.

**Run:**

```python
print(df.index)
```

**Look at the result:**

```text
RangeIndex(start=0, stop=3, step=1)
```

**Run:**

```python
df.index = ["student_1", "student_2", "student_3"]
df
```

**Look at the result:**

```text
             Name  Age  Score
student_1  Daniel   24     85
student_2    Sara   27     91
student_3   David   22     78
```

**Short explanation:** Point at three things in that table out loud:

- `student_1`, `student_2`, `student_3` → **Index labels** (identify each row).
- `Name`, `Age`, `Score` → **column labels** (identify each column).
- `Daniel`, `24`, `85`, etc. → the actual **values**.

> The Index identifies rows. It is not the same thing as a regular data column — you can't
> select it with `df["index"]`, and it doesn't count toward `df.shape`'s column number.

**Small task:** Give your Section 4 DataFrame custom row labels of your choice, then print it
and point out the Index labels, column labels, and values.

---

## Section 8 — `.loc[]` and `.iloc[]`

**What are we doing?** Selecting a full row, two different ways.

**Run:**

```python
df.loc["student_1"]
```

**Look at the result:**

```text
Name     Daniel
Age          24
Score        85
Name: student_1, dtype: object
```

**Short explanation:** `.loc[]` selects **by label** — you gave it the Index label
`"student_1"`, and it found that row.

**Run:**

```python
df.iloc[0]
```

**Look at the result:**

```text
Name     Daniel
Age          24
Score        85
Name: student_1, dtype: object
```

**Short explanation:** `.iloc[]` selects **by integer position** — `0` means "the first row,"
regardless of what its label is called. Same row, different way of asking for it.

```text
loc   → labels
iloc  → positions
```

**Small tasks:**

1. Get the second row using `.iloc[]`.
2. Get `student_3` using `.loc[]`.
3. Get just one value — `Score` for `student_2` — using `df.loc["student_2", "Score"]`.

---

## Section 9 — Add and Change a Column

**What are we doing?** Modifying the DataFrame directly.

**Run:**

```python
df["Passed"] = [True, True, False]
df
```

**Look at the result:**

```text
             Name  Age  Score  Passed
student_1  Daniel   24     85    True
student_2    Sara   27     91    True
student_3   David   22     78   False
```

**Short explanation:** Assigning a list to a new column name creates that column. The list needs
one value per row — same idea as building a Series from scratch in Section 2.

**Run:**

```python
df["Score"] = df["Score"] + 5
df
```

**Look at the result:**

```text
             Name  Age  Score  Passed
student_1  Daniel   24     90    True
student_2    Sara   27     96    True
student_3   David   22     83   False
```

**Short explanation:** `df["Score"]` is a Series (remember Section 6). Adding `5` to it applies
to every value at once — the exact same idea you practiced with NumPy arrays yesterday. Every
`Score` value went up by 5.

**Small task:** Add a new column to your own DataFrame, then update an existing numeric column
using a simple `+` or `*`.

---

## Section 10 — Simple Boolean Selection

**What are we doing?** Filtering rows based on a condition — same pattern as `arr[arr > 20]`
from the NumPy exercise, now on a DataFrame.

**Predict first:** What type do you think `df["Score"] > 90` returns?

**Run:**

```python
df["Score"] > 90
```

**Look at the result:**

```text
student_1    False
student_2     True
student_3    False
Name: Score, dtype: bool
```

**Short explanation:** This is a **Boolean Series** — `True`/`False` labeled with the same Index
as `df`. Nothing has been filtered yet; we only asked a yes/no question about every row.

**Run:**

```python
df[df["Score"] > 90]
```

**Look at the result:**

```text
           Name  Age  Score  Passed
student_2  Sara   27     96    True
```

**Short explanation:**

```text
Select the Score column
        ↓
Create a Boolean condition (Score > 90)
        ↓
Use that condition inside df[...] to keep only matching rows
```

Same two-step idea as NumPy's `arr[arr > 20]` — just applied to a whole row this time instead of
a single value.

**Small task:** On your own DataFrame, write one condition on a numeric column and use it to
filter the DataFrame down to matching rows.

---

## Section 11 — Create a Small CSV and Load It

**What are we doing?** Making a real (tiny) CSV file, then letting Pandas build the DataFrame
for us this time, instead of typing it by hand.

**Run this once** to create the file:

```python
csv_text = """Name,Age,City
Daniel,24,Haifa
Sara,27,Jerusalem
David,22,Tel Aviv
"""

with open("students.csv", "w") as f:
    f.write(csv_text)
```

**Now load it:**

```python
csv_df = pd.read_csv("students.csv")
csv_df
```

**Look at the result:**

```text
     Name  Age       City
0  Daniel   24      Haifa
1    Sara   27  Jerusalem
2   David   22   Tel Aviv
```

**Run:**

```python
print(type(csv_df))
print(csv_df.columns)
print(csv_df.index)
print(csv_df.shape)
```

**Look at the result:**

```text
<class 'pandas.core.frame.DataFrame'>
Index(['Name', 'Age', 'City'], dtype='object')
RangeIndex(start=0, stop=3, step=1)
(3, 3)
```

**Short explanation:** We never called `pd.DataFrame()` this time — `pd.read_csv()` built one
for us automatically. Everything you already know (columns, Index, shape, selecting a column,
`.loc[]`/`.iloc[]`) works on `csv_df` exactly the same way it worked on the DataFrame you typed
by hand.

---

## Section 12 — Load JSON

**What are we doing?** Same idea, different file format.

**Run this once** to create the file:

```python
json_text = """[
    {"Name": "Daniel", "Age": 24, "City": "Haifa"},
    {"Name": "Sara", "Age": 27, "City": "Jerusalem"},
    {"Name": "David", "Age": 22, "City": "Tel Aviv"}
]
"""

with open("students.json", "w") as f:
    f.write(json_text)
```

**Now load it:**

```python
json_df = pd.read_json("students.json")
json_df
```

**Look at the result:**

```text
     Name  Age       City
0  Daniel   24      Haifa
1    Sara   27  Jerusalem
2   David   22   Tel Aviv
```

**Ask yourself:** Is this still a DataFrame? Check with `type(json_df)`.

**Short explanation:** Notice this JSON file is a flat list of records — no data nested inside
other data. That's exactly why `pd.read_json()` could turn it straight into a DataFrame with no
extra steps. Different file format, same familiar structure, same tools.

---

## Section 13 — Load Data From a Simple API (Optional but Recommended)

**What are we doing?** Seeing that data doesn't have to come from a local file at all.

This section calls a real public API. If it's unavailable on your network, skip it — everything
else in this exercise still works without it.

**Run:**

```python
import requests

response = requests.get("https://jsonplaceholder.typicode.com/todos?_limit=5")
data = response.json()

api_df = pd.DataFrame(data)
```

**Inspect it:**

```python
api_df.head()
api_df.columns
api_df.shape
```

**Look at the result:**

```text
   userId  id                                              title  completed
0       1   1                                 delectus aut autem      False
1       1   2                 quis ut nam facilis et officia qui      False
2       1   3                                fugiat veniam minus      False
3       1   4                                   et porro tempora       True
4       1   5  laboriosam mollitia et enim quasi adipisci qui...      False

Index(['userId', 'id', 'title', 'completed'], dtype='object')
(5, 4)
```

**Short explanation:**

```text
API request  →  JSON response  →  data.json()  →  pd.DataFrame()  →  DataFrame
```

We're not learning about APIs today — `requests.get()` just fetches data, `.json()` turns the
response into plain Python data (a list of dictionaries, same shape as Section 5), and
`pd.DataFrame()` does the rest, exactly like it always has.

---

## Section 14 — Connect All Input Sources

```text
Manual Python data ─┐
CSV ────────────────┤
JSON ───────────────┼──→  DataFrame  →  same Pandas tools
API ────────────────┘
```

Whether your data started as a dictionary you typed, a `.csv` file, a `.json` file, or an API
response — once it becomes a DataFrame, every tool you practiced today (`.columns`, `.index`,
`.shape`, `[]`, `.loc[]`, `.iloc[]`, Boolean filtering) works exactly the same way, every time.

---

## Section 15 — Final Mini Exercise

Build this DataFrame from scratch, using only what you practiced today.

```python
data = {
    "Product": ["Mouse", "Keyboard", "Monitor", "Webcam", "Headset"],
    "Category": ["Accessories", "Accessories", "Displays", "Accessories", "Audio"],
    "Price": [25, 45, 210, 60, 80],
    "Stock": [120, 75, 30, 50, 40]
}
```

Using only commands from this exercise:

1. Create the DataFrame.
2. Print `.columns`, `.index`, and `.shape`.
3. Select the `Product` column and confirm it's a Series.
4. Select `Product` and `Price` together and confirm it's a DataFrame.
5. Retrieve one row using `.iloc[]`.
6. Give the DataFrame meaningful custom Index labels (e.g. `"item_1"`, `"item_2"`, ...), then
   retrieve one row using `.loc[]`.
7. Add a new column of your choice.
8. Write one Boolean condition on a numeric column.
9. Use that condition to filter the DataFrame.

This is purely structural practice — no cleaning, no investigation, just confirming you can
build, inspect, and select confidently.

---

## You Are Ready for a Real Dataset

Today, every DataFrame you touched had 3 to 5 rows, and you typed most of them yourself. That
was deliberate — with a tiny table, you could see every row, column, label, and value at once,
and always know exactly what Pandas handed back to you.

You now know:

- what a DataFrame and a Series are, and how to tell them apart,
- what the Index is, and that it's not a data column,
- how dictionary keys, column labels, Index labels, and values are four different things,
- how `[]`, `.loc[]`, and `.iloc[]` each select data,
- and that a CSV, a JSON file, an API response, and a plain Python dictionary can all become the
  same kind of DataFrame.

Next comes `Day_1_Guided.md` — the real `Online Retail` dataset. The structures are identical to
what you used today. The only real difference is scale: instead of 3–5 rows you can read in one
glance, you'll have hundreds of thousands — which is exactly why the habits you built today
(checking `.shape`, knowing what a selection returns, trusting `.loc`/`.iloc`) start to matter.
