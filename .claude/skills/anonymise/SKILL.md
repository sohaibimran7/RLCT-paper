---
name: anonymise
description: Anonymise the paper for review by switching main.tex from the deanonymised authblk/natbib layout back to the ICML 2025 template. Use when the user asks to anonymise the paper, hide author identities for submission, or switch back to the ICML template.
---

# Anonymise the paper (switch to ICML 2025 template)

Convert `main.tex` from the deanonymised layout (`authblk` + `natbib`, visible authors, Acknowledgments + Author contributions sections) to the anonymous ICML 2025 template layout. The ICML template hides author identities by default.

This is the inverse of the `deanonymise` skill.

## Preconditions

- Working directory should be `/Users/work/consistency-training-methods/paper/`.
- `main.tex` is currently in the deanonymised state. If unsure, grep for `\\usepackage{authblk}` (deanonymised) vs `\\usepackage{icml2025}` (anonymised). If it's already anonymised, stop and tell the user.
- The ICML template files live in `./ICML2025_Template/` (the `icml2025.sty` and `.bst` files).

## Transformations

Apply the following four edits with the `Edit` tool. Each edit is a single contiguous replacement — do not break them up.

### 1. Preamble: replace authblk/natbib block with ICML template loader

Replace:

```latex
\documentclass[twocolumn]{article}

\usepackage{microtype}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{hyperref}
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage{url}
\usepackage{amsfonts}
\usepackage{amsmath}
\usepackage{amssymb}
\usepackage{xcolor}
\usepackage{tikz}
\usetikzlibrary{positioning, calc, arrows.meta}
\usepackage[margin=1in]{geometry}
\usepackage{authblk}
\usepackage{natbib}
```

with:

```latex
\documentclass{article}

% ICML 2025 template
\usepackage{microtype}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{hyperref}
\makeatletter
\def\input@path{{ICML2025_Template/}}
\makeatother
\usepackage{icml2025}

\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage{url}
\usepackage{amsfonts}
\usepackage{amsmath}
\usepackage{amssymb}
\usepackage{xcolor}
\usepackage{tikz}
\usetikzlibrary{positioning, calc, arrows.meta}
```

### 2. Title block: replace authblk title/author block with ICML title block

The current deanonymised title block looks like:

```latex
\title{<TITLE>}

\author[1]{Sohaib Imran\thanks{Correspondence: \texttt{<EMAIL>}}}
\author[2]{Prakhar Gupta}
\author[3]{Jannes Elstner}
\author[4]{David Africa}
\affil[1]{Lancaster University}
\affil[2]{University of Michigan}
\affil[3]{<JANNES_AFFIL>}
\affil[4]{UK AI Security Institute}

\date{}

\begin{document}

\maketitle
```

Read the file first to capture the current values of:
- `<TITLE>` — the paper title between `\title{...}`
- `<EMAIL>` — the correspondence email inside `\thanks{Correspondence: \texttt{...}}`
- `<JANNES_AFFIL>` — Jannes Elstner's affiliation. May be `Apollo Research` or `Independent\thanks{Now at Apollo Research}`. Preserve whatever is there, but for ICML translate `\thanks{...}` to `\protect\footnote{...}`.

Replace the deanonymised block with:

```latex
\icmltitlerunning{Reinforcement Learning Consistency Training}

\begin{document}

\twocolumn[
\icmltitle{<TITLE>}

% Keep real authors here; ICML style anonymizes them unless [accepted] is used.
\begin{icmlauthorlist}
\icmlauthor{Sohaib Imran}{b}
\icmlauthor{Prakhar Gupta}{c}
\icmlauthor{Jannes Elstner}{e}
\icmlauthor{David Africa}{f}
\end{icmlauthorlist}

\icmlaffiliation{b}{Lancaster University}
\icmlaffiliation{c}{University of Michigan}
\icmlaffiliation{e}{<JANNES_AFFIL_ICML>}
\icmlaffiliation{f}{UK AI Security Institute}

\icmlcorrespondingauthor{Sohaib Imran}{<EMAIL>}

\icmlkeywords{Reinforcement Learning, Consistency Training, Robustness}

\vskip 0.3in
]

\printAffiliationsAndNotice{}
```

Where `<JANNES_AFFIL_ICML>` is the affiliation with any `\thanks{...}` rewritten as `\protect\footnote{...}` (the `icmlaffiliation` argument can't take a bare `\thanks`).

### 3. Remove the Acknowledgments and Author contributions sections

Delete the two `\section*{...}` blocks that appear immediately before `\bibliography{...}`. The block looks like:

```latex
\section*{Acknowledgments}
We thank Tim Hua ... compute for this research.

\section*{Author contributions}
JE and SI conceptualised the method. ... DA
supervised the project.

\bibliography{references, ai_safety}
```

Replace it with just:

```latex
\bibliography{references, ai_safety}
```

Save the deleted Acknowledgments and Author contributions text verbatim somewhere you can read later (e.g. note it in the response) so the `deanonymise` skill can restore it. Better: don't lose it — the `deanonymise` skill keeps a canonical version, but if you're worried about drift, mention the current text in your reply so the user can confirm.

### 4. Bibliography style

Replace:

```latex
\bibliographystyle{plainnat}
```

with:

```latex
\bibliographystyle{ICML2025_Template/icml2025}
```

## After editing

1. Build the PDF to confirm no compile errors:

```bash
pdflatex -interaction=nonstopmode -halt-on-error main.tex > /tmp/latex1.log 2>&1 \
  && bibtex main > /tmp/bibtex.log 2>&1 \
  && pdflatex -interaction=nonstopmode -halt-on-error main.tex > /tmp/latex2.log 2>&1 \
  && pdflatex -interaction=nonstopmode -halt-on-error main.tex > /tmp/latex3.log 2>&1
```

If any pass exits non-zero, inspect the relevant log and fix.

2. Report what changed (1–2 sentences). Do not commit unless the user asks.
