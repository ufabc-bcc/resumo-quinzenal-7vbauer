# Resumo 5 - Paradigmas de Programação
## Nome: Vitor Marini Blaselbauer
## RA: 11048115

---

## Semana 9 - Avaliação Preguiçosa

- Em Haskell, utilizando avaliação preguiçosa, não é realizada a avaliação da expressão mas sim colocada uma referência para a expressão, e ela só será avaliada quando for utilzada durante a execução.
- Função ``:sprint``.
- Expressões não avaliadas: ``thunk``.


```haskell
x = 5 + 10
:sprint x
> x = _

x
> 15

:sprint x
> x = 15
```
- Função de avaliar expressões (só até o primeiro nivel): ``seq :: a -> b -> b`` ,  ``seq x ()``.
- **Weak Head Normal Form (WHNF)** = Forma Normal Fraca.
- ``:force`` = Forma Normal Completa.
- No ghci ``:force ``.
- No programa: ``import Control.DeepSeq``, ``force``.


```haskell
--  Versão utilizando tempo linear, pois explora a avaliação preguiçosa do Haskell.
fibos :: [Integer]
fibos = 0 : 1 : zipWith (+) fibos (tail fibos)

-----------------------------------------------------------------------------------
-- Expressões calculadas uma vez são memorizadas e reaproveitadas se necessário futuramente.
-- Todo número primo maior que 5 é na forma 6k +- 1
primos :: [Int]
primos =
  2 : 3 : filter ePrimo candidatos
  where
    candidatos =  junta [5,11..] [7,13..] -- Lado esquerdo é 6k -1, lado direito é 6k + 1
    junta (a:as) (b:bs) = a : b : junta as bs -- Construimos uma lista única alternando a inserção dos dois lados em uma mesma lista.

ePrimo :: Int -> Bool
ePrimo x =
  all (\e -> x `rem` e /= 0) possiveisFatores
  where
    maxDiv = ((floor . sqrt) :: Double -> Int) $ fromIntegral x -- raiz é definida só para números de ponto flutuante, pra isso o fromIntegral.
    possiveisFatores = takeWhile (<= maxDiv) primos
```

### Haskell Paralelo

```cabal 
cabal 
      build-depends: 
        parallel, deepseq, time

      ghc-options:   
        -02 (Otimização ligada)
        -threaded (múltiplos threads) 
        -rtsopts (embute opções no programa) 
        -with-rtsopts=-N (número de threads) 
        -eventlog (criação de um log do uso de threads)
```
- Compilar com ``stack build --profile`` e executar com ``stack exec <projeto> --RTS -- +RTS -N1`` .
- Tipo de dados ``Eval a``, container, Monad.
- ``runEval`` (com Forma Normal Fraca), ``rpar`` e ``rseq`` .
- Utilizamos ``rpar`` e ``rseq`` para colocar expressões (**thunk**) dentro do ``Eval`` .
- ``rpar`` = vai começar a execução de uma expressão mas te devolve o controle em seguida.
- ``rseq`` = vai esperar a avaliação da expressão para devolver o controle.
- Função ``evaluate`` força a avaliação para a forma normal fraca.

```haskell
fib :: Integer -> Integer
fib 0 = 0
fib 1 = 1
fib n = fib (n - 1) + fib (n - 2)

main :: IO ()
main = do
    t0 <- getCurrentTime
    -- r  <- evaluate (runEval fparpar)
    -- r  <- evaluate (runEval fparseq)
    -- r  <- evaluate (runEval fparparseq)
    t1 <- getCurrentTime
    print (diffUTCTime t1 t0)
    print r 
    t2 <- getCurrentTime
    print (diffUTCTime t2 t0)


fparpar :: Eval (Integer, Integer)
fparpar = do 
        a <- rpar (fib 41)
        b <- rpar (fib 40)
        return (a, b)

-- =========================== fib 41
-- ==================          fib 40
-- ^ runeval

fparseq :: Eval (Integer, Integer)
fparseq = do 
        a <- rpar (fib 41)
        b <- rseq (fib 40)
        return (a,b)

-- =========================== fib 41
-- ==================          fib 40
--                  ^ runeval

fparparseq :: Eval (Integer, Integer)
fparparseq = do 
            a <- rpar (fib 41)
            b <- rpar (fib 40)
            rseq a
            rseq b
            return (a,b)

-- =========================== fib 41
-- ==================          fib 40
--                             ^ runeval
```
- Como organizar melhor?
- ``type Strategy = a -> Eval a``
---

- ``using :: a -> Strategy a -> a``
- ``x `using` s = runEval (s x)``
- O formato fica ``(fib 41, fib 40) using fparpar`` para realizar o cálculo com ``fparpar`` .
---
- Melhorando ainda mais...
```haskell
// Passando as estrategias (sa e sb) e as expressões (a, b)
evalPair :: Strategy a -> Strategy b -> Strategy (a, b)
evalPair sa sb (a, b) = do
                    a <- sa a
                    b <- sb b
                    return (a', b')

-- Faltando somente as expressões, usando rpar.
parPair2 :: Strategy (a, b)
parPair2 = evalPair rpar rpar

-- NFData é uma classe de tipos que podem ser totalemente avaliados.
-- Executa o x até o final e espera que ele termine.
rdeepseq :: NFData a => Strategy a
rdeepseq x = rseq (force x)

. . .
```
- ``parList`` para listas: ``fmap f xs `using` parList rseq``
- ``parMap :: (a -> b) -> [a] -> [b]``
- ``parMap f xs = map f xs `using` parList rseq ``


```haskell
-- Usando parList para calcular a média de valores de uma lista.
-- Cada elemento de xss vai ser potencialmente avaliado em paralelo.
-- Implementação com problemas.
mean :: [[Double]] -> [Double]
mean xss = map mean' xss  `using` parList rseq
    where
        mean' xs = sum xs / fromIntegral (length xs)

-- Dividimos a lista em partes menores e forçamos a execução até a forma normal completa.
meanPar :: [[Double]] -> [Double]
meanPar xss = concat medias
    where
        medias = map mean chunks `using` parList rdeepseq
        chunks = chunksOf 1000 xss
```

- ThreadScope: compilar e executar o código com o parâmetro ``-s`` para imprimir informações de tempo de execução.
- Usar ``-ls`` para no ``stack`` e em seguida ``$ threadscope media.eventlog`` para abrir uma janela com os gráficos.
- ``spark`` : promessa de algo a ser computado e que pode ser computado em paralelo, mas não necessariamente será.
- Cada elemento da lista gera um ``spark``, que são inseridos em um pool que alimenta os processos paralelos.
- - **par** = Tarefa proposta.
- - **created** = Adicionada ao pool.
- - **dud** = Avaliado, WHNF.
- - **overflow** = **Pool** de sparks cheia.
- - **converted** = Avaliado por um núcleo disponivel.
- - **fizzled** = Já foi avaliado.
- - **garbage collector** = Não foi necessário.

---
### Problemas
- Poucos sparks = Pode ser paralelizado ainda mais.
- Muitos sparks = Paralelizando demais.
- Muitos duds e fizzles = Estratégia não otimizada.
---

## Semana 10 - Concorrência
### Threads e forkIO

- Concorrente: Dado um intervalo de tempo, duas ou mais coisas avançaram.
- Varios processos compartilhando núcleos são concorrentes.
- Não implicam necessariamente em paralelismo.
- Threads foram originalmente criados para expressar concorrência.


```haskell
import Control.Concurrent
forkIO :: IO () -> IO ThreadID

import System.IO -- 
hSetBuffering stdout NoBuffering -- impressão imediata, sem buffer.

forkIO $ replicateM_ 100 (putChar 'A') -- Cria um novo thread e executa o que lhe foi passado.
replicateM_ 100 (putChar 'B')          -- Será executado logo depois, antes do forkIO terminar.
```

#### Comunicação entre Threads
```haskell
data MVar a -- Variavel mutável que contém um tipo a.

newEmptyMVar :: IO (MVar a)            -- Container vazio
newMVar      :: a -> IO (MVar a)       -- Variavel nova dentro do contexto de IO
takeMVar     :: MVar a -> IO a         -- Devolve o conteúdo de MVar em um IO
putMVar      :: MVar a -> a -> IO ()   -- Coloca 'a' em um MVar e te devolve um IO ()
```
- E a pureza ? resolvido com o ``IO`` .

```haskell
main = do
    m <- newEmptyMVar
    forkIO $ do putMVar m 'x'; putMVar m 'y'
    r1 <- takeMVar m -- espera o outro thread colocar 'x' em m, e em seguida toma o valor em r1
    print r1         -- imprime o resultado 
    r2 <- takeMVar m -- espera o outro thread colocar 'y' em m, e ...
    print r2         -- imprime o resultado
    r3 <- takeMVar m -- vai gerar erro, pois a outra thread terminou e nao há possiblidade de colocar um
    print r3         --  . . . valor em m novamente neste programa.
```

- Se uma thread chamar ``takeMVar`` e ela estiver vazia. ficará bloqueada até algum conteudo ser inserido.
- Se uma thread chamar ``putMVar`` e ela estiver cheia, ficará bloqueada em espera até que alguma thread utiliza takeMVar.

### Operações Assíncronas

```haskell
import Network.HTTP.Conduit --cabal http-conduit
import Control.Concurrent.Async

data Async a = Async (MVar a) -- do Control.Concurrent.Async

async :: IO a -> IO (Async a) -- Cria uma thread que será executada assincronamente.
async acao = do
    var <- newEmptyMVar
    forkIO $ do r <- acao; putMVar var r
    return $ Async var

wait :: Async a -> IO a
wait (Async mvar) = takeMVar mvar

. . .

-- Capturar páginas web com http de forma assincrona.
a1 <- async $ simpleHttp "..."
a2 <- async $ simpleHttp "..."

-- Aguarda assincronamente o recebimento das paginas nas MVar a1 e a2.
r1 <- wait a1
r2 <- wait a2

print (B.length r1, B.length r2)

readMVar -- Ao invez de retirar o valor do MVar, recebe o valor mas não o remove do MVar.
-- É implementada fazendo:
--                        a <- takeMVar m 
--                        putMVar m a
--                        return a

waitCatch :: Async a -> IO (Either SomeException a)

-- Sem Either
main = do
  as <- mapM (async . getURL) sites
  rs <- mapM wait as
  mapM_ (r -> print $ B.length r) rs

-- Com Either
main = do
  as <- mapM (async . getURL) sites
  rs <- mapM waitCatch as
  mapM_ printLen rs
```

### Estruturas de Dados Persistentes

- Por que é dificil desenvolver estruturas persistentes?
- - Inexistência de atualizações destrutivas;
- - Aumento de expectativa (Em uma estrutura imperativa, raramente esperamos que ela seja persistente).


---

- Estruturas de dados funcionais são sempre persistentes. 
- Atualizações em uma ED não destroem a versão antiga.
- Elementos afetados são copiados.
- Elementos não afetados são compartilhados entre as versões.

#### Estrutura Zipper
- Analogia a como uma formiga explora o mundo
- - A localização da formiga é o foco do **zipper**.
- - A formiga pode se deslocar para explorar o mundo seguindo os caminhos disponiveis.
- - Ela deixa por onde passou uma trilha de feromonio para poder retornar por onde ela veio.

```haskell
type ListZipper a = ([a], [a])

-- Começa com o foco no primeiro elemento, sem ter caminhado nada ainda.
toListZipper :: [a] -> ListZipper a
toListZipper xs = ([], xs)

-- A lista da esquerda fica na ordem inverso, por causa do historico do percurso.
-- Para voltar para a estrutura de lista deve-se reverter a ordem dessa lista da esquerda.
fromListZipper :: ListZipper a -> [a]
fromListZipper (es, ds) = reverse es ++ ds

-- Se a lista da direita está vazia, percorremos todos os elementos e o foco agora está na lista vazia.
-- Caso contrário, o foco é o primeiro elemento da lista da direita.
lzFoco :: ListZipper a -> Maybe a
lzFoco (_: []) = Nothing
lzFoco (_: x:_) = Just x

-- Mesmo caso anterior se o foco for o fim da lista.
-- Caso contrário, passa o primeiro elemento da lista da direita para a lista da esquerda.
lzDir :: ListZipper a -> Maybe (ListZipper a)
lzDir (_, []) = Nothing
lzDir (es, d:ds) = Just (d:es, ds)

-- Se estiver no inicio, não consgue andar para a esquerda.
-- Caso contrário, passa o primeiro elemento da lista da esquerda para a frente da lista da direita.
lzEsq :: ListZipper a -> Maybe (ListZipper a)
lzEsq ([]: _) = Nothing
lzEsq (e:es, ds) = Just (es, e:ds)

```

### Funções Úteis.
- ``floor`` - arredondamento para baixo
- ``sqrt`` - raiz quadrada
- ``all`` - verifica se todos os valores são ``True`` para uma função f.
- ``zipWith`` - alem das listas, recebe uma função apara combinar os termos.
- ``getCurrentTime`` - retorna a medida de tempo naquele momento.
- ``diffUTCTime t1 t0`` - diferença entre medidas de tempo, util com ``getCurrentTime``.
- ``chunksOf`` - separa uma lista em listas de 'n' elementos.
- ``forever`` - repete uma ação para sempre
- ``threadDelay <segundos>`` - sleep()
- ``simpleHttp`` - requisição http.
