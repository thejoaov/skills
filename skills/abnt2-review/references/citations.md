# Citações (NBR 10520:2002)

## Regras gerais

- Sistema padrão do abnTeX2: **autor-data** (com `abntex2cite`)
- Número de página é **obrigatório** em citações diretas; recomendado em indiretas
- As citações no texto devem corresponder exatamente à entrada na lista de referências

---

## Citação direta curta (≤ 3 linhas)

Inserida no próprio parágrafo, entre **aspas duplas**. Página obrigatória.

```latex
"A aprendizagem de máquina permite identificar padrões em grandes volumes de dados." (SILVA, 2015, p. 112).

Segundo Almeida (2008, p. 203), "a análise preditiva representa [...]".
```

**Erros comuns:**
- Aspas simples (`'...'`) em vez de duplas (`"..."`)
- Falta do número de página
- Ponto final antes do parêntese da citação — o ponto vai **depois**: `"[...]." (SILVA, 2015, p. 10).` ✗ → `"[...]" (SILVA, 2015, p. 10).` ✓

---

## Citação direta longa (> 3 linhas)

Parágrafo separado, sem aspas, com:
- Recuo de **4 cm** da margem esquerda
- Fonte **10pt**
- Espaçamento **simples**

```latex
\begin{citacao}
A predição de desempenho acadêmico constitui um campo emergente na
intersecção entre as ciências da educação e da computação, potencializado
pela crescente disponibilidade de dados educacionais em plataformas digitais
institucionais. \cite[p. 45]{silva2015}
\end{citacao}
```

**Erros comuns:**
- Citação longa entre aspas (não deve ter)
- Citação longa embutida no parágrafo sem ambiente `citacao`
- Falta de número de página

---

## Citação indireta (paráfrase)

Sem aspas. Página não obrigatória, mas recomendada.

```latex
Segundo Chiavenato (2014), a administração de pessoas envolve [...]

[...] conforme observado em estudos anteriores \cite[p. 78]{pereira2018}.
```

---

## Citação de citação (apud)

Usar quando não se teve acesso à obra original.

```latex
(FOUCAULT, 1975 \apud SILVA, 2010, p. 45)
```

**Regras:**
- Nas referências finais: listar **apenas** a obra efetivamente consultada (Silva, no exemplo)
- Mencionar a obra original em nota de rodapé (recomendado)
- Evitar `apud` sempre que possível — buscar a fonte primária

---

## Múltiplos autores simultâneos

Ordem **alfabética**, separados por **ponto e vírgula**:

```latex
\cite{almeida2010,carvalho2008,pereira2012}
% Resultado: (ALMEIDA, 2010; CARVALHO, 2008; PEREIRA, 2012)
```

---

## Supressões e acréscimos em citações

| Recurso | Símbolo | Uso |
|---|---|---|
| Supressão de trecho | `[...]` | Quando se omite parte do texto citado |
| Acréscimo do citante | `[palavra]` | Quando se insere algo para clareza |
| Ênfase do citante | `(grifo nosso)` | Ao final da citação |
| Ênfase do original | `(grifo do autor)` | Ao final da citação |

---

## Formas de chamada no texto

```latex
% Autor entre parênteses (mais comum em citação indireta)
\cite{silva2015}          → (SILVA, 2015)
\cite[p. 10]{silva2015}   → (SILVA, 2015, p. 10)

% Autor no texto (integrado à frase)
\citeonline{silva2015}          → Silva (2015)
\citeonline[p. 10]{silva2015}   → Silva (2015, p. 10)

% Apud
\apud{original1975}{consultado2010}  → (ORIGINAL, 1975 apud CONSULTADO, 2010)
```

---

## Citação de autor com mesmo sobrenome

Adicionar a inicial do prenome para distinguir:
```
(SILVA, A., 2010; SILVA, J., 2015)
```

## Citação de mesma obra no mesmo parágrafo

Na segunda e posteriores menções ao mesmo autor no mesmo parágrafo, pode-se abreviar:
- No corpo do texto: repetir a chamada completa
- Não usar "idem", "ibidem" ou "op. cit." — o abnTeX2 não usa essas expressões
