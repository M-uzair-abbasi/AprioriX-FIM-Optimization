# AprioriX-FIM: Comparison of Apriori with ProbDF (2024)

[![Python 3.12](https://img.shields.io/badge/python-3.12-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![IEEE Format](https://img.shields.io/badge/Report-IEEE%20Conference-success)](report/)

> **CS-378: Design and Analysis of Algorithms — Semester Project**  
> Ghulam Ishaq Khan Institute of Engineering Sciences and Technology, Topi, Pakistan  
> Spring 2026

A rigorous empirical comparison of the classical **Apriori algorithm** against **ProbDF (2024)** — a state-of-the-art approximate Frequent Itemset Mining algorithm — with two custom optimization strategies for Apriori, benchmarked on three FIMI datasets across five min-support thresholds.

---

## 👥 Team Members

| Name | Roll No | Email | Primary Contribution |
|------|---------|-------|----------------------|
| Bilal Ahmad Sheikh | 2023162 | 2023162@giki.edu.pk | Apriori Baseline, HashTree, ProbDF, Notebooks 1–4 |
| Muhammad Muzzammil Idrees | 2023481 | 2023481@giki.edu.pk | Bitset Tidset, Quality Validation, Literature Review |
| Raneem Sultan | 2023597 | 2023597@giki.edu.pk | Experimental Protocol, Figures, Discussion |
| Muhammad Uzair | 2023547 | 2023547@giki.edu.pk | Complexity Analysis, Conclusion, References |

---

## 🎯 Project Highlights

### Key Findings

🔥 **11.63× speedup** — ProbDF over Apriori Baseline on Chess@80%  
💾 **5.2 MB** memory — ProbDF on Chess@80% vs Tidset's 14.2 GB (set-based)  
🚫 **Connect intractable** — All Apriori variants fail at every threshold  
✅ **99.1% precision** — ProbDF at Chess@90% with exact validation  
📊 **8 publication-quality figures** with full IEEE-format report

### Performance Summary

| Algorithm | Chess@80% | Chess@90% | Accidents@90% | Connect |
|-----------|-----------|-----------|---------------|---------|
| Apriori-Baseline | 28.10 s | 2.26 s | 19.41 s |  Intractable |
| Apriori-HashTree | N/A | 1.98 s | 39.20 s |  Intractable |
| Apriori-Tidset | 17.41 s | 8.12 s | 7.84 s |  Intractable |
| **ProbDF (2024)** | **2.42 s** | **0.95 s** | **35.57 s** | **45.58 s (@90%)** |

---

## 📁 Repository Structure

```
AprioriX-FIM-Optimization/
│
├── 📓 notebooks/                       # All Jupyter notebooks (run in order)
│   ├── notebook1_data_eda.ipynb        # Dataset loading & exploratory analysis
│   ├── notebook2_apriori.ipynb         # Apriori + HashTree + Tidset benchmarks
│   ├── notebook3_probdf.ipynb          # ProbDF implementation & benchmarks
│   └── notebook4_comparison.ipynb      # Combined analysis, tables, figures
│
├── 📊 figures/                         # Publication-quality figures (PNG)
│   ├── fig1_time.png                   # Fig 1: Execution time (log scale)
│   ├── fig2_memory.png                 # Fig 2: Memory usage (log scale)
│   ├── fig3_direct_compare.png         # Fig 3: Baseline vs ProbDF
│   ├── fig4_speedup.png                # Fig 4: Speedup heatmap
│   ├── fig5_tractability.png           # Fig 5: Tractability map
│   ├── fig6_quality.png                # Fig 6: ProbDF precision/recall
│   ├── fig7_efficiency.png             # Fig 7: Memory efficiency frontier
│   └── fig8_level_analysis.png         # Fig 8: Level-wise candidate analysis
│
├── 📄 report/                          # Final IEEE conference paper
│   ├── fim_report_FINAL.tex            # LaTeX source (compile on Overleaf)
│   └── DAA_Algorithm_report.pdf        # Compiled PDF (final submission)
│
├── 📈 results/                         # Saved experiment outputs
│   ├── master_results.csv              # All measurements (4×3×5 grid)
│   ├── quality_metrics.csv             # ProbDF precision/recall data
│   └── candidate_counts.csv            # Apriori candidates per level
│
├── 💾 data/                            # Datasets (downloaded by NB1)
│   └── README.md                       # Download instructions
│
├── 📚 docs/                            # Supplementary documentation
│   ├── ALGORITHMS.md                   # Algorithm reference
│   ├── EXPERIMENTAL_PROTOCOL.md        # How experiments were run
│   └── VIVA_PREP.md                    # Q&A preparation
│
├── 📋 requirements.txt                 # Python dependencies
├── 📋 .gitignore                       # Files to exclude from git
├── 📋 LICENSE                          # MIT License
└── 📋 README.md                        # This file
```

---

## 🚀 Quick Start

### Prerequisites

- Python 3.12+
- Google Colab account (recommended) OR local Jupyter with ~12 GB RAM
- ~2 GB free disk space (for datasets)

### Installation

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/AprioriX-FIM-Optimization.git
cd AprioriX-FIM-Optimization

# Install dependencies
pip install -r requirements.txt
```

### Running the Experiments

Run the notebooks **strictly in order** — each depends on the outputs of the previous one:

```
1️⃣  notebook1_data_eda.ipynb        →  Downloads & parses FIMI datasets
2️⃣  notebook2_apriori.ipynb         →  Benchmarks all Apriori variants
3️⃣  notebook3_probdf.ipynb          →  Benchmarks ProbDF algorithm
4️⃣  notebook4_comparison.ipynb      →  Generates all 8 figures + tables
```

> 💡 **Recommended:** Use Google Colab with a High-RAM session for Notebook 2, since Tidset can consume up to 14 GB on Chess@60%.

### Google Drive Integration

The notebooks save intermediate results to:
```
/content/drive/MyDrive/CS378_FIM_Project/
```
Mount your Drive when prompted at the start of each notebook.

### Expected Runtimes (Colab Free Tier)

| Notebook | Runtime | Notes |
|----------|---------|-------|
| NB1 (EDA) | ~5 min | Downloads ~50 MB of data |
| NB2 (Apriori) | ~30–60 min | Use High-RAM session |
| NB3 (ProbDF) | ~30–60 min | Standard session OK |
| NB4 (Compare) | ~5 min | Just loads + visualizes |

---

## 📊 Datasets

All datasets from the [FIMI Repository](http://fimi.uantwerpen.be/data/).

| Dataset | Transactions | Items | Avg Length | Density | Type |
|---------|-------------|-------|------------|---------|------|
| **Chess** | 3,196 | 75 | 37.0 | 49.3% | Dense (game states) |
| **Connect** | 67,557 | 129 | 43.0 | 33.3% | Dense (game states) |
| **Accidents** | 340,183 | 468 | 33.8 | 7.3% | Sparse (traffic records) |

Datasets are downloaded automatically by `notebook1_data_eda.ipynb` — no manual download needed.

---

## 🧠 Algorithms Implemented

### 1. Apriori Baseline (Agrawal & Srikant, 1994)
Level-wise frequent itemset mining with prefix-join candidate generation and full (k-1)-subset verification pruning.

### 2. HashTree Optimization (Park et al., 1995)
Hash-partitioned candidate lookup with branch factor b=50.  
**Theoretical speedup:** O(|Ck|) → O(|Ck|/b) per transaction.  
**Empirical result:** 1.14× at Chess@90%.

### 3. Bitset Tidset Optimization (Zaki et al., 1997)
Vertical TID-set representation using Python `bytearray` bitmasks.  
**Memory reduction:** 224× vs Python set.  
**Empirical result:** 1.61× speedup, 5.5 MB at Chess@80%.

### 4. ProbDF — State-of-the-Art (Sadeequllah et al., 2024) 
Probabilistic Depth-First FIM with Probabilistic Support Prediction Model (PSPM).  
**Key innovation:** Stops reading transaction data after Phase 1.  
**Space complexity:** O(|F1|²) — independent of dataset size.  
**Empirical result:** 11.63× speedup at Chess@80%.

---

## 📖 Report

The full IEEE conference-format paper is available at:
- 📄 **PDF:** [`report/DAA_Algorithm_report.pdf`](report/DAA_Algorithm_report.pdf)
- 📝 **LaTeX source:** [`report/fim_report_FINAL.tex`](report/fim_report_FINAL.tex)

**Report sections:** Abstract → Introduction → Literature Review → Algorithms → Optimization Strategies → Experimental Setup & Results → Discussion → Conclusion → 10 References

---

## 📚 References

1. R. Agrawal, R. Srikant — *"Fast algorithms for mining association rules"*, VLDB 1994
2. J. Han, J. Pei, Y. Yin — *"Mining frequent patterns without candidate generation"*, ACM SIGMOD 2000
3. M. J. Zaki et al. — *"New algorithms for fast discovery of association rules"*, KDD 1997
4. J. S. Park, M.-S. Chen, P. S. Yu — *"An effective hash-based algorithm for mining association rules"*, ACM SIGMOD 1995
5. M. Sadeequllah et al. — *"Quick mining in dense data: applying probabilistic support prediction in depth-first order"*, PeerJ CS 2024 [DOI: 10.7717/peerj-cs.2334](https://doi.org/10.7717/peerj-cs.2334)

Full bibliography (10 references) in the report.

---

## 🎓 Academic Integrity

All implementations are **original work** by the team members listed above. External sources (papers, datasets) are properly cited. Code was written independently per CS-378 academic integrity guidelines.

This work was submitted as the final project for **CS-378: Design and Analysis of Algorithms** at GIKI, Spring 2026.

---

## 📜 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">


</div>
