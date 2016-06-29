# Iterables

Un objeto es considerado iretable si tiene una implemntación de la propiedad [`Symbol.iterator`](Symbols.md#symboliterator).
Algunos tipos preconstruidos como `Array`, `Map`, `Set`, `String`, `Int32Array`, `Uint32Array`, etc. tienen su propia proeidad ya implementada de  `Symbol.iterator`.
La función `Symbol.iterator` en un objeto es responsable de devolver la lista de valores a iterar.

## Declaración `for..of`

El bucle `for..of` sobre un objeto iterable invoca la propiedad `Symbol.iterator` en el objeto.
Aquí hay un bucle sencillo de `for..of` en un array:

```ts
let someArray = [1, "string", false];

for (let entry of someArray) {
    console.log(entry); // 1, "string", false
}
```

### Declaraciones `for..of` vs. `for..in`

Ambos, `for..of` y `for..in` iteran sobre listas, pero los valores iterados son diferentes.
Mientras que `for..in` devuelve una lista de claves (*keys*) del objeto iterado, `for..of` devuelve una lista de valores (*values*) de las propiedades numéricas del objeto iterado.

Un ejemplo que demuestra esta distinción:

```ts
let list = [4, 5, 6];

for (let i in list) {
   console.log(i); // "0", "1", "2",
}

for (let i of list) {
   console.log(i); // "4", "5", "6"
}
```

Otra distinción es que mientras `for..in` opera sobre cualquier objeto y sirve como una forma de inspeccionar las propiedades de un objeto, `for..of` está mayoritariamente interesado en los valores de los objetos iterables.
Objetos preconstruidos como `Map` y `Set` implementan la propiedad `Symbol.iterator` permitiendo acceso a los valores almacenados.

```ts
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets) {
   console.log(pet); // "species"
}

for (let pet of pets) {
    console.log(pet); // "Cat", "Dog", "Hamster"
}
```

### Generación de Código

#### Objetivos ES5 y ES3

Si nos enfocamos en ES5 o ES3, solamente están permitidos los iteradores en los valores del tipo `Array`.
Es un error usar bucles `for..of` en valores que no sean arrays, incluso si esos no arrays implementan la propiedad `Symbol.iterator`.

El compilador generará un simple bucle `for` para un bucle `for..of`, por ejemplo:

```ts
let numbers = [1, 2, 3];
for (let num of numbers) {
    console.log(num);
}
```

se generará como:

```js
var numbers = [1, 2, 3];
for (var _i = 0; _i < numbers.length; _i++) {
    var num = numbers[_i];
    console.log(num);
}
```

#### Orientándonos a ECMAScript 2015 y mayor

Cuando nos orientamos al motoro de compilación de ECMAScipt 2015, el compilador generará bucles `for..of` para orientar la implementación de iterador preconstruidos en el motor.
