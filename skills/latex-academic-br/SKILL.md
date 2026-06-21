---
name: latex-academic-br
description: Use when writing or setting up a Brazilian academic paper (TCC, dissertação, artigo) in LaTeX following ABNT standards via abnTeX2. Covers project structure, the LuaLaTeX + Mermaid diagram rendering workflow, Makefile build (PDF + DOCX via Pandoc), GitHub Actions CI, and the Markdown-first drafting approach. Trigger when the user asks about LaTeX for academic papers, ABNT formatting, abnTeX2, compiling a monograph, or embedding diagrams in a thesis.
---

# LaTeX Academic Paper (Brazilian ABNT)

Workflow for writing Brazilian academic papers (TCC, dissertação, artigo) in LaTeX using [abnTeX2](https://www.abntex.net.br/). Covers project structure, building to PDF and DOCX, embedding Mermaid diagrams, and CI automation.

## References

```text
references/
  project-structure.md   Directory layout, chapter files, config files, bibliography
  makefile.md            Build commands: pdf, docx, clean, distclean
  mermaid-in-latex.md    ltmermaid package: rendering Mermaid diagrams inside LuaLaTeX
  ci-workflow.md         GitHub Actions: LuaLaTeX + Pandoc + Mermaid in CI
  drafting-workflow.md   Markdown-first drafting before migrating to .tex
```

## When to use

- Setting up a new academic paper project
- Adding diagram support to an existing LaTeX document
- Configuring CI to compile and publish the PDF automatically
- Understanding ABNT chapter/section structure in abnTeX2

## Quick Reference

### Build commands

```bash
make all    # Build PDF + DOCX
make pdf    # Build only PDF (LuaLaTeX × 3 passes + BibTeX)
make docx   # Convert to DOCX via Pandoc (requires make pdf first)
make clean  # Remove LaTeX auxiliary files
```

### Embedding a Mermaid diagram

Requires `ltmermaid.sty` (local package) and `-shell-escape` flag (already set in the Makefile). Place the package file alongside `documento.tex`.

```latex
\usepackage{ltmermaid}

\begin{mermaid}
graph TD
  A[Coleta de Dados] --> B[Pré-processamento]
  B --> C[Treinamento]
  C --> D{Avaliação}
  D -->|Aprovado| E[Deploy]
  D -->|Reprovado| B
\end{mermaid}
```

At compile time, LuaLaTeX calls `npx -y @mermaid-js/mermaid-cli` to render each diagram to PNG and includes it. Requires Node.js available in `$PATH`.

### Recommended project structure

```
documento/
  documento.tex          Main entry point (includes estrutura/)
  estrutura/
    preambulo.tex        \usepackage declarations, custom commands
    pre-textual.tex      \include{pre-textual/*}
    textual.tex          \include{capitulos/*}
    pos-textual.tex      \include{pos-textual/*}
  capitulos/
    01-introducao.tex
    02-referencial.tex
    03-metodologia.tex
    04-resultados.tex
    05-conclusao.tex
  configuracoes/
    metadados.tex        \titulo, \autor, \data, \instituicao
    citacoes.tex         \citebracket, custom citation commands
  pre-textual/
    resumo.tex
    abstract.tex
    listas.tex           \listoffigures, \listoftables
  pos-textual/
    referencias.tex      \bibliography{bibliografia}
    apendices.tex
  bibliografia.bib
  ltmermaid.sty          Local Mermaid package
  Makefile
  output/                Generated PDF and DOCX (gitignore this)
```

### ABNT chapter structure (abnTeX2)

```latex
% In textual.tex
\textual

\chapter{Introdução}
\input{capitulos/01-introducao}

\chapter{Referencial Teórico}
\input{capitulos/02-referencial}

\chapter{Metodologia}
\input{capitulos/03-metodologia}

\chapter{Resultados e Discussão}
\input{capitulos/04-resultados}

\chapter{Conclusão}
\input{capitulos/05-conclusao}
```

### Markdown-first drafting

Write in `texto-base.md` as a scratchpad. Use `<!-- TODO: ... -->` comments for open questions. Migrate section by section to `.tex` files when the text stabilizes. The Markdown draft serves as a working memory for what still needs to be written, researched, or revised — not as an intermediate compilation step.
