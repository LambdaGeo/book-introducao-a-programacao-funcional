# Entrada/Saída (I/O) e Programação Interativa

Até este ponto, focamos puramente em expressões funcionais que recebem dados e computam saídas sem qualquer efeito colateral. No entanto, um programa útil na vida real precisa interagir com o mundo externo: ler dados do teclado, gravar arquivos em disco ou realizar requisições de rede. Neste capítulo, estudaremos como o Haskell resolve o paradoxo de interagir com o mundo impuro de forma segura através da **Monad de IO** e da **notação `do`**.

---

## 1. O Paradoxo do I/O na Programação Funcional Pura

Se o Haskell é uma linguagem funcional pura baseada em determinismo matemático (onde a mesma função com os mesmos argumentos sempre retorna o mesmo valor), como podemos implementar uma função como `lerTeclado`? Se o usuário digita coisas diferentes a cada execução, essa função seria impura e violaria as garantias de otimização e avaliação preguiçosa do compilador.

Para resolver este paradoxo, o Haskell separa rigorosamente o mundo das **expressões puras** do mundo das **ações de I/O**.

```text
  ┌────────────────────────────────┐
  │      Mundo Puro (Haskell)      │
  │  - Sem efeitos colaterais      │
  │  - Determinismo absoluto       │
  └───────────────┬────────────────┘
                  │  (Encapsulamento via Tipo IO)
                  ▼
  ┌────────────────────────────────┐
  │       Mundo Impuro (I/O)       │
  │  - Leitura/Escrita de Arquivos │
  │  - Interação com Teclado/Tela  │
  └────────────────────────────────┘
```

---

## 2. A Solução: O Tipo `IO a`

Em Haskell, uma ação que interage com o mundo externo tem o tipo **`IO a`**. Isso significa: *"uma receita/ação que, quando executada pelo sistema operacional, realizará efeitos colaterais e retornará um valor do tipo `a`"*.

* **`IO Char`**: Uma ação que realiza I/O e retorna um caractere (ex: `getChar`).
* **`IO ()`**: Uma ação que realiza I/O mas não retorna nenhum valor útil (representado pelo tipo unitário `()`, semelhante ao `void` de outras linguagens).

!!! info
    Existe uma diferença crucial entre o tipo `String` e o tipo `IO String`. Um valor do tipo `String` é apenas texto puro que pode ser avaliado com segurança. Um valor do tipo `IO String` é uma **ação pendente** (como ler uma linha do teclado) que só produzirá a string quando for executada. Você nunca pode "extrair" um valor de `IO` para o mundo puro sem que a função inteira também se torne uma ação de `IO`.

---

## 3. Ações Básicas de I/O

A biblioteca padrão do Haskell fornece as seguintes ações primitivas de I/O:

* **`getChar :: IO Char`**: Lê um único caractere do teclado.
* **`putChar :: Char -> IO ()`**: Escreve um único caractere na tela.
* **`getLine :: IO String`**: Lê uma linha inteira de texto do teclado.
* **`putStrLn :: String -> IO ()`**: Escreve uma string na tela seguida por uma nova linha.

---

## 4. Sequenciando Ações com a Notação `do`

Para realizar várias ações de I/O em sequência, utilizamos o construtor sintático **`do`**. Ele nos permite encadear ações de forma linear, assemelhando-se à programação imperativa:

```haskell
interagir :: IO ()
interagir = do
  putStrLn "Qual eh o seu nome?"
  nome <- getLine
  putStrLn ("Bem-vindo ao Haskell, " ++ nome ++ "!")
```

### O Operador de Atribuição de I/O (`<-`)
Note o uso do símbolo **`<-`**. Ele serve para extrair o valor puro produzido por uma ação de `IO` e vinculá-lo a um nome local (neste caso, extraindo a `String` de dentro de `IO String` retornada por `getLine`).

### O Significado de `return` em Haskell
Diferentemente de linguagens imperativas, onde `return` é um comando de controle que interrompe a execução de uma função, em Haskell o **`return` é uma função comum**. 

A função `return :: a -> IO a` pega um valor puro de tipo `a` e o envelopa em uma ação de `IO` que não realiza nenhum efeito colateral:

```haskell
pedirConfirmacao :: IO Bool
pedirConfirmacao = do
  putStrLn "Deseja continuar? (s/n)"
  resposta <- getChar
  if resposta == 's' 
    then return True 
    else return False
```

### Por Baixo do `do`: os Operadores `>>=` e `>>`
A notação `do` é apenas açúcar sintático. Por baixo, as ações são encadeadas com dois operadores:

* **`(>>=)`** (pronunciado *bind*): executa a ação da esquerda, extrai seu resultado e o passa para a função da direita. Tipo: `IO a -> (a -> IO b) -> IO b`.
* **`(>>)`**: executa a ação da esquerda, descarta o resultado e executa a da direita.

O programa `interagir` acima é equivalente a:

```haskell
interagir :: IO ()
interagir =
  putStrLn "Qual eh o seu nome?" >>
  getLine >>= \nome ->
  putStrLn ("Bem-vindo ao Haskell, " ++ nome ++ "!")
```

Cada linha `x <- acao` do bloco `do` vira `acao >>= \x -> ...`, e cada ação sem `<-` vira um `>>`. Saber essa correspondência ajuda a ler código Haskell mais avançado e a entender o que a monad `IO` realmente faz.

---

## 5. Manipulação de Arquivos

Além de ler e escrever no console, o Haskell fornece ações básicas para ler e gravar arquivos em disco:

* **`readFile :: FilePath -> IO String`**: Abre e lê todo o conteúdo de um arquivo de forma preguiçosa.
* **`writeFile :: FilePath -> String -> IO ()`**: Grava uma string em um arquivo (sobrescrevendo o conteúdo existente).
* **`appendFile :: FilePath -> String -> IO ()`**: Anexa uma string no final de um arquivo existente.

Exemplo de um programa que lê um arquivo e grava seu conteúdo em maiúsculas em outro arquivo:

```haskell
import Data.Char (toUpper)

converterArquivo :: FilePath -> FilePath -> IO ()
converterArquivo origem destino = do
  conteudo <- readFile origem
  let conteudoMaiusculo = map toUpper conteudo
  writeFile destino conteudoMaiusculo
  putStrLn "Arquivo processado com sucesso!"
```

Note o padrão típico dos programas Haskell: a ação impura (`readFile`/`writeFile`) fica nas bordas, enquanto a transformação em si (`map toUpper`) é uma função **pura**, ligada com `let` — fácil de testar isoladamente.

---

## 6. O Padrão `interact`: Filtros de Texto em Uma Linha

Para programas do estilo "lê da entrada padrão, transforma, escreve na saída" (filtros Unix), o Haskell oferece a função `interact :: (String -> String) -> IO ()`, que recebe uma **função pura** de transformação e cuida de todo o I/O. Um contador de linhas completo:

```haskell
-- WC.hs
main = interact wordCount
    where wordCount input = show (length (lines input)) ++ "\n"
```

```bash
$ runghc WC < arquivo.txt
7
```

Graças à avaliação preguiçosa, o `interact` processa a entrada de forma incremental — funciona até com entradas maiores que a memória. Esse é o desenho recomendado para programas Haskell: **concentre a lógica em funções puras e reduza o código impuro ao mínimo indispensável.** Grande parte do risco em software está na comunicação com o mundo exterior; como o sistema de tipos nos diz exatamente quais partes do código têm efeitos colaterais, a "superfície de ataque" fica pequena e bem vigiada.

---

## 7. Argumentos de Linha de Comando e o Padrão "Núcleo Puro, Casca Impura"

Programas reais frequentemente processam arquivos indicados na linha de comando, em vez de usar a entrada/saída padrão. O módulo `System.Environment` fornece `getArgs :: IO [String]`, que lê os argumentos passados ao executável. Um pequeno *framework* reutilizável para "ler um arquivo, transformar, gravar em outro arquivo":

```haskell
import System.Environment (getArgs)

interactWith :: (String -> String) -> FilePath -> FilePath -> IO ()
interactWith function inputFile outputFile = do
  input <- readFile inputFile
  writeFile outputFile (function input)

main :: IO ()
main = mainWith minhaFuncao
  where
    mainWith function = do
      args <- getArgs
      case args of
        [entrada, saida] -> interactWith function entrada saida
        _ -> putStrLn "erro: são necessários exatamente dois argumentos"

    -- troque "id" pela função que você quiser testar
    minhaFuncao = id
```

Compilando e executando (num projeto Cabal, este seria o conteúdo de `app/Main.hs`):

```bash
$ cabal build && cabal run meu-programa -- entrada.txt saida.txt
```

O ponto pedagógico é o **design**, não a sintaxe nova: `interactWith` é o "framework" fixo (I/O), e `minhaFuncao :: String -> String` é a única peça que muda de programa para programa — e é **pura**, portanto testável no GHCi sem tocar em disco algum. Esse é o mesmo padrão do capítulo de [funções de alta ordem](../unidade1/07_alta_ordem.md): isolar a lógica de negócio da mecânica de E/S.

No próximo capítulo, começamos a construir uma biblioteca completa de manipulação de JSON, aplicando os tipos algébricos e a IO deste capítulo a um projeto real.

---

## 📓 Lista 5: Entrada/Saída (IO) e Programação Interativa

Exercícios de fixação sobre os conceitos deste capítulo.

### Parte 1 – Fundamentos de IO

1. Escreva um programa que leia um caractere do teclado e o imprima duas vezes na tela.

    ```haskell
    -- exemplo de execução
    > a
    aa

    ```

2. Crie um programa que leia dois caracteres e os imprima separados por um espaço.

    ```haskell
    > a
    > b
    a b

    ```

3. Escreva uma versão simplificada do `putStrLn`, chamada `putStrSimples`, que recebe uma string e a imprime caractere por caractere, finalizando com `\n`.

---

### Parte 2 – Lendo e escrevendo Strings

1. Implemente uma função que peça o nome do usuário e o cumprimente.

    ```haskell
    > Qual eh o seu nome?
    Sergio
    Bem vindo Sergio!

    ```

2. Escreva um programa que leia duas linhas do teclado e depois imprima a concatenação delas.

---

### Parte 3 – Do notation

1. Reescreva o seguinte programa sem usar `do` (usando `>>=`):

    ```haskell
    main = do
      nome <- getLine
      putStrLn ("Oi, " ++ nome)

    ```

2. Escreva um programa que peça a idade do usuário e diga se ele é maior de idade ou não.

---

### Parte 4 – Manipulação de arquivos

1. Escreva um programa que leia o conteúdo de um arquivo `"entrada.txt"` e imprima na tela.
2. Crie uma função que copie o conteúdo de `"entrada.txt"` para `"saida.txt"`.
3. Usando o padrão `interactWith` deste capítulo, escreva uma função `removerStopwords :: String -> String` que remove de um texto as palavras "e", "de", "a" e "o" (uma por linha na saída), e adapte `interactWith` para que o nome dos arquivos de entrada e saída seja pedido ao usuário via teclado, em vez de vir da linha de comando.

---

### Parte 5 – Desafios

1. Implemente um programa que conte quantas linhas existem em um arquivo de texto.
2. Escreva uma função `main` que:
    - pergunte ao usuário um número `n`,
    - leia uma lista de números de um arquivo,
    - some apenas os `n` primeiros números,
    - grave o resultado em outro arquivo.
3. Escreva um mini "chat": o programa deve ler uma linha do usuário e imprimir de volta a mesma linha em maiúsculas até que ele digite `"sair"`.

---

> **Nota de atribuição:** partes deste capítulo adaptam material de *Real World Haskell*, de Bryan O'Sullivan, Don Stewart e John Goerzen ([book.realworldhaskell.org](http://book.realworldhaskell.org/read/)), sob a licença [Creative Commons Attribution-Noncommercial 3.0](http://creativecommons.org/licenses/by-nc/3.0/).