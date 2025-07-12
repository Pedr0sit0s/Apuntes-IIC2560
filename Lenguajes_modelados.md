## Modelando el significado

Este curso se centrará en el comportamiento asociado con la sintaxis, concretamente en la semántica de los lenguajes de programación.

### Modelado del significado

Para estudiar la semántica, utilizaremos lo que se denomina **semántica del intérprete**. La idea fundamental es sencilla: para explicar un lenguaje, debemos escribir un intérprete.

### Modelando la sintaxis

Independientemente de si escribimos:

- `3+4` (notación infija)
- `3 4 +` (notación postfija)
- `+ 3 4` (notación prefija)

siempre expresamos la misma operación: sumar 3 y 4.

En esencia, el contenido es un árbol donde la operación de suma actúa como raíz y los operandos como hojas.

Podemos representar esto en **Scheme** como:

```racket
(add (num 3) (num 4))
```

Otra operación como `(3-4)+7` se representaría como:

```racket
(add (minus (num 3) (num 4)) (num 7))
```

Una definición de datos para soportar estas operaciones podría ser:

```racket
(define-type AE
  [num (n number?)]
  [add (left-AE AE?) (right-AE AE?)]
  [minus (left-AE AE?) (right-AE AE?)])

;; SINTAXIS

;; [nombre-del-dato (dato tipo-dato?)]
```

donde **AE** significa **expresión aritmética**.

Con **define-type** creamos tipos de datos. Los tipos definidos son:

- **num**: representa un número.
- **add**: representa una suma, que contiene una expresión izquierda y una expresión derecha. Estas expresiones pueden ser a su vez operaciones más complejas, por lo que deben ser de tipo AE (de ahí el uso de `AE?`). El tipo AE engloba `num`, `add` y `minus`.
    
    Ejemplo:
    
    ```racket
    (add (num 3) (minus (num 4) (num 7)))
    
    ;; En este caso, `add` tiene:
    ;; - Expresión izquierda: (num 3)
    ;; - Expresión derecha: (minus (num 4) (num 7))
    ;;   A su vez, `minus` contiene:
    ;;   - Expresión izquierda: (num 4)
    ;;   - Expresión derecha: (num 7)
    ```
    
- **minus**: representa una resta, que también contiene una expresión izquierda y una expresión derecha, al igual que `add`.

### Introducción a los parsers

Nuestro intérprete debe procesar términos de tipo **AE**, pero escribirlos manualmente sería tedioso (imagina escribir constantemente expresiones como `(add (num 3) (num 4))`). Por ello, necesitamos un programa que convierta la **sintaxis concreta** (lo que un usuario escribe) en **sintaxis abstracta** (una representación independiente de la notación sintáctica). A este programa lo llamaremos **analizador sintáctico** o **parser**.

Utilizaremos una notación prefija con llaves `{}` en lugar de paréntesis, para distinguir la sintaxis concreta de Racket de la de nuestro lenguaje.

Ejemplos de programas válidos:

```racket
3
{+ 3 4}
{+ {- 3 4} 7}
```

Usaremos la función `read` de Scheme, que interpreta las entradas como **s-expressions**:

- `1 7 2 9` → número `1729`
- `c s 1 7 3` → símbolo `cs173`
- `(1 a)` → lista `(1 'a)`
- `{+ 3 4}` → lista `(+ 3 4)`

Nuestra gramática en **BNF** sería:

racket

```racket
#|
<AE> ::= <num>
        | {+ <AE> <AE>}
        | {- <AE> <AE>}
|#
```

El parser se implementaría así:

```racket
;; parse : sexp -> AE
(define (parse sexp)
  (match sexp
    [(? number? n) (num n)]
    [(list '+ l r) (add (parse l) (parse r))]
    [(list '- l r) (minus (parse l) (parse r))]
    [else (error 'parse "Sintaxis inválida: ~a" sexp)]))
```

Desglosemos el código:

1. Creamos una función `parse` que recibe una **s-expression** (`sexp`), como `{+ 3 5}`.
2. Usamos `match` para analizar la estructura de `sexp`:
    - **Caso 1**: Si `sexp` es un número (verificado con `(? number? n)`), creamos una estructura `(num n)`.
    - **Caso 2**: Si `sexp` es una lista que comienza con `+` (p.ej., `(+ 3 4)`), creamos una estructura `add` y aplicamos `parse` recursivamente a las subexpresiones `l` y `r`.
    - **Caso 3**: Si `sexp` es una lista que comienza con  (p.ej., `(- 3 4)`), creamos una estructura `minus` y aplicamos `parse` recursivamente a las subexpresiones `l` y `r`.
    - **Caso 4**: Si `sexp` no coincide con ningún patrón válido (p.ej., `{/ 5 3}`), lanzamos un error indicando que la sintaxis es inválida.