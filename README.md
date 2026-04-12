# prepx

One-line cleaning **and** exploratory data analysis for pandas DataFrames.

```python
from prepx import clean, eda

cleaned_df, clean_report = clean(df)
eda_report = eda(cleaned_df, target="price")
```

---

## Features

- **Automated Data Cleaning**: Handle duplicates, missing values, outliers, and more with a single function call
- **Smart Type Inference**: Automatically convert columns to numeric or datetime when possible
- **Column Standardization**: Auto-rename columns to snake_case
- **Outlier Detection**: Identify and cap outliers using IQR or Z-score methods
- **Full EDA Reports**: Generate comprehensive exploratory analysis with statistics, correlations, and warnings
- **Target Analysis**: When a target column is specified, includes class balance and feature correlations
- **Programmatic Access**: All report data is returned as dicts for further processing

---

## Installation

```bash
pip install prepx
```

Or install from source:

```bash
pip install .
```

---

## `clean(df, **kwargs)` → `(DataFrame, dict)`

Cleans the DataFrame and returns the cleaned version plus a full action report.

| Parameter | Default | What it does |
|---|---|---|
| `drop_duplicates` | `True` | Remove exact duplicate rows |
| `handle_missing` | `"auto"` | `"auto"`/`"fill"` → median/mode fill; `"drop"` → drop NaN rows; `"none"` → skip |
| `missing_threshold` | `0.6` | Drop columns with > this fraction missing |
| `fix_dtypes` | `True` | Coerce object columns to numeric or datetime when ≥80% parse |
| `strip_whitespace` | `True` | Strip leading/trailing whitespace from strings |
| `standardize_columns` | `True` | Rename columns to snake_case |
| `remove_outliers` | `False` | Cap outliers via IQR fences or Z-score |
| `outlier_method` | `"iqr"` | `"iqr"` or `"zscore"` |
| `outlier_threshold` | `3.0` | Z-score cut-off (only used for `"zscore"`) |
| `drop_constant_cols` | `True` | Drop columns with only one unique value |
| `verbose` | `True` | Print a human-readable report |

### Example output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  prepx  ·  Cleaning Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Initial shape                   1000 rows × 12 cols
  Final shape                      961 rows ×  9 cols
  Rows removed                      39
  Columns removed                    3
──────────────────────────────────────────────────────────────

  ✦ Column names → snake_case  (4 renamed)
  ✦ Whitespace stripped  (5 string columns)
  ✦ Type coercion  (2 columns updated)
      Numeric  : age, income
  ✦ Duplicate rows removed  : 12
  ✦ Constant columns dropped : id_copy
  ✦ High-missing columns dropped  (>60% NaN)
      notes
  ✦ Missing values filled  (3 columns)
      age       4 NaN → median (32.0)
      city      2 NaN → mode ('London')
```

---

## `eda(df, **kwargs)` → `dict`

Full exploratory analysis. Returns a rich dict and prints a report.

| Parameter | Default | What it does |
|---|---|---|
| `target` | `None` | Target column — adds class balance + per-feature correlations |
| `top_n_categories` | `10` | How many top categories to show per object column |
| `correlation_method` | `"pearson"` | `"pearson"`, `"spearman"`, or `"kendall"` |
| `verbose` | `True` | Print a human-readable report |

### Report dict keys

| Key | Contents |
|---|---|
| `overview` | Shape, duplicates, memory |
| `dtypes` | Column lists by type + per-column dtype |
| `missing` | Total missing, per-column counts and percentages |
| `numeric_stats` | Mean, std, min/max, percentiles, skewness, kurtosis, outliers |
| `categorical_stats` | Unique count, top-N values, mode, entropy |
| `correlations` | Full matrix + high-correlation pairs (|r| ≥ 0.7) |
| `target_analysis` | Class balance + feature correlations (if `target` set) |
| `warnings` | Auto-generated issues (duplicates, skew, multicollinearity, etc.) |

---

## Quick start

```python
import pandas as pd
from prepx import clean, eda

df = pd.read_csv("data.csv")

# 1 — clean
cleaned, clean_report = clean(df, remove_outliers=True)

# 2 — explore
eda_report = eda(cleaned, target="churn")

# 3 — use the report programmatically
print(eda_report["warnings"])
print(eda_report["numeric_stats"]["age"])
```