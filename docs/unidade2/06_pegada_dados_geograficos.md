# Pegada de Dados Geográficos: Geometria e Álgebra de Mapas com o TerraHS

O capítulo anterior mostrou o paradigma funcional construindo linguagens. Este mostra outra aplicação avançada, num domínio bem diferente: **Sistemas de Informação Geográfica (GIS)**. O projeto **[TerraHS](https://github.com/LambdaGeo/terrahs)** é uma reescrita, em Haskell puro, de um software de pesquisa originalmente construído entre 2005 e 2009 no INPE (Instituto Nacional de Pesquisas Espaciais) — e é uma boa ilustração de como os mesmos ingredientes dos capítulos anteriores (ADTs, typeclasses, tipos parametrizados, funções puras) modelam um domínio de aplicação real, fora do universo de "estruturas de dados de livro-texto".

---

## 1. Geometria como tipos algébricos

A base do TerraHS é modelar formas geométricas como ADTs — o mesmo padrão do capítulo de [Tipos Algébricos](01_declarando_tipos_classes.md), aplicado a coordenadas:

```haskell
data Coord = Coord { coordX :: !Double, coordY :: !Double }

newtype Point = Point { pointCoord :: Coord }

newtype Line = Line { lineCoords :: [Coord] }  -- invariante: ≥ 2 pontos

newtype Polygon = Polygon { polygonRing :: [Coord] }  -- invariante: anel fechado, ≥ 3 vértices
```

Repare que `Line` e `Polygon` não expõem seus construtores — só as funções seletoras (`lineCoords`, `polygonRing`). Quem quiser construir um `Line` precisa passar pela função `mkLine`, que valida a invariante (pelo menos dois pontos) e devolve `Maybe Line`:

```haskell
mkLine :: [Coord] -> Maybe Line
mkLine cs
  | length cs >= 2 = Just (Line cs)
  | otherwise      = Nothing
```

Esse padrão — esconder o construtor, expor só uma função "inteligente" que garante um invariante — chama-se **smart constructor**, e é uma das formas mais diretas de usar o sistema de tipos para tornar estados inválidos **irrepresentáveis**: depois de construído, todo `Line` do programa *sempre* tem pelo menos dois pontos, sem precisar checar isso de novo em nenhuma outra função.

### Uma typeclass para unificar as três formas

`Point`, `Line` e `Polygon` são tipos diferentes, mas compartilham operações (área, perímetro, centroide). Em vez de três famílias de funções (`pointArea`, `lineArea`, `polygonArea`...), o TerraHS usa uma **typeclass** — exatamente o mecanismo do capítulo de [Tipos Algébricos](01_declarando_tipos_classes.md#7-implementando-classes-de-tipos-personalizadas):

```haskell
class Geometry a where
  area      :: a -> Double
  perimeter :: a -> Double
  centroid  :: a -> Coord

instance Geometry Point where
  area _      = 0
  perimeter _ = 0
  centroid    = pointCoord

instance Geometry Polygon where
  area      = polygonArea       -- fórmula do shoelace
  perimeter = polygonPerimeter
  centroid  = polygonCentroid
```

Qualquer função que precise só de "alguma geometria" pode ser escrita de forma genérica sobre `Geometry a => a`, sem saber (nem precisar saber) se está lidando com um ponto, uma linha ou um polígono.

---

## 2. A álgebra de mapas: generalizando Tomlin com predicados espaciais

A parte mais avançada do TerraHS é a **álgebra de mapas** — o resultado de pesquisa que inspirou o projeto original. A ideia clássica (Tomlin, 1983) é que operações sobre mapas raster (grades) se dividem em quatro categorias: **local** (célula a célula), **focal** (vizinhança fixa), **zonal** (agregando por região) e **global** (o mapa inteiro).

O TerraHS parte de uma ideia mais geral, vinda do padrão OGC de *coverage*: um mapa não precisa ser uma grade fixa — pode ser qualquer **função discreta** de um domínio de elementos geográficos para um conjunto de valores:

```haskell
data Coverage a b = Coverage (a -> b) [a]

newCov :: [a] -> (a -> b) -> Coverage a b
newCov dom f = Coverage f dom

values :: Coverage a b -> [b]
values c@(Coverage f dom) = map f dom
```

Com essa representação, os operadores FOCAL e ZONAL de Tomlin — que, na formulação clássica, usam só duas relações espaciais fixas ("vizinho de" e "contido em") — viram **um único operador**, parametrizado por **qualquer** predicado espacial:

```haskell
spatial :: ([b] -> b) -> Coverage a b -> (a -> ref -> Bool) -> Coverage ref b -> Coverage ref b
spatial fn c predicate covRef =
  newCov (domain covRef) (\x -> compose fn (select c predicate x))
```

Na prática, isso permite trocar livremente o critério de vizinhança. Um exemplo real do projeto: dado um conjunto de pontos com um valor de desmatamento e um conjunto de polígonos representando áreas de proteção ambiental, a média de desmatamento *por área de proteção* é uma única chamada, usando `pointInPolygon` como predicado espacial:

```haskell
mediaPorArea :: Coverage Point Double -> Coverage Polygon Double
mediaPorArea pontos = spatial media pontos pointInPolygon areasDeProtecao
  where media xs = sum xs / fromIntegral (length xs)
```

Trocar `pointInPolygon` por um predicado de proximidade a uma estrada (`pointOnLine`, com uma tolerância) dá a mesma agregação, mas "ao longo da estrada" em vez de "dentro da área" — sem escrever nenhuma lógica de agregação nova. A separação entre *seleção espacial* (`select`, o predicado) e *agregação* (`compose`, a função `[b] -> b`) é o que permite essa reutilização.

---

## 3. Lendo dados geográficos de verdade

Além da álgebra, o TerraHS lê formatos de dados geoespaciais padrão da indústria — texto (**WKT**) e binário (**Shapefile**, o formato de fato da ESRI):

```haskell
parseWKT :: String -> Either String AnyGeometry
-- parseWKT "POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))"
--   ==> Right (AGPolygon (Polygon {polygonRing = [...]}))
```

O parser de WKT é escrito à mão, sem biblioteca externa, com o mesmo estilo de *parser combinator* do capítulo anterior (uma função `String -> Maybe (a, String)`, combinada com `Functor`/`Applicative`/`Monad`) — um bom exercício para comparar lado a lado com o Parsec usado no `scheme-hs`. Já o leitor de Shapefile trabalha diretamente sobre bytes binários (com a biblioteca `binary`), decodificando um formato que mistura *big-endian* e *little-endian* — um exercício de como um tipo `Get a` também é, por trás da API, uma função que consome um prefixo da entrada e devolve um valor mais o resto, o mesmo padrão conceitual de um parser de texto.

!!! info "Onde ler mais"
    O código completo — geometria, topologia, os parsers de WKT/GeoJSON/Shapefile, a álgebra de `Coverage`, e uma suíte de testes que reproduz os exemplos numéricos originais da pesquisa que deu origem ao projeto — está em **[github.com/LambdaGeo/terrahs](https://github.com/LambdaGeo/terrahs)**. O README do repositório tem instruções de instalação e um executável de demonstração (`cabal run terrahs-demo`) que roda os exemplos deste capítulo com dados sintéticos reproduzíveis.
