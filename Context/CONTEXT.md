# CONTEXT.md — Briefing para Claude Code

Este arquivo é o ponto de entrada para qualquer sessão de trabalho no TCC.
Leia este arquivo inteiro antes de qualquer ação.

---

## O Projeto

TCC de Lucas Garcia, curso de Engenharia de Computação e Informação, Escola
Politécnica / UFRJ (DEL — Departamento de Eletrônica e de Computação).

**Título:** Gestão de Contratos de Energia sob Incerteza: Comparação entre
Equivalente Determinístico (DEQ) e Programação Dinâmica Dual Estocástica (PDDE)

O trabalho é baseado em um artigo aceito na LVIII SBPO 2026 (já publicado) e em
um repositório de código público: https://github.com/GARCl-A/TCC-Lucas-Garcia

---

## O que o trabalho faz (resumo técnico)

Uma comercializadora de energia elétrica precisa decidir quais contratos bilaterais
comprar e vender ao longo de um horizonte mensal, antes de conhecer o PLD (Preço
de Liquidação das Diferenças) e a geração física. O problema é modelado como
programação estocástica multiestágio.

Duas abordagens são comparadas:
- **DEQ**: formulação determinística equivalente — monta toda a árvore de cenários
  em um único PL. Implementado em Julia com JuMP + HiGHS.
- **PDDE (SDDP)**: programação dinâmica dual estocástica — decompõe o problema
  por estágio, aproxima funções de valor por cortes de Benders. Implementado em
  Julia com SDDP.jl + HiGHS.

Resultado principal: DEQ escala exponencialmente (O(R^T) nós) e estoura memória
para instâncias maiores. PDDE resolve as mesmas instâncias em segundos.

### Pipeline de dados (Python)
O repositório de código tem um pipeline em `src/dataPipeline/` com 4 etapas:
1. `pld.py` — gera cenários de PLD via modelo PAR(1) log-normal calibrado em
   dados históricos da CCEE. Respeita limites regulatórios de PLD mín/máx por ano.
2. `generation_stochastic.py` — geração física por usina, condicionada ao PLD:
   hidráulicas com fator inversamente proporcional ao PLD, eólicas com raiz do fator.
3. `contracts.py` — carteira legada como percentual fixo da geração média esperada.
4. `trades.py` — oportunidades de negociação: curva forward sintética (média dos
   cenários de PLD) com spread bid/ask por duração do contrato.

### Modelos Julia (src/model/v3_models/)
- `deq.jl` — structs: DEQConfig, MarketData, ArvoreCenarios, DadosMercado.
  Limite hard-coded de 250.000 nós. Indexa PLD e geração por (nó, submercado).
- `sddp.jl` — PolicyGraph com nós (m,0) de decisão e (m,c) de liquidação.
  Trades passados como SDDP.State para memória entre decisão e liquidação.

---

## Template LaTeX

Usa o template oficial da Poli/UFRJ (DEL). Arquivos-chave na raiz:
- `main.tex` — entry point, usa `\book`, `\frontmatter`/`\mainmatter`
- `TesePack.tex` — pacotes (baseado no template da Poli, adaptado para UTF-8)
- `Capa.tex` — capa e folha de rosto
- `Preambulo.tex` — declaração, copyright, dedicatória, agradecimentos,
  resumo, abstract, siglas (tudo em um arquivo, como no template original)

### Estrutura de capítulos (modular)
Cada capítulo tem um arquivo orquestrador que faz `\input` das subseções:

```
main.tex
├── Capa.tex
├── Preambulo.tex
├── capitulos/
│   ├── introducao/
│   │   ├── introducao.tex        ← orquestrador
│   │   ├── contexto.tex
│   │   ├── problema.tex
│   │   ├── objetivos.tex
│   │   └── organizacao.tex
│   ├── revisao/
│   │   ├── revisao.tex
│   │   ├── mercado_energia.tex
│   │   ├── prog_estocastica.tex
│   │   ├── pdde_literatura.tex
│   │   ├── portfolio_literatura.tex
│   │   └── ferramentas.tex
│   ├── modelagem/
│   │   ├── modelagem.tex
│   │   ├── descricao_problema.tex   ← ESCRITO
│   │   ├── notacao.tex              ← ESCRITO
│   │   ├── deq.tex                  ← ESCRITO
│   │   └── pdde.tex                 ← ESCRITO
│   ├── implementacao/
│   │   ├── implementacao.tex
│   │   ├── arquitetura.tex
│   │   ├── pipeline_dados.tex
│   │   ├── geracao_pld.tex
│   │   ├── geracao_producao.tex
│   │   ├── carteira_trades.tex
│   │   ├── impl_deq.tex
│   │   └── impl_sddp.tex
│   ├── experimentos/
│   │   ├── experimentos.tex
│   │   ├── configuracao.tex
│   │   ├── escalabilidade.tex
│   │   ├── politica.tex
│   │   └── discussao.tex
│   └── conclusao/
│       └── conclusao.tex
├── referencias.bib
└── CONTEXT.md                       ← este arquivo
```

### Índice planejado completo

**Cap. 1 — Introdução**
1.1 Contexto e Motivação
1.2 O Problema de Gestão de Portfólio de Energia
1.3 Objetivos do Trabalho
1.4 Organização do Documento

**Cap. 2 — Revisão Bibliográfica**
2.1 O Mercado de Energia Elétrica Brasileiro
2.2 Programação Estocástica Multiestágio
2.3 Programação Dinâmica Dual Estocástica (PDDE)
2.4 Gestão de Portfólio de Contratos de Energia
2.5 Ferramentas Computacionais: JuMP e SDDP.jl

**Cap. 3 — Modelagem Matemática** ← CAPÍTULO COMPLETO
3.1 Descrição do Problema
3.2 Notação: Conjuntos, Parâmetros e Variáveis
3.3 Formulação Determinística Equivalente (DEQ)
3.4 Decomposição via PDDE

**Cap. 4 — Implementação Computacional**
4.1 Arquitetura Geral do Sistema
4.2 Pipeline de Geração de Dados (Python)
4.3 Geração de Cenários de PLD via PAR(1)
4.4 Geração Estocástica de Produção Física
4.5 Construção da Carteira e Oportunidades de Negociação
4.6 Implementação do DEQ em JuMP
4.7 Implementação da PDDE em SDDP.jl

**Cap. 5 — Experimentos Computacionais**
5.1 Configuração Experimental
5.2 Análise de Escalabilidade Computacional
5.3 Análise da Política de Contratação
5.4 Discussão dos Resultados

**Cap. 6 — Conclusão**
6.1 Conclusões
6.2 Limitações do Trabalho
6.3 Trabalhos Futuros

---

## Convenções adotadas

- Codificação: UTF-8 (o template original era latin1; adaptado para Overleaf/VSCode)
- Citações inline: `\citeonline{chave}` → "Pereira e Pinto (1991) mostraram que..."
- Citações no final de frase: `\cite{chave}`
- Estilo bibliográfico: `coppe` (arquivo `coppe.bst` necessário — vem com o
  template original da Poli)
- Acentuação: usar comandos LaTeX explícitos (`\c{c}`, `\~{a}`, etc.) nos
  arquivos de capa/preâmbulo para compatibilidade; nos capítulos pode usar
  UTF-8 direto (ex: `ção`, `ã`, `é`)
- Equações: numeradas com `\label{eq:nome}` e referenciadas com `\eqref{eq:nome}`
- Figuras: sempre com `\label{fig:nome}`, caption abaixo, fonte quando necessário
- `\emph{}` para termos técnicos em inglês na primeira ocorrência

---

## Status atual

| Capítulo | Status |
|---|---|
| Cap. 1 — Introdução | stub (TODO) |
| Cap. 2 — Revisão Bibliográfica | **COMPLETO** |
| Cap. 3 — Modelagem Matemática | **COMPLETO** |
| Cap. 4 — Implementação | stub (TODO) |
| Cap. 5 — Experimentos | stub (TODO) |
| Cap. 6 — Conclusão | stub (TODO) |
| Preambulo.tex | resumo/abstract preenchidos, dedicatória/agradecimentos TODO |
| referencias.bib | refs do artigo + Birge2011 + Shapiro2009 |

---

## Referências já no .bib

- `pereira1991` — Pereira & Pinto, Mathematical Programming, 1991 (paper original PDDE)
- `dowson2021` — Dowson & Kapelevich, INFORMS JoC, 2021 (SDDP.jl)
- `lubin2023` — Lubin et al., MPC, 2023 (JuMP 1.0)
- `bessa2006` — Bessa et al., LACTEC/COPEL, 2006
- `mme2002` — MME, Relatório de Progresso nº 2, 2002
- `ccee2024a` — CCEE, conceitos PLD
- `ccee2024b` — CCEE, dados de medição
- `ccee2024c` — CCEE, resultados leilões MCSD
- `birge2011` — Birge & Louveaux, Introduction to Stochastic Programming, 2011
- `shapiro2009` — Shapiro, Dentcheva & Ruszczyński, Lectures on SP, 2009

---

## Instruções para o agente

Quando receber uma tarefa de escrita:
1. Leia este CONTEXT.md primeiro
2. Verifique o status da seção pedida na tabela acima
3. Se precisar de contexto técnico do código, peça para ver os arquivos relevantes
   de `src/dataPipeline/` ou `src/model/v3_models/` do repo TCC-Lucas-Garcia
4. Escreva diretamente no arquivo `.tex` correspondente
5. Atualize a tabela de status neste CONTEXT.md após concluir
6. Não crie novos arquivos fora da estrutura definida acima sem confirmar primeiro
