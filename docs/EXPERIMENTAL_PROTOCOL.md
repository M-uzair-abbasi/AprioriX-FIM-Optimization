# Experimental Protocol

This document describes the methodology used to ensure reproducible and fair benchmarking across all algorithm-dataset-threshold combinations.

---

## Environment

| Component | Specification |
|-----------|---------------|
| Platform | Google Colab (CPU runtime) |
| RAM | 12 GB free tier; High-RAM session for dense thresholds |
| OS | Ubuntu 22.04 LTS |
| Python | 3.12 |
| Key libraries | NumPy, Pandas, Matplotlib, Seaborn, tracemalloc, gc |

---

## Measurement Methodology

### Wall-Clock Time
- Measured via `time.perf_counter()` for sub-microsecond resolution
- Averaged over **3 trials** when single-run time < 10 s
- **Single trial** when time ≥ 10 s (due to total experiment budget)
- `gc.collect()` called between trials to neutralize garbage collection variance

### Peak Memory
- Measured via Python's standard library `tracemalloc`
- `tracemalloc.start()` called immediately before algorithm execution
- `tracemalloc.get_traced_memory()[1]` captures peak after termination
- Memory measurement excludes the input transaction data (loaded before tracing begins)

### Frequent Itemset Counting
- Direct count of returned set from each algorithm
- Cross-validated between Baseline and Tidset on tractable combinations
- ProbDF count compared against exact ground truth where computable

---

## Tractability Criteria

A combination is marked **N/A (intractable)** if any of:
1. **Timeout:** Single run exceeds 5 minutes (300 s)
2. **Out of Memory:** Process killed by OOM, kernel crash, or RAM exhaustion
3. **No frequent itemsets found:** L₁ empty at threshold (rare, not encountered)

---

## Threshold Sweep

For each (dataset, algorithm) pair, we test 5 thresholds:
```
[0.50, 0.60, 0.70, 0.80, 0.90]
```
Higher thresholds typically tractable; lower thresholds typically intractable on dense data.

---

## Implementation Notes

### Apriori Tidset — Implementation Switch

A subtle but important detail in our experimental setup:

- **Chess @ 60% – 70%:** Tidset uses **set-based** TID representation (Python `set`)
- **Chess @ 80% – 90%:** Tidset uses **bitset** TID representation (`bytearray`)

The switch was motivated by RAM exhaustion (>51 GB) at low thresholds. The set-based version was retained at 60–70% because the bitset version is slower at very large itemset counts (its overhead grows with |Fₖ|).

This is documented in:
- Table II footnote (`†` in `fim_report_FINAL.tex`)
- Figure 2 annotations
- Discussion Section VI-A

### Validation Cross-Check

At each tractable threshold:
- **L₁ size:** All three Apriori variants must produce identical L₁ (verified)
- **Frequent count:** Baseline and Tidset must agree exactly (verified — see Notebook 2 Cell 16)
- **ProbDF correctness:** Per-itemset exact validation at Chess@90% (NB3 Cell 25)

---

## Cited Configuration

For full reproducibility, paper Table II uses these exact parameters:

```python
MIN_SUP_THRESHOLDS = [0.50, 0.60, 0.70, 0.80, 0.90]
HASHTREE_BRANCH_FACTOR = 50
HASHTREE_FALLBACK_THRESHOLD = 500    # |Cₖ| ≤ 500 → use direct issubset
PROBDF_MAX_K = 10                    # DFS depth limit
TIMEOUT_SECONDS = 300
TRIAL_COUNT_FAST = 3                 # used when t < 10 s
TRIAL_COUNT_SLOW = 1                 # used when t ≥ 10 s
```

---

## Known Limitations

1. **Python overhead.** All measurements include Python interpretation overhead, estimated at 50–200× slower than equivalent C++. Results are valid for *relative* comparisons within this study.

2. **Single-trial variance.** Combinations with runtime > 10s used 1 trial only — these have higher measurement variance.

3. **Memory not absolute.** `tracemalloc` measures Python-level allocations; native library allocations (e.g., NumPy internals) may not be fully captured.

4. **PSPM at k ≥ 4 simplified.** Our implementation uses recursive predicted-support storage rather than the full single-vector prefix technique in the original Sadeequllah et al. paper. This compounds prediction error at deeper levels.

---

## Reproduction Instructions

To exactly reproduce paper results:

1. Run `notebook1_data_eda.ipynb` to download/parse datasets
2. Run `notebook2_apriori.ipynb` for Apriori variants (use High-RAM Colab session)
3. Run `notebook3_probdf.ipynb` for ProbDF
4. Run `notebook4_comparison.ipynb` for tables and figures

The notebooks are deterministic — same inputs produce same outputs, modulo single-trial timing variance for runs > 10 s.
