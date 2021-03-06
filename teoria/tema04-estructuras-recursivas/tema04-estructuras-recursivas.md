
# Tema 4: Estructuras de datos recursivas

## Contenidos

- [1 Listas estructuradas](#1)
    - [1.1 Definición y ejemplos](#1-1)
    - [1.2 Funciones recursivas sobre listas estructuradas](#1-2)
- [2 Árboles](#2)
    - [2.1 Definición de árboles en Scheme](#2-1)
    - [2.2 Funciones recursivas sobre árboles](#2-2)

## Bibliografía - SICP

En este tema explicamos conceptos de los siguientes capítulos del libro *Structure and Intepretation of Computer Programs*:

- [1.2.2 - Tree Recursion](https://mitpress.mit.edu/sicp/full-text/book/book-Z-H-11.html#%_sec_1.2.2)
- [2.2.2 - Hierarchical Structures](https://mitpress.mit.edu/sicp/full-text/book/book-Z-H-15.html#%_sec_2.2.2)

## <a name="1"></a> 1 Listas estructuradas

Hemos visto que las listas en Scheme se implementan como un estructura
de datos recursiva, formada por una pareja que enlaza en su parte
derecha el resto de la lista y que termina con una parte derecha en la
que hay una lista vacía.

En este apartado vamos a volver a estudiar las listas desde un nivel
de abstracción alto, usando las funciones:

- `(car lista)` para obtener el primer elemento de una lista
- `(cdr lista)` para obtener el resto de la lista
- `(cons dato lista)` para construir una nueva lista con el dato como
  primer elemento

En la mayoría de funciones y ejemplos que hemos visto hasta ahora las
listas están formadas por datos y el recorrido por la lista es un
recorrido lineal, iterando por sus elementos.

En este apartado vamos a ampliar este concepto y estudiar cómo
trabajar con *listas que contienen otras listas*.

### <a name="1-1"></a> 1.1 Definición y ejemplos

Las listas en Scheme pueden tener cualquier tipo de elementos,
incluido otras listas.

Llamaremos **lista estructurada** a una lista que contiene otras
sublistas. Lo contrario de lista estructurada es una **lista plana**,
una lista formada por elementos que no son listas. Llamaremos
**hojas** a los elementos de una lista que no son sublistas.

A las listas estructuradas cuyas hojas son símbolos se les denomina en
el contexto de la programación funcional _expresiones-S_
([S-expression](http://en.wikipedia.org/wiki/S-expression)).

Por ejemplo, la lista estructurada:

```
{a b {c d e} {f {g h}}}
```

es una lista estructurada con 4 elementos:

- El elemento `'a`, una hoja
- El elemento `'b`, otra hoja
- La lista plana `{c d e}`
- La lista estructurada `{f {g h}}`

Se puede construir con cualquiera de las siguientes expresiones:

```
(define lista (list 'a 'b (list 'c 'd 'e) (list 'f (list 'g 'h))))
(define lista '(a b (c d e) (f (g h))))
```

Una lista formada por parejas la consideraremos una lista plana, ya
que no contiene ninguna sublista. Por ejemplo, la lista

```
{{a . 3} {b . 5} {c . 12}}
```

es una lista plana de tres elementos (hojas) que son parejas.

#### 1.1.1. Definiciones en Scheme

Vamos a escribir las definiciones anteriores de `hoja`, `plana` y
`estructurada` usando código de Scheme.

##### Función `(hoja? dato)`

Un dato es una hoja si no es una lista:

```
(define (hoja? dato)
   (not (list? dato)))
```

Utilizaremos esta función para comprobar si un determinado elemento de
una lista es o no una hoja. Por ejemplo, supongamos la siguiente
lista:

```
{{1 2} 3 4 {5 6}}
```

Es una lista de 4 elementos, siendo el primero y el último otras
sublistas y el segundo y el tercero hojas. Podemos comprobar si son o
no hojas sus elementos:

```
(define lista '((1 2) 3 4 (5 6)))
(hoja? (car lista)) ; ⇒ #f
(hoja? (cadr lista)) ; ⇒ #t
(hoja? (caddr lista)) ; ⇒ #t
(hoja? (cadddr lista)) ; ⇒ #f
```

La lista vacía no es una hoja

```
(hoja? '()) ; ⇒ #f
```

##### Función `(plana? lista)`

Una definición recursiva de lista plana:

>Una lista es plana si y solo si el primer elemento es una hoja y el
>resto es plana.

Y el caso base:

>Una lista vacía es plana.

Usando esta definición recursiva, podemos implementar en Scheme la
función `(plana? lista)` que comprueba si una lista es plana:

```
(define (plana? lista)
   (or (null? lista)
       (and (hoja? (car lista))
            (plana? (cdr lista)))))
```

Ejemplos:

```scheme
(plana? '(a b c d e f)) ; ⇒ #t
(plana? (list (cons 'a 1) "Hola" #f)) ; ⇒ #t
(plana? '(a (b c) d)) ; ⇒ #f
(plana? '(a () b)) ; ⇒ #f
```


##### Función `(estructurada? lista)`

Una lista es estructurada cuando alguno de sus elementos es otra lista:

```
(define (estructurada? lista)
   (if (null? lista)
      #f
      (or (list? (car lista))
          (estructurada? (cdr lista)))))
```

Ejemplos:

```scheme
(estructurada? '(1 2 3 4)) ; ⇒ #f
(estructurada? (list (cons 'a 1) (cons 'b 2) (cons 'c 3))) ; ⇒ #f
(estructurada? '(a () b)) ; ⇒ #t
(estructurada? '(a (b c) d)) ; ⇒ #t
```

Realmente bastaría con haber hecho una de las dos definiciones y
escribir la otra como la negación de la primera:

```
(define (estructurada? lista)
   (not (plana? lista)))
```

#### 1.1.2 Ejemplos de listas estructuradas

Las listas estructuradas son muy útiles para representar información
jerárquica en donde queremos representar elementos que contienen otros
elementos.

Por ejemplo, las expresiones de Scheme son listas estructuradas:

	(= 4 (+ 2 2))
	(if (= x y) (* x y) (+ (/ x y) 45))
	(define (factorial x) (if (= x 0) 1 (* x (factorial (- x 1)))))

El análisis sintáctico de una oración puede generar una lista
estructurada de símbolos, en donde se agrupan los distintos elementos
de la oración:

	{{Juan} {compró} {la entrada {de los Miserables}} {el viernes por la tarde}}

Una página HTML, con sus distintos elementos, unos dentro de otros,
también se puede representar con una lista estructurada:

	{{<h1> Mi lista de la compra </h1>}
	  {<ul> {<li> naranjas </li>}
            {<li> tomates </li>}
            {<li> huevos </li>} </ul>}}


#### 1.1.3 *Pseudo árboles* con niveles

Las listas estructuradas definen una estructura de niveles, donde la
lista inicial representa el primer nivel, y cada sublista representa
un nivel inferior. Los datos de las listas representan las hojas.

Por ejemplo, la representación en forma de niveles de la lista `{{a b
c} d e}` es la siguiente:

<img src="imagenes/expresion-e-1.png" width="350px"/>

Las hojas `d` y `e` están en el nivel 1 y en las posiciones 2 y 3 de
la lista y las hojas `a`, `b` y `c` en el nivel 2 y en la posición 1
de la lista.

> UNA LISTA ESTRUCTURADA NO ES UN ÁRBOL  
> Una lista estructurada no es un árbol propiamente dicho, porque
> todos los datos están en las hojas.

Otro ejemplo. ¿Cuál sería la representación en niveles de la siguiente
lista estructurada?:

	{let {{x 12}
	      {y 5}}
	   {+ x y}}}

<img src="imagenes/expresion-e-2.png" width="300px"/>

### <a name="1-2"></a> 1.2 Funciones recursivas sobre listas estructuradas

#### 1.2.1 Número de hojas

Veamos como primer ejemplo la función `(num-hojas lista)` que cuenta
el número de hojas de una lista estructurada.

Por ejemplo:

```
(num-hojas '((1 2) (3 4 (5) 6) (7))) ⇒ 7
```

Podemos definir la función obteniendo el primer elemento y el resto de
la lista, y contando recursivamente el número de hojas del primer
elemento y del resto. Al ser una lista estructurada, el primer
elemento puede ser a su vez otra lista, por lo que llamamos a la
recursión para contar sus hojas.

La definición de este caso general usando _pseudocódigo_ es:

> El número de hojas de una lista estructurada es la suma del número
> de hojas de su primer elemento (que puede ser otra lista) y del
> número de hojas del resto.

<img src="imagenes/num-hojas-estructurada.png" width="400px"/>

Como casos base, podemos considerar cuando la lista es vacía (el
número de hojas es 0) y cuando la lista no es tal, sino que es un dato
(una hoja), en cuyo caso es 1. La implementación en Scheme es:


```
(define (num-hojas lista)
   (cond
      ((null? lista) 0)
      ((hoja? lista) 1)
      (else (+ (num-hojas (car lista))
               (num-hojas (cdr lista))))))
```

Hay que hacer notar que el parámetro `lista` puede ser tanto una lista
como un dato atómico. Estamos aprovechándonos de la característica de
Scheme de ser débilmente tipeado para hacer un código bastante
conciso.


##### Versión con funciones de orden superior

Podemos usar también las funciones de orden superior `map` y
`fold-right` para obtener una versión más concisa.

Una lista estructurada tiene como elementos hojas o listas. Podemos
entonces mapear una expresión lambda con _la propia función que
estamos definiendo_ sobre sus elementos, poniendo como caso especial
el hecho de que la lista sea una hoja. El resultado será una lista de
números (el número de hojas de cada componente), que podemos sumar
haciendo un `fold-right` con la función `+`:

```scheme
(define (num-hojas-fos lista)
    (fold-right + 0 (map (lambda (sublista)
                           (if (hoja? sublista)
                               1
                               (num-hojas-fos sublista))) lista)))
```

Una explicación gráfica de cómo funciona la función sobre la lista `'(1 (2 3) (4) (5 (6 7) 8))`:

<img src="imagenes/map-lista.png" width="700px"/>


#### 1.2.2 Altura de una lista estructurada

La *altura* de una lista estructurada viene dada por su número de
niveles: una lista plana tiene una altura de 1, la lista `'((1 2 3) 4
5)` tiene una altura de 2.

Para calcular la altura de una lista estructurada tenemos que obtener
(de forma recursiva) la altura de su primer elemento, y la altura del
resto de la lista, sumarle 1 a la altura del primer elemento y
devolver el máximo de los dos números.

<img src="imagenes/altura-estructurada.png" width="300px"/>

Como casos base, la altura de una lista vacía o de una hoja (dato) es 0.

En Scheme:

```
(define (altura lista)
   (cond 
      ((null? lista) 0)
      ((hoja? lista) 0)
      (else (max (+ 1 (altura (car lista)))
                 (altura (cdr lista))))))

```

Por ejemplo:

```
(altura '(1 (2 3) 4)) ⇒ 2
(altura '(1 (2 (3)) 3)) ⇒ 3
```

##### Versión con funciones de orden superior

Y la segunda versión, usando las funciones de orden superior `map`
para obtener la altura de las sublistas y `fold-right` para quedarse
con el máximo.

```scheme
(define (altura-fos lista)
   (+ 1 (fold-right max 0 (map (lambda (sublista)
                                 (if (hoja? sublista)
                                     0
                                     (altura-fos sublista))) lista))))
```

#### 1.2.3 Otras funciones recursivas

Vamos a diseñar otras funciones recursivas que trabajan con la
estructura jerárquica de las listas estructuradas.

- `(aplana lista)`: devuelve una lista plana con todas las hojas de la lista
- `(pertenece-lista? dato lista)`: busca una hoja en una lista
  estructurada
- `(nivel-lista dato lista)`: devuelve el nivel en el que se encuentra
  un dato en una lista
- `(cuadrado-lista lista)`: eleva todas las hojas al cuadrado
  (suponemos que la lista estructurada contiene números)
- `(map-lista f lista)`: similar a map, aplica una función a todas las
  hojas de la lista estructurada y devuelve el resultado (otra lista
  estructurada)

##### `(aplana lista)`

Devuelve una lista plana con todas las hojas de la lista.

```scheme
(define (aplana lista)
  (cond
    ((null? lista) '())
    ((hoja? lista) (list lista))
    (else 
     (append (aplana (car lista))
             (aplana (cdr lista))))))
```

Por ejemplo:

```scheme
(aplana '(1 2 (3 (4 (5))) (((6)))))
; ⇒ {1 2 3 4 5 6}
```

Con funciones de orden superior:

```scheme
(define (aplana-FOS lista)
  (fold-right append
              '()
              (map (lambda (x)
                     (if (hoja? x)
                         (list x)
                         (aplana-FOS x))) lista)))
```

##### `(pertenece-lista? dato lista)`

Comprueba si el dato x aparece en la lista estructurada. 

```scheme
(define (pertenece? x lista)
  (cond 
    ((null? lista) #f)
    ((hoja? lista) (equal? x lista))
    (else (or (pertenece? x (car lista))
              (pertenece? x (cdr lista))))))
```

Ejemplos:

```scheme
(pertenece? 'a '(b c (d (a)))) ⇒ #t
(pertenece? 'a '(b c (d e (f)) g)) ⇒ #f
```

Con funciones de orden superior:

```scheme
#lang r6rs
(import (rnrs base)
        (rnrs lists (6)))
        
(define (pertenece-FOS? elem lista)
  (exists (lambda (x)
             (if (hoja? x)
                 (equal? elem x)
                 (pertenece-FOS? elem x))) lista))
```

##### `(nivel-lista dato lista)`

Veamos como última función que explora una lista estructurada la
función `(nivel-lista dato lista)` que recorre la lista buscando el
dato y devuelve el nivel en que se encuentra. Si el dato no se
encuentra en la lista, se devolverá -1. Si el dato se encuentra en más
de un lugar de la lista se devolverá el nivel mayor.

Ejemplos:

```scheme
(nivel-hoja 'b '(a b (c))) ; ⇒ 1
(nivel-hoja 'b '(a (b) c)) ; ⇒ 2
(nivel-hoja 'b '(a (b) d ((b)))) ; ⇒ 2
(nivel-hoja 'b '(a c d ((e)))) ; ⇒ 0
```

```scheme
(define (nivel-hoja dato lista)
  (cond
    ((null? lista) -1)
    ((hoja? lista) (if (equal? lista dato) 0 -1))
    (else (max (suma-1-si-mayor-igual-que-0 (nivel-hoja dato (car lista)))
               (nivel-hoja dato (cdr lista))))))
```

La función auxiliar se define de la siguiente forma:

```scheme
(define (suma-1-si-mayor-igual-que-0 x)
  (if (>= x 0)
      (+ x 1)
      x))
```

Con funciones de orden superior:

```scheme
(define (nivel-hoja-FOS dato lista)
  (suma-1-si-mayor-igual-que-0
       (fold-right max -1
                   (map (lambda (sublista)
                          (if (hoja? sublista)
                              (if (equal? sublista dato) 0 -1)
                              (nivel-hoja-FOS dato sublista)))  lista))))
```

##### `(cuadrado-lista lista)`

Devuelve una lista estructurada con la misma estructura y sus números
elevados al cuadrado.

```scheme
(define (cuadrado-lista lista)
  (cond ((null? lista) '())
        ((hoja? lista) (* lista lista))
        (else (cons (cuadrado-lista (car lista))
                    (cuadrado-lista (cdr lista))))))
```

Por ejemplo:

```scheme
(cuadrado-lista '(2 3 (4 (5)))) ⇒ (4 9 (16 (25))
```

Es muy interesante la versión de esta función con funciones de orden
superior:

```scheme
(define (cuadrado-lista-fos lista)
    (map (lambda (sublista)
           (if (hoja? sublista)
               (* sublista sublista)
               (cuadrado-lista-fos sublista))) lista))
```

Como una lista estructurada está compuesta de datos o de otras
sublistas podemos aplicar `map` para que devuelva la lista resultante
de transformar la original con la función que le pasamos como
parámetro.

##### `(map-lista f lista)`

Devuelve una lista estructurada igual que la original con el resultado
de aplicar a cada uno de sus hojas la función f
 
```
(define (map-lista f lista)
  (cond ((null? lista) '())
        ((hoja? lista) (f lista))
        (else (cons (map-lista f (car lista))
                    (map-lista f (cdr lista))))))
```
	
Por ejemplo:

```
(map-lista (lambda (x) (* x x)) '(2 3 (4 (5)))) ⇒ (4 9 (16 (25))
```

## <a name="2"></a>2 Árboles

### <a name="2-1"></a>2.1 Definición de árboles en Scheme

#### 2.1.1 Definición de árbol

Un **árbol** es una estructura de datos definida por un valor raíz,
que es el padre de toda la estructura, del que salen otros subárboles
hijos
([Wikipedia](https://en.wikipedia.org/wiki/Tree_(data_structure))).

Un **árbol** se puede definir recursivamente de la siguiente forma:

- Una colección de un **dato** (el valor de la raíz del árbol) y una
  **lista de hijos** que también son árboles.
- Una **hoja** será un árbol sin hijos (un dato con una lista de hijos
  vacía).

Un ejemplo de árbol:

<img src="imagenes/arbol-sencillo.png" width="250px"/>

El árbol anterior tiene como dato de la raíz es el símbolo `+` y tiene
3 árboles hijos:

<img src="imagenes/arboles-hijos.png" width="300px"/>

- El primer hijo es un árbol hoja, con valor 5 y sin hijos
- El segundo hijo es un árbol con valor `*` y dos hijos hoja, el 2 y
  el 3
- El tercer hijo es otro árbol hoja, con valor 10

#### 2.1.2 Representación de árboles con listas

En Scheme tenemos como estructura de datos principal la lista. ¿Cómo
construimos un árbol usando listas?

La forma de hacerlo será usar **una lista de _n+1_ elementos** para
representar un árbol con n hijos:

- el primer elemento la lista será el dato de la raíz
- el resto serán los árboles hijos

> Árbol: '(dato hijo-1 hijo-2 ... hijo-n)

Los nodos hoja serán por tanto listas de un elemento, el propio dato
(no tiene más elementos porque no tiene hijos)

> Nodo hoja: '(dato)

Por ejemplo, el árbol anterior lo representaremos en Scheme con la
siguiente lista:

```scheme
{+ {5} {* {2} {3}} {10}}
```

Los elementos de esta lista son:

- El primer elemento es el símbolo `+`, el dato valor de la raíz del
  árbol
- El segundo elemento es la lista `{5}`, que representa el árbol hoja
  formado por un 5
- El tercer elemento es la lista `{* {2} {3}}`, que representa el
  árbol con un dato `*` y dos hijos
- El cuarto elemento es la lista `{10}`, que representa el árbol hoja
  formado por un 10

Podríamos definir el árbol con la siguiente sentencia:

```scheme
(define arbol1 '(+ (5) (* (2) (3)) (10)))
```

Otro ejemplo más. ¿Cómo se implementa en Scheme el árbol de la siguiente figura?

<img src="imagenes/binario-2.png" width="300px"/>

Se haría con la lista de la siguiente sentencia:

```scheme
(define arbol2 '(40 (18 (3) (23 (29))) (52 (47))))
```

#### 2.1.2 Barrera de abstracción

Una vez definida la forma de representar árboles, vamos a definir las
funciones para manejarlos. Veremos las funciones para obtener el dato
y los hijos y la función para construir un árbol nuevo. Estas
funciones proporcionarán lo que se denomina _barrera de abstracción_
del tipo datos *árbol*.

Una _barrera de abstracción_ es un conjunto de funciones que permiten
trabajar con un tipo de datos escondiendo su implementación. Por
convenio, en todas las funciones ponemos el mismo sufijo, el nombre
del tipo de dato, en este caso **tree** (vamos a hacer una mezcla un
poco extraña, escribiendo el nombre del tipo de dato en inglés, y el
nombre de las funciones en castellano).

Definimos dos conjuntos de funciones: **constructores** para construir
un nuevo árbol y **selectores** para obtener los elementos del
árbol. Vamos a empezar por los selectores.

**Selectores**

Funciones que obtienen los elementos de un árbol:

```scheme
(define (dato-tree arbol) 
    (car arbol))

(define (hijos-tree arbol) 
    (cdr arbol))

(define (hoja-tree? arbol) 
   (null? (hijos-tree arbol)))
```

Es importante tener claro los tipos devueltos por las dos primeras
funciones:

- `(dato-tree arbol)`: devuelve el dato de la raíz del árbol.
- `(hijos-tree arbol)`: devuelve una lista de árboles hijos. En
  algunas ocasiones llamaremos *bosque* a una lista de árboles.

Por ejemplo, en el árbol `arbol1` las funciones anteriores devuelven
los siguientes valores:

```scheme
(dato-tree arbol1) ; ⇒ +
(hijos-tree arbol1) ; ⇒ {{5} {* {2} {3}} {10}}
(hoja-tree? (car (hijos-tree arbol1))) ; ⇒ #t
```

- La llamada `(dato-tree arbol1)` devuelve el dato que hay en la raíz
  del árbol, el símbolo `+`
- La invocación `(hijos-tree arbol1)` devuelve una lista de tres
  elementos, los árboles hijos: `{{5} {* {2} {3}} {10}}`:
    - El primer elemento es la lista `{5}`, que representa el árbol
      hoja formado por el `5`
    - El segundo es la lista `{* {2} {3}}`, que representa el árbol
      formado por el `*` en su raíz y las hojas `2` y `3`
    - El tercero es la lista `{10}`, que representa el árbol hoja
      `10`.
- El primer elemento de la lista de hijos es un árbol hoja:
  `(hoja-tree? (car (hijos-tree arbol1))) ⇒ #t`
				   
Es muy importante considerar en cada caso con qué tipo de dato estamos
trabajando y usar la barrera de abstracción adecuada en cada caso:

- La función `hijos-tree` siempre devuelve una lista de árboles, que
  podemos recorrer usando `car` y `cdr`.
- El `car` de una lista de árboles (devuelta por `hijos-tree`) siempre
  es un árbol y debemos de usar las funciones de su barrera de
  abstracción: `dato-tree` e `hijos-tree`.
- La función `dato-tree` devuelve un dato de árbol, del tipo que
  guardemos en el árbol.

Por ejemplo, para obtener el número 2 en el árbol anterior tendríamos
que hacer lo siguiente: acceder al segundo elemento de la lista de
hijos, después al primer hijo de éste y por último acceder a su
dato. Recordemos que `hijos-tree` devuelve la lista de árboles hijos,
por lo que utilizaremos las funciones `car` y `cdr` para recorrerlas y
obtener los elementos que nos interesen:

```scheme
(dato-tree (car (hijos-tree (cadr (hijos-tree arbol1))))) ; ⇒ 2
```

**Constructores**

Funciones que permiten construir un nuevo árbol:

```scheme
(define (make-tree dato lista-arboles)  
   (cons dato lista-arboles))

(define (make-hoja-tree dato) 
    (make-tree dato '()))
```

- La función `(make-tree dato lista-arboles)` recibe un dato y una
  lista de árboles y devuelve un árbol formado por el dato en su raíz
  y la lista de hijos.
- La función `(make-hoja-tree dato)` recibe un dato y devuelve un
  árbol hoja (un árbol sin hijos).

El árbol anterior se puede construir con las siguientes llamadas a los
constructores:

```scheme
(make-tree '+ (list (make-hoja-tree 5)
                             (make-tree '* 
                                        (list (make-hoja-tree 2) 
                                              (make-hoja-tree 3)))
                             (make-hoja-tree 10)))
```

#### 2.1.3 Diferencia entre árboles y listas estructuradas

Es importante diferenciar la barrera de abstracción de los árboles de
la de las listas estructuradas. Aunque un árbol se implementa en
Scheme con una lista estructurada, a la hora de definir funciones
sobre árboles hay que trabajar con las funciones definidas arriba.

El siguiente esquema resumen las características de la barrera de
abstracción de listas y árboles:

<img src="imagenes/barrera-abstraccion.png" width="550px">

### <a name="2-2"></a>2.2 Funciones recursivas sobre árboles

Vamos a diseñar las siguientes funciones recursivas:

* `(suma-datos-tree tree)`: devuelve la suma de todos los nodos
* `(to-list-tree tree)`: devuelve una lista con los datos del árbol
* `(cuadrado-tree tree)`: eleva al cuadrado todos los datos de un
  árbol manteniendo la estructura del árbol original
* `(map-tree f tree)`: devuelve un árbol con la estructura del árbol
  original aplicando la función f a subdatos.
* `(altura-tree tree)`: devuelve la altura de un árbol

Todas comparten un patrón similar de recursión mutua.

#### 2.2.1 `(suma-datos-tree tree)`

Vamos a implementar una función recursiva que sume todos los datos de
un árbol.

Un árbol siempre va a tener un dato y una lista de hijos (que puede
ser vacía) que obtenemos con las funciones `dato-tree` e
`hijos-tree`. Podemos plantear entonces el problema de sumar los datos
de un árbol como la suma del dato de su raíz y lo que devuelva la
llamada a una función auxiliar que sume los datos de su lista de hijos
(un bosque):

```scheme
(define (suma-datos-tree tree)
    (+ (dato-tree tree)
       (suma-datos-bosque (hijos-tree tree))))
```

Esta función suma los datos de **UN** árbol. La podemos utilizar
entonces para construir la siguiente función que suma una lista de
árboles:

```scheme
(define (suma-datos-bosque bosque)
   (if (null? bosque)
       0
       (+ (suma-datos-tree (car bosque)) (suma-datos-bosque (cdr bosque)))))
```

Tenemos una recursión mutua: para sumar los datos de una lista de
árboles llamamos a la suma de un árbol individual que a su vez llama a
la suma de sus hijos, etc. La recursión termina cuando calculamos la
suma de un árbol hoja. Entonces se pasa a `suma-datos-bosque` una
lista vacía y ésta devolverá 0.


```scheme
(suma-datos-tree arbol2) ; 212⇒
```

**Versión alternativa con funciones de orden superior**

Al igual que hacíamos con las listas estructuradas, es posible
conseguir una versión más concisa y elegante utilizando funciones de
orden superior:

```scheme
(define (suma-datos-tree-fos tree)
    (+ (dato-tree tree)
       (fold-right + 0 (map suma-datos-tree-fos (hijos-tree tree)))))
```	

La función `map` aplica la propia función que estamos definiendo a
cada uno de los árboles de `(hijos-tree tree)`, devolviendo una lista
de números. Esta lista de número la sumamos haciendo un `fold-right +
0`. Una traza de su funcionamiento sería la siguiente:

```scheme
(suma-datos-tree-fos '(1 (2 (3) (4)) (5) (6 (7)))) ⇒
(+ 1 (fold-right + 0 (map suma-datos-tree-fos '((2 (3) (4)) 
                                                (5)
                                                (6 (7)))))) ⇒
(+ 1 (fold-right + '(9 5 13))) ⇒
(+ 1 27) ⇒
28
```

#### 2.2.2 `(to-list-tree tree)`

Queremos diseñar una función `(to-list-tree tree)` que devuelva una
lista con los datos del árbol en un recorrido *preorden*.

```scheme
(define (to-list-tree tree)
   (cons (dato-tree tree)
         (to-list-bosque (hijos-tree tree))))

(define (to-list-bosque bosque)
   (if (null? bosque)
       '()
       (append (to-list-tree (car bosque))
               (to-list-bosque (cdr bosque)))))
```

La función utiliza una *recursión mutua*: para listar todos los nodos,
añadimos el dato a la lista de nodos que nos devuelve la función
`to-list-bosque`. Esta función coge una lista de árboles (un *bosque*)
y devuelve la lista *inorden* de sus nodos. Para ello, concatena la
lista de los nodos de su primer elemento (el primer árbol) a la lista
de nodos del resto de árboles (que devuelve la llamada recursiva).

Ejemplo:

```scheme
(to-list-tree '(* (+ (5) (* (2) (3)) (10)) (- (12)))) 
; ⇒ (* + 5 * 2 3 10 - 12)
```

Una definición alternativa usando funciones de orden superior:

```scheme
(define (to-list-tree-fos tree)
    (cons (dato-tree tree)
          (fold-right append '() (map to-list-tree-fos (hijos-tree tree)))))
```

Esta versión es muy elegante y concisa. Usa la función `map` que
aplica una función a los elementos de una lista y devuelve la lista
resultante. Como lo que devuelve `(hijos-tree tree)` es precisamente
una lista de árboles podemos aplicar a sus elementos cualquier función
definida sobre árboles. Incluso la propia función que estamos
definiendo (¡confía en la recursión!).

#### 2.2.3 `(cuadrado-tree tree)`

Veamos ahora la función `(cuadrado-tree tree)` que toma un árbol de
números y devuelve un árbol con la misma estructura y sus datos
elevados al cuadrado:

```scheme
(define (cuadrado-tree tree)
   (make-tree (cuadrado (dato-tree tree))
                   (cuadrado-bosque (hijos-tree tree))))  

(define (cuadrado-bosque bosque)
   (if (null? bosque)
       '()
       (cons (cuadrado-tree (car bosque))
               (cuadrado-bosque (cdr bosque)))))
```

Ejemplo:

```scheme
(cuadrado-tree '(2 (3 (4) (5)) (6))) 
; ⇒ (4 (9 (16) (25)) (36))
```

Versión 2, con `map`:

```scheme
(define (cuadrado-tree tree)
   (make-tree (cuadrado (dato-tree tree))
   	          (map cuadrado-tree (hijos-tree tree))))
```

#### 2.2.4 `map-tree`

La función `map-tree` es una función de orden superior que generaliza
la función anterior. Definimos un parámetro adicional en el que se
pasa la función a aplicar a los elementos del árbol.

```scheme
(define (map-tree f tree)
   (make-tree (f (dato-tree tree))
              (map-bosque f (hijos-tree tree))))  

(define (map-bosque f bosque)
   (if (null? bosque)
       '()
       (cons (map-tree f (car bosque))
             (map-bosque f (cdr bosque)))))
```

Ejemplos:

```scheme
(map-tree cuadrado '(2 (3 (4) (5)) (6)))
; ⇒ (4 (9 (16) (25)) (36))
(map-tree (lambda (x) (+ x 1)) '(2 (3 (4) (5)) (6)))
; ⇒ (3 (4 (5) (6)) (7))
```

Con `map`:

```scheme
(define (map-tree f tree)
  (make-tree (f (dato-tree tree))
             (map (lambda (x)
                    (map-tree f x)) (hijos-tree tree))))
```


#### 2.2.5. `altura-tree`

Vamos por último a definir una función que devuelve la altura de un
árbol (el nivel del nodo de mayor nivel). Un nodo hoja tiene de altura
0.

Solución 1:

```scheme
(define (altura-tree tree)
   (if (hoja-tree? tree)
       0
       (+ 1 (max-altura-bosque (hijos-tree tree)))))  

(define (max-altura-bosque bosque)
    (if (null? bosque)
        0
        (max (altura-tree (car bosque))
             (max-altura-bosque (cdr bosque)))))
```

Ejemplos:


```scheme
(altura-tree '(2)) ;  ⇒ 0
(altura-tree '(4 (9 (16) (25)) (36))) ; ⇒ 2
```

Solución con funciones de orden superior:

La función `max-altura-bosque` puede implementarse de una forma más
concisa todavía usando las funciones con funciones de orden superior:

```scheme
(define (max-altura-bosque-fos bosque)
   (fold-right max 0 (map altura-tree bosque)))
```
	
La función `map` mapea la función `altura-tree` a todos los elementos
del *bosque* (lista de árboles) devolviendo una lista de números, de
la que obtenemos el máximo plegando la lista con la función `max`.

----

Lenguajes y Paradigmas de Programación, curso 2016-17  
© Departamento Ciencia de la Computación e Inteligencia Artificial, Universidad de Alicante  
Domingo Gallardo, Cristina Pomares
