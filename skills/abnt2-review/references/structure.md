# Estrutura do Documento (NBR 14724:2011)

## Ordem obrigatória completa

```
PARTE EXTERNA
├── Capa                           obrigatório
└── Lombada                        opcional

ELEMENTOS PRÉ-TEXTUAIS             (não numerados, não constam no sumário)
├── Folha de rosto                 obrigatório
├── Errata                         opcional (folha avulsa, não paginada)
├── Folha de aprovação             obrigatório
├── Dedicatória                    opcional
├── Agradecimentos                 opcional
├── Epígrafe                       opcional
├── Resumo (pt-BR)                 obrigatório
├── Abstract (inglês)              obrigatório
├── Lista de ilustrações           obrigatório se ≥10 figuras
├── Lista de tabelas               obrigatório se ≥10 tabelas
├── Lista de abreviaturas/siglas   opcional
├── Lista de símbolos              opcional
└── Sumário                        obrigatório

ELEMENTOS TEXTUAIS                 (numerados, a partir da Introdução)
├── Introdução
├── Desenvolvimento (capítulos)
└── Conclusão

ELEMENTOS PÓS-TEXTUAIS            (sem numeração de seção; constam no sumário)
├── Referências                    obrigatório
├── Glossário                      opcional
├── Apêndice(s)                    opcional
├── Anexo(s)                       opcional
└── Índice                         opcional
```

**Regras de paginação:**
- Pré-textuais: contados a partir da folha de rosto, mas **não exibem número**
- Capa: **não contada**
- Numeração visível: começa na **primeira página textual** (Introdução)

---

## Capa (NBR 14724:2011 — 4.1)

Elementos em ordem, todos **centralizados**:

```
[Nome da Instituição]          ← maiúsculo, negrito
[Nome do Curso]               ← maiúsculo, negrito (quando exigido)
[Nome do(s) Autor(es)]        ← maiúsculo

[TÍTULO DO TRABALHO]          ← maiúsculo, negrito
[Subtítulo]                   ← minúsculo (se houver)

[Local]                       ← maiúsculo
[Ano]                         ← arábico
```

```latex
\imprimircapa  % comando abnTeX2 — gera capa automaticamente
               % com os dados de \titulo, \autor, \local, \data, \instituicao
```

---

## Folha de Rosto (NBR 14724:2011 — 4.2.1.1)

```
[Nome do Autor]                ← maiúsculo, centralizado

[TÍTULO]                       ← maiúsculo, negrito, centralizado

                    [Natureza do trabalho]   ← à direita (da metade da página)
                    [Objetivo]               ← espaçamento simples
                    [Instituição]            ← sem negrito
                    [Área de concentração]
                    
                    Orientador: [Nome]
                    Coorientador: [Nome] (se houver)

[Local]                        ← centralizado
[Ano]                          ← centralizado
```

```latex
\imprimirfolhaderosto  % comando abnTeX2
```

**Erros frequentes na folha de rosto:**
- Natureza do trabalho centralizada (deve estar à direita)
- Negrito na natureza do trabalho
- Falta de informação sobre objetivo (ex.: `apresentado como requisito...`) ou área
- Orientador antes da natureza do trabalho

---

## Resumo (NBR 6028:2003)

```latex
\begin{resumo}
Texto do resumo em terceira pessoa do singular, voz ativa, sem citações,
sem recuo de parágrafo, espaçamento 1,5. Deve conter objetivos, metodologia,
resultados e conclusões. Extensão: 150 a 500 palavras.

\vspace{\onelineskip}
\noindent
\textbf{Palavras-chave}: palavra um; palavra dois; palavra três.
\end{resumo}
```

**Palavras-chave:**
- 3 a 6 termos
- Iniciais **minúsculas** (exceto nomes próprios e científicos)
- Separadas por **ponto e vírgula**
- Finalizadas por **ponto**

---

## Sumário (NBR 6027:2012)

```latex
\tableofcontents  % abnTeX2 gera automaticamente
```

**Regras:**
- Reproduz fielmente a estrutura do trabalho (mesma ordem, grafia, numeração)
- Pré-textuais: **não aparecem** no sumário
- Pós-textuais (Referências, Apêndices, Anexos): aparecem **sem numeração de seção**
- Capítulos em caixa alta (opção `sumario=abnt-6027-2012`)
- Líderes de pontos conectando título à página: obrigatórios

**Configuração:**
```latex
\renewcommand{\ABNTEXchapterfont}{\normalfont\bfseries}
```

---

## Numeração de Seções (NBR 6024:2012)

**Até 5 níveis:**
```
1 CAPÍTULO
1.1 Seção secundária
1.1.1 Seção terciária
1.1.1.1 Seção quaternária
1.1.1.1.1 Seção quinária
```

**Regras:**
- Indicativo numérico separado do título por **um espaço**; sem ponto final
- Títulos **com** indicativo → alinhados à **esquerda**
- Títulos **sem** indicativo (resumo, dedicatória) → **centralizados**
- Nunca ter um único subnível: se existe `1.1`, deve existir `1.2`
- Títulos em caixa alta para capítulos (configurável no `.cls`)

---

## Apêndices e Anexos

```latex
% Apêndice (material do próprio autor)
\begin{apendicesenv}
\partapendices
\chapter{Questionário Aplicado}
Conteúdo...
\end{apendicesenv}

% Anexo (material de terceiros)
\begin{anexosenv}
\partanexos
\chapter{Regulamento Institucional}
Conteúdo...
\end{anexosenv}
```

**Identificação:**
```
APÊNDICE A – Título descritivo
APÊNDICE B – Outro título
...
APÊNDICE AA – Quando esgotar o alfabeto

ANEXO A – Título descritivo
```

**Citação obrigatória no texto:**
- `... conforme o questionário aplicado (Apêndice A) ...`
- `... (ver Anexo A) ...`

---

## Notas de rodapé

```latex
Texto do parágrafo.\footnote{Conteúdo da nota de rodapé.}
```

- Fonte: 10pt
- Espaçamento: simples
- Segunda linha em diante: alinhada abaixo da primeira letra do texto (recuo francês)
- Uso: comentários e informações complementares que não integram o corpo do texto
- **Não usar** para referências bibliográficas no sistema autor-data
