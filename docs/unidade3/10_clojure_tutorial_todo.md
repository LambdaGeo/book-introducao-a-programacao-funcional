# Trabalho Clojure: ToDo List Full-Stack

Este capítulo apresenta o **primeiro projeto avaliativo da Unidade 3**: a construção de uma aplicação **Todo List completa**, indo de um repositório Git vazio até um projeto **full-stack funcional**, usando o ecossistema Clojure moderno.

!!! success "Tutorial completo no site de tutoriais"
    O passo a passo detalhado — com todos os comandos, códigos, erros esperados e correções — está publicado como tutorial no site de tutoriais do LambdaGEO:

    **[Clojure e ClojureScript: Construindo uma Aplicação Todo List do Zero](https://lambdageo.github.io/tutoriais/clojure/)**

    Este capítulo é apenas a introdução e o contrato de entrega; siga o tutorial para desenvolver o projeto.

---

## 🧱 O que Você Vai Construir

* **Backend:** Clojure, com **Ring**, **Reitit** e **next.jdbc**;
* **Frontend:** ClojureScript, com **Reagent 2.0 (React 19)** e **Shadow-CLJS**;
* **Banco de Dados:** **SQLite**, para persistência real.

Mais do que um tutorial de "copiar e colar", o guia foi pensado para **ensinar arquitetura**: você vai entender por que a aplicação funciona — e por que ela quebra — explorando erros típicos do desenvolvimento real (CORS, formatos de dados incompatíveis, sincronização de estado) e aprendendo a corrigi-los.

## 🗺️ A Jornada (Fases do Tutorial)

1. **Fundação**: verificação do ambiente, Git e `.gitignore`;
2. **Backend mínimo**: um servidor "Hello World" com Jetty e Reitit;
3. **Banco em memória**: API REST criando e lendo tarefas com um `atom`;
4. **Frontend reativo isolado**: interface dinâmica com Reagent;
5. **Integração full-stack**: comunicação via API REST, lidando com CORS;
6. **Banco real**: migrando para SQLite com `next.jdbc`;
7. **CRUD completo**: marcar como concluída (*Update*) e remover (*Delete*);
8. **Documentação**: um `README.md` profissional e a preparação da entrega.

Repare como o projeto exercita os conceitos da unidade: **imutabilidade e `atom`** para estado, **mapas e threading macros** na API, **funções de alta ordem** na renderização com Reagent, e **interoperabilidade** com o ecossistema JVM/JavaScript.

## 📦 Requisitos de Entrega

1. **Repositório GitHub** com histórico de commits **incremental**, refletindo as fases do tutorial (Conventional Commits recomendado);
2. Aplicação funcional: criar, listar, concluir e remover tarefas, com persistência em SQLite;
3. `README.md` explicando como clonar, compilar e executar backend e frontend;
4. **Extensões além do tutorial são o diferencial de nota** — sugestões: filtros (todas/ativas/concluídas), edição de tarefas, datas de vencimento, testes com `clojure.test`.

A pontuação e os critérios detalhados estão no [plano da unidade](00_plano_unidade3.md).
