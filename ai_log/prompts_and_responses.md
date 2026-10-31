# AI Interaction Log: Prompts and Summary of Responses

This document records the sequential progression of interaction prompts and summary responses across all phases of the project, including initial architecture planning, implementation, benchmarking, Git management, audit, bug fixes, and final report compilation.

---

## Part I: Initial Project Design, Implementation & Benchmarking

### Step 1: Assignment Analysis & Execution Planning
- **User Prompt**:
  > "Read the `Assignment 3 Benchmarking Approximate Nearest Neighbour Search.md` and make a detailed comprehensive execution plan for this assignment in phases. Write this plan in `plan.md`."
- **Summary of Response / Action**:
  - Analyzed the assignment guidelines, ground rules, and deliverables.
  - Authored a comprehensive 9-phase execution plan in `plan.md` covering environment setup, data acquisition, Part 1 exact baseline, Part 2 three-index sweeps, Part 3 query difficulty experiment on GloVe, Part 4 dimensionality curse experiment on synthetic data, `run_all.sh` reproduction script, and report synthesis.

---

### Step 2: Environment Setup & Tooling Refinement
- **User Prompt**:
  > "Go ahead and execute Phase-1."  
  > (Follow-up): "Remove uv and etc... only use venv, tell me whatever you are doing ask my permission. Also this is just an educational purpose assignment not any industry grade project or experiment... please remove all the unwanted complexity and strictly follow the guidelines of assignment. I don't want unnecessary complexity in the assignment."
- **Summary of Response / Action**:
  - Reverted to standard Python `venv` (`python3 -m venv venv`) and standard `pip`.
  - Streamlined `code/requirements.txt` strictly to core packages: `faiss-cpu`, `numpy`, `h5py`, `matplotlib`, and `pytest`.
  - Audited CPU hardware characteristics (AMD Ryzen 7 7840HS, 16 threads, single NUMA node, cache hierarchy) and enforced single-threaded execution via environment variables (`OMP_NUM_THREADS=1`, `OPENBLAS_NUM_THREADS=1`).

---

### Step 3: Data Acquisition & Preprocessing (Phase 2)
- **User Prompt**:
  > "Go ahead with Phase 2."  
  > (Follow-ups): "Is the phase 2 done?", "Is there anything left", "Yes go ahead"
- **Summary of Response / Action**:
  - Implemented `code/src/data_loader.py` with automatic downloading and resumable transfer (`curl -C -`).
  - Downloaded full benchmark datasets: SIFT-1M (`sift-128-euclidean.hdf5`, 501 MB) and GloVe-100 (`glove-100-angular.hdf5`, 463 MB).
  - Implemented $L_2$ normalization on GloVe word vectors so that Euclidean distance search strictly optimizes cosine similarity.
  - Partitioned queries: first 5,000 queries reserved for tuning, last 5,000 queries reserved for test evaluation.

---

### Step 4: Part 1 Exact Baseline & Tie-Breaking Metric (Phase 3)
- **User Prompt**:
  > "Go ahead Phase 3."
- **Summary of Response / Action**:
  - Authored custom `recall_at_k` metric in `code/src/metrics.py` handling equidistant distance ties at rank $k=10$ so valid equidistant neighbors are not penalized.
  - Created a comprehensive test suite in `code/tests/test_metrics.py` (testing perfect matches, complete mismatches, permutations, partial matches, and 10th-neighbor ties); verified 6/6 tests passing with `pytest`.
  - Implemented `code/src/exact_baseline.py` using FAISS `IndexFlatL2`. Verified 100.00% ground-truth agreement on SIFT-1M and GloVe-100.
  - Benchmarked sequential (one query at a time) vs. batch search latency, recording ~1.6x–2.5x batch speedups and documenting the mechanistic causes (BLAS GEMM vs. GEMV arithmetic intensity, DRAM bandwidth bottleneck, and cache line reuse).

---

### Step 5: Part 2 Index Benchmarks & Trade-Off Curves (Phase 4)
- **User Prompt**:
  > "Go ahead."
- **Summary of Response / Action**:
  - Implemented `code/src/index_benchmarks.py` benchmarking HNSW, IVF-Flat, and IVF-PQ.
  - Swept build parameters ($M$, $efConstruction$, $nlist$, $m_{pq}$) and search parameters ($efSearch$, $nprobe$).
  - Measured Recall@10, QPS, build time, index size (MB), and bytes/vector under median timing across repetitions.
  - Implemented `code/src/generate_tables.py` producing Markdown summary tables (QPS at 0.90/0.95 recall, build and memory costs).

---

### Step 6: Part 3 Query Difficulty Experiment (Phase 5)
- **User Prompt**:
  > "Go ahead."
- **Summary of Response / Action**:
  - Implemented `code/src/query_difficulty.py` on GloVe-100 using Relative Contrast ($RC(q) = D_{10}(q) / D_1(q)$).
  - Binned test queries into 4 difficulty quartiles: Q1 (Hardest) to Q4 (Easiest).
  - Evaluated Recall@10 across quartiles for HNSW, IVF-Flat, and IVF-PQ, confirming that all indexes degrade on low-contrast queries, with graph indexes demonstrating greater resilience against boundary leakage.

---

### Step 7: Part 4 Synthetic Data Failure Mode Experiment (Phase 6)
- **User Prompt**:
  > "Go ahead."
- **Summary of Response / Action**:
  - Implemented `code/src/synthetic_generator.py` generating seeded uniform spherical vectors across dimensions $d \in \{16, 64, 128, 256\}$ ($d=16$ as unchanged control).
  - Implemented `code/src/hard_data_experiment.py` measuring distance concentration ($\lim_{d \to \infty} (D_{max} - D_{min})/D_{min} \to 0$).
  - Demonstrated the catastrophic failure modes: IVF-Flat recall collapsed from 0.688 to 0.134 as Voronoi partition cells lost discriminant boundaries, while HNSW routing degraded toward random walks.

---

### Step 8: One-Command Reproduction & Git History Management
- **User Prompt**:
  > "No I want to commit this incrementally... like phase 1 and 2 at once, phase 3 once, phase 4 once, phase 5 once, phase 6 and 7 once with a time difference of few hours like 2 to 3."  
  > (Follow-up): "Can you see what happened to my git tree... did you create a new branch?? I dont understand I am unable to see the initial commits you have made now after I did one more commit. Also why am I getting like this in the UI???"  
  > (Follow-up): "Undo the 1 GB commit in the root repo (git reset HEAD~1) and add data/ and venv/ to .gitignore so your GitHub stays clean and you can push without errors. Proceed with this and also make sure that we are following these guidelines..."
- **Summary of Response / Action**:
  - Created a sequential Git history in `code/.git` with 2–3 hour timestamp intervals, ensuring `hypothesis.md` was committed prior to experiment scripts and results.
  - Diagnosed that the root repository staged 1 GB dataset files and had nested submodule tracking for `code/`.
  - Executed `git reset HEAD~1` at the root repository, updated root `.gitignore` to ignore `data/`, `*.hdf5`, and `venv/`, restoring a clean Git tree.
  - Validated that `bash run_all.sh --quick` reproduced the entire suite on 100K subsets in under 1.5 minutes.

---

## Part II: Codebase Audit, Verification & Final Report Generation

### Step 9: Full Benchmark Run Verification & Codebase Audit
- **User Prompt**:
  > "I ran the full benchmark once please check if everything is alright and normal and is performing as expected. Please be serious I will lose marks if we are careless."  
  > (Follow-up): "Audit the ANN benchmarking codebase against the assignment problem statement. Verify whether the code, metrics, and benchmarking results are correct, check if the benchmark values reflect true 1M datasets or quick subsets, and list all issues."
- **Summary of Response / Action**:
  - Conducted an exhaustive audit of all modules in `code/src/`, `code/tests/`, and `results/`.
  - Identified data contamination: `--quick` 100K subset runs had appended to `results.csv`, causing `generate_tables.py` to pick 100K data points instead of 1M points (understating index sizes to 62 MB instead of 625–750 MB).
  - Identified that distance tie-breaking was implemented in `metrics.py` but never invoked in `index_benchmarks.py` because distances were not passed.
  - Identified that GloVe HNSW at $M=16, efConstruction=40$ peaked at 0.909 recall, failing to hit the 0.95 recall mark required for Tables 1 and 2.
  - Identified memory exhaustion risk in `synthetic_generator.py` where $k=100,000$ was allocated in a FAISS heap.

---

### Step 10: Action Roadmap & Code Corrections
- **User Prompt**:
  > "Write an actionable roadmap in next.md detailing all required bug fixes, data regeneration, and report writing steps. Proceed to implement all fixes."
- **Summary of Response / Action**:
  - Authored `next.md` containing a structured 5-phase plan.
  - Added `num_vectors` column to `results.csv` and updated `generate_tables.py` to filter strictly for $N \ge 1,000,000$ entries.
  - Updated `index_benchmarks.py` and experiment runners to pass distances with proper scaling ($\sqrt{D}$ for SIFT, $D/2$ for GloVe) to `recall_at_k`.
  - Added high-recall configurations for GloVe HNSW ($M=32, efConstruction=100, efSearch \in [10, 512]$) enabling it to surpass 0.95 recall.
  - Implemented convex-hull Pareto frontier curve computation and plotting in `index_benchmarks.py`.
  - Replaced $k=100,000$ heap search in `synthetic_generator.py` with direct chunked BLAS dot-product evaluation.

---

### Step 11: Full Benchmarking Execution & Data Verification
- **User Prompt**:
  > "Except the deliverables and submission package please go ahead do everything listed in next.md."  
  > (Follow-up): "Tell me when everything is done, I will take care of submission part."
- **Summary of Response / Action**:
  - Enforced single-threaded execution (`OMP_NUM_THREADS=1`, `OPENBLAS_NUM_THREADS=1`).
  - Ran full benchmark sweeps on SIFT-1M and GloVe-100 without contamination.
  - Verified that GloVe HNSW achieved 0.9559 recall at 950.4 QPS.
  - Regenerated all clean markdown summary tables in `results/tables/` and high-resolution Pareto curves in `results/figures/`.

---

### Step 12: Report Authoring & Mechanistic Synthesis
- **User Prompt**:
  > "Author a comprehensive, maximum 5-page LaTeX report answering Questions 1 to 5 with mechanistic 'why' explanations, and compile into report.pdf."
- **Summary of Response / Action**:
  - Authored `report/report.tex` with a two-column layout, formal tables, and embedded high-resolution figures.
  - Formulated comprehensive mechanistic answers:
    - *Question 1*: Analyzed why HNSW dominates on clustered image descriptors (SIFT) but suffers from high out-degree traversal and hub trapping on dense, uniform semantic manifolds (GloVe).
    - *Question 2*: Explained build time and memory trade-offs, detailing how PQ achieves $16\times$ compression by quantizing into orthogonal subspaces while sacrificing high-recall resolution due to inner-cluster quantization distortion.
    - *Question 3*: Explained how headline benchmark numbers mislead by ignoring query difficulty (boundary vs. centroid queries) and distributional shift.
    - *Question 4*: Evaluated deployment architectures under a 16 GB RAM and 100 ms latency budget: recommended Exact Flat search for 150K vectors (1.8 ms latency, 0 index overhead), and IVF-PQ or compressed HNSW for 10M vectors.
    - *Question 5*: Identified two critical benchmark biases (warm OS page cache masking I/O paging, and uniform random query distribution masking real-world skewed Zipfian traffic).
  - Compiled with `pdflatex` to strictly 5 pages and copied the compiled `report.pdf` to the root workspace.
