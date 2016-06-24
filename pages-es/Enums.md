# Enumeraciones (*Enums*)

Los enums nos permiten definir un conjunto de constantes numéricas.
Una enumeración puede ser definido usando la palabra clave `enum`.

```ts
enum Direction {
    Arriba = 1,
    Abajo,
    Izquierda,
    Derecha
}
```

El cuerpo de un enumeración consiste en cero o más miembros.
Los miembros de una enumeración tienen valores numéricos asociados a ellos y pueden ser incluso *constante* o *computerizados*.
Un miembro de la enumeración es considerado constante si:

* No tiene un inicializador y el miembro de la enumeración que le precede es constante.
    En este caso, el valor del miemrbo actual de la enumeración será el valor del que le precede más uno.
    Una excepción a esta regla es el primer elemento de una enumeración.
    Si no tiene inicializador le es asignado el valor `0`.
* Un miembro de la enumeración es inicializado con una constante enumerada.
    Una constante enumerada es un subconjunto de expresiones TypeScript que pueden ser completamente evualadas en tiempo de compilación.
    Una expresión es una constante enumerada si incluye:
    * Numérico literal.
    * Referencia a miembros constantes enumerados previamente definidos (pueden ser definidos en enumeraciones diferentes)
        Si el miembro es definido en la misma enumeración, puede ser referenciado usando nombres no cualificado.
    * Expresión constante enumerada entre parentesis.
    * `+`, `-`, `~` operadores unitarios aplicados a expresiones constantes enumeradas.
    * `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^` operadores binarios con expresiones constantes enumeradas como operadores.
    Es un error en tiempo de compilación para las expresiones constantes enumeradas que son evaluadas como `NaN` o `Infinity`.

En todos los otros casos los miembros enumerados son considerados computados.

```ts
enum FileAccess {
    // constant members
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // computed member
    G = "123".length
}
```

Las enumeraciones son objetos reales que existen en tiempo de ejecución.
Una razón es la habildiad para mantener un mapeo inverso de los valores de la enumeración a los nombres de la enumeración.

```ts
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[Enum.A]; // "A"
```

es compilado a:

```js
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
var a = Enum.A;
var nameOfA = Enum[Enum.A]; // "A"
```

En el código generado una enumeración es compilada a un objeto que almacena ambos, tanto al derecho (`name` -> `value`) como al revés (`value` -> `name`).
Las referencias a los miembros enumerados siempre se emiten como acceso de propiedad y no en línea.
En muchos casos esta es una solución perfectamente válida.
Sin embargo, algunas veces los requisitos son más estrictos.
Para evitar pagar el coste extra del código generado y la indirección cuando se accede a los valores enumerados, es posible usar constantes enumeradas.
Las constantes enumeradas son definidas con el modificador `const` que precede a la palabra clave `enum`.

```ts
const enum Enum {
    A = 1,
    B = A * 2
}
```

Las constantes enumeradas solamente pueden usar expresiones constantes enumeradas y en contra de las enumeraciones regulares, son completamentes eliminadas durante la compilación.
Las miembros constantes enumerados están inclinados a usar sitios de uso.
Esto es posible desde que las constantes enumeradas no pueden tener miembros computados.

```ts
const enum Directions {
    Arriba,
    Abajo,
    Izquierda,
    Derecha
}

let directions = [Directions.Arriba, Directions.Abajo, Directions.Izquierda, Directions.Derecha]
```

En código generado se convertirá

```js
var directions = [0 /* Arriba */, 1 /* Abajo */, 2 /* Izquierda */, 3 /* Derecha */];
```

# Enumeraciones ambientales

Las enumeraciones ambientales son usadas para describir la forma de enumraciones existentes.

```ts
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

Una importante diferencia entre las enumeraciones ambientales y no ambientales es que, en las enumraciones regulares, los miembros que no tienen un inicializador son considerados miembros constantes.
Para miembros no constantes de enumeraciones ambientales que no tienen inicializador son considerados computerizados.
