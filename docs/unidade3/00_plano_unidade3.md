# Unidade 3 — Clojure e Elixir (20h)

## Identificação

| | |
|---|---|
| **Unidade** | 3 de 3 |
| **Carga horária** | 20 horas (10 aulas de 2h) |
| **Pré-requisitos** | Unidades 1 e 2 concluídas (fundamentos do paradigma funcional em Haskell) |
| **Linguagens** | Clojure/ClojureScript (JVM) e Elixir (BEAM/Erlang VM) |

## Objetivos de aprendizagem

Ao final desta unidade, o estudante deverá ser capaz de:

1. **Transferir** os conceitos funcionais aprendidos em Haskell (imutabilidade, alta ordem, recursão) para linguagens dinâmicas da indústria.
2. **Programar** em Clojure utilizando coleções persistentes, funções de alta ordem, threading macros e lazy sequences.
3. **Programar** em Elixir utilizando pattern matching, o operador pipe, módulos e testes com ExUnit.
4. **Explicar** o modelo de atores da BEAM e construir programas concorrentes com processos e mensagens em Elixir.
5. **Desenvolver** aplicações web completas nos dois ecossistemas: full-stack Clojure/ClojureScript e tempo real com Elixir/Phoenix LiveView.

## Cronograma

| Aula | CH | Conteúdo | Capítulo |
|:---:|:---:|---|---|
| 1 | 2h | Clojure: sintaxe Lisp, formas especiais, REPL e fundamentos | [Clojure Fundamentos](01_clojure_fundamentos.md) |
| 2 | 2h | Coleções persistentes: vetores, mapas e imutabilidade estrutural | [Clojure Vetores e Mapas](02_clojure_colecoes.md) |
| 3 | 2h | Alta ordem, threading macros (`->`, `->>`) e mapas complexos | [Alta Ordem](03_clojure_alta_ordem.md) · [Threading](04_clojure_threading_mapas_complexos.md) |
| 4 | 2h | Lazy sequences, macros e interoperabilidade com Java | [Lazy Sequences](05_clojure_lazy_sequences.md) · [Macros](06_clojure_metaprogramacao.md) · [Interop](07_clojure_interop.md) |
| 5 | 2h | Revisão dos pilares e prática dirigida com exercícios resolvidos | [Revisão](08_clojure_revisao.md) · [Exercícios](09_clojure_exercicios.md) |
| 6 | 2h | Laboratório orientado: **projeto ToDo full-stack em Clojure/ClojureScript** | [Trabalho Clojure](10_clojure_tutorial_todo.md) |
| 7 | 2h | Elixir: fundamentos, pattern matching, pipe e coleções | [Fundamentos](11_elixir_fundamentos.md) · [Pattern Matching](12_elixir_colecoes_pattern_matching.md) |
| 8 | 2h | Módulos, funções e testes com ExUnit | [Módulos, Funções e Testes](13_elixir_modulos_funcoes_testes.md) |
| 9 | 2h | Processos, concorrência e noções de metaprogramação na BEAM | [Processos](14_elixir_processos.md) · [Metaprogramação](15_elixir_metaprogramacao.md) |
| 10 | 2h | Laboratório orientado: **projeto ToDo em Elixir/Phoenix LiveView** | [Trabalho Elixir](16_elixir_tutorial_todo.md) |

!!! note "Trabalho extraclasse"
    Os dois projetos (aulas 6 e 10) são iniciados em laboratório com orientação do professor, mas sua conclusão exige dedicação extraclasse. O desenvolvimento passo a passo é guiado pelos tutoriais publicados no site de tutoriais do LambdaGEO: **[Clojure/ClojureScript Todo List](https://lambdageo.github.io/tutoriais/clojure/)** e **[Elixir/Phoenix LiveView Todo List](https://lambdageo.github.io/tutoriais/elixir/)** — os capítulos correspondentes deste material contêm a introdução e os requisitos de entrega. Os capítulos de metaprogramação (Clojure e Elixir) são apresentados como panorama em aula e indicados como leitura de aprofundamento.

## Metodologia

A unidade aplica a estratégia de *transferência de paradigma*: cada conceito é apresentado retomando o equivalente já estudado em Haskell. As aulas 1–5 e 7–9 combinam exposição com codificação ao vivo no REPL (Clojure) e no IEx (Elixir); as aulas 6 e 10 são laboratórios dedicados aos projetos avaliativos.

## Avaliação

Dois projetos práticos, com peso igual:

- **Projeto Clojure (50%)** — aplicação ToDo List full-stack: backend Jetty/Reitit, banco SQLite (next.jdbc) e frontend reativo com ClojureScript e Reagent (React 19). Especificação no capítulo [Trabalho Clojure](10_clojure_tutorial_todo.md).
- **Projeto Elixir (50%)** — aplicação ToDo List em tempo real com Phoenix LiveView, Ecto e SQLite, integrando frontend e backend via WebSockets. Especificação no capítulo [Trabalho Elixir](16_elixir_tutorial_todo.md).

Cada projeto é avaliado por: funcionamento conforme a especificação, qualidade e idiomaticidade do código, e extensões implementadas além do tutorial (diferencial de nota).

## Bibliografia da unidade

- MILLER, Alex; HALLOWAY, Stuart; BEDRA, Aaron. *Programming Clojure*. 3. ed.
- THOMAS, Dave. *Programming Elixir ≥ 1.6*.
- Documentação oficial: [clojure.org](https://clojure.org) e [elixir-lang.org](https://elixir-lang.org).
