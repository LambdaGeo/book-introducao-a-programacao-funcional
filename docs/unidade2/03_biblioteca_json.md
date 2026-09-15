# Desenvolvendo uma Biblioteca JSON em Haskell

Neste capítulo, iniciamos o projeto central da Unidade 2: uma biblioteca completa para manipulação e serialização de dados em formato **JSON (JavaScript Object Notation)**, adaptando o clássico *Real World Haskell* (Capítulo 5) para um projeto moderno gerenciado pelo **Cabal**.

!!! success "Tutorial completo no site de tutoriais"
    O passo a passo detalhado — do `cabal init` até o pacote final, com todos os módulos, códigos e explicações — está publicado como tutorial no site de tutoriais do LambdaGEO:

    **[Construindo e Testando uma Biblioteca Haskell: JSON, Pretty Printing e QuickCheck](https://lambdageo.github.io/tutoriais/haskell/)**

    Este capítulo apresenta os conceitos e as decisões de projeto; siga a **Parte 1** do tutorial para construir a biblioteca.

!!! info "Leitura complementar"
    O capítulo original que inspirou este projeto: [*Real World Haskell*, Cap. 5 — Writing a library: working with JSON data](http://book.realworldhaskell.org/read/writing-a-library-working-with-json-data.html) (em inglês; o tutorial moderniza o código para GHC 9.x e Cabal).

---

## 📜 Um Tour Relâmpago pelo JSON

O JSON é uma representação pequena e simples para armazenar e transmitir dados estruturados — por exemplo, de um serviço web para uma aplicação no navegador. O formato é descrito em [www.json.org](http://www.json.org/) e na RFC 4627. Ele suporta quatro tipos básicos de valor:

```json
"uma string"   12345   true   null
```

E dois tipos compostos: o **array**, uma sequência ordenada de valores, e o **objeto**, uma coleção não ordenada de pares nome/valor (os nomes são sempre strings):

```json
[-3.14, true, null, "uma string"]
{"numeros": [1,2,3,4,5], "util": false}
```

## 🧩 A Ideia Central: Modelar o Domínio com um ADT

A essência da biblioteca é a correspondência direta entre a gramática do JSON e um tipo algébrico: **cada tipo do JSON vira um construtor de valor**:

```haskell
data JValue = JString String
            | JNumber Double
            | JBool Bool
            | JNull
            | JObject [(String, JValue)]
            | JArray [JValue]
            deriving (Eq, Ord, Show)
```

Note que `JObject` e `JArray` são **recursivos** — carregam outros `JValue` — exatamente como no capítulo de [tipos recursivos](01_declarando_tipos_classes.md). Com o tipo definido, todo o restante da biblioteca é pattern matching sobre ele: *accessors* seguros que retornam `Maybe` (`getString`, `getInt`, ...) e funções de renderização.

## 🏗️ As Decisões de Projeto

O tutorial constrói a biblioteca em três módulos, e cada um materializa uma lição de engenharia funcional:

1. **`SimpleJSON.hs`** — o ADT `JValue` e os accessors. *Lição:* modelagem de domínio com tipos e extração segura via `Maybe`.
2. **`Prettify.hs`** — uma biblioteca **genérica** de pretty printing baseada em um tipo abstrato `Doc`. *Lição:* renderizar direto para `String` funciona, mas amarra a implementação; ao basear a biblioteca em um tipo abstrato, podemos trocar a representação interna sem que os usuários percebam.
3. **`PrettyJSON.hs`** — o renderizador que converte `JValue` em `Doc`, cuidando de escapes de strings e caracteres Unicode. *Lição:* separar a *geração do formato* da *impressão em si* — a renderização é uma função **pura**, e o `IO` fica só nas bordas do programa.

Além dos módulos, o tutorial cobre a estrutura do projeto Cabal (biblioteca + executável), o sistema de exportações dos módulos e o empacotamento — conectando diretamente com a estrutura de projeto apresentada no capítulo de [Instalação do Ambiente](../unidade1/02_instalacao.md).

## 🎯 O que Você Deve Dominar ao Final

* Definir um ADT recursivo que modela um formato de dados real;
* Escrever accessors totais usando `Maybe`;
* Explicar por que a biblioteca usa um tipo abstrato `Doc` em vez de renderizar direto para `String`;
* Organizar módulos com listas de exportação explícitas em um projeto Cabal.

No próximo capítulo, garantiremos a **correção** da biblioteca com testes baseados em propriedades usando o QuickCheck — a Parte 2 do mesmo tutorial.
