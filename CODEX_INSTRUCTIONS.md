# Codex Build Notes / High-Level Instructions

This repo was built by following the user's instructions from the chat thread. The intent is to keep a faithful, high-level record of what was requested and how artifacts were produced.

## Inputs The User Pointed To
- Marin documentation:
  - Marin 8B retrospective: `https://marin.readthedocs.io/en/latest/reports/marin-8b-retro/`
  - Marin 32B retrospective: `https://marin.readthedocs.io/en/latest/reports/marin-32b-retro/`
  - Auto-generated experiments/history summary: `https://marin.readthedocs.io/en/latest/reports/summary/`
- Local exemplars (for structure/style): `other_papers/` contains LaTeX sources for:
  - OLMo 2 technical report
  - OLMo 3 technical report

## What The User Asked For (Chronological)
1. Confirm understanding that the project is an Overleaf/LaTeX document.
2. Read Marin retrospectives (8B + 32B) and the experiments/history summary closely.
3. Explain at a high level:
   - What Marin is.
   - What "8B" and "32B" refer to.
   - What the major "issues" were (stability, data mix, shuffling, contamination, etc.).
4. Create an academic-paper outline in `PAPER_OUTLINE.md`.
   - Model it after high-quality open-model technical reports (OLMo2/OLMo3/DeepSeek-style).
   - Make it very detailed.
   - Use specifics from the Marin documentation (datasets, phases, hyperparameters, evaluations).
5. Inspect `other_papers/` and read the OLMo2/OLMo3 TeX papers to get context.
6. Produce comprehensive citation-focused reading notes:
   - `OLMO2.md`: summarize takeaways; emphasize related work and why citations matter; cover most/all bib citations.
   - `OLMO3.md`: same emphasis (related work + citation intent), as comprehensive as possible.
7. Re-read the Marin documentation again and update `PAPER_OUTLINE.md` to be even more detailed.
   - Be explicit about datasets, mixtures, and what is interesting to compare against.
   - Sprinkle relevant BibTeX keys (from OLMo2/OLMo3 bibliographies) through the outline.
8. Write a full paper (LaTeX) from `PAPER_OUTLINE.md`.
   - Re-check the retrospectives.
   - Download and include the figures/images from the Marin retrospectives where appropriate.
   - Use OLMo2/OLMo3 citation context to shape Related Work and comparisons.
   - Create a multi-file LaTeX structure: `sections/intro.tex`, `sections/related.tex`, `sections/methods.tex`, `sections/experiments.tex`, `sections/conclusion.tex`.
   - Add a BibTeX file used by the paper.

## What Was Actually Generated In This Repo
- `OLMO2.md`
  - Citation-driven reading notes for OLMo 2.
  - Includes an annotated index of bib entries from the OLMo 2 build artifacts, plus a "Related Work Map" grouped by purpose.
- `OLMO3.md`
  - Citation-driven reading notes for OLMo 3.
  - Includes an annotated index of bib entries and a role-based "Related Work Map".
- `PAPER_OUTLINE.md`
  - A detailed academic paper plan for Marin 8B/32B, written in the "model flow" / fully-open technical report style.
  - Includes explicit dataset mix tables, phase structure, evaluation plan, and citation keys.
- Paper implementation (requested next):
  - `sections/` folder with per-section `.tex` files.
  - `figures/` folder containing images downloaded from Marin retrospectives.
  - `references.bib` containing BibTeX entries required by the LaTeX paper.
  - `main.tex` updated to compile the full paper.

## Operational Notes
- Marin retro figures were discovered by parsing the HTML pages and extracting referenced `.png` assets.
- Where possible, BibTeX entries were sourced from the OLMo2/OLMo3 paper bibliographies already present in `other_papers/`.
- For items not present in those bibliographies (e.g., project web pages, specific datasets, tooling repos), small `@misc` entries are added.
