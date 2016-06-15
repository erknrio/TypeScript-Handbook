# Declaración de Variables

Resumiré esta seccion todo lo posible ya que se presuponen conocimientos básicos sobre `JavaScript` y su declaración de variables.

Con `var` una variable obtendrá el último valor establecido independientemente de la cantidad de veces que la declaremos.

Por su lado, `let` funciona solamente en el bloque de código donde se declaró proporcionando un comportamiento diferente.

Por último, `const` actúa de forma similar a `let` pero impidiendo la redeclaración de la variable.
Lo que sí permite es cambiar el valor de una variable ya establecida.

Pongamos algunos ejemplos de uso:

[Documentación](https://www.typescriptlang.org/docs/handbook/variable-declarations.html).

# `var` vs `let`

Con var:

```ts
var a = 10;
var a = 20;
```

En el ejemplo anterior `a` valdrá `20`. Pero, ¿qué pasa si usamos let?

```ts
let x = 10;
let x = 20; // error
```
En el caso de `let` no podemos re-declarar `x` en el mismo ámbito.

Veamos un ejemplo un poco más complejo. ¿Que devolvería i?

```ts
for (var i = 0; i < 10; i++) {
    setTimeout(function() {console.log(i); }, 100 * i);
}
```

Si pensaste en esto:

```text
0
1
2
3
4
5
6
7
8
9
```

estabas equivocado, devuelve esto:

```text
10
10
10
10
10
10
10
10
10
10
```

Debido a que la función tiene un setTimeout de 100 milisegundos y toma el valor de i cuando el bucle ha terminado y la variable i tiene el valor de 10.
¿Cómo logramos este timeout con var?

```ts
for (var i = 0; i < 10; i++) {
    // capture the current state of 'i'
    // by invoking a function with its current value
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```

Pero con `let` si funciona como esperamos debido a que crea un nuevo ámbito por cada iteración del bucle for.

De esta forma setTimeout tendrá el valor que i tenía en cada iteración.

```ts
for (let i = 0; i < 10 ; i++) {
    setTimeout(function() {console.log(i); }, 100 * i);
}
```
El resultado:

```text
0
1
2
3
4
5
6
7
8
9
```

# `const` declarations

Debemos recordar que `const` no permite redeclarar una variable pero sí cambiar su valor.

```ts
const numVidasDelGato = 9;
const gatito = {
    nombre: "Aurora",
    numVidas: numVidasDelGato,
}

// Error
kitty = {
    nombre: "Danielle",
    numVidas: numVidasDelGato
};

// all "ok"
kitty.nombre = "Rory";
kitty.nombre = "Kitty";
kitty.nombre = "Cat";
kitty.numVidas--;
```

# Desestructuración (Destructuring)

Dividiremos esta explicación en array y objetos, para la información completa lea [este artículo de Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).

## Desestructuración de Array

Existen varias formas de desesturcturar. Pongamos varios ejemplos:

```ts
let input = [1, 2];
let [primero, segundo] = input;
console.log(primero); // outputs 1
console.log(segundo); // outputs 2
```

El ejemplo anterior crea dos variables llamadas `primero` y `segundo`.
Podemos usar una variable ya declarada como `input` o directamente los valores `[1,2]`. El equivalente en ES5 sería:

```ts
primero = input[0];
segundo = input[1];
```

Podemos usarlo con parámetros en una función. Ojo compatibilidad:

```ts
function f([primero, segundo]: [number, number]) {
    console.log(primero);
    console.log(segundo);
}
f(input);
```

¿Y si desconocemos la cantidad de valores que recibiremos? Podemos usar: `...variable`:

```ts
let [primero, ...resto] = [1, 2, 3, 4];
console.log(primero); // outputs 1
console.log(resto); // outputs [ 2, 3, 4 ]
```

No hace falta que tomemos todos los valores, podemos obtener solo los que nos interesen, por ejemplo:

```ts
let [, segundo, , fourth] = [1, 2, 3, 4];
```

## Destrcuturación de Objectos

Podemos declararlo de dos formas:

1.-

```ts
let o = {
    a: "foo",
    b: 12,
    c: "bar"
}
let {a, b} = o;
```
2.-

```ts
({a, b} = {a: "baz", b: 101});
```

### Renombrar propiedades

```ts
let {a: nuevoNombre1, b: nuevoNombre2} = o;
```

Resultaría en el siguiente código:

```ts
let nuevoNombre1 = o.a;
let nuevoNombre2 = o.b;
```

De todas formas no se mantienen los tipos de los valores en la deconstrucción, hay que especificarlos:

```ts
let {a, b}: {a: string, b: number} = o;
```

### Valores por defecto

Mantenemos las propiedades `a` y `b` del objeto aún si `b` es undefined.

```ts
function keepWholeObject(wholeObject: {a: string, b?: number}) {
    let {a, b = 1001} = wholeObject;
}
```

## Function declarations

Destructuring also works in function declarations.
For simple cases this is straightforward:

```ts
type C = {a: string, b?: number}
function f({a, b}: C): void {
    // ...
}
```

But specifying defaults is more common for parameters, and getting defaults right with destructuring can be tricky.
First of all, you need to remember to put the type before the default value.

```ts
function f({a, b} = {a: "", b: 0}): void {
    // ...
}
f(); // ok, default to {a: "", b: 0}
```

Then, you need to remember to give a default for optional properties on the destructured property instead of the main initializer.
Remember that `C` was defined with `b` optional:

```ts
function f({a, b = 0} = {a: ""}): void {
    // ...
}
f({a: "yes"}) // ok, default b = 0
f() // ok, default to {a: ""}, which then defaults b = 0
f({}) // error, 'a' is required if you supply an argument
```

Use destructuring with care.
As the previous example demonstrates, anything but the simplest destructuring expressions have a lot of corner cases.
This is especially true with deeply nested destructuring, which gets *really* hard to understand even without piling on renaming, default values, and type annotations.
Try to keep destructuring expressions small and simple.
You can always write the assignments that destructuring would generate yourself.
