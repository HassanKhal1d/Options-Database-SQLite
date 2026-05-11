# Dataset Datasheet | High-Frequency Options Data Capstone Project


| Category             | Details |
|----------------------|---------|
| **Underlying Asset** | S&P 500 Index (SPX) |
| **Data Type**        | 1-minute interval options chain snapshots |
| **Date Coverage**    | 2023-10-02 to 2023-10-03 (2 trading days) |
| **Total Records**    | 37,927 rows across 33 columns |
| **Option Types**     | Calls and Puts (combined wide format per row) |
| **Expiry Types**     | Daily, Weekly, Monthly |
| **Strike Range**     | 4,100 – 4,500 |
| **Primary Use**      | Options data engineering, liquidity filtering, SQLite pipeline construction, intraday strategy research |


---

### Motivation

**What task does this dataset help to solve?**

This dataset supports the construction of a high-frequency options data pipeline for the S&P 500 Index (SPX). Specifically, it provides the empirical basis for four interconnected tasks: preprocessing and standardising raw 1-minute options chain data; applying liquidity and strike-distance filters to isolate tradeable contracts; loading the cleaned data into a structured SQLite database with separate call and put tables; and querying that database to extract meaningful intraday and expiry-based data slices for options trading strategy research.

The dataset enables analysis of minute-level options activity across multiple expiry types — Daily, Weekly, and Monthly — covering a range of strikes relative to the underlying SPX price. This makes it suitable for studying intraday options behaviour, implied volatility dynamics, and the relationship between liquidity and strike distance for near-term contracts.

**Who created the dataset and who funded it?**

The dataset was provided as part of the High-Frequency Options Data Capstone Project within the ML and AI Certificate Programme delivered by Imperial College Business School in collaboration with Emeritus. The raw CSV files contain real 1-minute interval options data for the SPX index and were supplied as course materials. No external funding or third-party data vendor is disclosed by the course, and the dataset is provided exclusively for educational use within this programme.

---

### Composition

**What does it contain?**

The dataset comprises two CSV files, each covering one trading day of SPX options data at 1-minute resolution:

| File | Date | Rows | Unique Timestamps | Unique Strikes |
|------|------|------|-------------------|----------------|
| `SPX_minute_options_2023-10-02.csv` | 2023-10-02 | 23,069 | ~391 | 41 |
| `SPX_minute_options_2023-10-03.csv` | 2023-10-03 | 14,858 | ~391 | 41 |
| **Combined** | 2023-10-02 to 2023-10-03 | **37,927** | **782** | **41** |

Each row represents a single strike-expiry-timestamp combination. The data is structured in wide format, meaning both the call and put sides of each contract are stored together in one row, prefixed with `C_` and `P_` respectively. There are 33 columns in total.

**Columns and data types:**

| Column | Type | Description |
|---|---|---|
| `[QUOTE_DATETIME]` | string → datetime | Full timestamp of the 1-minute snapshot |
| `[QUOTE_DATE]` | string → date | Date portion of the quote |
| `[QUOTE_TIME]` | string → time | Time portion of the quote |
| `[SYMBOL]` | string | Underlying symbol — always `SPX` |
| `[UNDERLYING_LAST]` | float | Last traded price of the SPX index at the snapshot time |
| `[EXPIRE_DATE]` | string → date | Expiration date of the option contract |
| `[EXPIRY_TYPE]` | string (categorical) | Contract expiry category: `Daily`, `Weekly`, or `Monthly` |
| `[DTE]` | float | Days to expiration at time of quote |
| `[STRIKE]` | float | Strike price of the option contract |
| `[STRIKE_DISTANCE]` | float | Absolute difference between strike and underlying price |
| `[STRIKE_DISTANCE_PCT]` | float | Strike distance as a proportion of the underlying price |
| `[C_DELTA]` | float | Call delta (rate of change of call price with respect to underlying) |
| `[C_GAMMA]` | float | Call gamma (rate of change of delta) |
| `[C_VEGA]` | float | Call vega (sensitivity to implied volatility) |
| `[C_THETA]` | float | Call theta (time decay per day) |
| `[C_RHO]` | float | Call rho (sensitivity to interest rate changes) |
| `[C_IV]` | float | Call implied volatility |
| `[C_VOLUME]` | integer | Call volume at snapshot time |
| `[C_LAST]` | float | Last traded call price |
| `[C_SIZE]` | string | Call bid-ask size (format: `N x N`) |
| `[C_BID]` | float | Call bid price |
| `[C_ASK]` | float | Call ask price |
| `[P_DELTA]` | float | Put delta |
| `[P_GAMMA]` | float | Put gamma |
| `[P_VEGA]` | float | Put vega |
| `[P_THETA]` | float | Put theta |
| `[P_RHO]` | float | Put rho |
| `[P_IV]` | float | Put implied volatility |
| `[P_VOLUME]` | integer | Put volume at snapshot time |
| `[P_LAST]` | float | Last traded put price |
| `[P_SIZE]` | string | Put bid-ask size (format: `N x N`) |
| `[P_BID]` | float | Put bid price |
| `[P_ASK]` | float | Put ask price |

**Key distributional facts:**

- Underlying SPX price range across both days: **4,291.01 – 4,329.69**
- Strike range: **4,100 – 4,500** (41 unique strikes at 5-point intervals)
- DTE range: **0.33 – 18.60** calendar days
- Expiry dates present: **2023-10-03** (Daily), **2023-10-06** (Weekly), **2023-10-21** (Monthly)
- Call volume: min 2, median 34, max 223 (mean 56)
- Put volume: min 2, median 34, max 225 (mean 56)
- Call IV range: **0.205 – 5.514**
- Put IV range: **0.205 – 6.041**
- **No missing values** across any column in either file

---

### Collection Process

**How was the data acquired?**

The data was collected from a market data source capturing SPX options chain snapshots at 1-minute intervals throughout each trading day. Each snapshot records the full options chain — all available strikes and expiries — at that moment in time, producing one row per strike-expiry pair per minute. The data is provided as pre-formed CSV files and was not collected by the student; it was distributed directly as part of the capstone project course materials.

**What is the sampling mechanism?**

Data was recorded at fixed 1-minute intervals aligned to market hours (09:30 – 16:00 US Eastern Time). Each minute produces a cross-section of all available SPX option contracts active at that timestamp. The result is a balanced time-series panel structure: for each timestamp, all 41 strike levels are recorded across the three available expiry dates, yielding approximately 123 rows per minute.

**Time frame of data collection:**

The dataset covers two consecutive trading days: Monday 2 October 2023 and Tuesday 3 October 2023. Trading hours run from 09:30:00 to 16:00:00 Eastern Time. The earlier file contains 391 unique minute timestamps and the combined dataset contains 782, consistent with a full 6.5-hour trading session per day.

---

### Preprocessing, Cleaning and Labelling

**What preprocessing is required before use?**

The raw files require the following preprocessing steps before analysis or database loading:

1. **Column name standardisation** — all column names are enclosed in square brackets (e.g. `[QUOTE_DATETIME]`). Brackets, spaces, and any special characters must be stripped and uniform capitalisation applied.

2. **Datetime conversion** — `QUOTE_DATETIME`, `QUOTE_DATE`, `QUOTE_TIME`, and `EXPIRE_DATE` are stored as plain strings and must be converted to proper Python datetime or date objects using `pd.to_datetime()`.

3. **Numeric type enforcement** — most numeric columns are correctly read as `float64` or `int64` by pandas; however, type assertions should be applied to confirm and enforce this after column renaming.

4. **`C_SIZE` and `P_SIZE` columns** — these are stored as strings in `"N x N"` format (e.g. `"9 x 6"`) representing bid and ask size. They are not directly numeric and require parsing or exclusion depending on intended use.

5. **Missing value check** — the raw files contain no null values across any column, as confirmed by inspection. A null check should nonetheless be included as a defensive step in the pipeline.

**What filtering is applied?**

After preprocessing, the following filters are applied as part of the project pipeline:

- **Liquidity filter**: remove rows where call or put bid, ask, or volume are zero or negative
- **Bid-ask spread filter**: optionally remove contracts with a spread-to-ask ratio above a defined threshold (e.g. 50%) as a proxy for illiquidity
- **Strike distance filter**: retain only contracts within a defined percentage of the underlying price (e.g. within 10%), using the pre-calculated `STRIKE_DISTANCE_PCT` column or a derived version
- **DTE filter**: optionally exclude very long-dated contracts if the focus is on near-term intraday behaviour

**What derived columns are created?**

The following columns are constructed during preprocessing:

- `STRIKE_DISTANCE` — absolute difference between strike and underlying: `abs(STRIKE - UNDERLYING_LAST)`. Note this is already present in the raw file but may be recomputed after type standardisation.
- `DTE` — days to expiration: `(EXPIRE_DATE - QUOTE_DATE).days`. Also pre-calculated in the raw file.
- After splitting into calls and puts, the `C_` and `P_` prefixes are stripped from Greek and pricing columns so each table has clean, uniform column names.

**Were any labels applied?**

The dataset is not labelled in the supervised learning sense. The split into call and put DataFrames (and subsequently into separate database tables) constitutes the primary structural labelling applied during processing.

---

### Uses

**What is this dataset appropriate for?**

The dataset is intended for educational and research use in options data engineering and quantitative trading strategy development, specifically for:

- **ETL pipeline construction** — practising the full extract-transform-load workflow on real financial market data, including column standardisation, type correction, and database insertion
- **Liquidity and filter design** — developing and evaluating bid-ask spread, volume, and strike-distance filters that mirror real-world data cleaning workflows used in options trading desks
- **SQLite database design** — building and querying a structured relational database optimised for minute-level options data, with separate call and put tables
- **Intraday options analysis** — extracting time-sliced data to observe how implied volatility, Greeks, and bid-ask spreads evolve within a trading session across different expiry types
- **Expiry structure analysis** — comparing Daily, Weekly, and Monthly contracts on the same underlying across the same timestamps to study term structure effects

**What are the inappropriate uses?**

This dataset should not be used to:

- **Make live trading decisions** — the data covers only two trading days and represents a narrow snapshot of market conditions in early October 2023; no strategy derived from this data should be considered validated or generalisable
- **Claim statistically robust backtesting results** — two days of data is insufficient for any meaningful backtest; conclusions drawn from this dataset about strategy performance are anecdotal rather than statistically valid
- **Extrapolate market structure findings** — any patterns observed in SPX options on these two specific days may not reflect typical behaviour and should not be generalised to other assets, time periods, or market regimes
- **Substitute for a complete options dataset** — the strike range of 4,100–4,500 and limited expiry set cover only a portion of the full SPX options chain; deep in-the-money and far out-of-the-money contracts, and longer-dated expiries, are not represented

**Are there risks or limitations?**

The primary limitation is **temporal scope**: two trading days provide insufficient coverage for strategy validation, volatility regime analysis, or robust filtering threshold calibration. Any filter parameters derived from this dataset (e.g. a bid-ask spread threshold or a minimum volume cutoff) will be tuned to conditions specific to early October 2023 and may not transfer to other periods.

A secondary limitation is **data format**: the wide-format structure — where call and put data share a row — requires explicit reshaping before separate call and put analyses can be performed. The `C_SIZE` and `P_SIZE` columns are non-numeric strings and cannot be used directly in quantitative filters without parsing.

A third limitation is the **absence of trade data**: the dataset records quote snapshots (bid, ask, last) rather than confirmed trade executions. Volume figures represent cumulative intraday volume at each snapshot, not tick-by-tick transaction records. This means bid-ask spread analysis reflects quoted liquidity, not realised transaction costs.

---

### Distribution

**How has the dataset been distributed?**

The dataset is distributed as two CSV files — `SPX_minute_options_2023-10-02.csv` and `SPX_minute_options_2023-10-03.csv` — provided directly through the course platform as part of the capstone project materials in the `data_modules` folder. No public repository, API, or physical media distribution has been established.

**When will the dataset be made available?**

The dataset is available to enrolled students for the duration of the capstone project assessment. Any distribution beyond this context would require explicit approval from the course administrators responsible for providing the data.

**What are the terms of use?**

The dataset was provided as part of an academic module at Imperial College Business School in collaboration with Emeritus. Use is limited to educational and assessment purposes within this programme. The data should not be redistributed, published, or used for commercial purposes. Any derived work should acknowledge the course from which the data was obtained. The raw CSV files should not be committed to public repositories; a `.gitignore` entry is recommended.

---

### Maintenance

The dataset is static and frozen at the point of distribution. It consists of exactly two files covering two trading days and will not be updated, extended, or corrected by the course after distribution. No versioning system is applied to the raw files.

Data integrity is the responsibility of the student during the preprocessing phase. Any corrections to column types, naming conventions, or derived fields are applied programmatically in the pipeline code and documented there. The raw CSV files should be treated as immutable source inputs and stored separately from any processed outputs.

The SQLite database produced from this dataset is a derived artefact and constitutes the versioned, queryable output of the pipeline. It should be regenerated from source files rather than manually edited, and the generation script should be idempotent — dropping and recreating tables on each run to ensure consistency with the current pipeline logic.
