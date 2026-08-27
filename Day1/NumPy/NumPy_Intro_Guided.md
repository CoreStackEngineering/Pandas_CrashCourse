# NumPy Basics — Before We Start Pandas

You already know Python lists. Before we touch Pandas tomorrow, we need one short stop at
**NumPy** — the library Pandas is actually built on top of.

This is not a NumPy course. It's about 45–60 minutes, just enough so that when you see a
NumPy-flavored idea inside Pandas later, it already feels familiar.

By the end you'll know:

- what a NumPy array is, and how it's different from a plain Python list,
- how an array is structured (dimensions, shape, size, type),
- how to access and slice values in it,
- that NumPy can apply one operation across every value at once,
- and that this is exactly the foundation Pandas builds on.

---

# Commands We Will Use

### Creating Arrays

`np.array()`

### Understanding Arrays

`.ndim` · `.shape` · `.size` · `.dtype`

### Selecting Values

`[]` · slicing

### Basic Operations

`+` · `-` · `*` · `/`

### Basic Calculations

`.sum()` · `.min()` · `.max()` · `.mean()`

---

## A Very Short Introduction

A Python list stores a collection of values — you already know this. NumPy adds one new
building block: the **array** (`ndarray`), built specifically for working with numbers.

NumPy sits underneath most of the Python data ecosystem, and Pandas is one of the libraries
built directly on top of it. That's the only reason we're here today — not to master NumPy,
just to recognize its basics.

---

## Section 1 — Import NumPy

**What are we doing?** Loading the library so we can use it.

**Run:**

```python
import numpy as np
```

**Look at the result:** Nothing prints — this line just makes NumPy available.

**Short explanation:** `np` is a nickname for `numpy`. It's not required, but it's the
convention literally every NumPy user follows — so much so that `np.array()`,
`np.something()` will look instantly familiar in any code you read later.

---

## Section 2 — Create a 1D Array

**What are we doing?** Turning a normal Python list into a NumPy array.

**Run:**

```python
numbers = [10, 20, 30, 40]
arr = np.array(numbers)

print(arr)
print(type(arr))
```

**Look at the result:**

```text
[10 20 30 40]
<class 'numpy.ndarray'>
```

**Short explanation:** `numbers` is a plain Python list. `np.array()` wraps it into a NumPy
array. Notice the printed array has no commas between values — that's your visual clue you're
looking at an `ndarray`, not a list.

```text
Python list  →  np.array()  →  NumPy ndarray
```

**Small task:** Make your own list of 5 numbers and convert it into an array. Print it and
print its type.

---

## Section 3 — Array Structure

**What are we doing?** Asking the array a few basic questions about itself.

**Run:**

```python
print(arr.ndim)
print(arr.shape)
print(arr.size)
print(arr.dtype)
```

**Look at the result:**

```text
1
(4,)
4
int64
```

**Short explanation:**

- `.ndim` — how many dimensions the array has (this one is a single flat sequence, so `1`).
- `.shape` — the size of each dimension (`4` values, in one dimension).
- `.size` — the total number of values in the array (`4`).
- `.dtype` — the type of value stored inside (whole numbers here, so an integer type).

**Small task:** Run all four on the array you created in Section 2. Do the numbers make sense
for a list of 5 values?

---

## Section 4 — Indexing

**What are we doing?** Grabbing single values out of an array — exactly like a list.

**Run:**

```python
print(arr[0])
print(arr[1])
print(arr[-1])
```

**Look at the result:**

```text
10
20
40
```

**Short explanation:** NumPy indexing works exactly like Python list indexing: position `0` is
the first value, `-1` is the last one. Nothing new here — just familiar syntax on a new kind of
object.

**Small task:** Using `arr = np.array([10, 20, 30, 40])`, print:

1. the first value,
2. the third value,
3. the last value.

---

## Section 5 — Slicing

**What are we doing?** Grabbing a range of values at once.

**Run:**

```python
print(arr[1:4])
print(arr[:3])
print(arr[2:])
```

**Look at the result:**

```text
[20 30 40]
[10 20 30]
[30 40]
```

**Short explanation:** Same rule as list slicing: `start:stop` — `stop` is not included.
Leaving `start` empty means "from the beginning," leaving `stop` empty means "to the end."

**Small task:** Print just the middle two values of `arr` using a slice.

---

## Section 6 — 2D Arrays

**What are we doing?** Seeing that an array can hold more than one dimension — rows and
columns, like a small table.

**Run:**

```python
matrix = np.array([
    [10, 20, 30],
    [40, 50, 60],
    [70, 80, 90],
])

print(matrix.ndim)
print(matrix.shape)
print(matrix.size)
```

**Look at the result:**

```text
2
(3, 3)
9
```

**Short explanation:**

```text
1D array  →  one sequence:            [10, 20, 30, 40]
2D array  →  rows and columns:        [[10, 20, 30],
                                        [40, 50, 60],
                                        [70, 80, 90]]
```

`shape (3, 3)` means 3 rows and 3 columns — `ndim` went from `1` to `2` accordingly.

**Run:**

```python
print(matrix[0])
print(matrix[0, 1])
print(matrix[2, 2])
```

**Look at the result:**

```text
[10 20 30]
20
90
```

**Short explanation:** `matrix[0]` grabs the whole first row. `matrix[0, 1]` means "row 0,
column 1." `matrix[2, 2]` is the bottom-right corner. That's all we need for now — rows and
columns, nothing deeper.

**Small task:** Print the second row of `matrix`, and then print just the value in the middle
of the grid (row 1, column 1).

---

## Section 7 — Basic Array Operations

**What are we doing?** Applying one operation to every value in the array at once.

**Run:**

```python
arr = np.array([10, 20, 30, 40])

print(arr + 5)
print(arr * 2)
print(arr / 10)
```

**Look at the result:**

```text
[15 25 35 45]
[20 40 60 80]
[1. 2. 3. 4.]
```

**Short explanation:** With a plain Python list, `numbers + 5` would raise an error — lists
don't know how to "add 5" to every item without a loop. NumPy arrays do this automatically:

> An operation on a NumPy array applies to every value in it, all at once.

This single idea is one of the biggest reasons NumPy (and Pandas, later) feels so different
from plain Python.

**Small task:** Take `arr` and produce an array where every value is cut in half.

---

## Section 8 — Basic Aggregations

**What are we doing?** Summarizing an array with one number.

**Run:**

```python
prices = np.array([25, 40, 15, 60, 30])

print(prices.sum())
print(prices.min())
print(prices.max())
print(prices.mean())
```

**Look at the result:**

```text
170
15
60
34.0
```

**Short explanation:** Each of these looks at every value in the array and reduces it to a
single summary number — total, smallest, largest, average. You'll use this exact pattern
constantly once we reach Pandas.

**Small task:** Create an array of 5 delivery times (in days) and print its sum, min, max, and
mean.

---

## Section 9 — Simple Boolean Filtering

**What are we doing?** Asking a yes/no question about every value in an array at once.

**Run:**

```python
arr = np.array([10, 20, 30, 40])
print(arr > 20)
```

**Look at the result:**

```text
[False False  True  True]
```

**Short explanation:** `arr > 20` doesn't return one answer — it checks *every* value and
returns a same-shaped array of `True`/`False`. This is called a **Boolean array**.

Now use that Boolean array to actually pull out the matching values:

**Run:**

```python
print(arr[arr > 20])
```

**Look at the result:**

```text
[30 40]
```

**Short explanation:** Two steps happened here:

1. `arr > 20` builds a `True`/`False` array.
2. Putting that array inside `arr[...]` keeps only the values where it's `True`.

Hold onto this idea — you'll see the exact same two-step pattern again very soon, on Pandas
DataFrames.

**Small task:** Using `arr = np.array([10, 20, 30, 40])`, print only the values that are less
than 25.

---

## Section 10 — Final Mini Exercise

Put everything together. No new commands here — only what you've already used today.

```python
delivery_days = np.array([2, 5, 1, 8, 4, 10, 3, 6])
```

Using only the tools from this exercise:

1. Print its `shape` and `size`.
2. Find the minimum delivery time.
3. Find the maximum delivery time.
4. Calculate the average delivery time.
5. Print the 3rd value in the array.
6. Slice out the first 4 values.
7. Print only the delivery times greater than 5.

This should take about 10 minutes. If you get stuck, scroll back — every answer uses a command
from Sections 1–9.

---

## Why Did We Learn This?

You now recognize several ideas you're about to meet again immediately in Pandas:

```text
arrays
dimensions
shape
data types
indexing
Boolean conditions
operations across many values
```

Pandas' main structures — the DataFrame and the Series — are built directly on top of what you
practiced today. When you see `df["Quantity"] > 0` in the next exercise, it will look new, but
the *idea* behind it — a Boolean array selecting matching values — is exactly what you just did
with `arr[arr > 20]`.

Next up: the Pandas exercise, where these same ideas apply to labeled, tabular data.
