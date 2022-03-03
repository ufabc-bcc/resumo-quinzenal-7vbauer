## Paradigmas de Programação - Resumo 1
### Nome : Vítor Marini Blaselbauer
### RA : 11048115

## Paradigmas de Programação (Semana 1)
### Paradigmas
- Imperativo: sequência de comandos que alteram o estado atual.

- Estruturado: possuem um fluxo de controle com funções.

- Orientado a objetos: comunicação entre objetos que possuem dados.

- Declarativo: define algo que deve ser feito sem dizer como.

- Funcional: funções matemáticas com dados imutáveis.

- Lógico: conjunto de fatos e regras com um interpretador inferindo respostas para as perguntas.


### Paradigma Funcional


- Efeito colateral: alteração de variáveis, entrada e saída de dados.

- Funções puras (sem efeito colateral).
- Recursão
- Avaliação Preguiçosa (geração de promessas de execução, não avaliadas)


### Cálculo Lambda

- Sistema formal para expressar computação baseada em abstração de funções e aplicação usando atribuições de nomes e substituições.

- Linguagem : descrita em função da sua sintaxe e semântica.

#### Sintaxe do Calculo Lambda
- Variáveis
- Definição de Funções
- Aplicação de Funções

---
- λ por \\   
- .  por -> 
---

#### Programa: definido por uma Expressão "e" ou termos  "λ"
#### Pode assumir três formas:
- Variável (nome que assumirá um valor).
- Abstração (função anônima ou λ ).
- Aplicação (chamada de função).

---

- Escopo de variáveis : ligadas ou livres
- Expressões fechadas : Não possuem variáveis livres.

## Conceitos Básicos - Parte 1 (Semana 2)

#### Características do Haskell:
- Código conciso e declarativo.
- Imutabilidade
- Funções Recursivas
- Funções de Alta ordem 
- Tipos Polimórficos
- Avaliação Preguiçosa

#### Olá Mundo


```
	module Main where
	main :: IO()
	main = do
		putStrLn "hello world"
```

- Funções : ***<nome da função>*** ***<parametros separados por espaço>***

- ***Aplicação de funções tem a maior precedência***

- Nomes: começar com uma letra miunuscula e seguida por 0 ou mais letras, maiusculas ou minusculas, digitos, underscore e aspas simples (suporte para Unicode).

- ***Convenção de nomes para listas***: lista de ``n`` coisas é um ``ns``.

- Blocos são definidos com indentação.

- ``where`` define variáveis e funções intermediárias dentro da função.
---
- Comentários:

``` -- comentário```

ou
```
{-
	comentário
-{
```

- Tipo = coleção de valores relacionados entre si.
---
### Tipos básicos:
- Bool
- Char
- String
- Int (64 bits)
- Integer (Qualquer precisão)
- Float (Precisão simples)
- Double (Precisão dupla)
---
- **Listas**: elementos dentro de ``[]`` separados por vírgulas.

- **Tuplas**: sequências finitas de elementos, de um ou mais tipos diferentes.

- **Funções**: mapeiam argumentos de um tipo para outro.

```
funcao :: Bool -> Bool
funcao2 :: Int -> Bool
```
## Conceitos Básicos - Parte 2 (Semana 2)

### Variável de tipo:
``` 
length :: [a] -> Int
```
- ``a`` indica que o tipo dos elementos pode ser qualquer um.

### Overloaded Types

Restrição de classe:
```
(+) :: C a => a -> a -> a
```
- Indica que ``a`` é um tipo da classe ``C``. 
- Implica que todas as outras ocorrências de ``a`` devem ser do mesmo tipo passado no argumento.
- Podemos/Devemos definir instâncias da função para tipos diferentes da classe.

### Condicionais

```
if <X> then <SE X VERDADEIRO> else <SE X FALSO>
```
```
if (X) 
	then <SE X VERDADEIRO>
	else if (Y) 
		then <SE Y VERDADEIRO (X FALSO)> 
		else <CASO CONTRÁRIO>
```

### Expressões Guardadas

- Alternativa ao ``if then else``.
- Avaliadas de cima para baixo.
```
funcao x | x == 0	=  0
	 | x > 0	=  1
	 | otherwise    = -1
```

- ``error`` = Interrompe a execução do programa.

### Operadores
```
(:+) :: Num a => a -> a -> a
x :+ y = abs x + y
```
ou
```
(:+) :: Num a => a -> a -> a
(:+) x y = abs x + y
```
- ``mod 10 3`` como função, ou ``10 `mod` 3`` como operador.


### Listas

- Coleção de valores de um determinado tipo.
```
data [] a = [] | a : [a]
```
- ``:``  , lê-se **cons**, é o operador de concatenação.
- Lista ligada.

#### Avaliação Preguiçosa 
- ``[0..3] = [0, 1, 2, 3]``
- ``[0, 2..]`` = Lista infinita de números pares.

### Funções e Observações para Listas
- ``lista !! <posição>``.
- ``head <lista>``.
- ``tail <lista>``.
- ``take <número> <lista>``.
- ``drop <numero> <lista>``.
-  ``length <lista>``.
- ``sum <lista>``.
- ``product <lista>``.
- ``<lista> ++ <lista>``.
- ``[]``, ``(x:[]) ou [x]`` ,  ``(x : xs)``.

### Strings
- Strings são listas de ``Char``.
```
"Test" == ['T', 'e', 's', 't']
 ```
### Compreensão de Listas
- ``[x^2 | x <- [1..5]]`` = Realiza a operação ``^2`` para cada elemento da lista ``[1..5]`` e guarda os resultados em uma lista.

```
[(x,y) | x <- [1..4], y <- [4..5]]
```

### Função ``pairs``
- Junta duas listas, retornando uma lista de pares.
```
zip [1, 2, 3] [4, 5, 6] == [(1, 4), (2, 5), (3, 6)]
```
```
zip [1, 2, 3] ['a', 'b', 'c', 'd'] == [(1, 'a'), (2, 'b'), (3, 'c')]
```

### Função ``pairs``

- Retorna uma lista de pares dos elementos adjacentes
```
pairs [1, 2, 3] == [(1, 2), (2, 3)]
```
### Funções ``and``

- Retorna ``True`` se todos os elementos são ``True``.

### Recursão

- Composta por caso(s) base e uma chamada recursiva .
```
fatorial :: Integer -> Integer
fatorial 0 = 1                  -- caso base 1
fatorial 1 = 1                  -- caso base 2
fatorial x = x * fatorial (n-1) -- chamada recursiva
```

**-- Recursão com listas --**

```
sum :: Num a = > [a] -> a
sum [] = 0
sum (x:xs) = x + sum xs
```
