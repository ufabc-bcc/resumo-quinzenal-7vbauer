# Paradigmas de Programação - Resumo 3
## Nome : Vitor Marini Blaselbauer
## RA: 11048115

--- 
## Semana 05 - Monoid e Foldable, Functor.

### Monoid

- É um conjunto de valores com um operador binário associativo (``mappend`` ou ``<>``) e um elemento identidade (``mempty``).

```haskell
class Monoid a where
    mempty  :: a                   -- Elemento identidade
    mappend :: a -> a -> a         -- Como combinar dois elementos
    mconcat :: [a] -> a            
    mconcat = foldr mappend mempty -- Como resolver uma lista de elementos
```
- Definindo instancias de ``Monoid`` para ``[]`` e ``Maybe``.

```haskell
instance Monoid [a] where
    mempty = []
    mappend = (++)


instance Monoid a => Monoid (Maybe a) where -- 
    mempty = Nothing

    Nothing  `mappend`   my      = my
    mx       `mappend`   Nothing = mx
    Just x   `mappend`   Just y  = Just (x `mappend` y)
```

- Podemos fazer uma generalização, em ``fold``:
```haskell
fold :: Monoid a => [a] -> a     -- Monoid a => t a -> a, onde t é lista e "a" é o tipo dos elementos.
fold []     = mempty              
fold (x:xs) = x `mappend` fold xs
```

#### ``Foldable``

- Classe dos tipos dobráveis.
- Uma instância de ``Foldable`` deve ter pelo menos as implementações de ``foldMap`` e ``foldr``.
- **A importância dos ``Monoids`` está na generalização em como combinar uma lista de valores de um tipo que pertença a essa classe (``Monoid``) &rarr; ``Foldable``**.

```haskell
class Foldable t where
    fold     :: Monoid a => t a -> a                 
    foldMap  :: Monoid b => (a -> b) -> t a -> b
    foldr    :: (a -> b -> b) -> b -> t a -> b
    foldl    :: (a -> b -> a) -> a -> t b -> a
```
- ``foldMap`` &rarr; *Map each element of the structure into a monoid, and combine the results with (<>)* - Hackage Haskell

- Exemplo de ``foldMap`` para ``Sum``


```haskell
-- Definindo o tipo.
newtype Sum a = Sum a
   deriving (Eq, Ord, Show, Read)

-- Criando uma instância de Monoid para o tipo em questão.
instance Num a => Monoid (Sum a) where
   mempty = Sum 0
   Sum x `mappend` Sum y = Sum (x+y)

getSum (foldMap Sum [1..10])
> 55
```

- **MapReduce** = Modelo de programação. É a função ``foldMap``
- **Map** (``map``) é feito em paralelo e de maneira distribuida, e o **Reduce** corresponde ``fold``.
- Utilzação de ``Monoids``!




### Functor 

- São funções que fazem com que as funções de um certo tipo sejam aplicáveis a um tipo paramétrico contendo esse tipo.
- Se temos uma função ``g`` (``a`` &rarr; ``b``) e um tipo paramétrico ``f``, podemos fazer a aplicação de ``g`` em ``f a`` e obter ``f b``.
- Operador ``Functor`` = ``<$>``

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b -- f é de functor
    -- ou
    -- Tipo paramétrico f que armazena zero ou mais valores do tipo "a", e que
    -- toda função a -> b pode ser mapeada para uma função f a -> f b
    fmap :: (a -> b) -> (f a -> f b)

-- fmap deve seguir as propriedades:
fmap id = id
fmap f . fmap g = fmap (f.g)
```

- Exemplo, ``Functor`` para ``[]``, ``Maybe`` e ``Tree`` :

```haskell
instance Functor [] where
    fmap = map

instance Functor Maybe where
    fmap _ Nothing   =  Nothing
    fmap g (Just x)  = Just (g x)

instance Functor Tree where
    fmap g (Leaf x)     = Leaf (g x)
    fmap g (Node l x r) = Node (fmap g l) (g x) (fmap g r)
```
- Por exemplo, uma função para os inteiros, podemos usá-la para o tipo ``Maybe`` com ``fmap``.

```haskell
> fmap chr (Just 65)
Just 'A'
> fmap (+1) (Just 65)
Just 66
```

#### Aritmética Segura com ``Functor``


```haskell
data SafeNum a = NaN | NegInf | PosInf | SafeNum a      deriving Show

{--

NaN -> Casos de raiz de negativo, 0 dividido por 0, ...
NegInf -> Casos de Overflow.
PosInf -> ''     ''      ''.  
SafeNum -> Divisão segura.

--}

-- signum :: Num a => a -> a   -- 1 para positivos, 0 para zero, -1 para negativos

safeAdd :: Int -> Int -> SafeNum Int
safeAdd x y
    | signum x /= signum y = SafeNum z
    | signum z /= signum x = if signum x > 0
                then PosInf
                else NegInf
    | otherwise = SafeNum z
    where z = x + y

safeDiv :: Int -> Int -> SafeNum Int
safeDiv 0 0 = NaN
safeDiv x 0
    | x > 0     = PosInf
    | otherwise = NegInf
safeDiv x y = SafeNum $ x `div` y

> 0 `safeDiv` (-1)
SafeNum 0
> (-1) `safeDiv` 0
NegInf
> 0 `safeDiv` 0
NaN

-- negate :: Num a => a -> a           -- muda o sinal do número.
> negate <$> z
SafeNum (-12)
> negate <$> safeDiv 10 0
PosInf
```

- Implementando a instância de ``Functor`` para ``SafeNum``.

```haskell
boxedCoerce :: SafeNum a -> SafeNum b
boxedCoerce NaN      = NaN
boxedCoerce NegInf   = NegInf
boxedCoerce PosInf   = PosInf
boxedCoerce      _   = error "Não deveria ser usado para valores safe"

instance Functor SafeNum where
    fmap f (SafeNum n) = SafeNum $ f n
    fmap _ x           = boxedCoerce x


flatten :: SafeNum (SafeNum a) -> SafeNum a
flatten (SafeNum sn) = sn
flatten v            = boxedCoerce v

f1 :: Int -> Int -> SafeNum Int
f1 x y =
    let xy = safeDiv x y
        yx = safeDiv y x
        safeAddXY = fmap safeAdd xy
        safeXYPlusYX = fmap (`fmap` yx) safeAddXY
    in
        (flatten.flatten) safeXYPlusYX
```

## Semana 06 - Applicative e Traversable

### Aplicatives

- Ou Applicative Functors...

```haskell
class Functor f => Applicative f where
    pure  :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b

-- Exemplos
> pure (+) <*> [1,2] <*> [3,4]
[4,5,5,6]

-- Realiza a aplicação de (+) no tipo interno e depois
-- encapsula o resultado no tipo original, com pure.
> pure (+) <*> (Just 3) <*> (Just 2)
Just 5
```
- Instâncias de ``Applicative`` para ``Maybe``, ``[]`` e ``SafeNum``:
```haskell
instance Applicative [] where
    pure x    = [x]
    gs <*> xs = [g x | g <- gs, x <- xs]

instance Applicative Maybe where
    pure             = Just
    Nothing  <*> _   = Nothing
    (Just g) <*> mx  = fmap g mx

instance Applicative SafeNum where
    pure = SafeNum
    f <*> x = flatten $ fmap (`fmap` x) f

-- Exemplos

-- Para cada elemento aplica a função, e retorna no tipo original (lista).
> pure (+1) <*> [1,2,3]
[2,3,4]
> pure (*20) <*> [2,3,4]
[20,60,80]
> pure (+) <*> [1] <*> [2]
[3]
```

- No caso de uma função ``g :: a -> Maybe a `` a ser aplicada em ``[g x1, g x2, g x3]``, na avaliação preguiçosa cada elemento será avaliado em ordem arbitrária, dependendo da função. Não queremos seguir computando no caso de falhas, portanto podemos construir uma lista de ``Applicative``:

```haskell
pure (:) <*> g x1 <*> (pure (:) <*> g x2 <*> (pure (:) <*> g x3 <*> pure []))
```
- Caso falhe, podemos retornar ``Nothing`` imediatamente.

```haskell
sequenceA :: (Applicative f) => [f a] -> f [a]
sequenceA     []  = pure []
sequenceA (x:xs)  = pure (:) <*> x <*> sequenceA xs

-- Temos:
> sequenceA [Just 3, Just 2, Just 1]
Just [3,2,1]

> sequenceA [Just 3, Nothing, Just 1]]
Nothing

> sequenceA [[1,2,3],[4,5,6]]
[[1,4],[1,5],[1,6],[2,4],[2,5],[2,6],[3,4],[3,5],[3,6]]
```
- ***Sequenciamento é útil quando queremos ter controle da ordem das operações, que podem falhar ou gerar efeitos colaterais.***

### Traversable

- Para quando temos uma função que mapeia um tipo ``a`` para ``Maybe b``, por exemplo, e temos ``[a]``. Queremos retornar um ``Maybe [b]`` ao invés de ``[Maybe b]``. Possivel com ``Applicative`` para listas.

```haskell
class (Functor t, Foldable t) => Traversable t where
    traverse :: Applicative f => (a -> f b) -> t a -> f (t b)
    
instance Traversable [] where
    traverse g     [] = pure []
    traverse g (x:xs) = pure (:) <*> g x <*> traverse g xs

-- Exemplos:
-- dec :: Int -> Maybe Int
-- dec x | x <= 0    = Nothing
--       | otherwise = Just (x - 1)

> traverse dec [1,2,3]
Just [0,1,2]

> traverse dec [2,1,0]
Nothing
```
# Resumo
- ``Monoid`` = Tipo com um conjunto de valores, um elemento identidade e um operador binário.
- ``Foldable`` = Tipo dobrável, usando pelo menos ``foldr`` e ``foldMap``.
```haskell
--     "t a" é um tipo parametrico contendo "a", e depois do fold temos "a", um Monoid.
--     Combinamos esses elementos utilizando mempty e mappend.
fold     :: Monoid a => t a -> a                 
--     Mapeamos os elementos através de uma função (a->b) de a para um Monoid b, e combinamos
--     os elementos através de <>.
foldMap  :: Monoid b => (a -> b) -> t a -> b
```
- ``Functor`` = Quando temos uma função de ``a -> b``, queremos ``t b`` mas temos ``t a``. Utilizamos a função ``fmap :: (a -> b) -> f a -> f b, Functor f``. 
- ``Applicative`` = Dado um ``Functor f``, definimos ``pure`` e ``<*>``. 
```haskell
--  Usamos pure para retornar ao tipo paramétrico.
--  Usamos o operador <*> com uma função a->b e um Functor, para ir de f a para f b.
pure  :: a -> f a
(<*>) :: f (a -> b) -> f a -> f b
-- Exemplo
pure (+) <*> (Just 3) <*> (Just 2) -- Just 5
``` 
- ``Traversable`` = Usando um tipo ``Functor`` e ``Foldable``, para esta classe implementamos a função ``traverse``. Para quando temos `(a -> f b)` e ``t a``, e queremos ``f (t b)``. 

## Resumo do Resumo

- ``Monoid`` &rarr; ``mempty`` e ``mappend ou <>``.
- ``Foldable`` &rarr; ``fold(r)`` e ``foldMap``.
- ``Functor`` &rarr; ``fmap ou <$>``.
- ``Applicative`` &rarr; ``pure`` e ``<*>``.
- ``Traversable`` &rarr; ``traverse``.

### Resumo do Resumo do Resumo

- Apontando algumas relações entre as classes:
- ``Monoid`` &rarr; ``Foldable`` 
- ``Functor`` &rarr; ``Applicative``
- (``Functor``, ``Foldable``, ``Applicative``) &rarr; ``Traversable``
