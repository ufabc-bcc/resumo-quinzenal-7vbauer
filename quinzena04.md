# Resumo 4 - Paradigmas de Programação
## Nome: Vitor Marini Blaselbauer
## RA: 11048115

---

- "(...) o uso de Applicative é para sequências de computações que podem ter efeitos mas que são independentes entre si."
- Queremos agora uma sequência de computações com efeito, mas que uma computação dependa da anterior.
```haskell
pure maybeDiv <*> eval x <*> eval y
```
- Pegamos o resultado de ``mx`` para aplicar em ``g``.

```haskell
f mx g = case mx of
            Nothing -> Nothing
            Just x  -> g x
```

---

### Monoid de Monad

- Um Monad pode ser visto como um Monoid das categorias dos Functors
- Functors e Monoids: 
```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b -- f de functor

class Monoid a where
    mempty  :: a                   -- Elemento identidade
    mappend :: a -> a -> a         -- Como combinar dois elementos
    ...
```


---
### Monads

- "(...) permitem uma generalização de sequenciamento de computação quando precisamos de funções que geram efeitos colaterais (...)"
- Como manter a pureza precisando imprimir no console, salvar arquivos, etc ... ? &rarr; ``Monads!!``
- "Operações sequenciais que transportam o efeito"
- ``Monad`` é um ``Applicative`` quando você leva em conta a sequência.
- 1. Elemento identidade ``return``
- 2. Operador associativo ``>=>``, variação de ``>>=``.

- Operador ``bind (>>=)``
```haskell
    (>>=) ::     m a -> (a ->     m b) ->     m b
--  (>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
```
- Operador ``fish (>=>)``
- Duas funções que transformam um valor puro em um Monad podem ser combinadas formando outra função.
```haskell
    (>=>) :: Monad m => (a -> m b) -> (b -> m c) -> (a -> m c)
```

```haskell
m1 >>= \x1 ->
m2 >>= \x2 ->
...
mn >>= \xn ->
f x1 x2 ... xn

-- ou

do x1 <- m1
    x2 <- m2
    ...
    xn <- mn
    f x1 x2 ... xn
```

```haskell
class Applicative m => Monad m where
   return :: a -> m a
   (>>=)  :: m a -> (a -> m b) -> m b

   return = pure
```
### Exemplos
```haskell
> [1,2,3] >>= \x -> [4,5,6] >>= return (x,y)
[(1,4), (1,5), (1,6), (2,4), (2,5), (2,6), (3,4), (3,5), (3,6)]

> [1,2,3] >>= \x -> [4,5,6] >>= return (y)
[1, 1, 1, 2, 2, 2, 3, 3, 3] 

> [1,2,3] >>= \x -> [4,5,6] >>= return (x)
[4, 5, 6, 4, 5, 6, 4, 5, 6]
```

### Parcialidade
- Funções que podem não terminar.
- Valor \_|_ Bottom, sinalização de algo que pode falhar.

### Não determinismo
```haskell
instance Monad [] where
    >>= :: [a] -> (a -> [b]) -> [b]
    xs >>= f =  concat (map f xs)

-- a função map:
(a -> b) -> [a] -> [b]

pares xs ys = do
        x <- xs
        y <- ys
        return (x, y)

-- Que é igual a ...
pares xs ys = [(x, y) | x <- xs, y <- ys]

-- Que também é ...
pares xs ys = 
    xs >>= \x ->
    ys >>= \y -> 
    return (x, y)
    
-- e ...
pares xs ys = xs >>= \x -> ys >>= \y -> return (x, y)
```
- ``List Comprehension`` é um ``Syntax sugar``  do ``Do-notation`` que é um ``Syntax sugar`` do ``Bind``

- **``pure`` e ``return`` são a mesma coisa, só são usados em contextos diferentes.**

### Exceções Either

- ``Maybe`` apenas indica que houve um problema, mas não dá mais informações.
- Tipo ``data Either a b = Left a | Right b``
- ``Left`` = log do erro.
- ``Right`` = valor esperado
- Instância de ``Functor``, ``Applicative`` e ``Monad``:

```haskell
-- Só aplica a função se for Right
instance Functor (Either a) where
    fmap f (Left x) = Left x
    fmap f (Right x) = Right (f x)

-- Propaga os valores de Left, e aplica a função em Right.
instance Applicative (Either a) where
  pure x = Right x

  (Left f)  <*> _         = Left f
  _         <*> (Left x)  = Left x
  (Right f) <*> (Right x) = Right (f x)

--
instance Monad (Either a) where
    (Left x)  >>=  f  = Left x
    (Right x) >>=  f  = f x

```
### Efeitos Colaterais: Read Only
- ``postTweet :: (String, Config) -> String`` , pode ser escrito como:
- ``postTweet :: String -> Config -> String`` , que pode ser escrito como:
- ``postTweet :: String -> (Config, String)`` , e podemos criar o tipo ``Reader`` e reescrever como:
```haskell
data Reader e a = Reader (e -> a)
postTweet :: String -> Reader Config String
```
- Instâncias de ``Functor``, ``Applicative`` e ``Monad``:

```haskell 
-------------------------- Functor ----------------------------
instance Functor (Reader e) where
    -- fmap :: (a -> b) -> Reader e a -> Reader e b
    fmap f (Reader g) = Reader (f.g)

------------------------ Applicative --------------------------
-- runReader (Reader f) e = f e
    
instance Applicative (Reader e) where
    -- pure :: a -> Reader e a
    pure x = Reader (\e -> x)

    -- (<*>) :: Reader e (a -> b) -> Reader e a -> Reader e b
    rab <*> ra = Reader (\e -> ab e (a e))
        where
            ab = runReader rab -- :: e -> (a -> b)
            a  = runReader ra  -- :: e -> a
    -- ra  = Reader e a
    -- rab = Reader e (a -> b)
    -- ab  = Execução do runReader com Reader e (a -> b)
    -- a   = Execução do runReader com Reader e a
--------------------------- Monad -----------------------------
instance Monad (Reader e) where
    -- (>>=) :: Reader e a -> (a -> Reader e b) -> Reader e b
    -- arb = Pega um a e gera um Reader de b (Reader e b).
    ra >>= arb =
    Reader (\e -> let a  = runReader ra e -- a
                rb = arb a -- Reader e b
            in  runReader rb e)
---------------------------------------------------------------
```

### Efeitos Colaterais: Write Only
- Tendo as funções:
```haskell 
isEven x = even x
not'   b = not b
isOdd  x = (not' . isEven) x
```
- Se quisermos gravar o percurso das funções:
```haskell 
isEven x = (even x, " isEven ")
not'   b = (not b, " not' ")

-- trace1 e trace2 são os percursos, b1 e b2 os valores.
isOdd  x = let (b1, trace1) = isEven x
               (b2, trace2) = not' b
                 in  (b2, trace1 ++ trace2)
```
- Porém não podemos usar mais a composição ``(.)``.
- As funções seguem o formato ``a -> (b, String)``. Com ``m = (, String)`` temos ``a -> m b``.
- Podemos criar o tipo ``Writer``:
```haskell 
data Writer w a = Writer a w

runWriter :: Writer w a -> (a, w)
runWriter (Writer a w) = (a, w)

tell :: w -> Writer w ()
tell s = Writer () s
```
- ``runWriter``, análogo ao ``runReader``.
- O  ``w`` indica o tipo do efeito colateral desejado. Nos exemplos, o percurso das funções é o tipo ``String``.

- Instâncias de ``Functor``, ``Applicative`` e ``Monad``: 


```haskell 
---------------------- Functor ----------------------------------
instance (Monoid w) => Functor (Writer w) where
 -- fmap :: (a -> b) -> Writer w a -> Writer w b
    fmap f (Writer a w) = Writer (f a) w

-------------------- Applicative --------------------------------
instance (Monoid w) => Applicative (Writer w) where
    -- pure :: a -> Writer w a
    pure x = Writer x mempty
    -- (<*>) :: Writer w (a -> b) -> Writer w a -> Writer w b
    (Writer f m1) <*> (Writer a m2) = Writer (f a) (m1 <> m2)

----------------------- Monads ----------------------------------
instance (Monoid w) => Monad (Writer w) where
    -- (Writer w a) -> (a -> Writer w b) -> (Writer w b)
    (Writer a w) >>= k = let (b, w') = runWriter (k a)
                in  Writer b (w <> w')
-----------------------------------------------------------------
```
- Utilizando o operador ``fish >=>`` para composição de ``Monads`` podemos definir ``isOddW'`` mais facilmente:
```haskell 
isOddW' :: Integer -> Writer String Bool
isOddW' = (isEvenW' >=> notW')
```


### Read-Write

#### Estado

- Problema: Temos um tipo árvore ``Tree Char`` e queromos converter para ``Tree Int``, com os nós folhas recebendo números ``[0..]``, na sequencia em que são visitados.

```haskell 
data Tree a = Leaf a | Node (Tree a) (Tree a)
	      deriving Show

rlabel :: Tree a -> Tree Int
rlabel (Leaf _)   = Leaf n
rlabel (Node l r) = Node (rlabel l) (rlabel r)

```
- ... onde ``n`` seja uma variavel de estado. Sempre que a utilizarmos, ela deve ter seu estado alterado.
- Com a pureza e a imutabilidade da programação funcional, podemos incorporar o estado atual na declaração da função.
```haskell
rlabel :: Tree a -> Int -> (Tree Int, Int)
rlabel (Leaf _) n = (Leaf n, n + 1)
rlabel (Node l r) = (Node l' r', n'')
    where
        (l', n')   = rlabel l n  -- altera o estado de n
        (l'', n'') = rlabel r n' -- altera o estado de n'
```

- ``Reader a -> b``
- ``Writer (a, b)``
- ``Tree a -> Int -> (Tree Int, Int)`` para &rarr;  ``t -> (a -> (b, a))`` para  &rarr; ``t -> Reader a (Writer a b)``. 

### Transformador de Estado

- Uma caixa que recebe um estado ``s`` e retorna uma valor ``v`` e um novo estado ``s'``.
- Uma caixa que recebe um estado ``s`` e um valor ``c`` para agir no ambiente em que vive, e retorna uma valor ``v`` e um novo estado ``s'``

```haskell
            -- = Reader s (Writer s a)
data State s a = State (s -> (a, s))

rlabel :: Tree a -> Int -> (Tree Int, Int)
rlabel :: Tree a -> State Int (Tree Int)
```
- ``runState`` aplica um transformador de estado em um estado.
- ``runState :: State s a -> s -> (a, s)``
- ``runState    (State f) s = f s``

- ``Functor ST`` define como aplicar uma função pura do tipo ``(a -> b)`` na parte do valor do resultado de um ``ST a``, transformando-o em um ``ST b``.

- Instâncias de ``Functor``, ``Applicative`` e ``Monad``:
```haskell 
instance Functor (State s) where
  -- fmap :: (a -> b) -> State s a -> State s b
  fmap f (State g) = State (\s -> let (a, s') = g s
				                     in  (f a, s'))

instance Applicative (State s) where
-- pure :: a -> State s a
  pure x = State (\s -> (x, s)) 
   - (<*>) :: State s (a -> b) -> State s a -> State s b
  sab <*> sa = State (\s -> let (f, s1) = runState sab s
				                (a, s2) = runState sa  s1
			                        in  (f a, s2))

instance Monad (State s) where
  -- (>>=) :: State s a -> (a -> State s b) -> State s b
  sa >>= f = State (\s -> let (a, s1) = runState sa s
                                   sb = f a
			                        in  runState sb s1)
```
### Efeitos Colaterais IO

- **Funções de Entrada e Saída são impuras !**
- ``getChar`` : captura um caracter do teclado.
- ``putChar`` : imprime um caracter na saída padrão.
- Funções de ``I/O`` alteram estado (``newtype IO a = newtype ST a = State -> (a, State)``) com estado sendo ``type State = Environment``.
- ``Environment`` : Sistema operacional, mundo computacional, etc ...
```haskell
getchar :: IO Char           -- 
putChar :: Char -> IO ()     -- 
getLine :: IO String         -- 
return  :: a -> IO a         -- valor puro envolvendo 'a' em uma ação IO

do putChar 'a' --> (_, env')  = putChar 'a' env
   putChar 'a' --> (_, env'') = putChar 'a' env'
```

### Funções de Alta Ordem para Monads

- ``Control.Monad`` : funções de alta ordem com versões para ``Monads``.

```haskell
> mapM conv "1234"
Just [1,2,3,4]

> map conv "1234"
[Just 1, Just 2, Just 3, Just 4]

> mapM conv "12a4"
Nothing

> map conv "12a4"
[Just 1, Just 2, Nothing, Just 4]

> filterM (\x -> [True, False]) [1,2,3]
[[1,2,3],[1,2],[1,3],[1],[2,3],[2],[3],[]]

{--

Para cada elemento da lista ele cria duas ramificações: uma em que aquele elemento existe, e outra que ele não existe.

      []
    /    \
   [1]     []
  /  \    /   \
[1,2] [1] [2] []
...

--}


> filter (\x -> [True, False]) [1,2,3]
Erro.

{--
f x | even x = [False]
    | otherwise = [True, False]
--}

> filterM f [1, 2, 3]
[[1,3],[1],[3],[]]

```
- ``filter`` : seleciona elementos de uma lista.
- ``filterM`` : seleciona possiveis eventos de uma sequencia de computações.

