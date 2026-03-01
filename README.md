# Prompt Evaluation Tool

A browser-based tool suite for human annotation and quality scoring of AI prompts. No installation or server required.

## Overview

This project provides four standalone HTML tools to support the full lifecycle of prompt evaluation: creating annotation datasets, scoring prompts against structured criteria, and visualizing results across evaluators.

## Tools

| File | Title | Purpose |
|------|-------|---------|
| `annotator.html` | Prompt Data Annotator | Create and annotate prompt evaluation datasets |
| `index.html` | Prompt Evaluation Tool | Score individual prompts against evaluation criteria |
| `analyze.html` | Prompt Evaluation Dashboard | Aggregated results visualization with charts |
| `individual.html` | Individual Evaluator Dashboard | Per-evaluator performance breakdown |

## Usage

1. Clone or download this repository
2. Open any `.html` file directly in a browser — no server or build step needed
3. Load a data file from the `data/` directory (or use the bundled sample data)
4. Annotate prompts or review evaluation results
5. Export results as JSON for further analysis

## Evaluation Criteria

Scoring criteria are defined in `config.json`:

| Criterion | Description | Scale |
|-----------|-------------|-------|
| Clarity | How clear and understandable the prompt is | 1–5 |
| Role Assignment | Whether the prompt assigns a role/persona to the LLM | Yes / No |
| Prompt Structure Order | Role → Task → Output format ordering is followed | 1–3 |
| Target Audience | How explicitly the intended audience is specified | 1–5 |
| Scope Limitation | How well the prompt limits its topic scope | 1–5 |
| Output Format & Examples | Whether output format and concrete examples are provided | 1–5 |

## Project Structure

```
eval-score/
├── index.html          # Main prompt evaluation tool
├── annotator.html      # Annotation / data creation tool
├── analyze.html        # Aggregated results dashboard
├── individual.html     # Per-evaluator dashboard
├── config.json         # Evaluation criteria and scoring options
├── mapping.json        # Score label mappings for display
├── data.json           # Sample evaluation data
└── data/               # Evaluation result JSON files (per evaluator)
```

## Data Format

Evaluation results are stored as JSON files. The `data/` directory contains result files exported from the annotation tool. Sample data files (`data.json`, `data_2026-02-28_*.json`) are included for reference.

## Configuration

To customize evaluation criteria, edit `config.json`. Each criterion supports:
- `name` — criterion identifier
- `description` — display label
- `options` — array of `{ value, note }` scoring options
- `type: "boolean"` — for Yes/No criteria (optional)

## License

MIT
