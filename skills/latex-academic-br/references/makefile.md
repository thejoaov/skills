# Makefile: Build Commands

## Complete Makefile

```makefile
FILE = documento
OUTDIR = ./output

.PHONY: all pdf docx clean distclean help

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) \
	  | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}'

all: pdf docx ## Build PDF and DOCX

pdf: ## Build PDF (LuaLaTeX × 3 + BibTeX)
	@mkdir -p $(OUTDIR)
	lualatex -shell-escape -interaction=nonstopmode $(FILE).tex
	bibtex $(FILE).aux || true
	lualatex -shell-escape -interaction=nonstopmode $(FILE).tex
	lualatex -shell-escape -interaction=nonstopmode $(FILE).tex
	@mv $(FILE).pdf $(OUTDIR)/$(FILE).pdf
	@echo "PDF: $(OUTDIR)/$(FILE).pdf"

docx: pdf ## Convert to DOCX via Pandoc
	@mkdir -p $(OUTDIR)
	pandoc $(FILE).tex -o $(OUTDIR)/$(FILE).docx \
	  --bibliography=bibliografia.bib \
	  --citeproc \
	  --resource-path=. \
	  || echo "WARN: pandoc not installed or conversion failed"
	@echo "DOCX: $(OUTDIR)/$(FILE).docx"

clean: ## Remove LaTeX auxiliary files
	rm -rf \
	  *.aux *.bbl *.toc *.out *.log \
	  *.nls *.nlo *.lof *.lot *.blg *.ilg \
	  *.idx *.ind *.lop *.fls *.fdb_latexmk \
	  *.synctex.gz *.bcf *.run.xml *.xdv \
	  *.nav *.snm mermaid

distclean: clean ## Remove everything including generated PDFs
	rm -rf $(OUTDIR)
```

## Why three LuaLaTeX passes?

1. **First pass**: resolves internal references (labels, citations), writes `.aux`
2. **BibTeX**: reads `.aux`, generates `.bbl` from `bibliografia.bib`
3. **Second pass**: reads `.bbl`, resolves citation numbers/names
4. **Third pass**: ensures cross-references (page numbers in `\listoffigures`, etc.) are correct

For documents without `\ref` chains or lists of figures, two passes may suffice.

## `-shell-escape` requirement

Required by `ltmermaid.sty` to call `npx -y @mermaid-js/mermaid-cli` at compile time. Without this flag, Mermaid blocks will fail silently or raise an error. The flag is safe for local builds; CI should use a locked Node.js environment.

## Pandoc DOCX notes

Pandoc converts `.tex` to `.docx` on a best-effort basis. ABNT-specific formatting (capa, folha de aprovação) does not translate — the DOCX is useful for committee review of the text content only, not for submission.

If the institution requires DOCX, provide it alongside the PDF and note it is not the canonical version.
