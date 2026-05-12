# Viva Preparation — Q&A Sheet

This document contains anticipated questions for the viva (40% of project grade) along with concise, defensible answers. Every team member should be able to answer questions from any section.

---

## Section A — Apriori Fundamentals

### Q1. Explain the anti-monotonicity property.

> If an itemset is infrequent, all its supersets are also infrequent. Equivalently, every (k-1)-subset of a frequent k-itemset must itself be frequent. This is the basis of Apriori pruning — we discard a k-candidate the moment any (k-1)-subset is missing from L_{k-1}. The search space reduces from worst-case O(2^|I|) to something much smaller in practice.

### Q2. Why does Apriori scale poorly on dense data?

> Two reasons: (1) candidate generation explodes — |Cₖ| can grow as O(|F₁|^k / k!) for dense data, and (2) support counting requires a full DB scan per level, costing O(|T| × |Cₖ|) where both |T| and |Cₖ| are large. The product is exponential in practice on datasets like Chess (49.3% density).

### Q3. What's the difference between support and confidence?

> Support measures itemset frequency: σ(X) = |T_X| / |T|, where T_X is transactions containing X. Confidence is for rules — conf(X → Y) = support(X ∪ Y) / support(X), the conditional probability of Y given X. Apriori mines frequent itemsets via support; association rule generation uses confidence on top.

---

## Section B — ProbDF (Likely Heavy Focus)

### Q4. Walk through PSPM Equation (1) for a 3-itemset.

> For {A, B, C}, we estimate |A ∩ B ∩ C| ≈ (|AB| · |AC| · |BC| · N) / (|A| · |B| · |C|), where pair supports |AB|, |AC|, |BC| are exact (from Phase 1B) and |A|, |B|, |C| are 1-itemset exact counts. The intuition is conditional-probability factoring under partial independence: roughly, P(A∩B∩C) ≈ P(A∩B)·P(A∩C)·P(B∩C) / (P(A)·P(B)·P(C)).

### Q5. Explain the upper and lower bound clipping.

> The raw PSPM prediction can violate physical constraints, so we clip:
> - **Upper (Eq. 3):** Ub = min(|AB|, |AC|, |BC|) — any 3-itemset support cannot exceed the smallest pair it contains (anti-monotone).
> - **Lower (Eq. 4):** Lb = max(|AB|+|AC|-|A|, ..., 0) — derived from inclusion-exclusion: |A∩B∩C| ≥ |A∩B| + |A∩C| - |A|.
> Final predicted support = max(Lb, min(Ub, raw_prediction)).

### Q6. Why does ProbDF need only two database scans?

> Phase 1A: one scan to count 1-itemsets. Phase 1B: one scan to count all 2-itemset pairs. After Phase 1B, transaction data is no longer accessed — every k≥3 itemset support is predicted from the pair-count matrix via PSPM. The memory footprint becomes O(|F₁|²) for the pair matrix, independent of |T|.

### Q7. Why is ProbDF approximate at k≥3 but exact at k=1, 2?

> By construction. Phase 1 produces exact counts via DB scan. Phase 2 uses PSPM, a probabilistic estimator under partial-independence assumption. The clipping (Eqs. 3, 4) keeps predictions within mathematically sound bounds, but doesn't eliminate error within those bounds.

### Q8. What is promotion pruning?

> When σ(prefix ∪ {e}) = σ(prefix), item e appears in every transaction containing prefix. So in any frequent set containing prefix, e is also there. We can "promote" e into the prefix and skip enumerating {prefix, e, x} separately — saves work.
> **Important:** Our implementation does NOT include promotion pruning — it's listed as Future Work item (5) in our paper. Sadeequllah et al.'s original paper implements it; we simplified our DFS for clarity.

---

## Section C — Optimizations

### Q9. Why did you choose HashTree and Tidset?

> They target the two distinct Apriori bottlenecks orthogonally. HashTree addresses support-counting cost (O(|Cₖ|) → O(|Cₖ|/b) per transaction). Tidset addresses database-scan cost (eliminates all scans after level 1 via bitset intersection). Keeping them separate lets us isolate each optimization's contribution.

### Q10. Why is your HashTree branch factor b = 50?

> We chose 50 because our tractable thresholds in Python produce small candidate sets (|Cₖ| often < 1000), where finer bucket partitioning adds hash-computation overhead without meaningful comparison savings. For larger candidate sets at lower thresholds, b = 1009 or larger would be optimal — but those thresholds are intractable in our Python environment anyway.

### Q11. Why is the Tidset speedup only 1.61× when bitset operations should be much faster?

> Python's `bin(byte).count('1')` for popcount is interpreted Python, not native code — each byte costs ~hundreds of nanoseconds vs single CPU cycles for `__builtin_popcountll` in C. In C++ or with vectorized NumPy popcount, the same algorithm would likely be 10–50× faster. The 1.61× is a Python-specific result, not a fundamental algorithmic limit.

### Q12. Why does Tidset use less memory than the set-based version?

> Each TID in a Python `set` costs ~28 bytes (int object + set bucket overhead). A bitset packs 8 TIDs per byte — that's 8 × 28 / 1 = **224× memory reduction** per tidset. At Chess@60% with 246,864 itemsets × 1,918 TIDs, set-based exhausts 51 GB; bitset uses ~120 MB.

### Q13. Why is Tidset Chess@80% (17.4 s) slower than @70% (9.8 s)?

> Two factors: (1) measurement variance — both were single-trial runs (>8 s threshold) after kernel restarts, and (2) implementation difference — 60-70% used set-based Tidset (faster intersection on large sets), 80-90% used bitset (slower at large itemset counts due to Python bytearray overhead). This is documented in Table II's † footnote.

---

## Section D — Methodology & Limitations

### Q14. Why didn't you compare against FP-Growth?

> The project specification required a state-of-the-art algorithm published in 2022 or later. FP-Growth is from 2000 and serves as a reference, not SOTA. We chose ProbDF (2024) from PeerJ Computer Science. FP-Growth comparison is in our Future Work — particularly combining FP-Growth's prefix tree with ProbDF's PSPM.

### Q15. Why is Connect entirely intractable?

> Connect has 67,557 transactions × 33% density with average transaction length 43, and 129 frequent 1-items at 50%. The candidate explosion combined with Python's ~50–200× overhead makes runtime exceed our 5-minute timeout at every threshold. Even compiled Java ECLAT requires 98 s on Connect@90% per Sarieddeen et al. [7] — Python overhead pushes us past the wall.

### Q16. How did you measure memory?

> `tracemalloc.start()` called immediately before each algorithm; peak read after termination via `get_traced_memory()[1]`. We measure additional Python-level memory allocated by the algorithm, not the input data loaded before tracing begins. This is fair for relative comparison between algorithms; it's not absolute total RAM.

### Q17. Walk through your validation. Why is ProbDF Chess@90% precision 99.1%, not 100%?

> We performed exact per-itemset validation at Chess@90%: for each of ProbDF's 573 returned itemsets, we re-scanned the database to compute true support. 568 were truly frequent (TP), 5 were false positives (predicted ≥ θ·N but true support fell just below). Recall: 568 out of 622 ground-truth itemsets = 91.3%. Earlier bugs in PSPM at depth=1 caused 1,671 false positives at Chess@90%, fixed via the `known_sup` dictionary that propagates exact prefix-pair counts (see NB3 Cell 24).

### Q18. What are your limitations?

> Three main limitations:
> 1. **Python overhead** — 50–200× slower than compiled implementations; absolute numbers not comparable to literature benchmarks, only relative.
> 2. **PSPM at k≥4 simplified** — our recursive predicted-support storage compounds error at deeper levels. The original paper's single-vector prefix technique would give tighter bounds.
> 3. **Limited threshold coverage** — Apriori intractable below 80% on Chess. ProbDF extends coverage but isn't a like-for-like comparison.

---

## Section E — Tough Probing Questions

### Q19. Show me where promotion pruning is implemented.

> **Honest answer:** It's not implemented in our DFS — we simplified to a uniform recursion. We list it as Future Work item (5) in Section VII of our paper. Implementing it would require partitioning children into promoted (sup = parent_sup) and normal sets, and folding promoted items into the effective prefix.

### Q20. Why is your candidate generation called "prefix join with subset verification" rather than "F_{k-1} × F_{k-1} join"?

> They're the same thing. F_{k-1} × F_{k-1} join *is* prefix join — we join pairs from L_{k-1} that share their first k-2 items. The "subset verification" step is the standard Apriori pruning: after generating a candidate c, verify all its (k-1)-subsets are in L_{k-1}. Without this verification step, we missed 457 itemsets at Chess@90% (challenge C1 in our paper).

### Q21. Walk through PSPM Eq. 2 for a 5-itemset {a, b, c, d, e}.

> Pref = {a, b}, last three = c, d, e. So:
> |Pref ∩ c ∩ d ∩ e| ≈ (|Pref·cd| · |Pref·ce| · |Pref·de|) / (|Pref·c| · |Pref·d| · |Pref·e|) × |Pref|
> Each |Pref·X| term is itself a stored value from earlier DFS levels. So:
> - |Pref·c| = |abc| (a 3-itemset, predicted via Eq. 1)
> - |Pref·cd| = |abcd| (a 4-itemset, predicted via Eq. 2 recursively)
> This is why error compounds at deeper levels — predictions of predictions.

### Q22. Why does ProbDF slow down on Accidents@90%?

> Two reasons: (1) Phase 1B is O(N × avg_len²) = ~340,183 × 33.8² ≈ 388M operations in Python, dominating runtime. (2) At 90% threshold on Accidents there are only 31 frequent itemsets (all 1- and 2-itemsets), so Phase 2 is trivial. Baseline at the same threshold runs only k=2 levels — faster than ProbDF's full pair scan. ProbDF wins on dense data with many deep frequent itemsets; Accidents at 7.3% density isn't truly dense.

### Q23. Why does Tidset Chess@90% lose to Baseline (0.28× speedup)?

> With only 622 frequent itemsets, the overhead of maintaining and intersecting bitset structures exceeds the cost of direct candidate-subset checking in Baseline. The crossover point in our Python implementation is around 5,000–8,000 frequent itemsets. Below that, bitset bookkeeping costs more than it saves.

### Q24. What's the role of the `known_sup` dictionary in ProbDF?

> It stores exact support counts for 1- and 2-itemsets (from Phase 1) and predicted supports for k≥3 itemsets (from PSPM, propagated forward). Without it, PSPM at depth=1 was using stale or incorrect prefix supports, causing 1,671 false positives at Chess@90%. The fix dropped FPs to 5 (challenge C6 in our paper).

### Q25. If you had another month, what would you fix?

> Top priority: re-implement in Cython or C++ to remove Python's 50–200× overhead — this would enable proper benchmarking against the original ProbDF paper's results. Second: implement proper promotion pruning. Third: implement the full single-vector prefix-storage PSPM from Sadeequllah et al. for tighter k≥4 bounds. Fourth: extend to streaming data and GPU-accelerated bitset intersection.

---

## Day-Of Checklist

- [ ] Each member knows which section they "own" (per Author Contributions block)
- [ ] Each member can write Eq. (1) or (2) on the board from memory
- [ ] Each member can sketch one of Algorithms 1-4 from memory
- [ ] Coordinate: if one member is stuck, the team supplements (don't contradict)
- [ ] Bring printed PDF + laptop with notebooks open
- [ ] Have `figures/` accessible to point at specific data
- [ ] Quality results: 99.1% at Chess@90% is the only fully-validated number — be precise

---

## Assignment of Q-Ownership

| Member | Primary Questions | Secondary |
|--------|-------------------|-----------|
| Bilal | Q1, Q4-Q8, Q19-Q21 (algorithms & ProbDF math) | Q24 |
| Muzzammil | Q9, Q11-Q12, Q17 (Tidset & validation) | Q23 |
| Raneem | Q3, Q13-Q16, Q22 (experimental protocol) | Q25 |
| Uzair | Q2, Q10, Q14, Q18, Q20 (complexity & methodology) | Q25 |

Every member should be able to answer ANY question, but **especially own theirs**.
