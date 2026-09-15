# Unidade 1 — Paradigmas e Haskell Básico (20h)

## Identificação

| | |
|---|---|
| **Unidade** | 1 de 3 |
| **Carga horária** | 20 horas (10 aulas de 2h) |
| **Pré-requisitos** | Lógica de programação; experiência prévia com alguma linguagem imperativa |
| **Linguagem** | Haskell (GHC via GHCup, GHCi, Cabal) |

## Objetivos de aprendizagem

Ao final desta unidade, o estudante deverá ser capaz de:

1. **Diferenciar** o paradigma funcional dos paradigmas imperativo e orientado a objetos, explicando conceitos como imutabilidade, transparência referencial e funções puras.
2. **Configurar** o ambiente de desenvolvimento Haskell (GHCup, GHCi, Cabal) e utilizar o REPL como ferramenta de exploração.
3. **Escrever** funções puras usando casamento de padrões (*pattern matching*), guardas e expressões condicionais.
4. **Interpretar** assinaturas de tipos e utilizar as classes de tipos básicas (`Eq`, `Ord`, `Show`, `Num`).
5. **Manipular** listas com recursão, compreensão de listas e as funções de alta ordem `map`, `filter` e `fold`.

## Cronograma

| Aula | CH | Conteúdo | Capítulo |
|:---:|:---:|---|---|
| 1 | 2h | Paradigmas de programação: imperativo × funcional; motivação e história | [Introdução aos Paradigmas](01_paradigmas.md) |
| 2 | 2h | Instalação do ambiente (GHCup, Cabal), primeiros passos no GHCi e estrutura de um projeto Cabal | [Instalação do Ambiente](02_instalacao.md) |
| 3 | 2h | Fundamentos: expressões, definição de funções, precedência e aplicação | [Fundamentos](03_fundamentos.md) |
| 4 | 2h | Tipos básicos, tuplas, polimorfismo e classes de tipos | [Tipos e Classes](04_tipos_classes.md) |
| 5 | 2h | Funções: pattern matching, guardas, `where`, `let` e lambdas | [Funções](05_funcoes.md) |
| 6 | 2h | Listas: construção, recursão sobre listas e compreensão de listas | [Listas](06_listas.md) |
| 7 | 2h | Prática dirigida em laboratório: recursão e listas | [Listas](06_listas.md) |
| 8 | 2h | Funções de alta ordem: `map`, `filter`, `fold`, composição e aplicação parcial | [Funções de Alta Ordem](07_alta_ordem.md) |
| 9 | 2h | Resolução comentada em sala das listas de exercícios e preparação para a avaliação | [Lista de Exercícios](08_exercicios.md) |
| 10 | 2h | Revisão geral e **avaliação escrita da unidade** | — |

## Metodologia

Aulas expositivas dialogadas intercaladas com codificação ao vivo no GHCi. As aulas 7 e 9 são integralmente práticas, realizadas em laboratório com acompanhamento individual. Recomenda-se que o estudante reproduza todos os exemplos do material no REPL antes de cada aula.

## Avaliação

- **Avaliação escrita individual (70%)** — aula 10: questões conceituais sobre paradigmas e questões de codificação em Haskell básico (funções, listas, alta ordem).
- **Listas de exercícios (30%)** — entregas parciais ao longo da unidade, baseadas nos exercícios dos capítulos.

## Bibliografia da unidade

- LIPOVAČA, Miran. *Learn You a Haskell for Great Good!* — capítulos 1 a 6.
- O'SULLIVAN, Bryan; STEWART, Don; GOERZEN, John. *Real World Haskell* — capítulos 1 a 4 (partes adaptadas ao longo dos capítulos desta unidade).
- HUTTON, Graham. *Programming in Haskell*. 2. ed. — capítulos 1 a 7 (leitura complementar).
