# SliceGuard — Artifact

Code and experiment data for **SliceGuard**, an LLM-based framework for detecting
cryptographic API misuses in Python through progressive refinement
(static filtering → cryptography-aware slicing → slice-informed LLM detection).

This artifact contains only what is needed to inspect the method and reproduce the
paper's numbers.

## Layout

```
SliceGuard/           SliceGuard implementation (the method itself)
  Slice_Algorithm.py            Phase 2: cryptography-aware program slicing
  Filter_Algorithm.py           Phase 1: static crypto-API filtering
  LLM_Detection_Algorithm.py    Phase 3: slice-informed LLM detection
  LLM_Detection_Algorithm_FL.py Phase 3 variant used by the filtering-only baseline
  SliceGuard_Implement.py       End-to-end driver (Phase 1 → 2 → 3)
  Filter_LLM_Implement.py       Driver for the filtering-only baseline
  LLM_Cost_Statistics.py        Token / time / cost aggregation
  rule_source_py.py             The 18 cryptographic misuse rules
  calculate_lhr.py              Line Hit Rate (LHR|TP) metric
  compute_lr.py                 Strict line-recall (LR) metric (also covers the static tools)
  reproduce_tables.py           Reproduces Tables 5, 6, 8 from the shipped CSVs (union rule)
  count_lines.py                Line-count utility
  test/                         Small sample inputs

SATs-Implement/       Static-analysis-tool harness (Bandit, Dlint, Semgrep, Cryptolation)
  SATs_Implement.py             Runs all four tools over a target folder → CSV reports
  bandit_json2csv.py            Bandit JSON → CSV
  dlint_json2csv.py             Dlint/flake8 JSON → CSV
  semgrep_json2csv.py           Semgrep JSON → CSV
  crypto_rules.yaml             Semgrep rule set (cryptographic misuse patterns)
  src/                          Cryptolation implementation + shared utilities
  test/                         Small sample inputs

experiment-data/      All data used in the paper
  datasets/                     PyCryptoBench-LLM, Sub-PyCryptoBench-LLM, CryptoMisuseTestset
  results/                      Per-method verdicts and tool reports (see its README.md)
  figures/                      Figure sources
  README.md                     Dataset/result documentation + reproduction notes
```

## Requirements

Python ≥ 3.9. Install the SliceGuard and harness dependencies:

```bash
pip install -r requirements.txt
```

The four static tools are invoked as external commands and must be on `PATH`:

| Tool         | Install                                                   |
|--------------|-----------------------------------------------------------|
| Bandit       | `pip install bandit`                                      |
| Dlint        | `pip install flake8 dlint`                                |
| Semgrep      | `pip install semgrep` (or see semgrep.dev)                |
| Cryptolation | bundled under `SATs-Implement/src/` (no install)          |

## Configuration

SliceGuard calls a chat-completion API. **No credentials are stored in this
repository.** Set the API key through an environment variable:

```bash
# Linux / macOS
export SLICEGUARD_API_KEY="<your-api-key>"

# Windows (PowerShell)
$env:SLICEGUARD_API_KEY = "<your-api-key>"
```

The model name and endpoint are set in the driver scripts (e.g.
`SliceGuard/SliceGuard_Implement.py`), where the paper used
`qwen3-next-80b-a3b-instruct` via an OpenAI-compatible endpoint. Any
OpenAI-compatible provider works — point `API_Url` at its
`/v1/chat/completions`.

## Running SliceGuard

```bash
cd SliceGuard
# Edit source_directory / sliced_directory in SliceGuard_Implement.py to point at
# your input tree, then:
python SliceGuard_Implement.py
```

This runs the three phases in order and writes per-file verdicts to a CSV plus an
`*_Implement_log.txt` with time / token / cost statistics.

For the filtering-only baseline (used as an ablation), run
`Filter_LLM_Implement.py` instead.

## Running the static tools

```bash
cd SATs-Implement
# Set target_folder in SATs_Implement.py (a sub-folder next to the script), then:
python SATs_Implement.py
```

Produces `bandit_report_*.csv`, `dlint_report_*.csv`, `semgrep_report_*.csv`,
and `cryptolation_report_*.csv`.

## Reproducing the paper's numbers

See `experiment-data/README.md`. In short: join each method's
`analysis_result_*.csv` with the ground-truth categories in
`experiment-data/datasets/PyCryptoBench-LLM/metadata/benchmark_info.json`
(or the real-world labels in `CryptoMisuseTestset/ground_truth.xlsx`), then
compute P/R/F1. Two scripts automate this:

- `SliceGuard/reproduce_tables.py` prints Tables 5, 6, and 8 of the paper from the
  shipped CSVs under the **union** positive-decision rule used in the paper.
- `SliceGuard/compute_lr.py` computes the strict line-recall (LR) column for every
  method, including the static tools; `calculate_lhr.py` computes the LHR|TP metric.

## Notes

- Driver scripts keep commented-out input/output paths and model alternatives as a
  record of the experiments; only credentials were removed.
- Token-cost tables in `LLM_Cost_Statistics.py` reflect the providers' prices at the
  time of the experiments.
- Absolute paths from the original execution machine were replaced with placeholders
  in the static-tool reports under `experiment-data/`.

## License

MIT — see `LICENSE`. Copyright (c) 2026 The SliceGuard Authors.

> Replace the copyright holder with the real author list before publishing.
