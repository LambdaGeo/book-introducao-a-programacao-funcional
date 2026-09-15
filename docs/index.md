# Introdução à Programação Funcional

Bem-vindo ao material didático da disciplina de **Introdução à Programação Funcional**! Este livro serve como guia completo da disciplina, organizada em **três unidades de 20 horas cada (60 horas totais)**, que percorrem desde os fundamentos do paradigma funcional até o desenvolvimento de aplicações modernas e reativas.

---

## 📋 Plano da Disciplina

| | |
|---|---|
| **Carga horária total** | 60 horas |
| **Estrutura** | 3 unidades de 20 horas (10 aulas de 2h por unidade) |
| **Linguagens** | Haskell, Clojure/ClojureScript e Elixir |
| **Pré-requisitos** | Lógica de programação e experiência com alguma linguagem imperativa |

**Ementa:** Paradigmas de programação. Funções puras, imutabilidade e transparência referencial. Tipos algébricos e classes de tipos. Recursão, listas e funções de alta ordem. Programas interativos e efeitos. Testes baseados em propriedades. Linguagens funcionais na indústria: Clojure (JVM) e Elixir (BEAM). Concorrência com o modelo de atores. Desenvolvimento de aplicações web funcionais.

Cada unidade possui um **plano detalhado** com objetivos de aprendizagem, cronograma aula a aula, metodologia e critérios de avaliação — consulte a primeira página de cada unidade no menu.

---

## 🎯 Estrutura do Curso

### 📘 Unidade 1: Paradigmas e Haskell Básico (20h)
Exploramos os fundamentos teóricos do paradigma funcional em comparação aos paradigmas imperativo e orientado a objetos. Utilizando a linguagem **Haskell**, uma linguagem funcional pura com tipagem estática forte, estudamos os conceitos essenciais:

* Fundamentos da sintaxe e ambiente de desenvolvimento (GHCup, Cabal, REPL/GHCi)
* Funções puras, casamento de padrões (*pattern matching*) e guardas
* Manipulação e compreensão de listas, e recursão
* Funções de Alta Ordem (*High-Order Functions*: `map`, `filter`, `fold`)

*Avaliação: prova escrita individual (70%) e listas de exercícios (30%). Veja o [plano completo da unidade](unidade1/00_plano_unidade1.md).*

### 📙 Unidade 2: Haskell Avançado e Qualidade de Código (20h)
Avançamos para tópicos do ecossistema de produção em Haskell, adaptados às práticas do Haskell moderno:

* Definição de novos tipos algébricos (ADTs) e Typeclasses
* Programação interativa usando a Monad de Entrada/Saída (`IO`)
* Estrutura de projetos industriais com **Cabal**, empacotamento e gerenciamento de dependências
* Construção de uma biblioteca de manipulação e serialização de dados JSON (adaptado do clássico *Real World Haskell*, Capítulo 5)
* Testes baseados em propriedades com **QuickCheck** e garantia de qualidade de software (adaptado do *Real World Haskell*, Capítulo 11)
* Duas aplicações avançadas do paradigma funcional: construção de interpretadores/parsers (regex via derivadas, um interpretador de Scheme) e processamento de dados geoespaciais

*Avaliação: desenvolvimento prático da biblioteca JSON integrada com uma suíte de testes QuickCheck (100%). Veja o [plano completo da unidade](unidade2/00_plano_unidade2.md).*

### 📗 Unidade 3: Clojure e Elixir (20h)
Na última unidade, exploramos a diversidade do paradigma funcional através de duas linguagens modernas focadas na indústria:

1. **Clojure**: Um dialeto Lisp moderno e pragmático rodando sobre a Máquina Virtual Java (JVM), que traz forte ênfase em imutabilidade e metaprogramação.
2. **Elixir**: Uma linguagem funcional concorrente que roda sobre a BEAM (Erlang VM), famosa por sua escalabilidade, tolerância a falhas e reatividade em tempo real.

*Avaliação: dois projetos práticos com peso igual (veja o [plano completo da unidade](unidade3/00_plano_unidade3.md)):*

* **Clojure/ClojureScript ToDo List (50%)**: Uma aplicação full-stack com backend Jetty/Reitit, banco SQLite (next.jdbc) e frontend reativo com ClojureScript e Reagent (React 19).
* **Elixir/Phoenix LiveView ToDo List (50%)**: Uma aplicação de tempo real concorrente onde o frontend e o backend se integram nativamente via WebSockets e LiveView, utilizando Ecto e SQLite.

---

## 📚 Bibliografia e Referências

* **Haskell**: Lipovača, Miran. *Learn You a Haskell for Great Good!*
* **Real World Haskell**: O'Sullivan, Bryan; Stewart, Don; Goerzen, John. *Real World Haskell*. O'Reilly, 2008. Disponível gratuitamente em [book.realworldhaskell.org](http://book.realworldhaskell.org/read/) (com código modernizado para os padrões atuais do GHC nos capítulos deste livro).
* **Clojure**: Miller, Alex. *Programming Clojure*.
* **Elixir**: Thomas, Dave. *Programming Elixir*.
* **Materiais e Tutoriais Complementares**: [LambdaGEO Research Lab](https://lambdageo.github.io).
