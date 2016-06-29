# Introducción

Empezando con ECMAScript 2015, `symbol` es un tipo de dato primitivo, como `number` y `string`.

Los valores `symbol` son creados llamando al constructor de `Symbol`.

```ts
let sym1 = Symbol();

let sym2 = Symbol("key"); // optional string key
```

Los símbolos son inmutables y únicos.

```ts
let sym2 = Symbol("key");
let sym3 = Symbol("key");

sym2 === sym3; // false, symbols son unicos
```

Como las cadenas, los símbolos pueden ser usados como las claves de las propiedades de los objetos.

```ts
let sym = Symbol();

let obj = {
    [sym]: "value"
};

console.log(obj[sym]); // "value"
```

Los símbolos también pueden ser combinados con las declaraciones de propiedad computada para declarar propiedades y miembros de clase.

```ts
const getClassNameSymbol = Symbol();

class C {
    [getClassNameSymbol](){
       return "C";
    }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

# Símbolos Conocidos

Además de los símbolos definidos por el usuario, hay símbolos conocidos preconstruidos.
Los símbolos preconstruidos son usados para representar comportamientos internos del lenguaje.
Aquí tiene una lista de símbolos conocidos que podrá encontrar en su mayoría en la documentación de [MDN](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Symbol#Símbolos_bien_conocidos):

## `Symbol.hasInstance`

Un método que determina si un objeto constructor reconoce un objeto como una de las instancias del constructor.
Llamado por la semántica del operador instanceof.

## `Symbol.isConcatSpreadable`

Un valor Boolean indicando que un objeto se debe reducir a sus elementos de array por Array.prototype.concat.

## `Symbol.iterator`

Un método que retorna el iterador por defecto de un objeto.
Llamado por la semántica de la declaración for-of.

## `Symbol.match`

Un método de expresión regular que concuerda la expresión regular con una cadena.
Llamado por el método `String.prototype.match`.

## `Symbol.replace`

Un método de expresión regular que concuerda con una subcadena de una cadena.
Llamado por `String.prototype.replace`.

## `Symbol.search`

Un método de expresión regular que devuelve el índice dentro de una cadena que concuerda con la expresión regular.
Llamado por `String.prototype.search`.

## `Symbol.species`

Una valiosa propiedad de función que es la función constructora y es usada para crear opbjetos derivados.

## `Symbol.split`

Un método de expresión regular que divide una cadena a los índices que concerdan con la expresión regular.
Llamado por el método `String.prototype.split`.

## `Symbol.toPrimitive`

Un método que convierte un objeto en su correspondiente valor primitivo.
Llamado bor el operador abstracto `ToPrimitive`.

## `Symbol.toStringTag`

Una valor de cadena que es usada en la creación de una descripción en una cadena de un objeto.
Llamado por el método predefinidio `Object.prototype.toString`.

## `Symbol.unscopables`

Un objeto cuyas propias propiedades son excluídas de las ataduras del entorno del objeto asociado (*from the ‘with’ environment bindings of the associated objects*).
