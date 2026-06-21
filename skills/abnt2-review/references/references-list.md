# Lista de Referências (NBR 6023:2002)

## Formatação geral

```latex
% No abnTeX2, as referências são geradas automaticamente pelo BibTeX
% com o estilo abntex2-alf (autor-data) ou abntex2-num (numérico)
\bibliographystyle{abntex2-alf}
\bibliography{bibliografia}
```

Características:
- Alinhamento: **à esquerda** (não justificado)
- Espaçamento entre linhas: **simples**
- **Uma linha em branco** entre cada referência
- Ordem: **alfabética** (sistema autor-data)
- Sem numeração
- Fonte: 12pt

---

## Estruturas por tipo de documento

### Livro

```
SOBRENOME, Prenome. Título: subtítulo. Número da edição. ed. Local de publicação: Editora, ano.
```

```bibtex
@book{pressman2021,
  author    = {Pressman, Roger S. and Maxim, Bruce R.},
  title     = {Engenharia de Software: Uma Abordagem Profissional},
  edition   = {9},
  publisher = {AMGH},
  address   = {Porto Alegre},
  year      = {2021}
}
```

Resultado:
```
PRESSMAN, Roger S.; MAXIM, Bruce R. Engenharia de software: uma abordagem profissional.
9. ed. Porto Alegre: AMGH, 2021.
```

---

### Capítulo de livro organizado

```
SOBRENOME, Prenome. Título do capítulo. In: SOBRENOME, Prenome (org.). Título do livro.
ed. Local: Editora, ano. p. XX-YY.
```

```bibtex
@incollection{silva2020cap,
  author    = {Silva, João},
  title     = {Título do capítulo},
  booktitle = {Título do livro organizado},
  editor    = {Pereira, Maria},
  publisher = {Editora X},
  address   = {São Paulo},
  year      = {2020},
  pages     = {45--67}
}
```

---

### Artigo em periódico

```
SOBRENOME, Prenome. Título do artigo. Nome do Periódico, Local, v. X, n. X, p. XX-YY, mês ano.
```

```bibtex
@article{silva2019,
  author  = {Silva, João Victor},
  title   = {Título do artigo},
  journal = {Revista Brasileira de Informática na Educação},
  address = {Porto Alegre},
  volume  = {27},
  number  = {3},
  pages   = {100--120},
  month   = {set.},
  year    = {2019},
  doi     = {10.5753/rbie.2019.27.03.100}
}
```

---

### Trabalho acadêmico (TCC, dissertação, tese)

```
SOBRENOME, Prenome. Título. Ano de apresentação. X f. Tipo (Grau) – Instituição, Local, ano de defesa.
```

```bibtex
@mastersthesis{pereira2022,
  author  = {Pereira, Ana Lúcia},
  title   = {Predição de Evasão Escolar com Aprendizado de Máquina},
  school  = {Universidade Federal do Piauí},
  address = {Teresina},
  year    = {2022},
  type    = {Dissertação (Mestrado em Ciência da Computação)}
}
```

---

### Documento eletrônico (artigo online, site)

```
[referência completa conforme o tipo]. Disponível em: <URL>. Acesso em: DD mês. AAAA.
```

```bibtex
@misc{ibge2023,
  author       = {{IBGE}},
  title        = {Pesquisa Nacional por Amostra de Domicílios Contínua},
  year         = {2023},
  howpublished = {Disponível em: \url{https://www.ibge.gov.br/pnad}. Acesso em: 10 jan. 2024}
}
```

---

### Legislação

```
JURISDIÇÃO. Título, numeração, data. Dados da publicação oficial.
```

```
BRASIL. Lei nº 9.394, de 20 de dezembro de 1996. Estabelece as diretrizes e bases
da educação nacional. Diário Oficial da União, Brasília, DF, 23 dez. 1996.
```

---

## Regras de autoria

| Situação | Regra |
|---|---|
| 1 autor | `SOBRENOME, Prenome.` |
| 2-3 autores | Listar todos, separados por ponto e vírgula |
| 4 ou mais | Primeiro autor + `et al.` (em itálico) |
| Sem autor | Entrada pelo título (primeira palavra em CAIXA ALTA) |
| Pessoa jurídica/instituição | Nome por extenso ou sigla reconhecida em CAIXA ALTA |
| Organizador | Sobrenome, Prenome (org.). |
| Editor | Sobrenome, Prenome (ed.). |

**Formato do sobrenome:** MAIÚSCULO. Prenomes podem ser abreviados com inicial.
- `SILVA, João` ou `SILVA, J.`
- Em caso de sobrenomes compostos: `CASTELO BRANCO, Humberto`

---

## Erros comuns nas referências

| Erro | Correção |
|---|---|
| Referências em ordem não-alfabética | Reordenar; o BibTeX com `abntex2-alf` faz isso automaticamente |
| `et al.` sem itálico | `\textit{et al.}` no `.bib` ou usar campo BibTeX correto |
| URL sem `Acesso em:` | Sempre incluir a data de acesso para documentos eletrônicos |
| Sem local de publicação | Usar `[S. l.]` se desconhecido |
| Sem editora | Usar `[s. n.]` se desconhecida |
| Ano com `?` sem colchetes | Usar `[1986?]` ou `[entre 1980 e 1985]` para anos incertos |
| Título do livro sem diferenciação tipográfica | Títulos de livros em itálico |
| Título do artigo com itálico | Título de artigo sem itálico (itálico é só para o nome do periódico) |
| Falta de número de páginas em artigos | Sempre incluir `p. XX-YY` |
| `In` sem maiúscula | Deve ser `In:` (com dois pontos, maiúsculo) |
| Nome do mês por extenso | Usar abreviatura padrão: `jan., fev., mar., abr., maio, jun., jul., ago., set., out., nov., dez.` |
