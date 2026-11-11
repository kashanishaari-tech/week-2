_reflection.md — Cleaning Code Court & Reflection

## Cleaning Code Court (line-by-line verdicts)

I asked the AI for a cleaning script after finishing my own diagnosis. Below is each major line (or logical step) with my verdict and evidence.

### Step 1 — Load a working copy
```python
df = pd.read_csv("teaching_prices_messy.csv")
df_clean = df.copy()
```
**Verdict: KEEP**  
Evidence: Standard and necessary. No data changed.

### Step 2 — Drop exact duplicates
```python
df_clean = df_clean.drop_duplicates()
```
**Verdict: KEEP**  
Evidence: Before = 97 rows, after = 96 rows. The only removed row was the exact copy of 2026-01-08 Staple A City 1. Matches my pre-AI plan.

### Step 3 — Fix the mislabeled item
```python
df_clean.loc[df_clean['item'] == 'oil b', 'item'] = 'Staple A'
```
**Verdict: KEEP** (I deliberately used exact match, not `.str.lower()`, so real “Oil B” rows stay untouched)  
Evidence: Before — one “oil b”. After — zero. On 2026-02-05 we now have both Staple A City 1 (108.5) and Oil B City 1 (318.0). Panel is complete.

### Step 4 — Clean price strings and convert to numeric
```python
df_clean['price'] = (
    df_clean['price']
    .astype(str)
    .str.replace(',', '', regex=False)
    .replace('nan', pd.NA)
)
df_clean['price'] = pd.to_numeric(df_clean['price'], errors='coerce')
```
**Verdict: KEEP**  
Evidence: `"1,250"` became 1250.0, then later nulled. All other values converted cleanly. dtype is now float64.

### Step 5 — Null the extreme outliers
```python
df_clean.loc[df_clean['price'] > 1000, 'price'] = pd.NA
```
**Verdict: MODIFY** (I chose the threshold myself)  
Evidence: 9999 and 1250 both became NaN. I preferred nulling the price over dropping the whole row so that the date–item–city combination is preserved. AI had suggested dropping the rows; I rejected that.

### Step 6 — Parse dates
```python
df_clean['date'] = pd.to_datetime(df_clean['date'])
```
**Verdict: KEEP**  
Evidence: dtype becomes datetime64[ns]. All 12 weekly dates parse correctly.

### Step 7 — Final shape & missingness check
```python
print(df_clean.shape)
print(df_clean['price'].isna().sum())
```
**Verdict: KEEP**  
Evidence: 96 rows, 3 missing prices (original 1 + 2 outliers I nulled). Exactly what I expected from my pre-AI plan.

---

## Reflection

**Which of my decisions differed from the AI’s suggestions?**
- AI offered to drop the two outlier rows entirely. I kept the rows and only nullified the price values.
- AI suggested a simple mean imputation for the single missing Pulse C price. I refused imputation and left it as NaN.

**Which decisions were better supported?**
My versions are better supported by the assignment’s own philosophy (“a clean-looking table with undocumented changes is worse than a messy honest one”) and by the teaching nature of the data. Leaving the three NaNs visible is more honest than inventing numbers or silently deleting observations.

**What would I have missed alone?**
I already found all seven problems myself. The AI mainly saved typing time on the string-cleaning and type-conversion boilerplate. The critical judgement calls (keep vs drop, impute vs leave missing) remained mine.

**Final cleaned state**
- 96 rows (1 exact duplicate removed)
- 3 missing prices (1 original + 2 extreme values nulled)
- Consistent item names (4 products only)
- Proper dtypes (datetime + float)
- Ready for any further analysis while remaining honest about data quality.
