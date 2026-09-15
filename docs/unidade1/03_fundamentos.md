# Fundamentos do Haskell e o Interpretador GHCi

Neste capítulo, daremos os primeiros passos práticos na linguagem Haskell. Em vez de focarmos em programas completos e complexos, utilizaremos o ambiente interativo **GHCi** para conversar com o compilador em tempo real. Esta abordagem de desenvolvimento orientada a feedback facilitará a compreensão dos fundamentos da sintaxe, regras de avaliação e a transição mental de paradigmas.

---

## 1. A Mudança de Paradigma: Mutação vs. Definição

Para quem vem de linguagens imperativas (como Python, Java ou C), a primeira barreira no aprendizado do paradigma funcional é a forma como lidamos com a atribuição de valores. Considere o seguinte código comum em linguagens imperativas:

```python
# Em Python
x = 0
x = x + 1
```

Em Python, a instrução `x = x + 1` significa: *"Pegue o valor atual na célula de memória referenciada por `x`, adicione 1 a ele, e salve o novo valor de volta na mesma célula de memória"*.

Em Haskell, se tentarmos escrever algo equivalente:

```haskell
-- Em Haskell
x = x + 1
```

O compilador interpretará isso como uma **definição matemática**: *"x é igual a x + 1"*. Em termos matemáticos, a equação $x = x + 1$ não possui solução finita. Se tentarmos avaliar o valor de `x` no interpretador GHCi, o Haskell (devido à sua avaliação preguiçosa) substituirá recursivamente `x` por seu próprio corpo indefinidamente:

```haskell
x = (x + 1) + 1
x = ((x + 1) + 1) + 1
-- ... causando uma recursão infinita e estourando a pilha de execução (stack overflow)
```

Da mesma forma, se definirmos:

```haskell
x = 0
x = 1
```

O Haskell rejeitará o código com um erro de compilação, acusando múltiplas definições conflitantes para o mesmo identificador `x`. 

!!! info
    Em Haskell, identificadores são **constantes matemáticas imutáveis**, e não caixas de memória que podem mudar de valor ao longo do tempo. Uma vez que definimos o valor de um identificador, ele é fixo e garantido para toda a vida útil do escopo.

---

## 2. O Interpretador GHCi: Seu Laboratório Interativo

O Glasgow Haskell Compiler (GHC) possui um console interativo chamado **GHCi**. Nele, podemos digitar expressões Haskell, avaliá-las imediatamente, inspecionar tipos de dados e depurar nosso código. Se você estiver familiarizado com o console interativo do Python (`python`) ou Ruby (`irb`), o GHCi desempenha um papel semelhante.

### Comandos Essenciais do GHCi
Todos os comandos específicos do GHCi começam com um caractere de dois pontos (`:`):

| Comando | Atalho | Descrição |
| :--- | :--- | :--- |
| `:load <arquivo>` | `:l` | Carrega um arquivo de código-fonte Haskell no REPL. |
| `:reload` | `:r` | Recarrega todos os arquivos atualmente abertos (útil após edições). |
| `:type <expressão>` | `:t` | Inspeciona o tipo de dados de uma expressão ou função. |
| `:info <nome>` | `:i` | Exibe informações detalhadas sobre um identificador, operador ou classe de tipos. |
| `:quit` | `:q` | Sai do interpretador GHCi. |
| `:module +<Mod>` | `:m` | Carrega um módulo adicional da biblioteca (ex: `:m +Data.List`). |
| `:set +t` | | Passa a exibir o tipo de cada expressão avaliada (`:unset +t` desativa). |
| `:?` | | Abre o menu de ajuda interativa detalhada. |

### A variável especial `it`
O GHCi guarda o resultado da última expressão avaliada em uma variável especial chamada **`it`**. Isso permite usar o resultado anterior na próxima expressão:

```haskell
Prelude> "foo"
"foo"
Prelude> it ++ "bar"
"foobar"
```

Se a avaliação de uma expressão falhar, o valor de `it` não muda — então podemos experimentar expressões potencialmente inválidas com segurança. Combinando o `it` com as setas do teclado (que recuperam e editam as linhas anteriores), ganhamos uma ótima forma de experimentação interativa: o custo de errar é baixíssimo. Aproveite para cometer erros baratos e abundantes enquanto explora a linguagem!

!!! tip
    **Permaneça sem medo diante das mensagens de erro.** As mensagens do GHC podem parecer longas e intimidadoras no início (`No instance for (Num Bool)...`), mas elas têm uma finalidade: apontam a localização exata do problema e frequentemente sugerem uma correção. Elas nos fazem executar uma certa quantidade de depuração *antecipada*, antes mesmo de rodar o programa. No começo, descubra apenas o suficiente para progredir; com a experiência, as partes obscuras das mensagens se tornarão naturais.

---

## 3. Aritmética Básica no GHCi

Ao abrir o GHCi executando `ghci` no seu terminal (ou `cabal repl`, se estiver dentro de um projeto Cabal — veremos isso no próximo capítulo), você será recebido pelo prompt `Prelude>` (o módulo base de funções carregadas por padrão). Podemos usar o interpretador diretamente como uma calculadora de alta precisão:

```haskell
Prelude> 2 + 15
17
Prelude> 49 * 100
4900
Prelude> 1892 - 1472
420
Prelude> 5 / 2
2.5
```

Note que operadores aritméticos tradicionais (`+`, `-`, `*`, `/`) possuem regras de precedência padrão matemáticas (multiplicação e divisão possuem prioridade sobre soma e subtração).

### Parênteses e Números Negativos
Diferentemente de outras linguagens, o operador `-` pode ser ambíguo em Haskell. Quando queremos utilizar um número negativo em uma expressão, devemos **obrigatoriamente envolvê-lo entre parênteses**:

```haskell
Prelude> 5 + -3
-- ERRO! O compilador tentará aplicar o operador "+" e o operador "-" consecutivamente.

Prelude> 5 + (-3)
2
```

Envolver os números negativos em parênteses evita que o compilador confunda o sinal do número com a aplicação do operador de subtração.

Esse tratamento incomum dos números negativos representa um *trade-off* fundamentado: Haskell permite definir novos operadores a qualquer momento (um recurso que usaremos bastante), e os projetistas da linguagem aceitaram uma sintaxe um pouco mais pesada para números negativos em troca desse poder expressivo.

### Precedência e Associatividade dos Operadores
Haskell atribui valores numéricos de precedência aos operadores, de 1 (menor) a 9 (maior). Um operador de maior precedência é aplicado antes de um de menor precedência. Podemos inspecionar a precedência de qualquer operador no GHCi com o comando `:info`:

```haskell
Prelude> :info (+)
...
infixl 6 +
Prelude> :info (*)
...
infixl 7 *
Prelude> :info (^)
...
infixr 8 ^
```

A linha `infixl 6 +` indica que `(+)` tem precedência 6 e é **associativo à esquerda** (`infixl`); já `(^)` é **associativo à direita** (`infixr`). Como `(*)` tem precedência 7, maior que a do `(+)`, a expressão `1 + 4 * 4` é avaliada como `1 + (4 * 4)`.

!!! tip
    Não é necessário memorizar as regras de precedência: na dúvida, adicione parênteses. Expressões complexas que dependem totalmente da precedência dos operadores são fontes notórias de bugs — a presença de alguns parênteses ajuda os futuros leitores (incluindo você mesmo) a entender a intenção.

---

## 4. Álgebra Booleana e Operadores de Comparação

O GHCi também nos permite computar expressões lógicas e relacionais. Os booleanos em Haskell são representados pelos construtores de valor `True` e `False` (sensíveis a maiúsculas/minúsculas).

### Operadores Lógicos:
* `&&` (conjunção / E lógico)
* `||` (disjunção / OU lógico)
* `not` (negação lógica - note que é uma função, não um operador simbólico)

```haskell
Prelude> True && False
False
Prelude> False || True
True
Prelude> not True
False
```

### Operadores de Comparação:
Para comparar valores numéricos ou textuais, utilizamos os operadores relacionais padrão:

* `==` (igualdade)
* `/=` (desigualdade / diferente de - diferente do tradicional `!=` de outras linguagens)
* `>`, `>=`, `<`, `<=` (maior, maior-ou-igual, menor, menor-ou-igual)

```haskell
Prelude> 5 == 5
True
Prelude> 10 /= 9
True
Prelude> 'a' > 'b'
False
```

---

## 5. Aplicação de Funções: Espaços em Vez de Parênteses

Uma das maiores diferenças sintáticas em Haskell é a **aplicação de funções**. Em matemática tradicional e linguagens tradicionais, chamamos funções passando argumentos entre parênteses e separados por vírgulas, como `f(x, y)`. 

Em Haskell, **parênteses e vírgulas não são usados para passar argumentos**. Em vez disso, aplicamos uma função simplesmente separando seu nome e seus argumentos por **espaços**:

```haskell
Prelude> min 9 10
9
Prelude> max 3 2
3
Prelude> compare 5 10
LT
```

A função `min` recebe dois argumentos numéricos e retorna o menor. A função `compare` recebe dois argumentos comparáveis e retorna um tipo chamado `Ordering` (que pode ser `LT` - Less Than, `GT` - Greater Than, ou `EQ` - Equal).

### Precedência de Aplicação de Funções
A aplicação de função em Haskell possui a **maior prioridade de todas**. Isso significa que a expressão `f x + 1` é avaliada como `(f x) + 1`, e não como `f (x + 1)`.

Se desejarmos passar o resultado de uma operação como argumento para uma função, devemos delimitar explicitamente o argumento com parênteses:

```haskell
Prelude> succ 5 + 1
7
-- Avaliado como: (succ 5) + 1 = 6 + 1 = 7

Prelude> succ (5 + 1)
7
-- Avaliado como: succ 6 = 7
```

---

## 6. Definições Locais: `let` e `where`

Quando escrevemos programas funcionais reais, frequentemente precisamos declarar constantes auxiliares ou subdividir problemas complexos em pequenos blocos nomeados. Em Haskell, fazemos isso por meio de dois construtores de escopo local: **`let`** e **`where`**.

### 1. Cláusulas `let ... in ...`
A estrutura `let` define ligações locais que podem ser referenciadas apenas dentro da expressão demarcada pelo bloco `in`. É uma expressão declarativa e pode ser usada em qualquer lugar onde uma expressão normal seja válida:

```haskell
calcularRetangulo :: Float -> Float -> Float
calcularRetangulo largura altura = 
    let area = largura * altura
        perimetro = 2 * (largura + altura)
    in area + perimetro
```

### 2. Cláusulas `where`
A cláusula `where` é anexada no final de uma definição de função e permite declarar variáveis locais visíveis para todo o escopo de equações e guardas daquela função. É altamente idiomática em Haskell por manter o corpo da função principal limpo e focado no topo:

```haskell
calcularRetanguloWhere :: Float -> Float -> Float
calcularRetanguloWhere largura altura = area + perimetro
  where
    area = largura * altura
    perimetro = 2 * (largura + altura)
```

---

## 7. Avaliação Preguiçosa na Prática: Substituição e *Thunks*

Como o Haskell avalia uma expressão como `isOdd (1 + 2)`, onde:

```haskell
isOdd n = mod n 2 == 1
```

Em uma linguagem de avaliação **estrita** (C, Python, Java), os argumentos são avaliados *antes* da função ser aplicada: primeiro `1 + 2` viraria `3`, depois `isOdd` seria chamada com `3`.

Haskell escolhe outro caminho: a avaliação **não-estrita** (preguiçosa). A subexpressão `1 + 2` *não* é reduzida imediatamente para `3`. Em vez disso, é criada uma "promessa" de que, quando o valor for realmente necessário, ele será calculado. O registro usado para rastrear essa expressão não avaliada é chamado de **thunk**. Se o resultado nunca for usado, o cálculo nunca acontece.

Uma consequência elegante: operadores de "curto-circuito" não precisam de suporte especial da linguagem. Em Haskell, `(||)` é uma função comum — se o operando esquerdo avaliar para `True`, o direito simplesmente nunca é avaliado:

```haskell
meuOu :: Bool -> Bool -> Bool
meuOu a b = if a then a else b
```

A expressão `meuOu True (length [1..] > 0)` retorna `True` sem travar, mesmo com uma lista infinita no segundo argumento — algo impossível de escrever como função comum em uma linguagem estrita.

Um bom modelo mental para entender a avaliação em Haskell é a **substituição e reescrita**: substitua cada nome pela sua definição, repetidamente, avaliando apenas o suficiente de cada expressão para determinar o valor final.

No próximo capítulo, exploraremos como o sistema de tipos estáticos do Haskell garante que essas expressões operem de forma segura e otimizada.

---

> **Nota de atribuição:** partes deste capítulo adaptam material de *Real World Haskell*, de Bryan O'Sullivan, Don Stewart e John Goerzen ([book.realworldhaskell.org](http://book.realworldhaskell.org/read/)), sob a licença [Creative Commons Attribution-Noncommercial 3.0](http://creativecommons.org/licenses/by-nc/3.0/).