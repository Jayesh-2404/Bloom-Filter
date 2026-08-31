# Detect Duplicate Web URLs Using Bloom Filters

Big Data Analytics — Bloom Filter and Mode Calculation for Browser History.

This project detects duplicate URLs in browser history using a Bloom Filter and finds the most frequently visited URL (mode).

---

## Problem Statement

Detect duplicate web URLs in a browser history using Bloom Filters. Mode Calculation.

**Objective:**

1. **Duplicate Detection:** Use a Bloom Filter to identify which URLs have been visited before.
2. **Mode Calculation:** Find the most frequently visited URL — the statistical mode.

**Why Bloom Filters?**

Storing every URL in a HashSet is memory expensive in Big Data scenarios.

- 10 million URLs in a Python set: ~1 GB RAM
- 10 million URLs in a Bloom Filter (1% false positive rate): ~12 MB RAM

---

## What is a Bloom Filter?

A Bloom Filter is a space-efficient probabilistic data structure used to test whether an element is a member of a set. Invented by Burton Howard Bloom in 1970.

It consists of:
- A bit array of `m` bits, all initialized to `0`
- `k` independent hash functions, each mapping an element to one of the `m` positions

### How It Works

**Adding an element:**

URL `https://google.com` hashed with k functions gives indices 3, 7, 12. Bits at those positions are set to 1.

```
Before: [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
After:  [0,0,0,1,0,0,0,1,0,0,0,0,1,0,0]
                ^          ^          ^
              idx 3      idx 7     idx 12
```

**Checking an element:**

Hash the URL with the same k functions and check the bits.

- If any bit is 0: Definitely not in the set.
- If all bits are 1: Probably in the set (could be a false positive).

Example — `https://google.com` was added:
```
Hash -> 3 (1), 7 (1), 12 (1) -> Probably in set -> Duplicate
```

Example — `https://example.com` never added:
```
Hash -> 1 (0) -> Definitely not in set
```

### Key Properties

- No False Negatives: If the filter says not present, it is guaranteed not present.
- Possible False Positives: If the filter says present, there is a small probability it is wrong.
- No Deletion: Once a bit is set to 1, it stays 1 in a standard Bloom Filter.
- Space Efficient: Uses m bits regardless of the size of the stored elements.

### Mathematical Formulas

**1. Optimal Bit Array Size (m):**

```
m = -(n * ln(p)) / (ln 2)^2
```

Where n = expected number of elements, p = desired false positive rate.

Example: n=1000, p=0.01 -> m ~ 9585 bits (1.17 KB)

**2. Optimal Hash Functions (k):**

```
k = (m / n) * ln 2
```

Example: m=9585, n=1000 -> k ~ 7

**3. Actual False Positive Rate:**

```
P = (1 - e^(-k * n / m))^k
```

---

## What is Mode Calculation?

The mode is the value that appears most frequently in a dataset.

Browser History Example:

```
google.com:  3 visits
youtube.com: 2 visits
github.com:  1 visit

Mode = google.com
```

- Mode: URL with the highest visit count
- Multimodal: Multiple URLs share the same highest frequency
- Frequency Distribution: Count of visits per URL

---

## Project Structure

```
BDA/
├── app.py                 # Streamlit application
├── bloom_filter.py        # Bloom Filter implementation
├── data_generator.py      # Synthetic browser history generator
├── mode_calculator.py     # Frequency analysis and mode calculation
├── utils.py               # Helper utilities
├── requirements.txt       # Dependencies
└── data/
    └── browser_history.csv  # Dataset (generated/uploaded)
```

---

## Installation and Setup

Prerequisites: Python 3.8+ and pip

1. Navigate to project directory:

```
cd D:\college\project\BDA
```

2. Create virtual environment (optional):

```
python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # macOS/Linux
```

3. Install dependencies:

```
pip install -r requirements.txt
```

Dependencies: streamlit, mmh3, bitarray, pandas, plotly, numpy

---

## How to Run

Start the application:

```
streamlit run app.py
```

Opens at http://localhost:8501

Quick test without GUI:

```
from bloom_filter import BloomFilter
bf = BloomFilter(expected_items=1000, false_positive_rate=0.01)
bf.add('https://google.com')
print(bf.check('https://google.com'))     # True
print(bf.check('https://facebook.com'))   # False
```

---

## Application Features

**Overview:** Explains workflow, shows dataset statistics and Bloom Filter preview.

**Detection:** Processes URLs through the Bloom Filter and compares against ground truth (Python set). Shows True Duplicate, First Seen, False Positive, False Negative with accuracy, precision, recall, F1, and actual vs theoretical false positive rate. Includes confusion matrix and exportable metrics.

**Mode:** Displays the most visited URL, summary statistics, top 10 URLs table and bar chart, and top 20 frequency distribution.

**Charts:** Bit array heatmap, detection breakdown (pie + confusion matrix), memory comparison (Bloom vs HashSet), fill ratio over time, and false positive rate curve.

**Conclusion:** Summarizes what was built, memory savings, accuracy, and the trade-off.

---

## Results and Conclusion

- False Negatives are always 0 (Bloom Filter guarantee).
- With default 1% target rate, observed false positive rate stays near target.
- Memory savings are typically 95-99% compared to a HashSet.
- Fill ratio above 50% increases false positive rate rapidly.
- Bloom Filters are ideal as a first-pass filter in Big Data systems before exact checks.

---

## References

1. Bloom, B. H. (1970). Space/time trade-offs in hash coding with allowable errors. Communications of the ACM, 13(7), 422-426.
2. MurmurHash3: https://github.com/aappleby/smhasher
3. Bloom Filter Calculator: https://hur.st/bloomfilter/
4. Streamlit Documentation: https://docs.streamlit.io/
