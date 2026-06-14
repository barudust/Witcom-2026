# Gated Multimodal Fusion with SWA for Trading Decisions — WITCOM paper

LaTeX sources and compiled PDF for the WITCOM submission (Springer LNCS format).

The paper presents a four-branch multimodal architecture (price series + FinBERT
news + corporate filings) with gated fusion and availability masking, validated
on the FinMMEval Task 3 corpus.

## Contents

```
.
├── paper.tex              # Manuscript (Springer LNCS / llncs class)
├── paper.pdf              # Compiled output (12 pp.)
├── figures/               # Vector figures (PDF) used by the paper
│   ├── fig_equity_curves.pdf
│   ├── fig_per_asset_sharpe.pdf
│   ├── fig_training_curves.pdf
│   ├── fig_confusion_matrix.pdf
│   └── fig_class_distribution.pdf
└── scripts/               # Scripts that regenerate the figures from real results
    ├── 10_make_paper_figures.py
    └── 11_seed_variance.py
```

## Build

Requires a LaTeX distribution (TeX Live / MiKTeX). The `llncs` class is part of
the standard distributions and installs automatically on first run with MiKTeX.

```bash
pdflatex paper.tex
pdflatex paper.tex   # second pass resolves \ref / \cite
```

No BibTeX run is needed: references use an inline `thebibliography`.

## Notes for co-authors

Placeholders to fill before camera-ready:

- Author names, affiliations and e-mails (title block in `paper.tex`).
- Acknowledgments.
- Confirm the CLEF 2026 FinMMEval venue metadata (footnote 1).

All reported numbers are derived from the project's experimental artifacts; the
figures under `figures/` are produced by `scripts/10_make_paper_figures.py` and
the seed-variance table by `scripts/11_seed_variance.py`. These scripts depend on
the full training pipeline and are included for transparency and reproducibility
rather than as a one-click build.
