# Introducción

Los genéricos *(generics)* son componentes reusables que permiten trabajar con variedad de tipos en lugar de un solo.

# El Hola Mundo de los Genéricos

Empecemos con la función de identidad (*identity*).
Es una función que retorna cualquier cosa que le pasen, similar al comando `echo`.

Podemos usarla con un tipo específico:

```ts
function identity(arg: number): number {
    return arg;
}
```

O hacerla incluso más genérica con el tipo `any`:

```ts
function identity(arg: any): any {
    return arg;
}
```

En el caso de `any` estamos perdiendo la información del tipo.
Por eso, necesitamos capturar el tipo del argumento para indicar que está siendo devuelto.
Aquí usamos un tipo variable (*type variable*), un clase especial de variable que funciona con tipos en lugar de valores.

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

Hemos añadido el tipo variable `T` a la función de identidad.
Esto nos permite capturar el tipo proporcionado por el usuario para usarlo más tarde.
También usamos `T` como el tipo del valor devuelto lo que nos permite trabajar con este valor tanto por un lado de la función como por el otro.

Ahora que hemos escrito la función de indentidad genérica, podemos llamarla de dos formas:

```ts
let output = identity<string>("myString");  // el tipo de salida sera 'string'
```

En este caso especificamos que el tipo `T` es una cadena, nótese que el uso de `<>` alrededor de los argumentos en lugar de `()`.

El segundo caso y más común, usamos la inferencia de tipo de argument *type argument inference*, esto quiere decir que queremos que el compilador establezca el valor de `T` automáticamente basándose en el tipo del argumento pasado:

```ts
let output = identity("myString");  // el tipo de salida sera 'string'
```

Quizás necesite especificar el tipo del argumento cuando el compilador falle al identificar el tipo, cosa que podría pasar en ejemplos más complejos.

# Trabajando con Variable de Tipo Genérico

Cuando se empiezan a utilizar los genéricos, el compilador le instará a que utilice correctamente en el cuerpo de la función cualquiera de los parámetros genéricos que escribió.
Es decir, que en realidad se trata a estos parámetros como si pudieran ser de cualquier tipo.

Usemos nuestra función `identity` de antes:

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

¿Y si queremos registrar el tamaño (length) del argumento `arg` a la consola con cada llamada?
Podríamos estar tentados a escribir esto:

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T no tiene .length
    return arg;
}
```

Recordemos que este tipo de variables puede ser de cualquier tipo, por lo que alguien podría haber pasado un `number`, los cuales no tienen `.length`.

Digamos que hemos propiciado que esta función trabaje con arrays de `T` en lugar de `T` directamente. Al trabajar con arrays, el método `.length` debería estar disponible:

```ts
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

Puedes leer el tipo de `loggingIdentity` como "la función genérica `loggingIdentity` tiene el parámetro del tipo `T`, y un argumento `arg` que es un array de `T`s, y devuelve un arrray de `T`s."
Si pasamos un array de número recibiremos un array de números.
If we passed in an array of numbers, we'd get an array of numbers back out, as `T` would bind to `number`.
Esto nos permite usar la variable de tipo genérico `T` como parte del tipo con el que trabajamos en lugar del tipo completo, dándonos más flexibilidad.

Alternativamente podemos escribir el ejemplo de la siguiente manera:

```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array tiene .length, no mas errores
    return arg;
}
```

In la siguiente sección cubriremos como crear tu propio tipo genérico, como `Array<T>`.

# Tipos Genéricos

En secciones previas creamos functiones de identidad genéricas que trabajaban sobre un rango de tipos.
En esta sección exploraremos el propio tipo de las funciones y como creart interfaces genéricas.

El tipo de las funciones genéricas es como las no genéricas, con la lista de parámetros primero, similar a la declaración de una función:

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```

Podriamos haber usado un nombre diferente para los parámetros de tipo genérico, tan largo como el número de variables de tipo y cómo se utilizan las variables de tipo.

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <U>(arg: U) => U = identity;
```

También podemos escribir el tipo genérico como una llamada a un tipo de objeto literal:

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: {<T>(arg: T): T} = identity;
```

Lo que nos lleva a escribir nuestra primera interfaz genérica.
Cogamos el objeto literal del ejemplo previo y movámoslo a una interfaz:

```ts
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

En un ejemplo similar, es posible que desse mover el parámetro genérico de la interfaz completa.
Esto nos permite ver el/los tipo/s genérico (e.g. `Diccionario<string>` en lugar de solo `Diccionario`).
Esto hace el tipo del parámetro visible a todos los miembros de la interfaz.

```ts
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

Tenga en cuenta que nuestro ejemplo ha cambiado para ser algo ligeramente diferente.
En lugar de describir una función genérica, tenemos una función no genérica que es parte de un tipo genérico.
Cuando usamos `GenericIdentityFn`, ahora tenemos que especificar el tipo correspondiente al argumento (en este caso: `number`).
Entendiendo cuando poner los tipos de los parámetros directamente en la llamada y cuando ponerlo en la interfaz será de ayuda al describir que asceptos del tipo son genéricos.

Además de las interfaces genéricas, también podemos crear clases genéricas.
Fíjese que no es posible crear enums ni espacios de nombre (*namespaces*) genéricos.

# Clases Genéricas

Una clase genérica tiene una forma similar a una interfaz genérica.
Las clases genéricas tienen una lista de parámetros de tipo genérico en paréntesis angulares (`<>`) seguidos del nombre de la clase.

```ts
class NumeroGenerico<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let miNumeroGenerico = new NumeroGenerico<number>();
miNumeroGenerico.zeroValue = 0;
miNumeroGenerico.add = function(x, y) { return x + y; };
```

Este es un uso literal de la clase `NumeroGenerico`, pero es posible que haya notado que nada lo restringue sólo al tipo `number`.
Podríamos usar `string` en su lugar o incluso objetos más complejos.

```ts
let stringNumeric = new NumeroGenerico<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

alert(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

Al igual que con la interfaz, poniendo el tipo de parámetro en la propia clase nos asegura que todas las propiedades de la clase funcionan con el mismo tipo.

Como ya vimos en [nuestra sección sobre clases](./Classes.md), una clase tiene dos lados para cada tipo: el estático y la instancia.
Las clases genéricas son genéricas solamente en la instancia en vez de en la parte estática, entonces cuando trabajamos con clases, los miembros estáticos no pueden usar el parámetro del tipo de la clase.

# Restricciones Genéricas

Si recuerdas en un ejemplo anterior, a veces puedes querer escribir una función genérica que funcione en una asignación (*set*) de tipos en dónde tengas algún conocimiento sobre que capacidades tendrá esa asignación de tipos.
En nuestro ejemplo de `loggingIdentity`, queríamo ser capaces de acceder a la propiedad `.length` de `arg`, pero el compilador no podía probar que cada tipo tuviera una propiedad `.length`, así que nos avisa de que no podemos hacer ese supuesto.

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T no tiene .length
    return arg;
}
```

En lugar de trabajar con todos los tipos, nos gustaría restringir esta función para trabajar con todos los tipos que también tengan la propiedad `.length`.
Mientras que el tipo tenga este miembro, lo permitiremos, pero es requerido tener al mismo este miembro.
Para hacerlo así, debemos hacer una lista de los requisitos como una restricción de lo que T puede ser.

Para esto, crearemos una interfaz que describa nuestra restricción.
Aquí crearemos una una interfaz con una sóla propiedad `.length` y entonces usaremos esta interfaz y la palabra clave `extends` para indicar nuestra restricción:

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

Debido a que la función genérica está restringida, no funcionará más sobre todos los tipos.

```ts
loggingIdentity(3);  // Error, number no tiene la propiedad .length
```

En su lugar, necesitamos pasarle los valores cuyo tipo tenga todas las propiedades requeridas.

```ts
loggingIdentity({length: 10, value: 3});
```

## Use de los Tipos de Parámetro en Restricciones Genéricas

Puedes declarar un tipo de parámetro que esté restringido por otro tipo de parámetro.
Por ejemplo, aquí nos gustaría coger dos objetos y copiar las propiedades de uno al otro.
Nos gustaría asegurar que no hemos escrito accidentalmente ninguna propiedad extra de nuestra `fuente` (*source*), así que pondremos una restricción entre los dos tipos:

```ts
function copyFields<T extends U, U>(target: T, source: U): T {
    for (let id in source) {
        target[id] = source[id];
    }
    return target;
}

let x = { a: 1, b: 2, c: 3, d: 4 };

copyFields(x, { b: 10, d: 20 }); // okay
copyFields(x, { Q: 90 });  // error: property 'Q' isn't declared in 'x'.
```

## Uso de los Tipos de Clase Genéricos

Cuando se crean *¿factories?* en TypeScript usando genéricos, es necesario referirse al tipo de clase por sus funciones constructoras. Por ejemplo:

```ts
function create<T>(c: {new(): T; }): T {
    return new c();
}
```

Un ejemplo más avanzado usa la propiedad prototipo para inferir y restringir las relaciones entre la función constructora y la instacia.

```ts
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function findKeeper<A extends Animal, K> (a: {new(): A;
    prototype: {keeper: K}}): K {

    return a.prototype.keeper;
}

findKeeper(Lion).nametag;  // typechecks!
```
