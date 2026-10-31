# Action Plan & Next Steps: Completing Assignment 3 (ANN Benchmarking)

This document outlines all pending tasks, code fixes, data regeneration steps, report writing, and submission packaging needed to complete the assignment in accordance with the specification.

---

## 1. Codebase Bug Fixes & Improvements

### 1.1 Fix Result Contamination & CSV Overwriting
- **Problem**: `code/src/index_benchmarks.py` appends to `results/results.csv`. Running `--quick` followed by full benchmarks leaves 100K subset rows in `results.csv`. `generate_tables.py` then selects the 100K subset rows because of higher QPS, corrupting the SIFT-1M summary tables (showing 62 MB index size instead of 625 MB).
- **Actions**:
  1. In `code/src/index_benchmarks.py`, add a column `num_vectors` (or `mode: full|quick`) to `results.csv`.
  2. In `code/run_all.sh`, ensure `results.csv` is reset or backed up before fresh benchmark runs, or maintain separate `results_full.csv` and `results_quick.csv`.
  3. In `code/src/generate_tables.py`, filter rows to ensure only full-dataset runs (where `num_vectors > 500,000` or `mode == "full"`) are considered for final report tables.

### 1.2 Enable Distance Tie-Handling in Benchmarks
- **Problem**: `recall_at_k` was implemented with tie-handling in `code/src/metrics.py`, but `code/src/index_benchmarks.py`, `code/src/query_difficulty.py`, and `code/src/hard_data_experiment.py` only pass `(ids, gt_neighbors)`, omitting `retrieved_distances` and `ground_truth_distances`. As a result, distance tie-breaking at position 10 is never actually utilized during benchmarking.
- **Actions**:
  1. Update `search_index()` calls in `index_benchmarks.py` to pass `dists` and `gt_distances` into `recall_at_k(ids, gt_neighbors, dists, gt_distances, k=10)`.
  2. Note: For SIFT, retrieved FAISS distances are squared Euclidean ($L_2^2$), while ground truth distances in HDF5 are unsquared ($L_2$). Pass $\sqrt{D}$ or ensure consistent scale.

### 1.3 Add GloVe Exact Baseline to `run_all.sh`
- **Problem**: `run_all.sh` only runs `exact_baseline.py` for SIFT.
- **Actions**:
  1. Add `$PYTHON src/exact_baseline.py --dataset glove $QUICK_FLAG` to `run_all.sh` Step 2.

### 1.4 Improve Trade-off Plotting (Pareto Frontier Envelope)
- **Problem**: In `plot_tradeoff_curve()`, all parameter configurations of an index are plotted with the exact same line color, causing visual clutter.
- **Actions**:
  1. Extract and plot the **Pareto frontier** (convex hull / optimal operating curve) for each index family (HNSW, IVF-Flat, IVF-PQ) alongside the exact baseline.
  2. Differentiate individual parameter curves using dashed/light lines or distinct markers.

### 1.5 Optimize `synthetic_generator.py` Memory Usage
- **Problem**: Line 37 calls `gt_index.search(sample_queries, n_base)` with `n_base = 100,000`, causing FAISS to allocate a 100,000-element heap per query just to get min and max distance.
- **Actions**:
  1. Compute $D_{\min}$ using `gt_index.search(sample_queries, 1)` and $D_{\max}$ via matrix-vector maximum distance or dot products directly in NumPy.

### 1.6 Expand Search Sweep for GloVe HNSW (Optional but Recommended)
- **Problem**: On GloVe-100, HNSW only reached recall **0.909** because $efSearch$ stopped at 256.
- **Actions**:
  1. Add $efSearch = [10, 20, 40, 80, 160, 320, 512]$ and test $M=32, efConstruction=100$ or $M=48$ so HNSW can cross the $0.95$ recall threshold on GloVe.

---

## 2. Benchmark Execution & Data Regeneration

1. **Clean Slate**:
   - Backup existing `results/results.csv` and clear old generated figures/tables.
2. **Execute Full Suite**:
   ```bash
   cd code
   bash run_all.sh
   ```
3. **Verify Integrity of Generated Tables**:
   - `results/tables/sift_cost_table.md`: Ensure Index Size is $\approx 625\text{ MB}$ for HNSW and $\approx 496\text{ MB}$ for IVF-Flat.
   - `results/tables/sift_qps_table.md`: Verify QPS @ recall 0.95 reflects 1M vectors ($\approx 6,000 - 8,500\text{ QPS}$ for HNSW; $\approx 2,400 - 3,300\text{ QPS}$ for IVF-Flat).
   - `results/tables/glove_cost_table.md` & `glove_qps_table.md`: Verify consistency with 1.18M vectors.
4. **Verify Quick Suite Execution**:
   ```bash
   bash run_all.sh --quick
   ```
   - Must complete in under 15 minutes.

---

## 3. Authoring the Report (`report.pdf`)

Create a formal, maximum 5-page report answering the 5 mandated questions using mechanistic ("why") explanations:

### Page 1: Introduction, Datasets & Part 1 (Exact Search Baseline)
- Summary of dataset properties (SIFT 128d Euclidean vs GloVe 100d normalized angular).
- Unit tests & Recall@10 with tie-breaking validation (100% agreement with ground truth).
- **Table / Timing Comparison**: Sequential (1 query at a time) vs Batch (5,000 queries).
  - Full SIFT: $\approx 37.0\text{ QPS}$ ($27.0\text{ ms/q}$) sequential vs $\approx 117.9\text{ QPS}$ ($8.48\text{ ms/q}$) batch ($3.19\times$ speedup).
- **Mechanistic Explanation**:
  1. *BLAS GEMM vs GEMV arithmetic intensity*: Batching allows register tiling and SIMD throughput.
  2. *Cache locality & memory bandwidth*: Base dataset reused in L2/L3 cache across queries in batch mode, avoiding DRAM re-reads.
  3. *Call overhead*: Eliminating 5,000 individual C++/Python transition and heap allocation overheads.

### Page 2: Part 2 (Standard Index Benchmarking)
- **Figures**:
  - `sift_tradeoff.png`: Recall@10 vs QPS (log-scale) with Pareto frontiers for HNSW, IVF-Flat, IVF-PQ, and exact baseline.
  - `glove_tradeoff.png`: Trade-off curves on GloVe-100.
- **Tables**:
  - Table 1: QPS at Recall 0.90 and 0.95 for each index on both datasets.
  - Table 2: Build time, index size (MB), and bytes per vector at Recall $\ge 0.95$.
- **Analysis (Question 1 & Question 2)**:
  - *Which index is fastest at recall 0.95?* HNSW dominates on SIFT ($\approx 8,500\text{ QPS}$ vs $\approx 3,300\text{ QPS}$ for IVF-Flat). On GloVe, IVF-Flat reaches 0.95 at $\approx 415\text{ QPS}$, whereas HNSW requires high degree/search budget due to hubness.
  - *What do indexes cost in build time and memory?* HNSW requires high build time ($58\text{ s} - 260\text{ s}$) and graph overhead ($656\text{ B/vec}$ vs $512\text{ B}$ raw). IVF-Flat is faster to train ($1.2\text{ s} - 100\text{ s}$) and has minimal overhead ($408 - 496\text{ B/vec}$).
  - *What does PQ save, and what does it give up?* PQ compresses memory by $15\times - 20\times$ ($28 - 40\text{ B/vec}$), but quantization distortion creates a recall ceiling ($\approx 0.73$ on SIFT, $\approx 0.52$ on GloVe), failing to reach $0.90+$ without re-ranking.

### Page 3: Part 3 (Query Difficulty on GloVe-100)
- **Figure & Table**:
  - `glove_query_difficulty.png` (bar chart across quartiles Q1 to Q4).
  - Summary table showing Recall@10 across quartiles Q1 (hardest) to Q4 (easiest).
- **Hypothesis Evaluation**:
  - Discuss the pre-committed hypothesis: queries with low relative contrast ($D_{10}/D_1 \approx 1$) suffer severe recall degradation across all indices.
  - Explain why HNSW dropped significantly on hard queries: graph routing gets trapped in dense local hubs when neighbor distances lack separation.

### Page 4: Part 4 (Custom Hard Data: The Curse of Dimensionality)
- **1 Main Figure**:
  - `part4_dimension_curse.png`: Dual-panel plot:
    - (A) Distance concentration ratio $(D_{\max} - D_{\min})/D_{\min}$ and recall collapse from $d=16$ down to $d=256$.
    - (B) QPS degradation.
- **Summary Table**:
  - `part4_dimension_curse.md` integrated.
- **Mechanisms & Limitations**:
  - Distance concentration: $\lim_{d \to \infty} \frac{D_{\max} - D_{\min}}{D_{\min}} \to 0$.
  - Voronoi partitioning failure: cluster centroids lose discriminative partition power.
  - Graph routing failure: greedy routing degrades into random walks.
  - What was not tested: non-spherical distributions, clustered high-dimensional data, OPQ/re-ranking.

### Page 5: Synthesis, Design Scenarios & Critical Evaluation
- **Question 3**: Headline numbers mislead because they average over query difficulty and assume static, low-dimensional manifolds.
- **Question 4 (Engineering Decision Scenario)**:
  - Budget: $100\text{ ms}$ per query, $16\text{ GB}$ RAM.
  - *Case A: 150,000 vectors*: Exact search takes $\approx 4\text{ ms}$ ($\ll 100\text{ ms}$) and uses $< 100\text{ MB}$ RAM. **Recommendation: Use exact search** (no indexing complexity, 100% recall).
  - *Case B: 10,000,000 vectors*:
    - Exact search would take $\approx 270\text{ ms}$ ($> 100\text{ ms}$ budget) and raw vectors take $\approx 5.1\text{ GB}$ RAM.
    - HNSW would exceed 16 GB RAM ($10\text{M} \times 656\text{ B} \approx 6.5\text{ GB}$ base $+$ graph indices $+$ memory fragmentation).
    - **Recommendation: IVF-Flat** with $nlist \approx 16,384$, $nprobe \approx 32 - 64$ (fits comfortably in $5 - 6\text{ GB}$ RAM, executes in $< 10\text{ ms}$ per query, easily satisfying latency and RAM budgets).
- **Question 5 (Two Ways the Benchmark Could Be Misleading)**:
  1. *Single-threaded batching bias / unrealistic concurrent traffic*: Real-world systems run multi-threaded concurrent queries under QPS load rather than single-threaded sequential queries.
  2. *Synthetic static distribution vs dynamic insertion/deletion*: Benchmarks test static read-only indices; graph fragmentation and re-clustering under streaming updates were not tested.

---

## 4. Documentation & Packaging Deliverables

1. **Fix `ai_log/`**:
   - Create `ai_log/prompts_and_responses.md` documenting key LLM prompts and code assistance interactions.
   - Update `ai_log/ai_statement.md` to remove references to non-existent files (`env_info.py`, `timer.py`, `plotter.py`) and list the actual files present in `code/src/`.
2. **Review `code/README.md`**:
   - Verify instructions for environment setup, `bash run_all.sh`, and `bash run_all.sh --quick`.
3. **Verify Git History**:
   - Confirm `git log` inside `code/` retains the commit sequence showing `hypothesis.md` committed prior to experiment code.
4. **Prepare Submission Zip**:
   - Name: `<rollnumber>_A3.zip`.
   - Contents check (must contain **only** these items, no datasets, no `.index` files):
     ```
     <rollnumber>_A3.zip
     ├── report.pdf
     ├── code/
     │   ├── .git/
     │   ├── src/
     │   ├── tests/
     │   ├── hypothesis.md
     │   ├── README.md
     │   ├── requirements.txt
     │   └── run_all.sh
     ├── results/
     │   ├── results.csv
     │   ├── figures/
     │   └── tables/
     └── ai_log/
         ├── ai_statement.md
         └── prompts_and_responses.md
     ```

