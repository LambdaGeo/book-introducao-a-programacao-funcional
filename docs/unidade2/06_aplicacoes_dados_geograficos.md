# Aplicações Avançadas: Dados Geoespaciais

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

## 3. Modelos dinâmicos espaciais: o comonad ao contrário do parser

O capítulo anterior usou `Functor`, `Applicative` e `Monad` para escrever parsers — cada um é, por trás da API, uma função que **produz** um valor a partir de um contexto (`String -> Maybe (a, String)`, no caso do parser). Existe a classe simétrica, menos comum de se ver em código do dia a dia, mas natural para um problema específico: um **comonad** é um contexto que já *tem* um valor, e sabe como olhar para valores vizinhos a partir dele. `TerraHS.CA` — mais um componente independente do pacote (`terrahs-ca`), ao lado do `terrahs-render` da próxima seção — usa exatamente essa ideia para modelar sistemas dinâmicos espaciais: autômatos celulares aplicados a um domínio geográfico, o tema original do artigo *"Modelos dinâmicos espaciais em programação funcional"* (Costa et al.) que deu origem a essa parte do projeto.

O tipo usado é `Store` (do pacote `comonad`): um valor de `Store e s` é, ao mesmo tempo, uma posição atual (`pos`), uma função de qualquer posição para seu estado (`peek`), e o estado da posição atual (`extract`, o valor de `peek` na própria posição). Para um autômato celular, isso é exatamente "onde estou, e como consultar qualquer vizinho sem sair daqui". A operação central é `extend`, que aplica uma regra `Store e s -> s` a **todas** as posições de uma vez, produzindo um novo `Store` — um passo de simulação inteiro, sem laço explícito:

```haskell
stepCA :: (Store e s -> s) -> Store e s -> Store e s
stepCA = extend

runCA :: (Store e s -> s) -> Store e s -> [Store e s]
runCA rule = iterate (stepCA rule)
```

`runCA` substitui diretamente o `sim :: Monad m => [t] -> (t -> s -> m s) -> s -> m s` do artigo original — a mesma ideia ("aplique a regra repetidamente, gerando uma sequência de estados"), só que como uma lista preguiçosa em vez de uma recursão monádica escrita à mão.

Um exemplo concreto: um modelo de difusão sobre seis polígonos, onde uma zona contaminada contamina toda zona vizinha no passo seguinte. A vizinhança é o mesmo tipo de predicado espacial da seção anterior — só que agora combinando duas condições com o `Monoid` de `Predicate` (`<>` vira "E lógico"):

```haskell
adjacent :: Predicate (Zone, Zone)
adjacent = notSelf <> touchesVia zonePoly intersects

diffusionRule :: Store Zone Bool -> Bool
diffusionRule w = extract w || or (neighborValues adjacent zones w)
```

`extract w` é "esta zona já está contaminada?"; `neighborValues adjacent zones w` lê o estado de toda zona vizinha (via `peek`) sem o autômato nunca precisar saber onde ele está numa grade — a "posição" é só mais um valor do domínio, um `Polygon`, não um índice `(x, y)`. Rodar `runCA diffusionRule` a partir de uma zona contaminada produz a sequência inteira de estados; `notSelf` e `touchesVia` são só duas funções auxiliares de `TerraHS.CA` para montar esse tipo de predicado sem repeti-lo em cada modelo.

!!! info "Três modelos, mesma máquina"
    O repositório tem três exemplos completos sobre essa base — `life-demo` (o Jogo da Vida de Conway, num tabuleiro infinito), `diffusion-demo` (o modelo acima) e `fire-demo` (incêndio florestal: floresta / queimando / queimado) — cada um só troca o domínio, a regra e o tipo de estado; a máquina (`stepCA`/`runCA`/`seedCA`) é a mesma para os três. Cada um também checa seu resultado contra um valor conhecido de antemão (o planador do Jogo da Vida reproduzindo-se deslocado após 4 gerações; a propagação da difusão contra uma busca em largura feita à mão) em vez de só imprimir a saída.

---

## 4. Lendo dados geográficos de verdade

Além da álgebra, o TerraHS lê formatos de dados geoespaciais padrão da indústria — texto (**WKT**) e binário (**Shapefile**, o formato de fato da ESRI):

```haskell
parseWKT :: String -> Either String AnyGeometry
-- parseWKT "POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))"
--   ==> Right (AGPolygon (Polygon {polygonRing = [...]}))
```

O parser de WKT é escrito à mão, sem biblioteca externa, com o mesmo estilo de *parser combinator* do capítulo anterior (uma função `String -> Maybe (a, String)`, combinada com `Functor`/`Applicative`/`Monad`) — um bom exercício para comparar lado a lado com o Parsec usado no `scheme-hs`. Já o leitor de Shapefile trabalha diretamente sobre bytes binários (com a biblioteca `binary`), decodificando um formato que mistura *big-endian* e *little-endian* — um exercício de como um tipo `Get a` também é, por trás da API, uma função que consome um prefixo da entrada e devolve um valor mais o resto, o mesmo padrão conceitual de um parser de texto.

---

## 5. Simplificação de geometria: dividir-para-conquistar com números reais

Um polígono lido de uma fonte real costuma ter vértices demais para o que se vai fazer com ele. A malha municipal do Maranhão do IBGE tem 217 municípios, 280 partes de polígono (ilhas contam separado) e **810.584 vértices no total** — um município só, Amarante do Maranhão, tem mais de 17 mil. Testar cada pixel de um mapa de ~550×740px contra esses vértices (a mesma função `pointInPolygon` da seção 1) é caro, e a maior parte desse custo não compra nada: uma curva menor que meio pixel de tela é invisível, não importa quão precisa seja a medição original.

O algoritmo clássico para isso é **Ramer-Douglas-Peucker**: a mesma receita de *dividir-para-conquistar* do capítulo de [Recursão](../unidade1/06_listas.md), só que sobre coordenadas em vez de números. Dada uma tolerância `epsilon`, mantém sempre o primeiro e o último ponto de uma cadeia; acha o ponto mais distante (perpendicularmente) da reta entre esses dois; se essa distância passa de `epsilon`, o ponto importa — divide a cadeia ali e simplifica cada metade recursivamente; senão, a cadeia inteira colapsa para só os dois pontos das pontas:

```haskell
simplifyCoords :: Double -> [Coord] -> [Coord]
simplifyCoords epsilon coords
  | length coords < 3 = coords
  | farthestDist > epsilon =
      simplifyCoords epsilon before ++ drop 1 (simplifyCoords epsilon after)
  | otherwise = [first, end]
  where
    first = head coords
    end   = last coords
    (farthestIdx, farthestDist) = -- ponto mais distante da reta first-end
      maximumBy (comparing snd) (zip [0 ..] (map (perpendicularDistance first end) coords))
    before = take (farthestIdx + 1) coords  -- inclui o ponto de corte nos dois lados
    after  = drop farthestIdx coords
```

Repare no `before`/`after`: as duas metades **compartilham** o ponto de corte (`take (farthestIdx + 1)` de um lado, `drop farthestIdx` do outro) — se fosse um `splitAt` comum, esse ponto ficaria só de um lado e desapareceria do resultado final. Foi exatamente esse bug que apareceu na primeira versão, pego por um teste que checava se a área de um polígono simplificado batia com a original: em vez de um quadrado, saía um triângulo degenerado.

Antes de ir para o dado real, o efeito é mais fácil de enxergar numa cadeia pequena, à mão: uma linha reta com um único "solavanco" de 0,05 no meio.

```haskell
ghci> let bumpy = [Coord 0 0, Coord 1 0, Coord 2 0.05, Coord 3 0, Coord 4 0]

ghci> simplifyCoords 0.1 bumpy    -- tolerância maior que o solavanco (0,05)
[Coord 0 0,Coord 4 0]

ghci> simplifyCoords 0.04 bumpy   -- tolerância menor que o solavanco
[Coord 0 0,Coord 2 5.0e-2,Coord 4 0]
```

Com `epsilon = 0.1`, o ponto do meio está a só 0,05 da reta entre as pontas — menos que a tolerância, então some: a cadeia inteira colapsa para os dois extremos, os únicos que `simplifyCoords` sempre preserva. Com `epsilon = 0.04`, a mesma distância já passa da tolerância, e o ponto fica — a cadeia é dividida ali, e cada metade (só dois pontos cada) já não tem mais nada pra simplificar. É a mesma pergunta, feita uma vez só — "esse ponto muda a forma o suficiente para valer a pena guardar?" — que o mapa real do Maranhão faz 810 mil vezes:

A tolerância natural para desenhar é *metade de um pixel de render* — uma curva menor que isso nunca muda a cor de nenhum pixel. Rodando essa mesma simplificação antes de desenhar o mapa real do Maranhão:

| | Original | Simplificado |
|---|---|---|
| Vértices | 810.584 | 10.133 (−98,7%) |
| Tempo de render | ~29 s | ~0,45 s (~65× mais rápido) |

E a imagem resultante é visualmente indistinguível da original — o ganho vem inteiro de vértices que nunca poderiam ter mudado um pixel de qualquer forma. `TerraHS.Geometry.Simplify` mora na biblioteca `terrahs` principal, não num componente à parte como `terrahs-ca`/`terrahs-render`: é aritmética pura sobre coordenadas, sem depender de nada novo — ao contrário de renderizar (que precisa de `JuicyPixels`) ou dos autômatos (que precisam de `comonad`/`contravariant`).

**Na prática**, sem medir nem comparar nada — só carregar um shapefile qualquer e desenhar o mapa — o Simplify entra como mais um passo no meio, entre ler o dado e montar a `Coverage` que vai pro renderer:

```haskell
main :: IO ()
main = do
  Right feats <- readVectorFile "dados/meu-shapefile.shp"
  let pares   = [ (poly, nome) | (poly, attrs) <- asPolygonPairs feats
                                , Just nome <- [attrAs "NOME" attrs] ]
      box     = foldr1 union (map (envelope . fst) pares)
      epsilon = 1 / (2 * 80)  -- meio pixel, pro mesmo scalePx do renderer
      cov     = fromPairs [ (simplifyPolygon epsilon poly, nome) | (poly, nome) <- pares ]
  renderPolygonFillWith corPara "mapa.png" box cov
```

Quatro chamadas fazem o trabalho todo: `readVectorFile`/`asPolygonPairs`/`attrAs` carregam o shapefile e extraem um atributo (a mesma leitura da seção 4); `simplifyPolygon` roda **em cada polígono, antes** de virar `Coverage` — simplifica a geometria, não a imagem depois de pronta; `renderPolygonFillWith` desenha. É exatamente a receita que `ibge-map-demo` usa — ele só chama `renderPolygonFillWith` duas vezes (original e simplificado) e mede o tempo de cada uma, pra produzir a tabela acima; quem só quer o mapa faz a versão de cima, sem a segunda chamada nem a comparação.

!!! info "Onde ler mais"
    O código completo — geometria, topologia, os parsers de WKT/GeoJSON/Shapefile, a álgebra de `Coverage`, os modelos dinâmicos comonádicos (`terrahs-ca`), a simplificação de geometria, e uma suíte de testes que reproduz os exemplos numéricos originais da pesquisa que deu origem ao projeto — está em **[github.com/LambdaGeo/terrahs](https://github.com/LambdaGeo/terrahs)**. O README do repositório tem instruções de instalação; `cabal run terrahs-demo` roda os exemplos deste capítulo com dados sintéticos reproduzíveis, `cabal run life-demo` / `diffusion-demo` / `fire-demo` rodam os três modelos da seção 3, e `cabal run ibge-map-demo` reproduz a comparação original/simplificado da tabela acima, com o mapa real do Maranhão desenhado dos dois jeitos.
