# Formatação: Margens, Fonte, Espaçamento, Paginação

## Margens (NBR 14724:2011 — 5.1)

Papel **A4** (210 × 297 mm).

| Margem | Anverso (frente) | Verso |
|---|---|---|
| Superior | 3 cm | 3 cm |
| Inferior | 2 cm | 2 cm |
| Esquerda | 3 cm | 2 cm |
| Direita | 2 cm | 3 cm |

```latex
% abnTeX2 / memoir
\setlrmarginsandblock{3cm}{2cm}{*}
\setulmarginsandblock{3cm}{2cm}{*}
\checkandfixthelayout
```

Para impressão simples face (`oneside`), usar sempre as margens do anverso.

---

## Fonte (NBR 14724:2011 — 5.1)

Fontes permitidas: **Times New Roman, Arial, Calibri ou Verdana**. A mesma família deve ser usada em todo o documento.

| Elemento | Tamanho | Estilo |
|---|---|---|
| Corpo do texto | 12pt | Normal |
| Títulos de capítulos | ≥ 14pt (abnTeX2: `\Huge`) | Negrito, maiúsculo |
| Títulos de seções | ≥ 12pt (abnTeX2: `\Large`) | Negrito, maiúsculo |
| Títulos de subseções | 12pt | Negrito |
| Abaixo de subseção | 12pt | Normal |
| Citação direta longa | 10pt | Normal |
| Notas de rodapé | 10pt | Normal |
| Fonte de figuras/tabelas | 10pt | Normal |
| Paginação | 10pt | Normal |
| Natureza (folha de rosto) | 12pt | Normal (sem negrito) |

```latex
% Times New Roman
\usepackage{times}
% ou Times via mathptmx
\usepackage{mathptmx}
% ou se quiser Arial/Helvetica
\usepackage{helvet}\renewcommand{\familydefault}{\sfdefault}
```

---

## Espaçamento (NBR 14724:2011 — 5.2)

**Regra geral: 1,5 entrelinhas** para todo o texto corrido.

```latex
\OnehalfSpacing  % abnTeX2 — equivale a 1,5 entrelinhas
```

### Elementos com espaçamento simples (1,0):

| Elemento | Como implementar |
|---|---|
| Notas de rodapé | Automático no abnTeX2 |
| Natureza do trabalho (folha de rosto) | `\begin{SingleSpacing}` |
| Ficha catalográfica | `\begin{SingleSpacing}` |
| Citações diretas longas | Automático no ambiente `citacao` |
| Lista de referências | Automático no abnTeX2 |
| Títulos/fontes de figuras e tabelas | Automático com `\caption` |

---

## Indentação e Alinhamento

| Elemento | Recuo | Alinhamento |
|---|---|---|
| Texto corrido | 1,5 cm na primeira linha | Justificado |
| Resumo/Abstract | Sem recuo | Justificado |
| Citação longa | 4 cm da margem esquerda | Justificado |
| Referências | Sem recuo | À esquerda (não justificado) |
| Títulos numerados | Sem recuo | À esquerda |
| Títulos não numerados | Sem recuo | Centralizado |

```latex
\setlength{\parindent}{1.5cm}       % recuo de parágrafo
\setlength{\ABNTEXcitacaorecuo}{4cm} % recuo de citação longa
```

---

## Paginação (NBR 14724:2011)

- **Contagem** começa na folha de rosto (página 2)
- **Indicação numérica** começa a aparecer na **primeira página textual** (Introdução)
- Pré-textuais: contados mas **não numerados**
- Capa: **não contada** e não numerada
- **Posição**: superior direito no anverso; superior esquerdo no verso
- Fonte: arábicos, 10pt
- No abnTeX2 com `oneside`: sempre superior direito

```latex
% No abnTeX2, a paginação é configurada automaticamente.
% O comando \textual ativa a exibição dos números.
\pretextual   % desativa numeração visível
% ... pré-textuais ...
\textual      % ativa numeração a partir daqui
```
