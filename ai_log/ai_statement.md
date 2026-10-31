# AI Statement

In accordance with course academic integrity policies, this document details the AI tools utilized, their purpose, and the categorization of each source file.

## Tools Used
- Google Antigravity / Gemini 3.8 Flash: Used as an interactive pair-programming assistant for boilerplate scaffolding, data loading scripts, experiment runner scripts, and generating metric evaluation plots.

## Source File Attribution

| File Path | Attribution Category | Notes / Rationale |
|---|---|---|
| `code/requirements.txt` | AI-assisted | Dependency list based on project specifications |
| `code/hypothesis.md` | Human-written & AI-assisted | Hypotheses formulated and refined prior to experiments |
| `code/src/env_info.py` | AI-assisted | System auditing and single-thread enforcement |
| `code/src/data_loader.py` | AI-assisted | HDF5 loader, Euclidean distance & GloVe normalization |
| `code/src/metrics.py` | AI-assisted | Recall@10 with exact tie-breaking logic |
| `code/src/exact_baseline.py` | AI-assisted | FAISS IndexFlatL2 implementation & validation |
| `code/src/timer.py` | AI-assisted | High-precision median timing and warmup utility |
| `code/src/index_benchmarks.py` | AI-assisted | HNSW, IVF-Flat, IVF-PQ wrappers |
| `code/src/synthetic_generator.py` | AI-assisted | Deterministic high-dimensional vector space generator |
| `code/src/plotter.py` | AI-assisted | Matplotlib Pareto curve & benchmark plotting |
| `code/tests/test_metrics.py` | AI-assisted | Pytest suite for Recall@10 edge cases and ties |
| `code/run_all.sh` | AI-assisted | End-to-end execution and quick verification script |
| `code/README.md` | Human-written & AI-assisted | Reproduction and architectural documentation |
| `report.pdf` | Human-written & AI-assisted | Analysis, mechanistic interpretation, and final synthesis |
