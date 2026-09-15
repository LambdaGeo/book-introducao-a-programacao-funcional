# Aplicações Avançadas: Interpretadores

Os capítulos anteriores desta unidade tratam de Haskell como linguagem para *aplicações* — bibliotecas, testes, programas interativos. Mas o paradigma funcional também é, historicamente, a ferramenta de escolha para construir **linguagens**: parsers, interpretadores e compiladores. Álgebra de tipos, recursão estrutural e funções de alta ordem — os mesmos ingredientes dos capítulos anteriores — são exatamente o que se precisa para representar e processar programas como dados.

Este capítulo ilustra essa aplicação avançada com dois projetos pequenos e completos:

1. **[PlayReg](https://github.com/LambdaGeo/playreg)** — um motor de casamento de expressões regulares construído sobre **derivadas de Brzozowski**, generalizado para casamento *ponderado* via semirings.
2. **[scheme-hs](https://github.com/LambdaGeo/scheme-hs)** — um interpretador de **Scheme** (um dialeto de Lisp), com parser combinators, variáveis mutáveis, *closures* e um REPL.

---

## 1. PlayReg: expressões regulares como tipo algébrico

### 1.1 A ideia: regex é um ADT

A base do PlayReg é reconhecer que uma expressão regular **é** um tipo algébrico de dados — exatamente a construção que vimos no capítulo de [Tipos Algébricos](01_declarando_tipos_classes.md):

```haskell
data Reg
  = Eps           -- ε: casa a string vazia
  | Sym Char      -- a: casa exatamente um caractere
  | Alt Reg Reg   -- α|β: alternativa
  | Seq Reg Reg   -- αβ: sequência (concatenação)
  | Rep Reg       -- α*: repetição (zero ou mais vezes)
```

Cada construtor de `Reg` corresponde a um operador clássico de expressões regulares. Um casador de padrões "por força bruta" é uma função recursiva direta sobre esse tipo — para cada forma de `Reg`, uma regra de casamento:

```haskell
accept :: Reg -> String -> Bool
accept Eps u       = null u
accept (Sym c) u   = u == [c]
accept (Alt p q) u = accept p u || accept q u
accept (Seq p q) u = or [accept p u1 && accept q u2 | (u1, u2) <- split' u]
accept (Rep r) u   = or [and [accept r ui | ui <- ps] | ps <- parts u]
```

Essa implementação é didaticamente transparente, mas **exponencial**: para casar `Seq`, ela testa todas as formas de dividir a string em duas partes; para `Rep`, todas as formas de particioná-la. Funciona para exemplos pequenos, mas não escala.

### 1.2 Casamento eficiente: derivadas de Brzozowski

A alternativa eficiente vem de uma ideia de 1964 (Janusz Brzozowski): a **derivada** de uma expressão regular `r` em relação a um caractere `c`, escrita `∂c(r)`, é uma nova expressão regular que casa exatamente as strings que `r` casaria *depois* de consumir o caractere `c`. Aplicando a derivada caractere a caractere ao longo da entrada, o casamento vira uma dobra (`fold`) — linear no tamanho da entrada, sem backtracking.

O PlayReg implementa isso com um tipo `REG` que carrega, em cada símbolo, um `Bool` indicando se aquele símbolo é a marca de "aceito" (`final`):

```haskell
data REG
  = EPS
  | SYM Bool Char
  | ALT REG REG
  | SEQ REG REG
  | REP REG

shift :: Bool -> REG -> Char -> REG
shift _ EPS _       = EPS
shift m (SYM _ x) c = SYM (m && x == c) x
shift m (ALT p q) c = ALT (shift m p c) (shift m q c)
shift m (SEQ p q) c =
  SEQ (shift m p c)
      (shift (m && empty p || final p) q c)
shift m (REP r) c = REP (shift (m || final r) r c)

match :: REG -> String -> Bool
match r []       = empty r
match r (c : cs) = final (foldl (shift False) (shift True r c) cs)
```

`empty r` diz se `r` casa a string vazia; `final r` diz se, no estado atual (depois de já ter "andado" por alguns caracteres), a expressão está numa posição de aceitação. `shift` avança um caractere, atualizando essas marcas — é a derivada, espalhada pela estrutura da árvore.

### 1.3 Generalizando com semirings: contar, e não só aceitar/rejeitar

A parte mais interessante do projeto é notar que `Bool` (aceita/rejeita) é só **um** dos possíveis resultados de um casamento. Se em vez de `Bool` usarmos outro tipo com as mesmas duas operações que `Bool` tem sob "ou" (`||`) e "e" (`&&`) — um **semiring** —, a mesma lógica de derivadas serve para calcular outras coisas:

```haskell
class Semiring s where
  zero, one    :: s
  (<+>), (<.>) :: s -> s -> s

instance Semiring Bool where
  zero = False; one = True
  (<+>) = (||); (<.>) = (&&)

instance Semiring Int where
  zero = 0; one = 1
  (<+>) = (+); (<.>) = (*)
```

Com `Semiring Int`, a mesma função de casamento (generalizada para `REGw c s`, parametrizada pelo semiring `s`) não diz só "casa ou não casa" — ela **conta quantas derivações diferentes** levam ao casamento. E com dois semirings customizados do projeto, `Leftmost` e `LeftLong`, a mesma lógica encontra a **posição** do casamento mais à esquerda (ou mais à esquerda *e* mais longo) dentro de uma string maior — a base de como um `grep` de verdade funciona por baixo dos panos:

```haskell
submatchw ab "xxabaxx" :: LeftLong
-- LeftLong (Range 2 4)   -- achou "aba" nas posições 2 a 4
```

!!! info "Onde ler mais"
    A técnica de generalizar casamento de regex via semirings é o tema do artigo curto *["A Play on Regular Expressions"](https://dl.acm.org/doi/10.1145/1863543.1863594)* (Fischer, Huch e Wilke), um *functional pearl* — um estilo de artigo que a comunidade Haskell usa para mostrar como um problema clássico fica elegante quando resolvido com os tipos certos. O código completo, testado e documentado, está em **[github.com/LambdaGeo/playreg](https://github.com/LambdaGeo/playreg)**.

---

## 2. scheme-hs: um interpretador via parser combinators

### 2.1 Por que Scheme?

Scheme (um dialeto de Lisp) é a linguagem clássica para aprender a escrever interpretadores: sua sintaxe é a própria estrutura de dados que o interpretador manipula (código Lisp é literalmente uma árvore de listas — a propriedade chamada *homoiconicidade*). O `scheme-hs` segue o tutorial ["Write Yourself a Scheme in 48 Hours"](https://en.wikibooks.org/wiki/Write_Yourself_a_Scheme_in_48_Hours), construindo o interpretador em camadas: primeiro o parser, depois um avaliador cada vez mais completo.

### 2.2 Parsers como funções: a biblioteca Parsec

O parser do `scheme-hs` é escrito com a biblioteca **Parsec** — parsers em Haskell são, literalmente, **valores funcionais**: um `Parser a` é (por trás da API) uma função de `String` para "um `a` e o resto da string não consumida", e parsers pequenos se combinam em parsers maiores com operadores como `<|>` (alternativa) e a notação `do` (sequenciamento), do mesmo jeito que combinamos qualquer outra função em Haskell:

```haskell
parseAtom :: Parser LispVal
parseAtom = do
  first <- letter <|> symbol
  rest  <- many (letter <|> digit <|> symbol)
  let atom = first : rest
  return $ case atom of
    "#t" -> Bool True
    "#f" -> Bool False
    _    -> Atom atom

parseExpr :: Parser LispVal
parseExpr =
  parseAtom
    <|> parseString
    <|> parseNumber
    <|> parseQuoted
    <|> do
      _ <- char '('
      x <- try parseList <|> parseDottedList
      _ <- char ')'
      return x
```

Repare que o parser inteiro é construído combinando pedaços pequenos (`parseAtom`, `parseString`, `parseList`...) — o mesmo princípio de composição que já apareceu nas funções de alta ordem da Unidade 1. Todos os valores que o parser produz têm o mesmo tipo, `LispVal`: um ADT que representa qualquer expressão Scheme válida.

```haskell
data LispVal
  = Atom String
  | List [LispVal]
  | DottedList [LispVal] LispVal
  | Number Integer
  | String String
  | Bool Bool
```

### 2.3 O avaliador: de puro a `IO`

A primeira versão de um avaliador Scheme pode ser uma função pura: `eval :: LispVal -> LispVal`. Mas assim que o interpretador precisa de **variáveis** (`define`, `set!`), a pureza esbarra num problema real: uma variável definida numa expressão precisa ficar visível nas próximas — é *estado que muda*. A solução do `scheme-hs` é o mesmo padrão do capítulo de [Programas Interativos](02_programas_interativos.md): isolar o estado mutável atrás do tipo `IO`, usando `IORef` para o ambiente de variáveis:

```haskell
type Env = IORef [(String, IORef LispVal)]

eval :: Env -> LispVal -> IOThrowsError LispVal
eval env (List [Atom "if", predicate, conseq, alt]) = do
  result <- eval env predicate
  case result of
    Bool False -> eval env alt
    _          -> eval env conseq
eval env (List [Atom "define", Atom var, form]) =
  eval env form >>= defineVar env var
eval env (List (Atom "lambda" : List ps : bodyExprs)) =
  makeNormalFunc env ps bodyExprs
```

O resultado mais interessante disso são as ***closures***: uma função Scheme criada com `lambda` guarda uma referência para o ambiente onde foi criada (`closure`), então consegue "lembrar" de variáveis que já saíram de escopo:

```scheme
(define (make-counter)
  (define n 0)
  (lambda () (begin (set! n (+ n 1)) n)))
(define c1 (make-counter))
(c1)  ; => 1
(c1)  ; => 2
(c1)  ; => 3
```

Cada chamada de `(c1)` enxerga e modifica o *mesmo* `n`, privado daquela closure — nenhuma outra parte do programa consegue acessá-lo diretamente. É o mesmo conceito de encapsulamento de outras linguagens (um atributo privado de objeto), só que construído a partir de peças mais simples: um ambiente mutável e uma função que o captura.

!!! info "Onde ler mais"
    O código completo — parser, avaliador, REPL, e uma suíte de testes cobrindo aritmética, condicionais, variáveis, funções recursivas e closures — está em **[github.com/LambdaGeo/scheme-hs](https://github.com/LambdaGeo/scheme-hs)**. O README do repositório documenta exatamente o que está implementado (e o que falta, como `let` e macros) e como compilar e rodar.
