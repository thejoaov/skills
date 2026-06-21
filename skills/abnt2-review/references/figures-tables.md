# Figuras, Tabelas e Quadros (NBR 14724:2011)

## Regra fundamental: título acima, fonte abaixo

Para **todos** os elementos (figura, tabela, quadro, gráfico, mapa, esquema):
- **Título**: imediatamente **acima** do elemento
- **Fonte**: imediatamente **abaixo** do elemento

Este é o erro mais comum. O título nunca vai abaixo da figura.

---

## Formato do título

```
Tipo Número – Descrição do conteúdo
```

- Separador: espaço + **travessão** (`–`, U+2013) + espaço
- **Não usar dois pontos** como separador
- Fonte: 12pt, centralizado na largura do elemento
- Negrito (opcional, mas consistente)

```latex
\caption{Distribuição dos estudantes por faixa de desempenho}
% Resultado: Figura 1 – Distribuição dos estudantes por faixa de desempenho
```

**Exemplos corretos:**
```
Figura 1 – Arquitetura do modelo preditivo proposto
Tabela 3 – Distribuição dos participantes por situação acadêmica
Quadro 2 – Comparativo entre algoritmos de classificação
Gráfico 1 – Evolução da acurácia por número de épocas
```

**Erros comuns no título:**
- `Figura 1: Descrição` → trocar `:` por `–`
- `Figura 1.1` (por capítulo) → usar numeração contínua: `Figura 1`, `Figura 2`...
- Título após a figura
- Título sem número

---

## Fonte (obrigatória mesmo para elaboração própria)

```latex
% No abnTeX2, usar \source{} ou nota após o caption
\fonte{Elaboração do próprio autor (2025).}
% ou
\fonte{Adaptado de \citeonline[p. 45]{silva2019}.}
% ou
\fonte{\citeonline[p. 45]{silva2019}.}
```

**Formatos de fonte:**

| Situação | Formato |
|---|---|
| Elaboração própria | `Fonte: Elaboração do próprio autor (ano).` |
| Adaptado de terceiro | `Fonte: Adaptado de SOBRENOME (ano, p. X).` |
| Reproduzido de terceiro | `Fonte: SOBRENOME (ano, p. X).` |
| Dados primários coletados | `Fonte: Dados da pesquisa (ano).` |

- Fonte: 10pt, alinhada à esquerda na margem do elemento
- Espaçamento: simples

**Erros comuns na fonte:**
- Fonte ausente
- Fonte acima do elemento
- `Fonte: Elaboração própria` sem o ano
- Fonte com 12pt

---

## Figuras (ilustrações)

Inclui: desenhos, mapas, esquemas, gráficos, fórmulas, fotografias, diagramas, fluxogramas, organogramas.

```latex
\begin{figure}[H]
  \centering
  \caption{Fluxograma do processo de treinamento do modelo}
  \label{fig:fluxo-treinamento}
  \includegraphics[width=0.8\textwidth]{imagens/fluxo-treinamento.png}
  \fonte{Elaboração do próprio autor (2025).}
\end{figure}
```

**Regras:**
- Inserir o mais próximo possível do trecho que a referencia no texto
- Deve ser **citada no texto** antes de aparecer: `... conforme a Figura 1 ...` ou `... (Figura 1)`
- Numeração **contínua** em todo o documento (não reinicia por capítulo)
- Lista de figuras: obrigatória a partir de **10 figuras**

```latex
% Numeração contínua (remover reinício por capítulo):
\counterwithout{figure}{chapter}
```

---

## Tabelas

Tabelas contêm **dados estatísticos/numéricos**. Seguem o padrão do IBGE:

**Diferença visual das figuras:** sem traços verticais nas bordas laterais.

```latex
\begin{table}[H]
  \centering
  \caption{Métricas de desempenho dos modelos avaliados}
  \label{tab:metricas}
  \begin{tabular}{lccc}
    \toprule
    Modelo & Acurácia & F1-Score & Precisão \\
    \midrule
    Árvore de Decisão & 0,87 & 0,86 & 0,85 \\
    Floresta Aleatória & 0,92 & 0,91 & 0,90 \\
    \bottomrule
  \end{tabular}
  \fonte{Dados da pesquisa (2025).}
\end{table}
```

**Estrutura:**
- `\toprule` e `\bottomrule`: traço horizontal duplo (topo e fundo)
- `\midrule`: traço simples separando cabeçalho do corpo
- **Sem** traços verticais nas colunas externas (bordas laterais abertas)
- Traços verticais internos: permitidos apenas para separar grupos no cabeçalho

**Dados especiais:**
- Dado inexistente: `—`
- Dado não disponível/desconhecido: `…`
- Valor < metade da unidade: `0`

---

## Quadros

Quadros contêm **dados qualitativos** (texto, categorias, comparações).

**Diferença das tabelas:** quadros têm **moldura completa** (traços em todos os lados).

```latex
\begin{quadro}[H]
  \centering
  \caption{Comparativo entre abordagens de aprendizado de máquina}
  \label{qdr:comparativo}
  \begin{tabular}{|l|l|l|}
    \hline
    Abordagem & Vantagem & Desvantagem \\
    \hline
    Árvore de Decisão & Interpretável & Suscetível a overfitting \\
    \hline
    Floresta Aleatória & Robusto & Menos interpretável \\
    \hline
  \end{tabular}
  \fonte{Elaboração do próprio autor (2025).}
\end{quadro}
```

Requer o pacote `trivfloat` e a definição do float `quadro` (já configurado no abnTeX2-IFPI).

---

## Numeração

| Tipo | Numeração | Lista obrigatória a partir de |
|---|---|---|
| Figuras | Sequencial contínua: Figura 1, 2, 3... | 10 figuras |
| Tabelas | Sequencial contínua: Tabela 1, 2, 3... | 10 tabelas |
| Quadros | Sequencial contínua: Quadro 1, 2, 3... | 10 quadros |

Cada tipo tem sua própria sequência independente.

---

## Equações e Fórmulas

```latex
\begin{equation}
  E = mc^2
  \label{eq:einstein}
\end{equation}
```

- Centralizadas na página
- Numeradas à direita, em arábicos entre parênteses: `(1)`, `(2)`...
- Referenciadas no texto: `... como mostra a Equação 1 ...` ou `... (Equação 1)`
