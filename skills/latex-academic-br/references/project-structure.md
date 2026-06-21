# Project Structure Reference

## Directory layout

```
documento/
  documento.tex                 Main entry point — \input{estrutura/*}
  abntex-ifpi/ (or abntex2/)    Local abnTeX2 customization layer (institution-specific)
  estrutura/
    preambulo.tex               All \usepackage declarations and custom commands
    pre-textual.tex             Aggregates pre-textual elements
    textual.tex                 Aggregates body chapters
    pos-textual.tex             Aggregates post-textual elements
  configuracoes/
    metadados.tex               Author, title, institution, date, keywords
    citacoes.tex                Custom citation commands, \citebracket etc.
    tipografia.tex              Font choices, encoding
    espacamento.tex             ABNT line/paragraph spacing
    pdf.tex                     \hypersetup for PDF metadata
  capitulos/                    One .tex per chapter (numbered prefix for ordering)
  pre-textual/
    resumo.tex
    abstract.tex
    listas.tex                  \listoffigures, \listoftables, \lstlistoflistings
    siglas.tex                  \begin{siglas} ... \end{siglas}
  pos-textual/
    referencias.tex             \bibliography{bibliografia}
    apendices.tex
    anexos.tex
  imagens/                      All figures (PDF, PNG, JPG)
  bibliografia.bib              BibTeX database
  ltmermaid.sty                 Local Mermaid package (copy alongside documento.tex)
  Makefile
  texto-base.md                 Working draft (not compiled, just a scratchpad)
  output/                       Generated artifacts — add to .gitignore
```

## Main entry point (`documento.tex`)

```latex
\documentclass[
  12pt,
  openright,
  twoside,
  a4paper,
  chapter=TITLE,
  section=TITLE,
  brazil,
  english
]{abntex2}

\input{configuracoes/metadados}
\input{estrutura/preambulo}

\begin{document}

\pretextual
\input{estrutura/pre-textual}

\textual
\input{estrutura/textual}

\postextual
\input{estrutura/pos-textual}

\end{document}
```

## Bibliography format (BibTeX)

```bibtex
@article{silva2024,
  author  = {Silva, João and Souza, Maria},
  title   = {Título do Artigo},
  journal = {Nome do Periódico},
  year    = {2024},
  volume  = {10},
  number  = {2},
  pages   = {100--120},
  doi     = {10.1000/exemplo}
}

@inproceedings{oliveira2023,
  author    = {Oliveira, Pedro},
  title     = {Título do Trabalho},
  booktitle = {Anais do Simpósio},
  year      = {2023},
  pages     = {45--52},
  address   = {São Paulo, Brasil}
}

@book{pressman2021,
  author    = {Pressman, Roger S. and Maxim, Bruce R.},
  title     = {Engenharia de Software: Uma Abordagem Profissional},
  edition   = {9},
  publisher = {AMGH},
  year      = {2021}
}
```

## ABNT citation commands (abnTeX2)

```latex
\cite{silva2024}           % (SILVA; SOUZA, 2024)
\citeonline{silva2024}     % Silva e Souza (2024)
\apud{silva2024}{jones2020} % (SILVA; SOUZA, 2024 apud JONES, 2020)
```
