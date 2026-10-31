# AI Interaction Log: Prompts and Summary of Responses

This document records the primary interaction prompts used during the development, benchmarking, debugging, and report authoring phases of Assignment 3.

---

### Phase 1: Problem Understanding & Codebase Audit
- **Prompt**: "Audit the ANN benchmarking codebase against the assignment problem statement. Verify whether the code, metrics, and benchmarking results are correct, check if the benchmark values reflect true 1M datasets or quick subsets, and list all issues."
- **Summary of Response / Action**:
  - Audited all modules in `code/src/`, `code/tests/`, and `results/`.
  - Identified data contamination where `--quick` 100K subset runs appended to `results.csv`, causing `generate_tables.py` to pick 100K data points instead of 1M points (understating index sizes to 62 MB instead of 625-750 MB).
  - Identified that distance tie-breaking was implemented in `metrics.py` but never invoked in `index_benchmarks.py` or experiment scripts because distances were not passed.
  - Identified heap allocation in `synthetic_generator.py` for $k=100,000$ when computing max distance.

---

### Phase 2: Action Roadmap & Code Corrections
- **Prompt**: "Write an actionable roadmap in next.md detailing all required bug fixes, data regeneration, and report writing steps. Proceed to implement all fixes."
- **Summary of Response / Action**:
  - Implemented `num_vectors` tracking and filtered out sub-million vector quick runs in `generate_tables.py`.
  - Passed squared/unsquared distances with proper scaling ($\sqrt{D}$ for SIFT, $D/2$ for GloVe) to `recall_at_k` to enable exact tie-handling.
  - Added GloVe exact baseline execution to `run_all.sh`.
  - Implemented convex-hull Pareto frontier curve computation and plotting in `index_benchmarks.py`.
  - Replaced $k=100,000$ FAISS heap allocation in `synthetic_generator.py` with direct BLAS dot-product evaluation.

---

### Phase 3: Benchmarking Execution & Verification
- **Prompt**: "Run the complete benchmarks single-threaded on SIFT-1M and GloVe-100 and regenerate all figures and markdown tables."
- **Summary of Response / Action**:
  - Enforced `OMP_NUM_THREADS=1` and `OPENBLAS_NUM_THREADS=1`.
  - Evaluated exact baseline (sequential vs batch) for SIFT and GloVe.
  - Ran parameter sweeps for HNSW ($M \in \{16, 32\}$, $efConstruction \in \{40, 100\}$, $efSearch \in [10, 512]$), IVF-Flat ($nlist \in \{256, 1024, 4096\}$, $nprobe \in [1, 128]$), and IVF-PQ ($m \in \{10, 16, 20, 32\}$).
  - Evaluated Query Difficulty quartiles on GloVe-100 and Dimensionality Curse on synthetic spherical data.
  - Verified regenerated tables and figures in `results/tables/` and `results/figures/`.

---

### Phase 4: Report Authoring & Question Synthesis
- **Prompt**: "Author a comprehensive, maximum 5-page LaTeX report answering Questions 1 to 5 with mechanistic 'why' explanations, and compile into report.pdf."
- **Summary of Response / Action**:
  - Authored `report/report.tex` with two-column layout, formal tables, and embedded high-resolution figures.
  - Formulated mechanistic answers explaining BLAS GEMM vs. GEMV arithmetic intensity, cache line reuse, HNSW semantic hub trapping, Voronoi cell boundary leakage, Beyer et al. distance concentration, and engineering deployment trade-offs.
  - Successfully compiled with `pdflatex` to strictly 5 pages and copied `report.pdf` to root workspace.

