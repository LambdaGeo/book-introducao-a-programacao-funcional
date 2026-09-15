# Declaração de Tipos Algébricos e Classes de Tipos (ADTs)

No desenvolvimento de sistemas complexos, a capacidade de modelar o domínio do problema por meio de tipos precisos é uma das maiores vantagens do Haskell. Neste capítulo, estudaremos como estender o sistema de tipos definindo **Sinônimos de Tipos**, **Tipos Algébricos de Dados (ADTs)**, **Sintaxe de Registro (Records)** e como criar ou estender **Classes de Tipos** (Typeclasses).

---

## 1. Sinônimos de Tipos: Criando Apelidos (`type`)

A forma mais simples de definir um novo tipo em Haskell é por meio de um **Sinônimo de Tipo** utilizando a palavra-chave `type`. Um sinônimo funciona apenas como um apelido legível para um tipo existente, facilitando a documentação e legibilidade do código:

```haskell
type Coordenada = (Double, Double)
type ID = Int
type Pessoa = (ID, String, Coordenada)
```

Os sinônimos de tipo podem aceitar parâmetros de tipo, funcionando de forma genérica:

```haskell
type Par a = (a, a)
type ListaAssoc k v = [(k, v)]
```

!!! warning
    Como os sinônimos de tipo são apenas "apelidos", o compilador não impede que você passe um par comum `(Double, Double)` para uma função que espera uma `Coordenada`. Para obter isolamento completo e segurança de tipos em tempo de compilação, devemos utilizar `newtype` ou `data`.

    Além disso, sinônimos de tipo **não podem ser recursivos** (por exemplo, tentar declarar `type Arvore = (Int, [Arvore])` resultará em erro).

---

## 2. Tipos Novos com Isolamento: `newtype`

Quando queremos um tipo **distinto** (não apenas um apelido) que envolve exatamente **um construtor com um único campo**, usamos `newtype`:

```haskell
newtype Dolar = Dolar Float
newtype CustomerID = CustomerID Int
```

Diferentemente de `type`, o compilador trata `Dolar` e `Float` como tipos incompatíveis — impossível passar um `Float` cru onde se espera `Dolar`. Diferentemente de `data`, o `newtype` não tem custo em tempo de execução: o "envelope" existe apenas em tempo de compilação e é removido pelo compilador. É a ferramenta ideal para dar segurança de tipos a valores primitivos (IDs, moedas, unidades de medida).

---

## 3. Tipos Algébricos de Dados (ADTs) (`data`)

Para criar tipos de dados inteiramente novos com seus próprios construtores de valor, utilizamos a palavra-chave **`data`**. Um **Tipo Algébrico de Dados** é composto por uma combinação de dois padrões matemáticos: **Tipos Soma** (alternativas) e **Tipos Produto** (combinações).

```text
               ┌────────────────────────┐
               │    Tipos Algébricos    │
               │      de Dados          │
               └──────────┬─────────────┘
                          │
            ┌─────────────┴─────────────┐
            ▼                           ▼
       [Tipo Soma]                [Tipo Produto]
      (Alternativas)              (Combinações)
  Vermelho | Verde | Azul       Circulo Float | Retangulo Float Float
```

### 1. Tipos Soma (Enumerações)
Representam valores que podem assumir uma entre várias alternativas distintas:

```haskell
data Cor = Vermelho | Verde | Azul | Amarelo
data Resposta = Sim | Nao | Talvez
```

Aqui, `Vermelho`, `Verde`, `Sim` e `Nao` são chamados de **Construtores de Valor**. Eles funcionam como constantes que representam os dados concretos do tipo.

### 2. Tipos Produto
Representam valores compostos por múltiplos campos de dados combinados:

```haskell
data Forma = Circulo Float
           | Retangulo Float Float
           | Quadrado Float
```

O tipo `Forma` possui três construtores de valor. O construtor `Circulo` carrega um `Float` (o raio). O construtor `Retangulo` carrega dois valores `Float` (largura e altura). Podemos usar casamento de padrões para inspecionar e extrair esses valores:

```haskell
area :: Forma -> Float
area (Circulo r)       = pi * r ^ 2
area (Retangulo l a)   = l * a
area (Quadrado l)      = l ^ 2
```

### Analogia com Outras Linguagens
Os ADTs unificam, em um único mecanismo, várias construções que outras linguagens oferecem separadamente:

* **`struct` do C / classe de dados**: um ADT com um único construtor e vários campos agrupa valores relacionados — a diferença é que os campos em Haskell são posicionais e anônimos (a menos que usemos a sintaxe de registro, a seguir).
* **`enum` do C/Java**: um tipo soma de construtores sem argumentos (`data Cor = Vermelho | Verde | Azul`) representa valores simbólicos — mas, ao contrário do C, os construtores **não** são inteiros disfarçados, e o compilador impede misturá-los com números.
* **`union` do C**: um tipo soma com argumentos é uma "união disjunta" **segura**: o valor lembra qual construtor o criou, e o pattern matching nos obriga a tratar cada caso — em C, é o programador quem precisa rastrear manualmente qual alternativa a união contém, uma fonte clássica de bugs.

### ADTs vs. Tuplas
Dois pares `(String, String)` são sempre do mesmo tipo, mesmo que um represente um animal e outro um móvel — o sistema de tipos não pode nos proteger da mistura. Já dois ADTs estruturalmente idênticos com nomes diferentes são tipos **distintos**:

```haskell
data Cetaceo = Cetaceo String String
data Movel   = Movel String String
```

Regra prática: se você usa um valor composto amplamente no código, declare um `data` — ganha segurança de tipos e legibilidade; para usos pequenos e localizados, uma tupla resolve.

---

## 4. Sintaxe de Registro (Record Syntax)

Quando definimos tipos produto com muitos campos, acessar os valores por posição em casamento de padrões torna-se confuso e propenso a erros. Para resolver isso, Haskell fornece a **Sintaxe de Registro** (*Record Syntax*), que permite nomear os campos:

```haskell
data Usuario = Usuario 
  { nome    :: String
  , idade   :: Int
  , ativo   :: Bool
  } deriving (Show, Eq)
```

### Vantagens da Sintaxe de Registro:
1. **Funções Seletoras Automáticas**: O compilador cria automaticamente funções com o nome de cada campo para extrair o valor de um registro. Ex: `nome meuUsuario` retorna a `String` do nome.
2. **Atualização Prática de Registros**: Podemos atualizar campos facilmente por meio de uma sintaxe de cópia imutável:
   ```haskell
   aniversario :: Usuario -> Usuario
   aniversario u = u { idade = idade u + 1 }
   ```

---

## 5. Tipos Parametrizados e o `Maybe`

Assim como as listas, nossos próprios tipos podem ser **polimórficos**: basta introduzir variáveis de tipo na declaração. O exemplo mais importante do Prelude é o tipo `Maybe`, que representa um valor que pode estar presente ou ausente:

```haskell
data Maybe a = Just a
             | Nothing
```

A variável `a` indica que `Maybe` recebe outro tipo como parâmetro: `Maybe Int`, `Maybe String`, `Maybe [Bool]` são tipos distintos construídos a partir dele.

```haskell
Prelude> Just 1.5
Just 1.5
Prelude> :type Just "bicicleta"
Just "bicicleta" :: Maybe [Char]
```

### Tratando Erros: `error` vs. `Maybe`
Haskell oferece a função `error :: String -> a`, que aborta a execução imediatamente com uma mensagem. O problema: quem chama a função não consegue distinguir um erro recuperável de uma falha fatal. Compare:

```haskell
-- Versão que "explode":
meuSegundo :: [a] -> a
meuSegundo xs = if null (tail xs)
                then error "lista muito curta"
                else head (tail xs)

-- Versão total e segura com Maybe:
segundoSeguro :: [a] -> Maybe a
segundoSeguro (_:x:_) = Just x
segundoSeguro _       = Nothing
```

Na versão com `Maybe`, devolvemos `Nothing` quando a lista é curta demais — e o **chamador** decide o que fazer, em vez de o programa cair. Note também como o pattern matching `(_:x:_)` torna a versão segura mais concisa: o padrão casa apenas com listas de pelo menos dois elementos, ligando `x` ao segundo.

---

## 6. Tipos Recursivos: Modelando Estruturas de Dados

Os tipos declarados com `data` podem ser recursivos, ou seja, conter referências a si mesmos em seus campos. Isso nos permite modelar estruturas de dados complexas, como árvores binárias:

```haskell
data Arvore a = Folha
              | No a (Arvore a) (Arvore a)
              deriving (Show, Eq)
```

Uma árvore binária ou é um nó com um valor e dois filhos (que também são árvores binárias), ou é uma folha vazia. Em Java, a classe equivalente usaria `null` para indicar a ausência de um filho; em Haskell não existe `null` — usamos explicitamente o construtor `Folha`, e o compilador nos obriga a tratá-lo no pattern matching. Toda uma classe de erros (`NullPointerException`) simplesmente não existe.

Podemos processar essa árvore utilizando recursão sobre sua estrutura:

```haskell
altura :: Arvore a -> Int
altura Folha      = 0
altura (No _ e d) = 1 + max (altura e) (altura d)
```

---

## 7. Implementando Classes de Tipos Personalizadas

Vimos na Unidade 1 que classes de tipos (como `Eq` e `Ord`) governam comportamentos comuns. Haskell nos permite declarar novas classes de tipos por meio da palavra-chave `class`:

```haskell
class Serializavel a where
  serializar :: a -> String
```

Aqui, declaramos uma classe `Serializavel` que exige que qualquer tipo participante implemente a função `serializar`.

### Declarando Instâncias (`instance`)
Para fazer um tipo de dados fazer parte de uma classe de tipos, escrevemos uma declaração de **`instance`**:

```haskell
instance Serializavel Cor where
  serializar Vermelho = "#FF0000"
  serializar Verde    = "#00FF00"
  serializar Azul     = "#0000FF"
  serializar Amarelo  = "#FFFF00"
```

### Derivação Automática (`deriving`)
Para classes de tipos muito comuns da biblioteca padrão (`Show`, `Read`, `Eq`, `Ord`), o compilador do Haskell é inteligente o bastante para gerar as implementações de instância automaticamente. Basta adicionar a cláusula `deriving` no final da declaração do tipo:

```haskell
data Ponto = Ponto Double Double
  deriving (Show, Eq)
```

No próximo capítulo, veremos como utilizar o encapsulamento de tipos algébricos para lidar com a manipulação de Entrada/Saída de forma segura através da Monad de IO.

---

## 📕 Lista 4: Tipos Algébricos, Sinônimos e Pattern Matching

Exercícios de fixação sobre os conceitos deste capítulo.

### **1. Sinônimos de tipos (`type`)**

1. Crie um **sinônimo de tipo** `Nome` para `String` e `Idade` para `Int`.

    Escreva uma função `apresenta :: Nome -> Idade -> String` que retorne `"Meu nome é X e tenho Y anos"`.

2. Explique a diferença entre `type` e `newtype` em Haskell.
3. Crie um **sinônimo de tipo** `Ponto` para `(Float, Float)`.

    Escreva uma função `distancia :: Ponto -> Ponto -> Float` que calcule a distância euclidiana entre dois pontos.

---

### **2. Novos tipos (`newtype`)**

1. Crie um `newtype` chamado `Dolar` que envolve `Float`.

    Escreva uma função `converterParaReal :: Dolar -> Float` considerando 1 Dólar = 5,50 BRL.

2. Explique por que `newtype` é mais eficiente que `data` para tipos que envolvem **um único construtor com um único campo**.

---

### **3. Tipos algebricos (`data`)**

1. Defina um tipo `DiaSemana` com os sete dias da semana.

    Escreva uma função `proximoDia :: DiaSemana -> DiaSemana`.

2. Defina um tipo `Cor` com `Vermelho | Verde | Azul | Amarelo`.

    Crie uma função `ehPrimaria :: Cor -> Bool` que retorne `True` se a cor for primária.

3. Crie um tipo `Forma` com `Circulo Float | Retangulo Float Float | Quadrado Float`.

    Escreva uma função `area :: Forma -> Float` que calcule a área de qualquer forma.

4. Crie um tipo `Conta` com `Corrente Float | Poupanca Float Float` (saldo e juros).

    Escreva uma função `saldoTotal :: Conta -> Float` que retorne o saldo total considerando juros na poupança.

---

### **4. Pattern Matching**

1. Reescreva a função `area` do exercício 3.3 (tipo `Forma`) usando **pattern matching**.
2. Escreva uma função `ehFimDeSemana :: DiaSemana -> Bool` usando pattern matching.
3. Crie uma função `descricaoConta :: Conta -> String` que descreva a conta (`"Conta Corrente: R$..."` ou `"Conta Poupança: R$... com juros..."`).
4. Escreva uma função `somaPontos :: Ponto -> Ponto -> Ponto` usando pattern matching em tuplas.

---

### **5. Funções que usam tipos algebricos**

1. Crie um tipo `Resultado` com `Sucesso String | Erro String`.

    Escreva uma função `processar :: Bool -> Resultado` que retorne `Sucesso "OK"` se `True`, ou `Erro "Falha"` se `False`.

2. Crie um tipo `Expressao` para representar expressões matemáticas simples:

    `Valor Int | Soma Expressao Expressao | Multiplicacao Expressao Expressao`.

    Escreva uma função `avaliar :: Expressao -> Int` que calcule o valor da expressão.

3. Crie um tipo `Opcao` com `Sim | Nao | Talvez`.

    Escreva uma função `resposta :: Opcao -> String` que retorne `"Aceito"`, `"Recuso"` ou `"Indeciso"`.

---

### **6. Exercícios mistos avançados**

1. Crie um tipo `Pessoa` com `Nome String` e `Idade Int`.

    Escreva uma função `maiorIdade :: Pessoa -> Bool` usando pattern matching.

2. Crie um tipo `Livro` com `Titulo String | Autor String | Ano Int`.

    Escreva uma função `descricaoLivro :: Livro -> String`.

3. Defina um tipo `Arvore a` com `Folha a | No (Arvore a) (Arvore a)`.

    Escreva uma função `contaNos :: Arvore a -> Int` que conte os nós de uma árvore.

4. Escreva uma função `mapArvore :: (a -> b) -> Arvore a -> Arvore b` que aplique uma função a todos os elementos da árvore.

---

> **Nota de atribuição:** partes deste capítulo adaptam material de *Real World Haskell*, de Bryan O'Sullivan, Don Stewart e John Goerzen ([book.realworldhaskell.org](http://book.realworldhaskell.org/read/)), sob a licença [Creative Commons Attribution-Noncommercial 3.0](http://creativecommons.org/licenses/by-nc/3.0/).