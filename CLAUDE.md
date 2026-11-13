# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Exploratory data analysis of user experience reports from the French public-services feedback platform ("Services Publics +", hence `spplus` in notebook names). Work happens in Jupyter notebooks under `notebooks/`, numbered sequentially (`01_exploration_spplus.ipynb`, …). Code comments, outputs and folder names are in French — follow that convention.

## Environment

- Python >= 3.14, managed with `uv` (`pyproject.toml` + `uv.lock`, virtualenv in `.venv/`).
- `uv sync` — install dependencies (including the `dev` group, which provides `ipykernel` for notebooks).
- `uv add <pkg>` / `uv add --dev <pkg>` — add a dependency; keep `uv.lock` committed.
- `uv run python …` or `.venv/bin/python …` — run scripts against the project env.
- No test suite, linter or build step exists yet.

## Data

- `data/brutes/` holds raw exports (gitignored); `data/traitees/` is the intended location for processed outputs (also gitignored). Never commit data files.
- Main dataset: `data/brutes/export-experiences_<date>.parquet` (~170k rows, 70 columns), one row per "expérience" (a user's account of a procedure). Load with `pd.read_parquet`.
- Shape of the export, which matters for any analysis:
  - Repeated column groups numbered 1–3: `… Typologie 1/2/3` (procedure type, channel, postcode) and `… structure 1/2/3` (the administration concerned, its response, visitor/author ratings of the response, AI-similarity rate). Groups 2 and 3 are almost entirely null; reshape to long format when analysing across them.
  - Column names are irregular: double spaces (`'Statut réponse structure  1'`, `'Evaluation  réponse structure 1 par auteur'`) and trailing spaces (`'Code Postal typologie 2 '`, `'ID structure 3 '`). Copy names exactly from `df.columns`, or normalise them (`" ".join(c.split())`) right after loading.
  - Date columns (`Écrit le`, `Date de publication`, `Date de premiere réponse`) load as `object` and need `pd.to_datetime`.
  - Some columns that should be numeric are strings (e.g. `Taux de similarité réponse IA structure 2/3`).
  - Free-text fields: `Titre`, `Description`, `Réponse structure N`. Thematic criteria columns (`Accessibilité`, `Information/Explication`, `Relation`, `Réactivité`, `Simplicité/Complexité`) and `Ressenti usager` carry the sentiment signals.
