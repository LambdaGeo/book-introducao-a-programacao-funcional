# Unidade 2 — Haskell Avançado e Qualidade de Código (20h)

## Identificação

| | |
|---|---|
| **Unidade** | 2 de 3 |
| **Carga horária** | 20 horas (10 aulas de 2h) |
| **Pré-requisitos** | Unidade 1 concluída (Haskell básico: funções, listas, alta ordem) |
| **Linguagem** | Haskell (projetos com Cabal e QuickCheck) |

## Objetivos de aprendizagem

Ao final desta unidade, o estudante deverá ser capaz de:

1. **Definir** tipos algébricos de dados (ADTs), tipos recursivos e instâncias de classes de tipos próprias.
2. **Construir** programas interativos compreendendo a separação entre código puro e ações `IO`.
3. **Estruturar** um projeto Haskell moderno com Cabal, organizando biblioteca, executável e suíte de testes.
4. **Implementar** uma biblioteca completa de serialização JSON, aplicando ADTs e pretty-printing (adaptado de *Real World Haskell*, cap. 5).
5. **Garantir** a qualidade do código com testes baseados em propriedades usando QuickCheck (adaptado de *Real World Haskell*, cap. 11).
6. **Reconhecer** aplicações avançadas do paradigma funcional em dois domínios reais: construção de interpretadores e parsers (casamento de expressões regulares, um interpretador de Scheme) e processamento de dados geoespaciais (geometria, álgebra de mapas).

## Cronograma

| Aula | CH | Conteúdo | Capítulo |
|:---:|:---:|---|---|
| 1 | 2h | Declaração de tipos: `type`, `data`, tipos parametrizados e recursivos | [Declarando Custom Types](01_declarando_tipos_classes.md) |
| 2 | 2h | Classes de tipos próprias, instâncias, derivação e Lista 4 (laboratório) | [Declarando Custom Types](01_declarando_tipos_classes.md) |
| 3 | 2h | Programas interativos: a monad `IO`, `do`-notation, entrada e saída | [Programas Interativos (IO)](02_programas_interativos.md) |
| 4 | 2h | Arquivos, `interact`, argumentos de linha de comando e Lista 5 (laboratório) | [Programas Interativos (IO)](02_programas_interativos.md) |
| 5 | 2h | Biblioteca JSON — parte 1: modelagem do tipo `JValue` e serialização | [Escrevendo a Biblioteca JSON](03_biblioteca_json.md) |
| 6 | 2h | Biblioteca JSON — parte 2: pretty-printing e refinamento da API | [Escrevendo a Biblioteca JSON](03_biblioteca_json.md) |
| 7 | 2h | Testes baseados em propriedades com QuickCheck | [Testes com QuickCheck](04_testes_qualidade.md) |
| 8 | 2h | Pegada avançada: parsers, casamento de padrões e um interpretador de Scheme | [Pegada de Compiladores](05_pegada_compiladores.md) |
| 9 | 2h | Pegada avançada: geometria e álgebra de mapas com dados geográficos | [Pegada de Dados Geográficos](06_pegada_dados_geograficos.md) |
| 10 | 2h | Laboratório orientado, entrega e defesa do trabalho prático | [Trabalho Prático](07_avaliacao.md) |

## Metodologia

A unidade é orientada a projeto: os conceitos das aulas 1–4 convergem para a construção incremental da biblioteca JSON (aulas 5–7), que é também o objeto da avaliação. As aulas 8 e 9 são capítulos de leitura e demonstração — mostram o paradigma funcional aplicado a projetos reais e completos (não são pré-requisito para o trabalho prático, mas ilustram para onde os conceitos da unidade levam fora de um exercício de livro). A aula 10 é laboratório de desenvolvimento e apresentação.

Os capítulos 5 e 6 (numeração deste material) apresentam os conceitos e as decisões de projeto da biblioteca JSON; o desenvolvimento passo a passo é guiado pelo tutorial **[Construindo e Testando uma Biblioteca Haskell: JSON, Pretty Printing e QuickCheck](https://lambdageo.github.io/blog/tutorial-haskell-json-quickcheck/)**, no blog do LambdaGEO.

## Avaliação

- **Trabalho prático (100%)** — desenvolvimento da biblioteca JSON estruturada com Cabal e integrada com uma suíte de testes QuickCheck, conforme especificado no capítulo [Trabalho Prático](07_avaliacao.md). A nota considera: corretude e completude da biblioteca, qualidade e cobertura das propriedades testadas, organização do projeto e defesa individual na aula 10.

## Bibliografia da unidade

- O'SULLIVAN, Bryan; STEWART, Don; GOERZEN, John. *Real World Haskell* — capítulos 5 e 11 (código modernizado para os padrões atuais do GHC).
- LIPOVAČA, Miran. *Learn You a Haskell for Great Good!* — capítulos 7 a 9.
- FISCHER, Sebastian; HUCH, Frank; WILKE, Thomas. *A Play on Regular Expressions* (ICFP 2010) — base teórica do capítulo [Pegada de Compiladores](05_pegada_compiladores.md).
- COSTA, Sérgio S.; CÂMARA, Gilberto; PALOMO, Danilo. *TerraHS: Integration of Functional Programming and Spatial Databases for GIS Application Development*, in *Advances in Geoinformatics* (Springer, 2007) — base teórica do capítulo [Pegada de Dados Geográficos](06_pegada_dados_geograficos.md).
