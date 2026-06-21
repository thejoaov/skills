# CI: GitHub Actions for LaTeX

## Workflow

```yaml
# .github/workflows/build-latex.yml
name: Build LaTeX

on:
  push:
    paths:
      - "documento/**"
  pull_request:
    paths:
      - "documento/**"
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install Mermaid CLI
        run: npm install -g @mermaid-js/mermaid-cli

      - name: Compile LaTeX
        uses: xu-cheng/latex-action@v3
        with:
          root_file: documento.tex
          working_directory: documento
          latexmk_use_lualatex: true
          extra_system_packages: "nodejs npm chromium"
          pre_compile: "npm install -g @mermaid-js/mermaid-cli"
          args: "-shell-escape -interaction=nonstopmode -file-line-error"

      - name: Generate DOCX
        run: |
          sudo apt-get install -y pandoc
          cd documento
          pandoc documento.tex -o output/documento.docx \
            --bibliography=bibliografia.bib \
            --citeproc \
            --resource-path=. \
            || echo "DOCX conversion failed — continuing"

      - name: Upload PDF
        uses: actions/upload-artifact@v4
        with:
          name: tcc-pdf
          path: documento/documento.pdf

      - name: Upload DOCX
        uses: actions/upload-artifact@v4
        with:
          name: tcc-docx
          path: documento/output/documento.docx
        continue-on-error: true
```

## Notes

- `xu-cheng/latex-action@v3` uses a full TeX Live container. `extra_system_packages` installs OS-level packages (`nodejs npm chromium`) needed by Mermaid CLI.
- The `pre_compile` step runs inside the container before LaTeX compilation, ensuring `mmdc` is available in `$PATH`.
- `paths: ["documento/**"]` ensures CI only runs when the document changes — not on every commit.
- Chromium is required by `@mermaid-js/mermaid-cli` for headless rendering.

## Generating a release PDF

To publish the compiled PDF as a GitHub Release asset on tag push:

```yaml
on:
  push:
    tags:
      - "v*"

# After the build job:
- name: Create release
  uses: softprops/action-gh-release@v2
  with:
    files: documento/documento.pdf
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
