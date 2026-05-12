# Algorithm Reference

Detailed pseudocode and complexity analysis for all four algorithms implemented in this project.

---

## 1. Apriori Baseline (Agrawal & Srikant, 1994)

### Pseudocode

```
Algorithm: Apriori Baseline
Input:  Transactions T, min_sup threshold θ
Output: All frequent itemsets F

1.  L₁ ← {frequent 1-itemsets via DB scan}
2.  F ← L₁
3.  k ← 1
4.  while Lₖ ≠ ∅ do
5.    k ← k + 1
6.    Cₖ ← CandGen(Lₖ₋₁)
7.    for all t ∈ T do
8.      for all c ∈ Cₖ with c ⊆ t do
9.        c.count += 1
10.   Lₖ ← {c ∈ Cₖ : c.count ≥ ⌈θ|T|⌉}
11.   F ← F ∪ Lₖ
12. return F
```

### Candidate Generation

We use **F_{k-1} × F_{k-1} prefix join** with **full (k-1)-subset verification pruning**.

### Complexity

- **Time:** O(2^|I|) worst case; O(|T| × |Cₖ|) per level
- **Space:** O(|Cₖ| + |T|)
- **Database scans:** 2k for depth-k itemsets

---

## 2. HashTree Optimization (Park et al., 1995)

### Pseudocode

```
Algorithm: Apriori with HashTree
Input:  Candidates Cₖ, branch factor b, transactions T

1.  Build HashTree: c → bucket[hash(c[0]) mod b]
2.  for all t ∈ T, |t| ≥ k do
3.    for all item x ∈ t do
4.      key ← x mod b
5.      for all c ∈ bucket[key] with c ⊆ t do
6.        c.count += 1
```

### Parameters

- **Branch factor:** b = 50
- **Fallback:** Direct `issubset` used when |Cₖ| ≤ 500

### Complexity

- **Time:** O(|T| × |Cₖ|/b) per level (theoretical)
- **Space:** O(|Cₖ|)
- **Empirical speedup:** 1.14× at Chess@90%

---

## 3. Bitset Tidset Optimization (Zaki et al., 1997)

### Pseudocode

```
Algorithm: Apriori with Bitset Tidset
Input:  Transactions T, min_sup θ

1.  Build 1-item bitsets: tid[i] ← bitmask of TIDs containing i
2.  L₁ ← {i : popcount(tid[i]) ≥ ⌈θ|T|⌉}
3.  while Lₖ ≠ ∅ do
4.    Cₖ₊₁ ← CandGen(Lₖ)
5.    for all c = A ∪ {e} ∈ Cₖ₊₁ do
6.      tid[c] ← tid[Aₖ₋₁] & tid[A'ₖ₋₁]
7.      σ(c) ← popcount(tid[c])
8.    Lₖ₊₁ ← {c : σ(c) ≥ ⌈θ|T|⌉}
```

### Implementation Details

- TID-sets stored as Python `bytearray` of length ⌈|T|/8⌉ bytes
- Support intersection: bitwise AND across byte arrays
- Popcount: `bin(byte).count('1')` per byte
- Explicit `gc.collect()` after each level

### Complexity

- **Memory per tidset:** O(|T|/8) bytes vs O(28|T|) for Python set — **224× reduction**
- **Support counting:** O(|T|/64) 64-bit word operations
- **Database scans after level 1:** 0
- **Empirical speedup:** 1.61× at Chess@80% (5.5 MB vs 51 GB set-based)

---

## 4. ProbDF — State-of-the-Art (Sadeequllah et al., 2024)

### PSPM — Probabilistic Support Prediction Model

**For 3-itemset {A, B, C}:**
```
|A ∩ B ∩ C| ≈ (|AB| · |AC| · |BC|) / (|A| · |B| · |C|) × N
```

**For k ≥ 4 with prefix Pref and last three items A, B, C:**
```
|Pref ∩ A ∩ B ∩ C| ≈ (|Pref·AB| · |Pref·AC| · |Pref·BC|) /
                     (|Pref·A| · |Pref·B| · |Pref·C|) × |Pref|
```

**Clipping bounds:**
```
Upper:  Ub = min(|AB|, |AC|, |BC|)
Lower:  Lb = max(|AB|+|AC|-|A|, |AB|+|BC|-|B|, |AC|+|BC|-|C|, 0)
```

### Pseudocode

```
Algorithm: ProbDF
Input:  Transactions T, min_sup θ
Output: Frequent itemsets F (approximate for k ≥ 3)

1.  Phase 1A: Scan T → compute σ(i) for all i ∈ I
2.  F₁ ← {i : σ(i) ≥ ⌈θN⌉}
3.  Phase 1B: Scan T → compute σ({i,j}) for all i,j ∈ F₁
4.  T no longer accessed in Phase 2
5.  Phase 2: DFS(∅, N, F₁)

procedure DFS(prefix, σ(prefix), candidates)
6.  for all e ∈ candidates do
7.    if |prefix| ≤ 1 then
8.      σ̂ ← σ(prefix ∪ {e})        ▷ exact from Phase 1
9.    else
10.     σ̂ ← PSPM(prefix, e)         ▷ Eq. 1 or 2
11.   if σ̂ ≥ ⌈θN⌉ then
12.     F ← F ∪ {prefix ∪ {e}}
13.     DFS(prefix ∪ {e}, σ̂, candidates > e)
```

### Complexity

- **Phase 1:** O(2 × |T| × avg_len²) for two DB scans
- **Phase 2:** O(|F₁|^h) DFS, O(h) stack memory
- **Total space:** O(|F₁|²) for pair matrix + O(h) DFS stack
- **Key advantage:** No transaction data accessed after Phase 1

### Empirical Performance

| Dataset | Threshold | Time | Memory | Speedup vs Baseline |
|---------|-----------|------|--------|---------------------|
| Chess | 80% | 2.42 s | 5.2 MB | **11.63×** |
| Chess | 90% | 0.95 s | 0.3 MB | 2.37× |
| Connect | 90% | 45.6 s | 17.1 MB | N/A (Baseline intractable) |
| Accidents | 90% | 35.6 s | 0.0 MB | 0.55× |

---

## Comparison Summary

| Aspect | Baseline | HashTree | Tidset | ProbDF |
|--------|----------|----------|--------|--------|
| Year | 1994 | 1995 | 1997 | **2024** |
| Type | Exact | Exact | Exact | **Approximate** |
| DB scans | 2k | 2k | 1 | 2 |
| Memory | O(\|Cₖ\|+\|T\|) | O(\|Cₖ\|) | O(\|Fₖ\|·\|T\|/8) | **O(\|F₁\|²)** |
| Best for | Small data | Many candidates | Sparse-moderate | **Dense** |

---

## References

Full citations in [`../report/fim_report_FINAL.tex`](../report/fim_report_FINAL.tex).
