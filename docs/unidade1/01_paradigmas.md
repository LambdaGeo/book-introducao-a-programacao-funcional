# Paradigmas de Programação e Introdução à Programação Funcional

Para entender a Programação Funcional (PF), é necessário primeiro compreender o conceito de **paradigma de programação** e a evolução histórica que nos trouxe até as linguagens modernas.

---

## 🧠 O que é um Paradigma de Programação?

Um paradigma de programação é um estilo ou uma forma de estruturar e organizar o raciocínio para resolver problemas através do código. Não se trata apenas de sintaxe, mas de um modelo mental sobre como os dados e as operações sobre esses dados devem se relacionar.

Os três paradigmas dominantes no desenvolvimento de software são:

### 1. Paradigma Imperativo
O modelo mental imperativo é focado em **como** fazer. O programa é visto como uma sequência de comandos que alteram o estado global da memória passo a passo.

* **Características**: Variáveis mutáveis, loops explicitos (`for`, `while`), estruturas condicionais e execução sequencial.
* **Analogia**: Uma receita de bolo passo a passo: *"Adicione 2 ovos, bata por 5 minutos, adicione farinha..."*

### 2. Paradigma Orientado a Objetos (POO)
Uma evolução do imperativo que agrupa estados (atributos) e comportamentos (métodos) em unidades conceituais chamadas **objetos**.

* **Características**: Encapsulamento, herança, polimorfismo. O estado do sistema é distribuído entre vários objetos que interagem enviando mensagens entre si.
* **Analogia**: Um teatro onde cada ator (objeto) tem seu próprio papel (método) e memória interna (atributos), conversando com outros atores.

### 3. Paradigma Declarativo / Funcional
O modelo mental funcional é focado em **o que** deve ser feito, e não em como. Os programas são construídos através da composição de funções matemáticas puras, sem alteração de estado global ou dados mutáveis.

* **Características**: Imutabilidade, funções como cidadãs de primeira classe, ausência de efeitos colaterais (*side effects*) e avaliação preguiçosa (*lazy evaluation*).
* **Analogia**: Uma linha de montagem matemática. Dados entram por um lado, passam por transformações (funções) e saem como novos dados do outro lado, sem alterar os dados originais.

---

## 📌 Histórico do Paradigma Funcional

A programação funcional baseia-se no **Cálculo Lambda (λ-cálculo)**, um formalismo matemático criado pelo lógico **Alonzo Church** na década de 1930 para estudar a computabilidade. Curiosamente, o cálculo lambda surgiu na mesma época em que **Alan Turing** criou a Máquina de Turing.

* **Máquina de Turing** (Base do Paradigma Imperativo): Baseada em transições de estado físicos e escrita/leitura em fitas magnéticas (memória).
* **Cálculo Lambda** (Base do Paradigma Funcional): Baseado em aplicações e definições de funções puras, substituição de variáveis e recursão.

Embora matematicamente equivalentes (Tese de Church-Turing), as duas abordagens levaram a caminhos diferentes na computação:

1. **Lisp (1958)**: Criada por John McCarthy — a primeira pessoa a perceber a conexão entre a prática de programação e o cálculo lambda —, foi a primeira linguagem a implementar seus conceitos, utilizando S-expressions. Lisp é a ancestral comum do Clojure.
2. **Fundamentos teóricos (1960s)**: Durante os anos 1960, os cientistas da computação começaram a reconhecer a importância do cálculo lambda. **Peter Landin** e **Christopher Strachey** desenvolveram os fundamentos das linguagens de programação: como entender o que os programas *fazem* (semântica operacional) e o que eles *significam* (semântica denotacional).
3. **Família ML (1970s)**: Robin Milner criou a linguagem ML — originalmente para auxiliar provas automatizadas de teoremas — trazendo sistemas de tipos estáticos fortes, polimorfismo e casamento de padrões. Deu origem a Standard ML e OCaml.
4. **As linguagens preguiçosas (1970s–80s)**: A década de 1970 viu o surgimento da avaliação preguiçosa como estratégia: David Turner desenvolveu SASL, KRC e Miranda, enquanto Rod Burstall e John Darlington desenvolveram NPL e Hope. Essas linguagens influenciaram outras nos anos 1980, como Lazy ML e Clean.
5. **Haskell (1990)**: No final dos anos 1980, os esforços de pesquisa em linguagens funcionais preguiçosas estavam espalhados por mais de uma dezena de linguagens. Preocupados com essa dispersão, os pesquisadores formaram um comitê para criar uma linguagem comum e, após três anos de trabalho, publicaram a especificação Haskell 1.0 em 1990 — batizada em homenagem ao lógico **Haskell Curry**. Muita gente desconfia de *design by committee*, mas o comitê do Haskell é um belo contraexemplo: produziu um projeto de linguagem elegante e unificou os esforços da comunidade de pesquisa. Do emaranhado de linguagens preguiçosas de 1990, apenas Haskell continua ativamente em uso. O padrão passou por revisões sucessivas — as mais importantes sendo o **Haskell 98** e o **Haskell 2010** —, e hoje o compilador **GHC** funciona como padrão de fato da linguagem.

### Da Academia ao Mundo Real

Durante a década de 1990, Haskell cumpriu um papel duplo: era uma linguagem estável para pesquisadores experimentarem com avaliação preguiçosa e, ao mesmo tempo, uma linguagem de ensino. Fora da academia, porém, poucos a conheciam — o slogan informal da comunidade era *"evitar o sucesso a todo custo"*.

Enquanto isso, o mainstream da programação experimentava ajustes relativamente pequenos — de C para C++, de C++ para Java. Na periferia, surgiam as linguagens dinâmicas: Guido van Rossum projetou o Python, Larry Wall criou o Perl e Yukihiro Matsumoto desenvolveu o Ruby. À medida que se popularizavam, elas espalharam três ideias fundamentais: os programadores não estavam trabalhando em linguagens suficientemente **expressivas**; muitas vezes vale a pena sacrificar desempenho de execução em troca de um grande ganho de **produtividade**; e — não por acaso — várias dessas linguagens **emprestaram conceitos da programação funcional**.

Impulsionado em parte por essa visibilidade, Haskell escapou da academia e hoje tem uma comunidade vibrante de uso comercial e open source. O movimento se completou: as linguagens mainstream de hoje (Java, C++, Python, JavaScript) incorporaram lambdas, `map`/`filter` e imutabilidade, e linguagens funcionais nativas da indústria — como o **Clojure** e o **Elixir**, que estudaremos na Unidade 3 — sustentam sistemas de larga escala.

---

## 🛡️ Pilares da Programação Funcional

### 1. Pureza Funcional e Efeitos Colaterais
Uma função é dita **pura** se ela atende a dois requisitos básicos:

1. Retorna sempre o mesmo valor para os mesmos argumentos (determinismo).
2. Não causa efeitos colaterais visíveis (como modificar uma variável global, escrever em arquivos, alterar parâmetros recebidos ou imprimir texto na tela).

### 2. Imutabilidade
Uma vez que um valor é criado, ele nunca muda. Para "alterar" um dado, criamos uma nova versão dele aplicando uma função. Isso elimina completamente problemas de concorrência e condições de corrida (*race conditions*), pois múltiplos threads podem ler o mesmo dado sem medo de que ele seja modificado no meio do caminho.

### 3. Funções como Cidadãs de Primeira Classe
Em linguagens funcionais, funções são tratadas como qualquer outro tipo de dado:

* Podem ser passadas como argumentos para outras funções.
* Podem ser retornadas por outras funções.
* Podem ser armazenadas em variáveis ou coleções de dados.

Funções que recebem ou retornam outras funções são chamadas de **Funções de Alta Ordem** (*High-Order Functions*).

### 4. Avaliação Preguiçosa (*Lazy Evaluation*)
Em Haskell, os cálculos são adiados até que seus resultados sejam realmente necessários. Esse recurso não é apenas um modo de postergar a avaliação: ele afeta profundamente a forma como escrevemos programas.

Um exemplo clássico: suponha que queremos encontrar os *k* menores elementos de uma lista não ordenada. Em uma linguagem tradicional, a abordagem óbvia — ordenar a lista inteira e pegar os *k* primeiros — seria cara, e escreveríamos uma função especial que encontra os valores em uma única passagem. Em Haskell, a abordagem "ordene e pegue" funciona bem:

```haskell
minima k xs = take k (sort xs)
```

A avaliação preguiçosa garante que a lista só será ordenada *o suficiente* para encontrar os *k* menores elementos. O código resultante é limpo, pequeno e eficiente.

---

## 💪 Por que Programação Funcional? Por que Haskell?

Além dos pilares conceituais, vale entender os argumentos práticos que motivam o estudo do paradigma — este livro os revisitará constantemente.

**Código puro é mais fácil de testar.** Quando uma função responde apenas às suas entradas visíveis, podemos afirmar propriedades do seu comportamento que devem ser *sempre* verdadeiras — e testá-las automaticamente contra milhares de entradas aleatórias (é exatamente o que faremos com o QuickCheck na Unidade 2). Como há muito menos código impuro do que haveria em uma linguagem tradicional, ganhamos muito mais confiança na consistência do software.

**Concisão comparável às linguagens dinâmicas, com segurança estática.** Comparado às linguagens estáticas tradicionais (C, Java), o sistema de tipos de Haskell é mais flexível e, graças à inferência de tipos, quase não exige anotações — reduzindo a redundância do código. Comparado às linguagens dinâmicas (Python, Ruby), Haskell oferece concisão semelhante, mas o compilador prova a ausência de erros de tipo antes da execução: um programa Haskell que compila dificilmente sofre de erros triviais em produção. É a escolha entre a perspectiva de **segurança** que Haskell enfatiza e a de **flexibilidade** que a tipagem dinâmica favorece.

**Uso real na indústria.** Haskell e as demais linguagens funcionais deste curso saíram há muito da academia: bancos usam Haskell para medir risco de carteiras de derivativos; empresas de biotecnologia criam modelos matemáticos com ele; o sistema de controle de versão Darcs e o próprio compilador GHC são escritos em Haskell. Clojure e Elixir, que veremos na Unidade 3, sustentam sistemas web de larga escala (Nubank, WhatsApp/Erlang, Discord).

---

## 🗺️ Roteiro do Livro e Conexão de Conceitos

Este livro foi planejado como um curso integrado, unindo materiais e práticas consolidadas em diferentes semestres para oferecer uma visão completa e moderna da programação funcional. O roteiro de aprendizado divide-se em três partes:

1. **Unidade 1: Paradigmas e Haskell Básico (Fundamentos Matemáticos)**
   Aqui estabelecemos os alicerces teóricos e a sintaxe de Haskell. Ao trabalhar com uma linguagem funcional pura e estaticamente tipada, você aprenderá a pensar em termos de imutabilidade, casamento de padrões, recursão e funções de alta ordem. É o treinamento conceitual mais rigoroso.
   
2. **Unidade 2: Haskell Avançado e Qualidade de Código (Prática Industrial)**
   Subimos o nível para o desenvolvimento de software modular. Unificamos o clássico *Real World Haskell* com as ferramentas modernas do compilador GHC (através do **Cabal**). Você aprenderá a modelar uma biblioteca de dados JSON e a garantir sua correção matemática usando testes baseados em propriedades com o **QuickCheck**, unindo teoria e engenharia de software de alta qualidade — e verá duas aplicações mais avançadas do paradigma funcional, em construção de interpretadores e em processamento de dados geoespaciais.
   
3. **Unidade 3: Lisp (Clojure) e Atores (Elixir) (A Diversidade do Paradigma)**
   Após dominar o rigor estático e puro de Haskell, exploramos dois mundos fundamentais na indústria:
   * **Clojure**: A flexibilidade e poder de metaprogramação de um Lisp dinâmico e hospedado na JVM, culminando no desenvolvimento de uma aplicação web ToDo List.
   * **Elixir**: A reatividade de tempo real e tolerância a falhas do modelo de Atores na máquina virtual BEAM, culminando em uma aplicação ToDo List interativa com Phoenix LiveView.

Ao final desta jornada, você terá saído de conceitos puros do Cálculo Lambda até a criação de sistemas web concorrentes e tolerantes a falhas, dominando os principais pilares da programação funcional contemporânea.

---

> **Nota de atribuição:** partes deste capítulo adaptam material de *Real World Haskell*, de Bryan O'Sullivan, Don Stewart e John Goerzen ([book.realworldhaskell.org](http://book.realworldhaskell.org/read/)), sob a licença [Creative Commons Attribution-Noncommercial 3.0](http://creativecommons.org/licenses/by-nc/3.0/).

