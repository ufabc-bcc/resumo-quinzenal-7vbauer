# Resumo 2 Paradigmas de Programação

## Nome: Vitor Marini Blaselbauer
## RA:   11048115

# Semana 3 - QuickCheck

### QuickCheck

- Ferramenta para teste de código.
- 1. Definir uma invariante do algoritmo.
- 2. Geração automatica de casos de teste aleatórios.

```haskell
import Test.QuickCheck

(...)

> quickCheck propX
```

### Propriedades
- Propósito de verificar se algo vale ou não depois de uma chamada de função. (?)

```haskell
-- Exemplo.
fatorial :: Integral a => a -> a
fatorial n
    | n == 0 = 1
    | otherwise = n * fatorial (n - 1)

-- Testa a propriedade (n + 1)! = (n + 1).n! , utilizando o fatorial implementado acima.
prop_fatorialX n = (NonNegative n) -- Somente inteiros não negativos.
    ==> fatorial n * (n + 1) == fatorial (n + 1)
```

### Função ``filter``
- "Filtra" elementos de uma lista dado uma função que retorna um ``Bool``.
- Se ``False``, não seleciona o elemento. 

```haskell
filter :: (a -> Bool) -> [a] -> [a]
filter p xs = [x | x <- xs, p x]

-- Exemplo.
filter (/= ' ') "abc def ghi"
> "abcdefghi"
```


### Operador ``==>``
- Quando não quisermos utilizar instâncias de um certo tipo nos testes:

```haskell
    propX :: Ord a => [a] -> Bool
    -- ou
    propX :: Ord a => [a] -> Property

    propX xs = not (null xs) -- Não utilizar listas nulas nos testes.
        => head (qsort xs) == minimum xs
```

### Função ``sort``
- Realize a ordenação de uma lista de elementos ordenáveis (``Ord``).
- Utiliza o algoritmo MergeSort.

## **! O QuickCheck não prova uma propriedade verdadeira ou falsa, apenas diz que não foi possivel encontrar um contra-exemplo para ela !**

# Semana 3 - Funções de Alta Ordem

### Funções como Tipos

- Em haskell podemos pensar em uma função como um tipo de dado.

```haskell
-- Recebe um inteiro e retorna outro inteiro.
soma :: Int -> Int -> Int

-- ou

-- Recebe um inteiro e retorna uma função que soma dois inteiros.
-- Esta segunda função então recebe um inteiro e retorna outro inteiro.
soma :: Int -> (Int -> Int)
```

```haskell
-- Exemplo.

soma x y = x + y

soma3 = soma 3

soma3 2 
```

### Aplicação Parcial
- Pode ser utilizado em qualquer função com mais do que um argumento.

```haskell
-- Exemplos.

soma3 = (+3)

dobra = (*2)
```

### Funções de Alta Ordem
- Haskell permite a passagem de funções como parâmetros.
- Funções que recebem uma ou mais fuções como argumento ou devolvem uma função são denominadas **funções de alta ordem** .
- Provê um aumento na expressividade.

### Função ``map``
- Transforma uma lista de um tipo ``a`` para um tipo ``b`` utilizando uma função ``f :: (a -> b)``.

```haskell
map :: (a -> b) -> [a] -> [b]
map f xs = [f x | x <- xs]
```

### Operador pipe ``($)``

- Para separar aplicações das funções e remover parênteses.

```haskell
($) :: (a -> b) -> a -> b
```

### Função ``foldr``
- Generalização de um formato de função ja visto anteriormente.

```haskell
f [] = v
f (x:xs) = g x (f xs) -- Conferir se é mesmo g (?)

-- Foldr
foldr :: (a -> b -> b) -> b -> [a] -> b
foldr f v []     = v
foldr f v (x:xs) = f x (foldr f v xs) 

-- a1 `f` (a2 `f` (a3 ``f` v))
-- Vai no ultimo elemento, aplica a função f com o valor passado (...)
--  e o resultado é passado para a função f com o próximo (anterior) elemento. 
```

```haskell
-- Exemplo.
foldr (+) 0 [1, 2, 3, 4]
> 10 -- (1 + (2 + (3 + (4 + 0))))
```

### Função ``foldl``
- "*Folding caudal*".

```haskell
foldl :: (a -> b -> b) -> a -> [b] -> a
foldl f v []     = v
foldl f v (x:xs) = foldl f (f v x) xs
```

## ***Quando a função `f` é associativa, a aplicação de ``foldr`` e ``foldl`` não faz diferença.***

### Função ``length`` com ``foldr`` e ``foldl``
```haskell
length = foldr (\_ n -> n + 1) 0

length = foldl (\n _ -> n + 1) 0
```

### Função ``reverse`` com ``foldr`` e ``foldl`` 
```haskell
reverse = foldr (\x xs -> xs ++ [x]) []

reverse = foldl (\xs x -> x:xs) []
```

### Função ``product`` com ``foldr`` e ``foldl`` 
```haskell
product = foldr (*) 1

product = foldl (*) 1
```

### Função ``sum`` com ``foldr`` e ``foldl`` 
```haskell
sum = foldr (+) 0

sum = foldl (+) 0
```

### **... pois '+' e '*' são associativas ...**

### ``foldr`` vs ``foldl``
- Se a lista é infinita : ``foldr``.
- Se o operador utilizado pode gerar curto-circuito: ``foldr``.
- Se a lista é finita e o operador não gera curto-circuito: ``foldl``.
- Se faz sentido trabalhar com a lista invertida: ``foldl``.

### Composição de Funções
- Utilização do operador '.' .
```haskell
--      /  f \      /  g  \    / f . g \
(.) :: (b -> c) -> (a -> b) -> (a -> c)
f . g = \x -> f (g x)
```

### ``($)`` vs ``(.)`` 
- Se o desempenho for o mesmo optar pela versão mais legível.

### Outras funções Úteis

- ``reverse`` : inverte a ordem de uma lista de trás para frente.
- ``even`` : ``True`` se o valor é par, ``False`` se o valor é impar.
- ``all`` : com uma função booleana e uma lista de valores, retorna ``True`` se ***todos os elementos*** com a aplicação da função retornam ``True``. ``False caso contrário. 
- ``any`` : similar a ``all``, porém retorna ``True`` se ***pelo menos um elemento*** retornar ``True`` com a aplicação da função. 
- ``takeWhile`` : toma valores de uma lista enquanto o elemento atual satisfaz uma função booleana.
- ``dropWhile`` : não seleciona elementos de uma lista enquanto o elemento atual satisfizer uma função booleana. 


---

# Semana 4 - Tipos de Dados Algébricos

### Keyword ``type``
- Cria apenas um "apelido" para um tipo.
- **Não** permite a criação de tipos mais complexos (e.g. recursivos).
```haskell
type Produto = (Integer, String, Double)
```

```haskell
-- Exemplos

type Produto = (Integer, String, Double)
type Cliente = (Integer, String, Double)

(...)

troco :: Produto -> Cliente -> Double

------------------------------------------

type String = [Char]
```
## Tipos de Dados Algébricos
- Tipos completamente novos.
- Pode conter tipos primitivos.
- Permite checagem em tempo de compilação.
- Permite expressividades.

### Tipo **Soma**
```haskell
data Bool = True | False

data Dir = Norte | Sul | Leste | Oeste
```

### Tipo **Produto**
```haskell
data Ponto = MkPonto Double Double
```
- *MkPonto* é um construtor, função usada para criar uma valor do tipo ``Ponto``.
- Os nomes do tipo e do construtor podem ser os mesmos, mas vivem em espaços diferentes e não são a mesma coisa.

### Misturando Soma e Produto
```haskell
data Forma = Circulo Ponto Double
    |   Retangulo Ponto Double Double
```
### Keyword ``deriving``
- Utilizado para derivar um tipo de outra classe.
```haskell
data Ponto = MkPonto Double Double
    deriving Show
```

### Tipo ``Show``
- Tipo imprimível.
- ``deriving Show``  cria automaticamente uma instância da função ``show`` para o tipo.



### Tipo ``Maybe``
- Tipos podem ser parametrizados.
- Oferece um melhor controle de erros e exceções.


```haskell
-- Pode ser nada, ou só um valor do tipo "a".

data Maybe a = Nothing | Just a

-- Exemplo com expressão "case ... of".
div :: Int -> Int -> Int
div m n = case (maybeDiv m n) of
    Nothing -> error "Divisão por zero."
    Just x -> x
```

### Tipo ``Either``
- Permite que uma função retorne dois tipos diferentes.

```haskell
data Either a b = Left a | Right b 

-- Exemplo.
divEither :: Int -> Int -> Either String Int
divEither _ 0 = Left "Divisão por zero."
divEither m n = Right (div m n)
```

### Keyword ``newtype``

- Permite apenas um construtor
- ``newtype`` define um novo tipo, ``type`` define apenas um sinônimo.
- ``newtype`` é um novo tipo até ser compilado e depois substituido como um sinonimo, garantindo a checagem em tempo de execução. 
```haskell
newtype = Nat = N Int
```

## Tipos Recursivos

### Tipo Arvore Binária


```haskell
-- Nó pode ser folha ou conter valor e arvores.
-- > Node l v r
data Tree a = Leaf a | Node (Tree a) a (Tree a)
```

### Record Type
- Forma alternativa de definição para tipo produto.

```haskell
-- Cria automaticamente as funções ``coordX`` e ``coordY`` para fazer o retorno dos valores.
data Ponto2D = Ponto {coordX :: Double,
            , coordY :: Double}
```
---
## Álgebra dos Tipos


```haskell
data Zero -- Nenhum valor. Não podemos fazer a passagem desse tipo para uma função

data Um = () -- Possui um unico valor.
```

- Tipo ``Zero`` é conhecido como ``Void`` e ``Um`` como ( ) (``unit``).

- ``Either`` representa a soma de tipos.
- ``Pair`` representa a operação de produto.


```haskell
data Pair a b = (a, b)
```

- Tipo ``Either Void ()`` : Soma de 0 e 1, apenas um único valor possivel.
- Exemplo: Tipo ``Bool = False | True`` &rarr; ``Either () ()`` &rarr; Soma de dois valores unitários.

### ***Tipo ``Either`` define a soma de dois tipos ou a união disjunta do conjunto definido por dois tipos.***

- Todo tipo soma pode ser definido por encadeamento ``Either``.


```haskell
-- Exemplo
data Cod_prod = Either Char Char --> 256 + 256 = 512
```

- Utilizando ``Pair`` . . .

```haskell
data ZeroVzsUm = Pair Void () -- Nenhum valor possivel, 0 x 1 = 0.

data UmvzsUm = Pair () () -- Um valor possivel, 1 x 1 = 1.
```

- Todo tipo produto pode ser construido genericamente com um ``Pair``.

```haskell
-- Exemplo
data vCod = Pair Char Bool -- 256 x 2 = 512.
```

- Propriedades algébricas funcionam para os ADTs (***Algebric Data Type***  's)
 
- Funções caracterizam um tipo exponencial

```haskell
-- Exemplo
f :: a -> b -- b^a combinações. 

f :: Bool -> a -- a^2 combinações. 
```

- Tipos das funções restringem suas possiveis implementações &rarr; **alguns erros de implementação podem ser verificados em tempo de compilação** . 
- Funções parametrizadas são ainda mais restritas.
---
## Zippers

### Para Listas

```haskell
data List a = Empty | Cons a (List a)
```
- Para notação algébrica: ``List a`` = *L ( A )*
- *L ( A ) = 1 + a . L ( A )*
- Rearranjando, *L ( A ) = 1 / ( 1 - a )*
- A série de Taylor (*1 + a + a^2 + a^3 + . . .*) da expressão acima repsenta o fato de que uma lista guarda informação sobre 0 elementos (``Empty``) ou de 1 elemento, ou de 2 elementos, etc ...
- Derivando o tipo *L ( A )* temos *L' ( A ) = (1 / (1 - a)) . (1 / (1 - a))*
- Então, a derivada de uma lista são duas listas.

```haskell
type DiffList a = Pair (List a) (List a)
```
- Estrutura conhecida como ``Zipper`` e traz diversos beneficios para o uso de estruturas **atravessáveis**.

```haskell
-- Forma de caminhar para frente e para trás na lista com tempo constante.
data Zipper a = Z [a] [a] deriving Show

focus :: Zipper a -> a
focus (Z _ [])     = error "Lista vazia!"
focus (Z _ (x:xs)) = x

walkRight, walkLeft :: Zipper a -> Zipper a
walkRight (Z lft (x:rgt)) = Z (x:lft) rgt 
walkLeft  (Z (x:lft) rgt) = Z lft (x:rgt)

-- Exemplo

> let lista = [1, 2, 3, 4]

> putStrLn $ show $ walkRight (Z [] lista)
Z [1] [2,3,4]

> putStrLn $ show $ focus (walkRight (Z [] lista)) 
2
```


### Para Árvores
- Derivando 
```haskell 
data Tree a = Leaf | Node { left  :: Tree a
			  , node  :: a
			  , right :: Tree a
			  }
```
- . . . extraimos:

```haskell
-- o foco é composto pelo elemento, a direção de onde veio e a árvore acima deste foco.
data From     = Lft | Rgt
data Zipper a = Zipper { left  :: Tree a
		               , right :: Tree a
		               , focus :: [(From, a, Tree a)]
		               }
```
- . . .  e gerando . . .
```haskell
data Zipper a = Zipper { focus :: Tree a
		               , histo :: [Either (Tree a) (Tree a)]
		               } deriving Show
```
- ``Tree`` para ``Zipper``, colocando ramos da esquerda e da direita nos seus respectivos campos e criando uma lista com o nó raiz.
```haskell
toZipper :: Tree a -> Zipper a
toZipper Leaf = Zipper Leaf []
toZipper    n = Zipper    n []
```

- ``Zipper`` para ``Tree``, com foco na raiz.
```haskell
fromZipper :: Zipper a -> Tree a
fromZipper tz =
  case histo tz of
             []   -> focus tz
              _   -> fromZipper (goUp tz)
```
- ``goLeft`` e ``goRight`` para árvores.
```haskell
goLeft :: Zipper a -> Zipper a
goLeft tz =
  case focus tz of
    Node Leaf _ _ -> tz
    Node l x r    -> Zipper l (Left (Node Leaf x r) : histo tz)

goRight :: Zipper a -> Zipper a
goRight tz =
  case focus tz of
    Node _ _ Leaf -> tz
    Node l x r    -> Zipper r (Right (Node l x Leaf) : histo tz)
```
- Para "subir".

```haskell
goUp :: Zipper a -> Zipper a
goUp (Zipper f [])     = Zipper f []
goUp (Zipper f (h:hs)) =
  case h of
    Left  (Node _ x r) -> Zipper (Node f x r) hs
    Right (Node l x _) -> Zipper (Node l x f) hs
```
---

## Classes de Tipo

- Definem grupos de tipos, que devem conter algumas funções especificadas.
- Keyword ``class``.

```haskell
-- Para um tipo a pertencer a Eq deve ter uma implementação de (==) e (/=).
class Eq a where
    (==), (/=) :: a -> a -> Bool
    x /= y = not (x == y)
```


### Instâncias da Classe

- Keywords ``instance ... where`` .
- Apenas tipos definidos com ``data`` e ``newtype`` podem ser instâncias de alguma classe.
- Classe em Haskell &rarr; coleção de tipos.
- Instância de uma Classe em Haskell &rarr; pertence àquela coleção de tipos.

```haskell
instance Eq Bool where
    False == False = True
    True  == True  = True
        _ ==     _ = False
```

- Tipo: coleção de valores reacionados.
- Classe: coleção de tipos (...).
- Métodos: funções requisitos de uma classe.
- Instância: tipo que pertence a uma classe.

### Algumas Classes...

#### ``Eq``
- Tipos comparáveis por igualdade e desigualdade.
```haskell
(==) :: a -> a -> Bool
(/=) :: a -> a -> Bool
```
#### ``Ord``
- Tipos ordenáveis.
```haskell
(<) :: a -> a -> Bool
(<=) :: a -> a -> Bool
(>) :: a -> a -> Bool
(>=) :: a -> a -> Bool
min :: a -> a -> a
max :: a -> a -> a
```
#### ``Show``
- Tipos imprimíveis.
```haskell
show :: a -> String
```
#### ``Read``
- Tipos legíveis.
- Como fazer a tradução para um tipo, de uma ``String`` .
```haskell
read :: String -> a

(...)

> read "12.5" :: Double
> read "[1,3,4]" :: [Int]
```
#### ``Num``
- Tipos numéricos.
- ***Valores negativos devem ser escritos entre parenteses para remover ambiguidades com o operador subtração.***
```haskell
(+) :: a -> a -> a
(-) :: a -> a -> a
(*) :: a -> a -> a
negate :: a -> a
abs :: a -> a
signum :: a -> a
fromInteger :: Integer -> a
```
#### ``Integral``
- Tipos numéricos inteiros.
- ``quot`` e ``rem`` arredondam para 0.
- ``div`` e ``mod`` arredondam para -∞.
```haskell
quot :: a -> a -> a
rem :: a -> a -> a
div :: a -> a -> a
mod :: a -> a -> a
quotRem :: a -> a -> (a, a)
divMod :: a -> a -> (a, a)
toInteger :: a -> Integer
```

#### ``Fractional``
- Tipos numéricos fracionários.
```haskell
(/) :: a -> a -> a
recip :: a -> a

(...)

> recip 10
0.1
```

#### ``Floating``
- Tipos numéricos de ponto flutuante.
```haskell
class Fractional a => Floating a where
    pi :: a
    exp :: a -> a
    log :: a -> a
    sqrt :: a -> a
    (**) :: a -> a -> a
    logBase :: a -> a -> a

    sin :: a -> a
    cos :: a -> a
    tan :: a -> a
    asin :: a -> a
    acos :: a -> a
    atan :: a -> a
    sinh :: a -> a
    cosh :: a -> a
    tanh :: a -> a
    asinh :: a -> a
    acosh :: a -> a
    atanh :: a -> a
```

#### ``Enum``
- Tipos que são enumeráveis.
```haskell
succ, pred, toEnum, fromEnum
```

### Derivação de Instâncias
- Keyword ``deriving``, já visto anteriormente.

### Info

- No ghci, digitar ``:info`` para informações sobre classes e tipos.

````haskell
> :info Integral
(...)

> :info Bool
(...)
````
