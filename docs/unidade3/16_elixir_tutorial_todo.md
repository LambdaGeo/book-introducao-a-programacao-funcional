# Trabalho Elixir: ToDo List com Phoenix LiveView

Este capítulo apresenta o **segundo projeto avaliativo da Unidade 3**: construir uma aplicação de **Lista de Tarefas (Todo List)** do zero, usando a stack moderna de **Elixir com Phoenix LiveView** — um framework funcional e reativo para o desenvolvimento web full-stack.

!!! success "Tutorial completo no site de tutoriais"
    O passo a passo detalhado — com todos os comandos, códigos e explicações — está publicado como tutorial no site de tutoriais do LambdaGEO:

    **[Elixir e Phoenix LiveView: Construindo uma Aplicação Todo List do Zero](https://lambdageo.github.io/tutoriais/elixir/)**

    Este capítulo é apenas a introdução e o contrato de entrega; siga o tutorial para desenvolver o projeto.

---

## 🔁 Um Mesmo Problema, Dois Caminhos Funcionais

Este projeto é o **segundo lado de uma mesma jornada**: no [trabalho de Clojure](10_clojure_tutorial_todo.md) resolvemos o mesmo problema com frontend reativo no navegador e comunicação via API REST. Aqui, faremos o mesmo *conceitualmente*, mas com uma arquitetura radicalmente diferente:

| Aspecto | Clojure | Elixir |
|---|---|---|
| Paradigma | Funcional puro (imutabilidade explícita) | Funcional concorrente (processos leves) |
| Renderização | Frontend reativo com Reagent (React) | LiveView (renderização no servidor em tempo real) |
| Comunicação | API REST + JSON | Canal WebSocket interno (phx) |
| Persistência | next.jdbc + SQLite | Ecto + SQLite |

Comparar as duas soluções — e argumentar sobre os *trade-offs* de cada arquitetura — faz parte do aprendizado esperado.

## ⚙️ A Stack

* **Linguagem:** Elixir 1.17+ (Erlang/OTP 26+);
* **Framework Web:** Phoenix 1.8 + LiveView 1.1;
* **Banco de Dados:** SQLite (via Ecto);
* **Estilo:** Tailwind CSS v4 + daisyUI (já integrados ao Phoenix).

## 🗺️ A Jornada (Fases do Tutorial)

1. **Fundação**: ambiente, Git e scaffolding do projeto Phoenix;
2. **Hello World LiveView**: prova de conceito da renderização reativa;
3. **Estado em memória**: adicionando tarefas no processo LiveView;
4. **Persistência**: Ecto, *migrations* e *schemas*;
5. **LiveView + banco**: *changesets* e validação de formulários;
6. **Delete**: remoção de tarefas com tratamento de eventos;
7. **Toggle**: marcar tarefas como concluídas;
8. **Tema**: personalização visual com Tailwind/daisyUI.

Repare como o projeto exercita os conceitos da unidade: **pattern matching** nos handlers de eventos, o **operador pipe** nas consultas Ecto, e os **processos da BEAM** — cada usuário conectado é um processo LiveView isolado, aplicando diretamente o modelo de atores estudado no capítulo de [processos e concorrência](14_elixir_processos.md).

## 📦 Requisitos de Entrega

1. **Repositório GitHub** com histórico de commits **incremental**, refletindo as fases do tutorial (Conventional Commits recomendado);
2. Aplicação funcional: criar, listar, concluir e remover tarefas, com persistência e atualização em tempo real (sem JavaScript manual);
3. `README.md` explicando como configurar e executar o projeto;
4. **Extensões além do tutorial são o diferencial de nota** — sugestões: PubSub para sincronizar múltiplos usuários em tempo real, filtros, edição de tarefas, testes com ExUnit.

A pontuação e os critérios detalhados estão no [plano da unidade](00_plano_unidade3.md).
