# Action Plan & Status: Assignment 3 (ANN Benchmarking)

All code fixes, benchmarking data regenerations, experimental verifications, and report authoring are **100% completed**. Per user instruction, packaging of deliverables (`<rollnumber>_A3.zip`) will be performed by the user.

---

## 1. Codebase Bug Fixes & Improvements [COMPLETED]

### 1.1 Fix Result Contamination & CSV Overwriting [COMPLETED]
- Added `num_vectors` column to `results/results.csv`.
- Filtered `code/src/generate_tables.py` by `num_vectors` to ensure `--quick` subset runs never contaminate full 1M tables.
- Updated `code/run_all.sh` to safely back up existing results prior to new full runs.

### 1.2 Enable Distance Tie-Handling in Benchmarks [COMPLETED]
- Corrected `search_index()` calls across `code/src/index_benchmarks.py`, `code/src/query_difficulty.py`, and `code/src/hard_data_experiment.py` to pass scaled retrieved and ground-truth distances ($\sqrt{D}$ for SIFT, $D/2$ for GloVe) to `recall_at_k()`.
- Validated with unit tests in `code/tests/test_metrics.py` (6/6 passing).

### 1.3 Add GloVe Exact Baseline to `run_all.sh` [COMPLETED]
- Added GloVe exact baseline execution to Step 2 in `run_all.sh` ($5.02\times$ batch GEMM speedup over sequential GEMV).

### 1.4 Improve Trade-off Plotting (Pareto Frontier Envelope) [COMPLETED]
- Implemented `get_pareto_frontier()` in `code/src/index_benchmarks.py` to plot prominent Pareto envelopes over individual sweep lines.
- Generated updated figures in `results/figures/sift_tradeoff.png` and `glove_tradeoff.png`.

### 1.5 Optimize `synthetic_generator.py` Memory Usage [COMPLETED]
- Replaced $k=100,000$ FAISS heap allocation with direct vectorized matrix-vector dot products in NumPy.

### 1.6 Expand Search Sweep for GloVe HNSW [COMPLETED]
- Expanded $efSearch$ sweep up to $512$ with $M=32, efConstruction=100$.
- GloVe HNSW crossed the 0.95 recall threshold ($0.9559$ recall at $950.4$ QPS).

---

## 2. Benchmark Execution & Data Regeneration [COMPLETED]

- Executed full single-threaded suite (`bash run_all.sh`) on 1,000,000 SIFT vectors and 1,183,514 GloVe vectors.
- Verified all generated markdown tables in `results/tables/`:
  - `sift_cost_table.md`: SIFT HNSW ($747.8$\,MB, $784.1$\,B/vec), IVF-Flat ($497.9$\,MB, $522.1$\,B/vec).
  - `sift_qps_table.md`: SIFT HNSW reaches $8,694.1$\,QPS at recall $0.95$; IVF-Flat reaches $3,354.0$\,QPS.
  - `glove_cost_table.md` & `glove_qps_table.md`: GloVe HNSW ($950.4$\,QPS @ 0.95), IVF-Flat ($674.4$\,QPS @ 0.95).
  - `glove_query_difficulty.md`: Validated Q1 ($0.402$ recall) vs. Q4 ($0.722$ recall) degradation.
  - `part4_dimension_curse.md`: Distance contrast collapse ($2.6137 \to 0.3171$) and recall collapse ($0.9490 \to 0.0590$).
- Fast verification mode (`bash run_all.sh --quick`) completes in under 1 minute.

---

## 3. Technical Report (`report.pdf`) [COMPLETED]

- Authored `report/report.tex` with comprehensive mechanistic ("why") explanations answering all 5 core questions.
- Compiled with `pdflatex` to strictly **5 pages** (maximum allowed limit).
- Copied final compiled document to root `./report.pdf`.

---

## 4. Documentation & AI Transparency [COMPLETED]

- Updated `ai_log/ai_statement.md` with full attribution matching all active source files.
- Created `ai_log/prompts_and_responses.md` documenting interaction phases.
- Preserved pre-committed hypothesis git history in `code/hypothesis.md`.

---

## 5. Submission Checklist (For User)

When packaging the final submission:
1. Verify the archive name: `<rollnumber>_A3.zip`.
2. Ensure the zip contains **only** the following structure (no `.index` files, no raw `.hdf5` datasets, no cache directories):
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
3. Test archive integrity before upload:
   ```bash
   unzip -l <rollnumber>_A3.zip
   ```

