# Testes e Garantia de Qualidade com QuickCheck

Neste capítulo, aprenderemos a metodologia de **Testes Baseados em Propriedades** (*Property-Based Testing*) com a biblioteca **QuickCheck**, adaptando o capítulo 11 do *Real World Haskell* para o ambiente de testes de um projeto **Cabal** moderno — e a aplicaremos à biblioteca JSON do capítulo anterior.

!!! success "Tutorial completo no blog"
    O passo a passo — configuração da suíte no arquivo `.cabal`, todas as propriedades, a instância `Arbitrary` do `JValue` e a medição de cobertura — está na **Parte 2** do tutorial no blog do LambdaGEO:

    **[Construindo e Testando uma Biblioteca Haskell: JSON, Pretty Printing e QuickCheck](https://lambdageo.github.io/blog/tutorial-haskell-json-quickcheck/)**

    Este capítulo apresenta os conceitos; siga o tutorial para implementar a suíte completa.

!!! info "Leitura complementar"
    O capítulo original que inspirou este material: [*Real World Haskell*, Cap. 11 — Testing and quality assurance](http://book.realworldhaskell.org/read/testing-and-quality-assurance.html) (em inglês; o tutorial do blog moderniza o código para QuickCheck 2.14 e o HPC atual).

---

## 🧪 Por que Testar Propriedades?

Os testes unitários tradicionais fornecem entradas específicas e checam saídas esperadas (`soma 2 3 == 5`). Embora úteis, exigem que o desenvolvedor pense manualmente em todos os casos especiais. Os testes baseados em propriedades invertem essa lógica:

1. Definimos **propriedades universais** (invariantes) que o código deve obedecer para *qualquer* entrada;
2. O **QuickCheck** gera automaticamente centenas de entradas aleatórias e testa as invariantes;
3. Ao encontrar uma falha, ele **encolhe** (*shrinking*) a entrada até o menor caso que reproduz o erro.

## 💡 O Sabor da Coisa: Propriedades de uma Ordenação

Para um algoritmo de ordenação `qsort`, que invariantes devem valer para **qualquer** lista?

```haskell
import Test.QuickCheck
import Data.List (sort, (\\))

-- Idempotência: ordenar duas vezes == ordenar uma vez
prop_idempotente :: [Int] -> Bool
prop_idempotente xs = qsort (qsort xs) == qsort xs

-- O menor elemento vem primeiro (só para listas não vazias)
prop_minimo :: [Int] -> Property
prop_minimo xs = not (null xs) ==> head (qsort xs) == minimum xs

-- Teste baseado em modelo: concorda com o sort da biblioteca padrão
prop_modelo :: [Int] -> Bool
prop_modelo xs = qsort xs == sort xs
```

```text
ghci> quickCheck prop_idempotente
+++ OK, passed 100 tests.
```

Três conceitos importantes aparecem aqui:

* **Implicação (`==>`)**: descarta entradas inválidas antes de testar (note que o tipo muda de `Bool` para `Property`);
* **Teste baseado em modelo**: comparar com uma implementação de referência correta (ainda que ineficiente) é uma técnica poderosíssima — grandes projetos Haskell mantêm suítes de propriedades executadas a cada commit;
* **QuickCheck como "lint" de API**: se uma propriedade é difícil de enunciar, talvez a interface esteja mal desenhada.

!!! tip
    É por isso que **código puro é mais fácil de testar** (como prometido no primeiro capítulo do livro): uma função que só depende das entradas visíveis pode ser bombardeada com milhares de entradas aleatórias sem *mocks* nem preparação de ambiente.

## 🎲 Gerando Dados Customizados: a Classe `Arbitrary`

Para testar o `JValue`, precisamos ensinar o QuickCheck a gerar valores aleatórios do nosso tipo, implementando a classe `Arbitrary`:

```haskell
class Arbitrary a where
  arbitrary :: Gen a
```

A biblioteca fornece combinadores (`elements`, `choose`, `oneof`, `listOf`) para construir geradores. O desafio interessante do `JValue` é que ele é **recursivo**: um gerador ingênuo pode criar objetos infinitamente profundos. A solução — detalhada no tutorial — usa o combinador `sized` para limitar a profundidade, reduzindo o "orçamento" a cada nível de recursão.

## 📊 Cobertura com HPC

Milhares de testes passando é reconfortante — mas quais partes do código os testes *realmente* exercitam? O **HPC** (*Haskell Program Coverage*) responde com precisão:

```bash
cabal test --enable-coverage
```

O relatório HTML mostra expressões, ramos e funções cobertos, destacando os trechos que nenhum teste alcança — o guia perfeito para escrever as próximas propriedades.

## 🎯 O que Você Deve Dominar ao Final

* Enunciar invariantes de uma função como propriedades QuickCheck (`prop_*`);
* Usar `==>` para restringir entradas e explicar o tipo `Property`;
* Implementar `Arbitrary` para um tipo recursivo com controle de profundidade (`sized`);
* Interpretar um relatório de cobertura HPC e usá-lo para direcionar novos testes.

Esses são exatamente os itens cobrados no [trabalho prático da unidade](07_avaliacao.md). Os próximos dois capítulos mostram o paradigma funcional em ação em dois domínios mais avançados — construção de interpretadores e processamento de dados geoespaciais — antes de você colocar a mão na massa no trabalho prático.

---

> **Nota de atribuição:** este capítulo adapta material do capítulo 11 de *Real World Haskell*, de Bryan O'Sullivan, Don Stewart e John Goerzen ([book.realworldhaskell.org](http://book.realworldhaskell.org/read/)), sob a licença [Creative Commons Attribution-Noncommercial 3.0](http://creativecommons.org/licenses/by-nc/3.0/).
