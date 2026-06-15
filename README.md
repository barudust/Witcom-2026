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
└── figures/               # Figures used by the paper
    ├── *.pdf              #   vector versions — used by paper.tex when compiling
    └── *.png              #   raster versions — for quick preview (GitHub, slides)
```

## Build

Requires a LaTeX distribution (TeX Live / MiKTeX). The `llncs` class is part of
the standard distributions and installs automatically on first run with MiKTeX.

```bash
pdflatex paper.tex
pdflatex paper.tex   # second pass resolves \ref / \cite
```

No BibTeX run is needed: references use an inline `thebibliography`.

## A note on figure formats

`paper.tex` embeds the **PDF** figures on purpose. PDF is a *vector* format: the
plots stay perfectly sharp at any zoom or print size, and the LNCS guidelines
explicitly recommend vector graphics over rasterized images. This is the correct
setup for the paper.

For convenience, an equivalent **PNG** of each figure is included as well, since
PNGs preview inline on the GitHub web interface and are easy to drop into slides
or documents. The paper does not use the PNGs; they are previews only. To switch
the paper to PNGs (not recommended), change the extensions in the
`\includegraphics{...}` calls inside `paper.tex`.

## Notes for co-authors

Placeholders to fill before camera-ready:

- Author names, affiliations and e-mails (title block in `paper.tex`).
- Acknowledgments.
- Confirm the CLEF 2026 FinMMEval venue metadata (footnote 1).

All reported numbers are derived from the project's experimental artifacts.
