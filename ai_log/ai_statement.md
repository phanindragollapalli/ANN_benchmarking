# AI Statement

In accordance with course academic integrity policies, this document details the AI tools utilized, their purpose, and the categorization of each source file.

## Tools Used
- Google Antigravity / Gemini: Used as an interactive pair-programming assistant for auditing code, resolving benchmarking bugs, enforcing single-threaded CPU benchmarks, creating unit tests, generating Pareto frontier plots, and compiling the technical report.

## Source File Attribution

| File Path | Attribution Category | Notes / Rationale |
|---|---|---|
| `code/requirements.txt` | AI-assisted | Dependency list based on project specifications |
| `code/hypothesis.md` | Human-written & AI-assisted | Pre-committed hypotheses formulated prior to experiments |
| `code/src/data_loader.py` | AI-assisted | HDF5 loader, Euclidean distance & GloVe unit normalization |
| `code/src/metrics.py` | AI-assisted | Recall@10 with exact distance boundary tie-handling |
| `code/src/exact_baseline.py` | AI-assisted | FAISS IndexFlatL2 implementation, sequential vs batch benchmarks |
| `code/src/index_benchmarks.py` | AI-assisted | HNSW, IVF-Flat, IVF-PQ sweeps and Pareto frontier plotting |
| `code/src/generate_tables.py` | AI-assisted | Summary tables generator for QPS, build time, and memory costs |
| `code/src/query_difficulty.py` | AI-assisted | Relative contrast quartiles evaluation on GloVe-100 |
| `code/src/synthetic_generator.py` | AI-assisted | Seeded hypersphere generator with NumPy distance spread |
| `code/src/hard_data_experiment.py` | AI-assisted | Dimensionality curse failure mode experiment ($d \in \{16, 64, 128, 256\}$) |
| `code/tests/test_metrics.py` | AI-assisted | Pytest test suite for Recall@10 tie-breaking and edge cases |
| `code/run_all.sh` | AI-assisted | End-to-end execution and `--quick` reproduction runner |
| `code/README.md` | Human-written & AI-assisted | Setup, system requirements, and reproduction instructions |
| `report.pdf` | Human-written & AI-assisted | LaTeX report covering all 5 core analytical questions |
