# Data Engineering with Python — Pandas Track

Until now you've built systems that create, store, and move data. This unit shifts the focus to
the data itself: understanding an unfamiliar dataset, finding its problems, and fixing only what
you can justify. The goal isn't memorizing Pandas commands — it's building a reliable process
and using a small toolkit correctly.

## Order

1. `Day_1/NumPy/NumPy_Intro_Guided.md`
2. `Day_1/Pandas_Intro/Pandas_Intro_Guided.md`
3. `Day_1/Day_1_Guided.md`
4. `Day_1/Day_1_Independent.md`
5. `Day_1/Day_1_Script_Guide.md`
6. `Day_2/Day_2_Guided.md`
7. `Day_2/Day_2_Independent.md`
8. `Day_2/Day_2_Script_Guide.md`

Each Guided exercise is done with the instructor. Each Independent exercise right after it uses
only the tools just taught — no new commands. Each Script Guide then shows the same process
turned into a small, reusable `.py` file.

## Day 1 vs. Day 2

| | Day 1 | Day 2 |
|---|---|---|
| Question | Can we trust this data? | What do we do about it? |
| Mindset | Observe → Measure → Investigate → Conclude | Measure → Decide → Clean → Verify |
| Action | Investigation only — nothing is changed | Justified changes only, each one verified |
| Deliverable | A Data Quality Report | A cleaned dataset + Cleaning Report |

## Datasets

| File | Used in | Purpose |
|---|---|---|
| `Online Retail.xlsx` | Day 1 & 2 Guided | Real UK retail transactions |
| `ecommerce_fulfillment_dirty.csv` | Day 1 & 2 Independent | Order fulfillment data with realistic quality issues |

Both are raw input — never overwrite them. Always save results to a new file.

## Instructor-only files

Don't open these before finishing the matching exercise — they contain expected answers:

- `Day_1/Instructor_Reference.md`, `Day_2/Instructor_Reference.md`
- `E-Commerce Order Fulfillment Dataset (50K Records).csv`, `generate_dirty_fulfillment_data.py`,
  `dirty_data_ground_truth.json`, `dirty_data_summary.md`
