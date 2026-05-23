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
│   ├── revisao/                  ← Cap. 2: Referencial Teórico
│   │   ├── revisao.tex
│   │   ├── mercado_energia.tex
│   │   ├── prog_estocastica.tex
│   │   ├── pdde_literatura.tex
│   │   ├── portfolio_literatura.tex
│   │   └── ferramentas.tex
│   ├── revisao_lit/              ← Cap. 3: Revisão da Literatura (NOVO)
│   │   ├── revisao_lit.tex       ← orquestrador
│   │   ├── portfolio_contratos.tex
│   │   ├── metodos_sddp.tex
│   │   ├── modelagem_pld_lit.tex
│   │   └── posicionamento.tex
│   ├── modelagem/                ← Cap. 4: Definição Formal do Problema
│   │   ├── modelagem.tex
│   │   ├── descricao_problema.tex   ← ESCRITO
│   │   ├── notacao.tex              ← ESCRITO
│   │   ├── deq.tex                  ← ESCRITO
│   │   └── pdde.tex                 ← ESCRITO
│   ├── implementacao/            ← Cap. 5: Metodologia Proposta
│   │   ├── implementacao.tex
│   │   ├── arquitetura.tex
│   │   ├── pipeline_dados.tex
│   │   ├── geracao_pld.tex
│   │   ├── geracao_producao.tex
│   │   ├── carteira_trades.tex
│   │   ├── impl_deq.tex
│   │   └── impl_sddp.tex
│   ├── experimentos/             ← Cap. 6: Experimentos Computacionais
│   │   ├── experimentos.tex
│   │   ├── configuracao.tex
│   │   ├── escalabilidade.tex
│   │   ├── politica.tex
│   │   └── discussao.tex
│   └── conclusao/                ← Cap. 7: Conclusões e Trabalhos Futuros
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

**Cap. 2 — Referencial Teórico**
2.1 O Mercado de Energia Elétrica Brasileiro
2.2 Programação Estocástica Multiestágio
2.3 Programação Dinâmica Dual Estocástica (PDDE)
2.4 Gestão de Portfólio de Contratos de Energia
2.5 Ferramentas Computacionais: JuMP e SDDP.jl

**Cap. 3 — Revisão da Literatura** ← NOVO (ESCRITO)
3.1 Otimização de Portfólio de Contratos de Energia
3.2 Métodos de Solução para Programação Estocástica Multiestágio
3.3 Modelagem Estocástica do PLD
3.4 Posicionamento do Trabalho e Lacunas Identificadas

**Cap. 4 — Definição Formal do Problema** ← COMPLETO (era Cap. 3)
4.1 Descrição do Problema
4.2 Notação: Conjuntos, Parâmetros e Variáveis
4.3 Formulação Determinística Equivalente (DEQ)
4.4 Decomposição via PDDE

**Cap. 5 — Metodologia Proposta** ← stub (era Cap. 4 "Implementação Computacional")
5.1 Arquitetura Geral do Sistema
5.2 Pipeline de Geração de Dados (Python)
5.3 Geração de Cenários de PLD via PAR(1)
5.4 Geração Estocástica de Produção Física
5.5 Construção da Carteira e Oportunidades de Negociação
5.6 Implementação do DEQ em JuMP
5.7 Implementação da PDDE em SDDP.jl

**Cap. 6 — Experimentos Computacionais** ← stub (era Cap. 5)
6.1 Configuração Experimental
6.2 Análise de Escalabilidade Computacional
6.3 Análise da Política de Contratação
6.4 Discussão dos Resultados

**Cap. 7 — Conclusões e Trabalhos Futuros** ← stub (era Cap. 6)
7.1 Conclusões
7.2 Limitações do Trabalho
7.3 Trabalhos Futuros

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
| Cap. 2 — Referencial Teórico | **COMPLETO** |
| Cap. 3 — Revisão da Literatura | **ESCRITO** |
| Cap. 4 — Definição Formal do Problema | **COMPLETO** (era Cap. 3) |
| Cap. 5 — Metodologia Proposta | stub (TODO) (era Cap. 4) |
| Cap. 6 — Experimentos Computacionais | stub (TODO) (era Cap. 5) |
| Cap. 7 — Conclusões e Trabalhos Futuros | stub (TODO) (era Cap. 6) |
| Preambulo.tex | resumo/abstract preenchidos, dedicatória/agradecimentos TODO |
| referencias.bib | 9 novas refs adicionadas e verificadas contra os PDFs |

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
- `shapiro2011` — Shapiro, EJOR, 2011 (convergência PDDE)
- `reis2013` — Reis, tese USP, 2013 (PAR(p) para vazões)
- `pedrini2019` — Pedrini, dissertação UFSC, 2019 (dois estágios + eólica)
- `guerreiro2017` — Ribeiro (Mário Guerreiro), dissertação FGV, 2017 (CVaR, ACL)
- `gunn2012` — Gunn (Laura Keiko), tese doutorado UNICAMP, 2012 (portfólio energia novos empreendimentos)
- `munhoz2018` — Munhoz (Letícia Leite), TCC UnB/FGA, 2018 (portfólio contratação ACL)
- `josemaria2018` — Maria (Thaisy Cristina José), TCC UFJF, 2018 (contratos e riscos ACL)
- `carvalho2021` — Carvalho (Tiago Paixão de), TCC UTFPR, 2021 (estratégias comercialização ACL)
- `shapiro2013` — Shapiro, Tekaya, da Costa (Joari Paulo), Soares — EJOR 224 (2013) 375–391
- `homemdemello2011` — Homem-de-Mello, de Matos, Finardi — Energy Systems 2 (2011) 1–31
- `philpott2012` — Philpott, de Matos — EJOR 218 (2012) 470–483
- `street2020` — Street, Valladão, Lawson, Velloso — Applied Energy 280 (2020) 115939
- `detzel2011` — Detzel et al. (7 autores), XIX SBRH, Maceió, 2011 (geração sintética AR(1), Sul do Brasil)

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
