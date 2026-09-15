# Instalação do Ambiente e Estrutura de um Projeto Cabal

A abordagem moderna e padrão recomendada hoje pela comunidade Haskell para instalar e gerenciar o compilador e suas ferramentas de build é o **GHCup** (Haskell Toolchain Installer). O GHCup gerencia a instalação do compilador **GHC**, do gerenciador de pacotes **Cabal**, do servidor de linguagem **HLS** (Haskell Language Server) e, opcionalmente, do Stack.

Neste livro, utilizaremos o **Cabal** como ferramenta de build e gerenciamento de projetos: é a ferramenta mantida pelo próprio time do GHC, distribuída junto com o compilador, e a recomendação oficial atual em [haskell.org](https://www.haskell.org/get-started/). Não é preciso instalar nada além do GHCup — Cabal já vem incluído.

---

## 💻 Como Instalar o Haskell com GHCup

### 1. No Linux e macOS
Abra o seu terminal e execute o comando oficial do GHCup:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```
Durante a instalação interativa:

* Pressione **Enter** para aceitar os caminhos padrão do diretório de instalação.
* Quando perguntado se deseja adicionar os caminhos ao seu `PATH` (no arquivo `.bashrc` ou `.zshrc`), responda **Yes (Y)**.
* Quando perguntado se deseja instalar o **Stack**, você pode responder **No (N)** — não vamos precisar dele neste livro.
* Quando perguntado se deseja instalar o **HLS** (Haskell Language Server, essencial para autocompletar e linting no VS Code), responda **Yes (Y)**.

Após a conclusão da instalação, reinicie o seu terminal ou execute `source ~/.bashrc` (ou seu equivalente) para carregar os caminhos de execução. Confirme que tudo está no `PATH`:
```bash
ghc --version
cabal --version
```

!!! warning "No Linux: uma biblioteca do sistema"
    O Cabal compila algumas dependências que precisam de aritmética de precisão arbitrária (GMP) para linkar. Se a compilação falhar com um erro do tipo `cannot find -lgmp`, falta o pacote de desenvolvimento do GMP no seu sistema — no Debian/Ubuntu:
    ```bash
    sudo apt install libgmp-dev
    ```
    O runtime do GHC já depende do GMP, então normalmente já está instalado; falta especificamente o pacote `-dev` com os símbolos de link.

### 2. No Windows
Abra o console do PowerShell (de preferência como Administrador) e execute o script oficial:
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://get-ghcup.haskell.org/install_haskell.ps1'))
```
Siga as instruções exibidas na tela e selecione as opções para instalar o **HLS** (o Stack, novamente, é opcional e não é necessário para este livro).

!!! tip "Se o `cabal build`/`cabal update` falhar com erro de assinatura"
    Versões de `cabal-install` muito antigas (por exemplo, as empacotadas pelo `apt` de distribuições Linux mais velhas) às vezes não conseguem validar o índice atual do Hackage, e falham com uma mensagem parecida com `<repo>/root.json does not have enough signatures signed with the appropriate keys`. Isso acontece quando o Hackage rotaciona as chaves de assinatura do índice e o `cabal-install` instalado é velho demais para reconhecer as novas. A correção é instalar um `cabal-install` atual via GHCup (como fizemos acima) em vez de depender do pacote do sistema operacional.

---

## 🛠️ O REPL: `ghci` e `cabal repl`

Fora de qualquer projeto, `ghci` sozinho abre um REPL com só a biblioteca `base` carregada — é a calculadora interativa que usamos no capítulo anterior. Dentro de um projeto Cabal (que vamos criar já já), `cabal repl` abre o mesmo REPL, mas já com os módulos e as dependências do seu projeto carregados.

## 📁 Criando um Projeto

Para a atividade deste livro, vamos criar o projeto `hs2json` (o mesmo que será o trabalho prático da Unidade 2). O Cabal tem um assistente interativo para gerar a estrutura inicial:

```bash
cabal init --interactive
```

Ele faz uma série de perguntas (nome do pacote, versão, se você quer uma biblioteca/executável/suíte de testes, licença, linguagem...). Para este livro, responda que sim para biblioteca, executável e suíte de testes. O resultado é uma árvore de diretórios como esta:

```text
hs2json/
├── app/
│   └── Main.hs          # Ponto de entrada executável (função main)
├── src/
│   └── MyLib.hs          # Código-fonte da biblioteca reutilizável
├── test/
│   └── Main.hs            # Suíte de testes automatizados
└── hs2json.cabal          # Descrição do pacote: metadados, dependências, módulos
```

### O arquivo `.cabal`

O `hs2json.cabal` é o único arquivo de configuração — sem a duplicação `package.yaml`/`stack.yaml` de outras ferramentas. Ele descreve, em seções (`library`, `executable`, `test-suite`), quais módulos cada parte do projeto expõe e de quais bibliotecas depende:

```cabal
library
    exposed-modules:  MyLib
    hs-source-dirs:   src
    build-depends:    base >=4.14
    default-language: Haskell2010

executable hs2json
    main-is:          Main.hs
    hs-source-dirs:   app
    build-depends:    base >=4.14, hs2json
    default-language: Haskell2010

test-suite hs2json-test
    type:             exitcode-stdio-1.0
    main-is:          Main.hs
    hs-source-dirs:   test
    build-depends:    base >=4.14, hs2json
    default-language: Haskell2010
```

Cada arquivo `.hs` dentro de `src/` deve declarar seu **nome de módulo** de forma correspondente ao seu caminho, e precisa estar listado em `exposed-modules` pra que o executável e os testes consigam importá-lo.

### Comandos essenciais

| Comando | Descrição |
| :--- | :--- |
| `cabal build` | Compila todo o projeto (biblioteca, executáveis e testes). |
| `cabal run` | Executa o binário principal do projeto. |
| `cabal test` | Executa a suíte de testes do projeto. |
| `cabal repl` | Abre o REPL carregando os módulos e dependências do projeto. |

```bash
cabal build
cabal run
```

A primeira vez que você rodar `cabal build` num projeto novo, ele vai buscar o índice de pacotes do Hackage (`cabal update`, se ainda não tiver rodado) e baixar as dependências — pode demorar um pouco.

---

## 🧪 Teste: adicionando uma dependência e rodando QuickCheck

Vamos ver como adicionar uma biblioteca e rodar testes com QuickCheck. Primeiro, em `src/MyLib.hs`, uma função mais interessante que a padrão — uma (propositalmente falha) implementação de quicksort:

```haskell
module MyLib
    ( someFunc
    , qsort
    ) where

qsort :: Ord a => [a] -> [a]
qsort []     = []
qsort (x:xs) = qsort lhs ++ [x] ++ qsort lhs
    where lhs = filter  (< x) xs
          rhs = filter (>= x) xs

someFunc :: IO ()
someFunc = putStrLn "someFunc"
```

Para testar essa função, vamos importar a biblioteca QuickCheck em `test/Main.hs`:

```haskell
import Test.QuickCheck

main :: IO ()
main = putStrLn "Test suite not yet implemented"
```

Rodando o teste agora:

```bash
$ cabal test
...
Main.hs:1:1: error:
    Could not find module `Test.QuickCheck'
    ...
```

O import não está disponível porque `QuickCheck` ainda não é uma dependência do projeto. Adicionamos no `.cabal`, na seção `test-suite`:

```cabal
test-suite hs2json-test
    type:             exitcode-stdio-1.0
    main-is:          Main.hs
    hs-source-dirs:   test
    build-depends:
        base >=4.14,
        hs2json,
        QuickCheck
    default-language: Haskell2010
```

Agora podemos testar novamente:

```bash
$ cabal test
...
Test suite hs2json-test: RUNNING...
Test suite not yet implemented
Test suite hs2json-test: PASS
```

Agora vamos implementar um teste de verdade para a função `qsort`: uma propriedade que qualquer boa ordenação deveria obedecer. Uma invariante útil e que aparece com frequência em código puramente funcional é a **idempotência** — aplicar a função duas vezes deve dar o mesmo resultado que aplicar uma vez. Para uma rotina de ordenação, isso deveria ser sempre verdade:

```haskell
prop_idempotent xs = qsort (qsort xs) == qsort xs
```

O funcionamento dessa biblioteca será estudado em detalhes no capítulo de [Testes com QuickCheck](../unidade2/04_testes_qualidade.md); a referência original é o capítulo 11 de [*Real World Haskell*](http://book.realworldhaskell.org/read/testing-and-quality-assurance.html). O objetivo aqui é só apresentar o `cabal`. Então, por enquanto, assuma que vamos atualizar `test/Main.hs` desse jeito:

```haskell
{-# LANGUAGE TemplateHaskell #-}

import Test.QuickCheck
import MyLib

prop_idempotent xs = qsort (qsort xs) == qsort xs

return []
runTests = $quickCheckAll

main :: IO ()
main = runTests >>= \passed -> if passed then putStrLn "Passou em todos os testes."
                                          else putStrLn "Alguns testes falharam."
```

Agora podemos rodar os testes:

```bash
$ cabal test
...
=== prop_idempotent from test/Main.hs:6 ===
*** Failed! Falsified (after 5 tests and 2 shrinks):
[0,-1]

Alguns testes falharam.
```

Depois de 5 testes, ocorreu uma falha. Voltando ao código, encontramos o erro — uma linha que devia usar `rhs` está usando `lhs` de novo:

```haskell
qsort lhs ++ [x] ++ qsort lhs   -- errado
```

O correto:

```haskell
qsort :: Ord a => [a] -> [a]
qsort []     = []
qsort (x:xs) = qsort lhs ++ [x] ++ qsort rhs
    where lhs = filter  (< x) xs
          rhs = filter (>= x) xs
```

Rodando os testes de novo:

```bash
$ cabal test
...
=== prop_idempotent from test/Main.hs:6 ===
+++ OK, passed 100 tests.

Passou em todos os testes.
```

Esse pequeno ciclo — escrever uma propriedade, deixar o QuickCheck gerar centenas de entradas, achar o bug, corrigir — é exatamente o que vamos aprofundar no capítulo de [Testes com QuickCheck](../unidade2/04_testes_qualidade.md) da Unidade 2, aplicado a um projeto bem maior: uma biblioteca de manipulação de JSON.
