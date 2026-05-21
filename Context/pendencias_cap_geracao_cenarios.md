# Pendências para o Capítulo de Geração de Cenários / Experimentos

Itens identificados durante auditoria do Referencial Teórico e da Modelagem que
não cabem em nenhum dos dois capítulos já escritos. Devem ser incorporados quando
o capítulo de Geração de Cenários (ou Experimentos) for redigido.

---

## 1. Tabela de Coeficientes PAR(1) Estimados

**Origem:** atualmente em `mercado_energia.tex` subsec. `par` como Tabela~\ref{tab:phi1m}.  
**Por que mover:** são resultados empíricos calculados pelo autor (via `par1_tabela.py`),
não conteúdo de literatura. Mantê-la no Referencial implica, erroneamente, que a CCEE
publicou esses valores.

**O que incluir no capítulo futuro:**
- A tabela com os coeficientes $\phi_{1,m}$ por mês e submercado, com marcação de
  significância estatística ($p < 0{,}05$)
- Legenda corrigida: **"Elaboração própria com dados da CCEE (2001–2025)"**
- Análise dos coeficientes: variação sazonal da autocorrelação, quebra em janeiro,
  valores próximos de 1 nos meses de período seco

**Referência cruzada a inserir no Referencial** (para substituir a tabela removida):
> "Os coeficientes $\phi_{1,m}$ estimados sobre a série histórica da CCEE (2001–2025)
> são apresentados no Capítulo~\ref{cap:experimentos}, Tabela~X."

---

## 2. Procedimento Completo de Geração de Cenários (Monte Carlo com PAR(1))

**Origem:** implementado em `src/dataPipeline/generators/pld.py`, mas não descrito
em nenhum trecho do TCC.  
**Por que é crítico:** a banca certamente perguntará como os cenários foram gerados.
Sem essa seção, o trabalho não é reproduzível.

**Algoritmo a descrever (conforme implementado em `pld.py`):**

### Etapa 1 — Calibração (por submercado $s$ e mês $m$)
Dados os valores históricos $\{\text{PLD}_{r,m}\}$ (CCEE, 2001–2025):
$$
\mu_{s,m} = \frac{1}{N}\sum_r \log(\text{PLD}_{s,r,m}), \quad
\sigma_{s,m} = \text{std}\bigl(\log(\text{PLD}_{s,r,m})\bigr)
$$
$$
\phi_{1,s,m} = \text{Pearson}\bigl(\log(\text{PLD}_{s,r,m}),\; \log(\text{PLD}_{s,r,m-1})\bigr)
$$

### Etapa 2 — Simulação (para cada cenário $\omega = 1,\ldots,2000$ e estágio $t$)
Sorteia-se **um único** choque $\varepsilon_t \sim \mathcal{N}(0,1)$, compartilhado
entre os 4 submercados (hipótese de correlação espacial perfeita — ver item 3 abaixo).

Para cada submercado $s$:
$$
z_{s,t} = \phi_{1,s,m(t)}\, z_{s,t-1} + \sqrt{1 - \phi_{1,s,m(t)}^2}\;\varepsilon_t
$$
$$
\log\widehat{\text{PLD}}_{s,t} = \mu_{s,m(t)} + \sigma_{s,m(t)}\, z_{s,t}
$$

### Etapa 3 — Retransformação
$$
\widehat{\text{PLD}}_{s,t} = \exp\!\bigl(\log\widehat{\text{PLD}}_{s,t}\bigr)
$$

### Etapa 4 — Truncagem regulatória
$$
P_{s,t} = \operatorname{clip}\!\bigl(\widehat{\text{PLD}}_{s,t},\;
\underline{p}_{\text{ano}(t)},\; \overline{p}_{\text{ano}(t)}\bigr)
$$
onde $\underline{p}$ e $\overline{p}$ são os limites de piso e teto da ANEEL para o
ano correspondente (tabela `LIMITES_PLD` em `config.py`).

**Observação importante:** a escala logarítmica garante positividade das simulações,
mas **não** os limites regulatórios — estes são impostos exclusivamente pela truncagem
da Etapa 4. Esse ponto deve ser explicitado no texto para evitar ambiguidade.

**Parâmetros de implementação a documentar:**
- Número de cenários: $N_\omega = 2.000$
- Horizonte de simulação: $T = 60$ meses
- Semente aleatória: fixar para reprodutibilidade (verificar se já está fixada no código)

---

## 3. Hipótese de Correlação Espacial Perfeita entre Submercados

**Origem:** implementado em `pld.py` — um único $\varepsilon_t$ para todos os submercados.  
**Por que deve ser documentado:** é uma hipótese de modelagem com impacto direto na
estrutura de risco do portfólio. Correlação perfeita entre PLDs dos submercados elimina
qualquer diversificação geográfica; uma correlação parcial (ou nula) produziria cenários
mais dispersos e um risco de portfólio diferente.

**O que escrever no capítulo:**
- Declarar explicitamente: "Adota-se a hipótese de correlação espacial perfeita entre
  submercados: o choque $\varepsilon_t$ é sorteado uma única vez por estágio e
  compartilhado entre SE/CO, Sul, NE e Norte."
- Justificar ou reconhecer como limitação: "Essa hipótese simplifica a implementação
  e é conservadora do ponto de vista do risco, pois impede que posições em submercados
  distintos se compensem por movimentos descorrelacionados de preço."
- Possível extensão futura: usar um vetor de choques correlacionados com matriz de
  covariância empírica entre os log-PLDs dos submercados.

---

## Onde estes itens se encaixam na estrutura do TCC

```
Capítulo X — Geração de Cenários e Dados de Entrada
  X.1  Dados históricos de PLD (fonte: CCEE, período)
  X.2  Calibração do PAR(1)          ← item 2, Etapas 1
  X.3  Geração de cenários            ← item 2, Etapas 2–4
       X.3.1  Hipótese de correlação espacial  ← item 3
  X.4  Parâmetros calibrados          ← item 1 (Tabela phi_1m aqui)
  X.5  Validação das séries geradas   (a ser desenvolvido)
```
