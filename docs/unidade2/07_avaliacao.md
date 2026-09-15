# Avaliação da Unidade 2: Projeto hs2json

Esta atividade avaliativa consiste em construir um projeto completo em Haskell moderno, aplicando técnicas de desenvolvimento de bibliotecas, modularização em Cabal e testes baseados em propriedades com QuickCheck.

!!! success "Tutorial guiado no blog"
    O desenvolvimento completo do projeto é guiado pelo tutorial no blog do LambdaGEO:

    **[Construindo e Testando uma Biblioteca Haskell: JSON, Pretty Printing e QuickCheck](https://lambdageo.github.io/blog/tutorial-haskell-json-quickcheck/)**

---

## 📋 Especificações do Projeto

Você deverá criar um projeto Cabal chamado `hs2json` estruturado como uma biblioteca reutilizável e uma suíte de testes robusta.

O projeto deve conter as seguintes funcionalidades mínimas:

### A. Tipo de Dados e Accessors (`SimpleJSON.hs`)
O arquivo `src/SimpleJSON.hs` deve definir o tipo algébrico de dados `JValue` que representa JSON, além de funções para extração segura de dados (*accessors*):

* `getString :: JValue -> Maybe String`
* `getInt :: JValue -> Maybe Int`
* `getDouble :: JValue -> Maybe Double`
* `getBool :: JValue -> Maybe Bool`
* `getObject :: JValue -> Maybe [(String, JValue)]`
* `getArray :: JValue -> Maybe [JValue]`
* `isNull :: JValue -> Bool`

### B. O Pretty Printer JSON (`Prettify.hs` e `PrettyJSON.hs`)
Você deve desenvolver uma biblioteca de formatação baseada em um tipo abstrato `Doc`. O renderizador deve:

* Escapar strings corretamente (caracteres Unicode, quebras de linha `\n`, tabs `\t`, etc.).
* Imprimir objetos JSON formatados com indentação e quebra de linhas para fácil legibilidade.

### C. Testes com QuickCheck (`test/Spec.hs`)
O projeto deve conter uma suíte de testes rodando via `cabal test` com, no mínimo:

1. Implementação da Typeclass `Arbitrary` para o tipo `JValue` de forma a gerar dados aleatórios corretos e não-recursivos infinitos.
2. No mínimo **5 propriedades** escritas para testar as invariantes da sua biblioteca (ex: idempotência de renderização, corretude dos escapes de string, corretude de acesso nas funções `get`).

---

## ⚙️ Requisitos do Repositório e Entrega

1. **Estrutura Cabal**: O projeto deve compilar limpo rodando `cabal build`.
2. **Qualidade dos Testes**: A suíte de testes deve rodar com sucesso via `cabal test`.
3. **Organização do Git**: O histórico de commits no repositório GitHub deve ser incremental e refletir o avanço por etapas do desenvolvimento do projeto (Conventional Commits recomendado).
4. **Arquivo `README.md`**: Explicando claramente como clonar, compilar, testar e executar a aplicação.
