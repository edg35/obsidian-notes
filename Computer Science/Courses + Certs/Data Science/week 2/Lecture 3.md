---
status: done
tags:
  - pandas
  - data_manipulation
date: 2026-01-27
---
### Key Data Structures

Pandas is a foundational library for data and ML engineers
- **Series:** A 1D labeled array capable of holding any data type. It only has **axis 0**.
- **DataFrame:** A 2D structure defined by rows and columns.
    - **Axis 0:** Referred to as "index" or "rows".
    - **Axis 1:** Referred to as "columns".

---

### Creating & Loading Data

- **From Arrays/Lists:** Use `pd.DataFrame(numpy_array)` or `pd.Series(list)`.
- **From Files:** * `pd.read_csv("file.csv")`.
    - `pd.read_excel("file.xlsx")`.
- **Basic Inspection:**
    - `df.head(n)`: View top $n$ rows.
    - `df.shape`: Get dimensions (rows, cols).
    - `df.describe()`: Summary statistics for the data.
    - `df.info()`: Check data types and memory usage.

---

#### Merging & Handling Missing Data

##### Joins and Concatenation

- **`pd.concat([df1, df2])`**: Appends DataFrames together.
- **`pd.merge(left, right, on='key')`**: SQL-style joins.

|**Join Type**|**Description**|
|---|---|
|**Inner**|Returns only rows where keys match in both tables.|
|**Outer**|Returns all rows from both tables (Union).|
|**Left**|Keeps all rows from the left table, adds matches from the right.|
|**Right**|Keeps all rows from the right table, adds matches from the left.|

#### Null Value Management

- **Detection:** `df.isnull().sum()` identifies null counts per column.
- **Removal:** `df.dropna(axis=1)` drops columns containing nulls.
- **Replacement:** `df['col'].fillna(value, inplace=True)` replaces missing values (e.g., with 0).

---

### Indexing & Filtering

#### Selection Operators

- **`[]`**: Primarily for column selection (e.g., `df['col_name']`). Numeric slices (e.g., `df[0:3]`) yield rows.
- **`.loc`**: Label-based indexing. Access rows/columns by their names.
- **`.iloc`**: Integer-positional indexing. Ignores labels, strictly uses numerical index.

#### Boolean Filtering

Extract data satisfying specific criteria using boolean arrays:
- **Simple:** `df[df['midterm'] > 50]`.
- **Multiple:** Use `isin` for lists (e.g., `df[df['Party'].isin(['Rep', 'Dem'])]`).

---

### GroupBy Operations

The GroupBy process follows the **Split-Apply-Combine** strategy:

1. **Split:** Segregate data into groups based on criteria.
2. **Apply:** Execute a function (sum, mean, etc.) on each group independently.
3. **Combine:** Re-integrate results into a new structure.

- **MultiIndex:** Grouping by multiple columns (e.g., `df.groupby(['Party', 'Result'])`) creates a hierarchical index.

---

### Memory Optimization

Crucial for "very large" files (multiple gigabytes).
#### Optimization Techniques

- **Numeric Downcasting:** Convert `float64` to `float32` or `int64` to `int32` to save ~50% memory. Use `pd.to_numeric(df['col'], downcast='float')`.
    
- **Categorical Data:** Convert "object" (string) columns with low cardinality (<50% unique values) to the `category` type.
    - **Benefit:** Can reduce memory usage by up to 90%.
    - **Drawback:** Cannot perform arithmetic or use `.min()`/`.max()` on category columns.

> **Note:** Use `df.info(memory_usage='deep')` for an accurate measurement of memory footprint.