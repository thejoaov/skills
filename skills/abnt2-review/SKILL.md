---
name: abnt2-review
description: Use when reviewing, correcting, or auditing a Brazilian academic paper (TCC, dissertação, artigo, monografia) for conformance with ABNT standards (NBR 14724, NBR 6023, NBR 10520, NBR 6028) implemented via abnTeX2, and for orthographic and stylistic correctness in Portuguese (pt-BR). Covers formatting rules (margins, font, spacing, pagination), document structure (capa, folha de rosto, sumário, elementos pré/pós-textuais), citations, references, figures, tables, quadros, appendices, and academic writing register. Trigger when the user asks to review, correct, proofread, or check a LaTeX TCC or academic paper against ABNT norms.
---

# ABNT2 Review — Revisão de Trabalhos Acadêmicos Brasileiros

Esta skill guia a revisão completa de um trabalho acadêmico brasileiro (TCC, artigo, dissertação) contra as normas ABNT implementadas pelo abnTeX2.

A revisão cobre duas dimensões:
1. **Conformidade ABNT** — estrutura, formatação e normas técnicas
2. **Revisão ortográfica e de estilo** — português acadêmico (pt-BR)

## References

```text
references/
  formatting.md        Margens, fonte, espaçamento, paginação, indentação
  structure.md         Ordem dos elementos, pré-textuais, pós-textuais, capa, folha de rosto
  citations.md         NBR 10520: citações diretas, indiretas, longas, apud, múltiplos autores
  references-list.md   NBR 6023: lista de referências, tipos de documento, regras de autoria
  figures-tables.md    Figuras, tabelas, quadros: títulos, fontes, numeração, posicionamento
  academic-writing.md  Registro acadêmico, ortografia, voz verbal, estrutura de parágrafos
```

## Processo de revisão

Execute os blocos abaixo em sequência. Para cada problema encontrado, indique:
- **Localização** (capítulo, seção, parágrafo ou linha do `.tex`)
- **O problema** (o que está errado)
- **A correção** (como deve ficar)
- **A norma** (qual NBR ou regra se aplica)

---

## Bloco 1 — Estrutura do documento

Verifique se todos os elementos obrigatórios estão presentes e na ordem correta.

**Ordem obrigatória (NBR 14724:2011):**

```
PARTE EXTERNA
  Capa ✓ obrigatório

PRÉ-TEXTUAIS (não numerados)
  Folha de rosto ✓ obrigatório
  Folha de aprovação ✓ obrigatório
  Dedicatória ○ opcional
  Agradecimentos ○ opcional
  Epígrafe ○ opcional
  Resumo (pt-BR) ✓ obrigatório
  Abstract (en) ✓ obrigatório
  Lista de figuras ✓ obrigatório se ≥10 figuras
  Lista de tabelas ✓ obrigatório se ≥10 tabelas
  Lista de abreviaturas ○ opcional
  Sumário ✓ obrigatório

TEXTUAIS (numerados)
  Introdução
  Desenvolvimento
  Conclusão

PÓS-TEXTUAIS (sem numeração de seção, mas constam no sumário)
  Referências ✓ obrigatório
  Glossário ○ opcional
  Apêndice(s) ○ opcional — material do próprio autor
  Anexo(s) ○ opcional — material de terceiros
```

Perguntas de verificação:
- Elementos pré-textuais aparecem no sumário? **Não devem aparecer.**
- Referências, apêndices e anexos aparecem no sumário? **Devem aparecer, sem numeração de seção.**
- Existe algo na ordem errada?

---

## Bloco 2 — Formatação (NBR 14724:2011)

Verifique no preâmbulo (`.cls`, `preambulo.tex`, `configuracoes/`) e no resultado compilado:

| Elemento | Regra | Verificação |
|---|---|---|
| Papel | A4 (210×297 mm) | `a4paper` na classe |
| Margem superior | 3 cm | `\setulmarginsandblock{3cm}{...}` |
| Margem inferior | 2 cm | `\setulmarginsandblock{...}{2cm}` |
| Margem esquerda | 3 cm (anverso) | `\setlrmarginsandblock{3cm}{...}` |
| Margem direita | 2 cm (anverso) | `\setlrmarginsandblock{...}{2cm}` |
| Fonte do corpo | 12pt, Times ou Arial | `12pt` + `\usepackage{times}` / `lmodern` |
| Espaçamento corpo | 1,5 entrelinhas | `\OnehalfSpacing` |
| Espaçamento citação longa | Simples | Ambiente `citacao` do abnTeX2 |
| Espaçamento notas de rodapé | Simples | automático no abnTeX2 |
| Espaçamento referências | Simples | automático no abnTeX2 |
| Recuo de parágrafo | 1,5 cm | `\setlength{\parindent}{1.5cm}` |
| Alinhamento | Justificado | `\justifying` ou padrão abnTeX2 |
| Paginação início | 1ª página textual (Introdução) | `\textual` antes do capítulo |
| Paginação posição | Superior direito (anverso) | padrão abnTeX2 |
| Fonte da paginação | 10pt | padrão abnTeX2 |

---

## Bloco 3 — Capa e Folha de Rosto

### Capa (NBR 14724:2011 — 4.1)

Elementos obrigatórios, todos centralizados e na ordem:
1. Nome da instituição — maiúsculo, negrito
2. Nome do tipo de trabalho — maiúsculo, negrito
3. Nome do(s) autor(es) — maiúsculo
4. Título — maiúsculo, negrito
5. Subtítulo (se houver) — minúsculo
6. Local (cidade) — maiúsculo
7. Ano — arábico

### Folha de rosto (NBR 14724:2011 — 4.2.1.1)

- Nome do autor — centralizado
- Título — centralizado, negrito, maiúsculo
- **Natureza do trabalho**: alinhada da metade da página à margem direita; espaçamento simples; sem negrito; inclui tipo do trabalho, objetivo, instituição e área
- Nome do orientador (e coorientador) — logo abaixo da natureza
- Local e ano — centralizados, na parte inferior

Erros comuns:
- Natureza do trabalho centralizada (deve estar à direita)
- Orientador listado antes da natureza
- Negrito na natureza do trabalho (não deve ter)

---

## Bloco 4 — Resumo e Abstract (NBR 6028:2003)

Verifique:
- [ ] Escrito na **terceira pessoa do singular, voz ativa**
- [ ] **150 a 500 palavras** (contar e informar o número real)
- [ ] Sem recuo de parágrafo
- [ ] Sem citações
- [ ] Sem siglas (usar o nome por extenso)
- [ ] Contém: objetivos, metodologia, resultados e conclusões
- [ ] **Palavras-chave:** 3 a 6 termos; iniciais minúsculas (exceto nomes próprios); separadas por ponto e vírgula; finalizadas por ponto
- [ ] Espaçamento 1,5 entrelinhas

Erros comuns:
- Uso de primeira pessoa ("Este trabalho analisou..." é correto; "Analisamos..." é errado)
- Citações no resumo
- Palavras-chave com iniciais maiúsculas indevidas
- Resumo com menos de 150 ou mais de 500 palavras
- Faltando um dos quatro elementos obrigatórios (objetivo, metodologia, resultados, conclusões)

---

## Bloco 5 — Citações (NBR 10520:2002)

### Citação direta curta (≤ 3 linhas)
- Entre aspas duplas, no corpo do texto
- Referência: `(SOBRENOME, ano, p. X)` ou `Sobrenome (ano, p. X)`
- Número de página é **obrigatório**

Erros comuns:
- `(SILVA, 2015)` sem número de página em citação direta → `(SILVA, 2015, p. 112)`
- Aspas simples em vez de aspas duplas
- Citação direta sem aspas

### Citação direta longa (> 3 linhas)

No LaTeX (abnTeX2):
```latex
\begin{citacao}
Texto da citação aqui. \cite[p. 45]{silva2015}
\end{citacao}
```

Deve ter:
- Recuo de **4 cm** da margem esquerda
- Fonte tamanho **10**
- Espaçamento **simples**
- **Sem aspas**

Erros comuns:
- Citação longa com aspas
- Citação longa misturada no parágrafo sem `\begin{citacao}`
- Falta de número de página

### Citação indireta (paráfrase)
- Sem aspas, página não obrigatória mas recomendada
- Correto: `Segundo Silva (2015), ...` ou `... (SILVA, 2015).`

### Apud
```
(AUTOR ORIGINAL, ano apud AUTOR CONSULTADO, ano, p. X)
```
- Nas referências finais: listar **apenas o autor consultado**
- Mencionar o original em nota de rodapé

### Múltiplos autores simultâneos
- Ordem **alfabética**, separados por ponto e vírgula
- `(ALMEIDA, 2010; CARVALHO, 2008; PEREIRA, 2012)`

### Supressões e acréscimos
- Supressão: `[...]`
- Acréscimo: `[palavra]`
- Ênfase do citante: `(grifo nosso)` após a citação
- Ênfase do original: `(grifo do autor)` após a citação

---

## Bloco 6 — Lista de Referências (NBR 6023:2002)

### Formatação
- [ ] Alinhada à **esquerda** (não justificada)
- [ ] Espaçamento **simples** entre linhas
- [ ] **Uma linha em branco** entre cada referência
- [ ] Ordem **alfabética** pelo sobrenome do primeiro autor
- [ ] Sem numeração

### Estruturas por tipo — conferir cada referência:

**Livro:**
```
SOBRENOME, Prenome. Título: subtítulo. ed. Local: Editora, ano.
```

**Artigo em periódico:**
```
SOBRENOME, Prenome. Título. Nome da Revista, local, v. X, n. X, p. XX-YY, mês ano.
```

**Trabalho acadêmico:**
```
SOBRENOME, Prenome. Título. ano. X f. Tipo (Grau) – Instituição, Local, ano.
```

**Documento eletrônico:**
```
[referência completa]. Disponível em: <URL>. Acesso em: DD mês. AAAA.
```

**Regras de autoria:**
- 1-3 autores: listar todos
- 4+: primeiro autor + `et al.` (em itálico: `\textit{et al.}`)
- Sem local: `[S. l.]`
- Sem editora: `[s. n.]`

Erros comuns:
- Referências em ordem não alfabética
- URL sem data de acesso
- `et al.` sem itálico
- Títulos de livros sem itálico (devem ser em itálico ou negrito — escolher e manter)
- Falta de informação de volume/número em artigos
- `In:` não capitalizado em capítulos de livros

---

## Bloco 7 — Figuras, Tabelas e Quadros

### Figuras e Quadros — título **acima**
```latex
\begin{figure}[h]
  \caption{Título descritivo da figura}  % ACIMA
  \label{fig:exemplo}
  \includegraphics[width=\textwidth]{imagens/exemplo.png}
  \source{Elaboração do próprio autor (2025).}  % ABAIXO
\end{figure}
```

### Tabelas — título **acima** (mesmo padrão)

Erros comuns:
- Título abaixo da figura/tabela (deve ser **acima**)
- Fonte ausente — mesmo que seja elaboração própria, escrever `Fonte: Elaboração do próprio autor (2025).`
- Fonte acima da figura (deve ser **abaixo**)
- Figura/tabela não citada no texto antes de aparecer
- Numeração por capítulo em vez de sequencial contínua (`Figura 1`, `Figura 2`, não `Figura 1.1`)

### Tabelas vs. Quadros
- **Tabela**: dados numéricos/estatísticos → **sem** traços verticais nas bordas laterais
- **Quadro**: dados qualitativos/textuais → **com** moldura completa (traços em todos os lados)

### Formato do título
```
Figura 1 – Descrição da figura
Tabela 3 – Distribuição dos participantes por faixa etária
Quadro 2 – Comparativo entre metodologias
```
Separador: espaço + travessão (`–`) + espaço. **Não usar dois pontos.**

---

## Bloco 8 — Numeração de Seções (NBR 6024:2012)

- Até **5 níveis**: `1`, `1.1`, `1.1.1`, `1.1.1.1`, `1.1.1.1.1`
- Títulos **com** indicativo numérico → alinhados à **esquerda**
- Títulos **sem** indicativo (dedicatória, agradecimentos, resumo) → **centralizados**
- Separar título do texto por uma linha em branco
- Sem ponto final nos títulos
- Não usar apenas um subnível (se existe `1.1`, deve existir `1.2`)

---

## Bloco 9 — Apêndices e Anexos

| | Apêndice | Anexo |
|---|---|---|
| Autoria | Do próprio autor | De terceiros |
| Identificação | `APÊNDICE A – Título` | `ANEXO A – Título` |
| Letras | A, B, C... AA, BB... | A, B, C... AA, BB... |

- Devem ser **citados no texto** antes de aparecer
- Aparecem no sumário sem numeração de seção
- `APÊNDICE` e `ANEXO` em maiúsculo

---

## Bloco 10 — Revisão Ortográfica e de Estilo (pt-BR Acadêmico)

### Ortografia e gramática
- [ ] Acordo Ortográfico de 2009 em vigor (verificar palavras como `ideia`, `assembleia`, `voo`, `zoo`)
- [ ] Acentuação correta: verbos do tipo `ele crê`, `ele vê`; hiato `saúde`
- [ ] Crase: obrigatória antes de `a` + feminino determinado; proibida antes de masculino, pronomes, plural indeterminado
- [ ] Concordância nominal e verbal
- [ ] Regência verbal: verbos de uso frequente em textos técnicos (`assistir a`, `obedecer a`, `visar a`)
- [ ] Hífen pós-reforma: `pré-requisito`, `super-homem`, `autoavaliação` (sem hífen), `autorretrato` (sem hífen)

### Registro acadêmico
- [ ] **Impessoalidade**: evitar `eu`, `nós`, `a gente`; usar terceira pessoa ou voz passiva
  - Errado: `Nós analisamos os dados`
  - Correto: `Os dados foram analisados` / `Analisou-se os dados` / `Este trabalho analisa`
- [ ] **Objetividade**: frases diretas, sem rodeios ou coloquialismos
- [ ] **Precisão**: usar terminologia técnica correta; definir termos na primeira ocorrência
- [ ] **Coesão**: verificar conectivos entre parágrafos; evitar repetição de sujeito desnecessária
- [ ] **Sem gírias ou expressões informais**: `agora`, `então`, `aí`, `tipo`, `legal` em sentido informal

### Siglas
- Na **primeira ocorrência**: nome por extenso seguido da sigla entre parênteses
  - Correto: `Instituto Federal do Piauí (IFPI)`
  - Erro: usar a sigla antes de defini-la
- Após a definição: usar apenas a sigla
- Verificar consistência: se definiu em uma seção, não redefinir em outra

### Pontuação
- [ ] Vírgula antes de orações subordinadas adverbiais antepostas
- [ ] Dois-pontos: seguido de espaço, sem maiúscula após (exceto nomes próprios)
- [ ] Ponto final após cada referência bibliográfica
- [ ] Ponto final nos itens de listas numeradas
- [ ] Ponto e vírgula entre palavras-chave (exceto após a última, que leva ponto final)

### Parágrafos
- [ ] Cada parágrafo desenvolve **uma ideia central**
- [ ] Primeiro parágrafo de cada seção contextualiza o que vem a seguir
- [ ] Parágrafos muito curtos (1-2 linhas) indicam ideia incompleta
- [ ] Parágrafos muito longos (>15 linhas) comprometem a leitura — verificar se podem ser divididos

---

## Relatório de revisão

Ao concluir, apresentar o resultado neste formato:

```markdown
## Relatório de Revisão ABNT

### Problemas Críticos (impedem aprovação)
- [ ] [localização] — [problema] → [correção] (NBR XXXXX)

### Problemas Relevantes (devem ser corrigidos)
- [ ] [localização] — [problema] → [correção]

### Sugestões de Melhoria (melhoram a qualidade)
- [ ] [localização] — [sugestão]

### Estatísticas
- Total de problemas encontrados: X
- Críticos: X | Relevantes: X | Sugestões: X
- Contagem de palavras do resumo: X (mínimo 150, máximo 500)
```
