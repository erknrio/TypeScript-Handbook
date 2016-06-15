# Introducción

Los tipos de variables son: boolean, number, string, array, enum, any y void. Los que no encontramos en JavaScript directamente se explican a continuación.

[Documentación](https://www.typescriptlang.org/docs/handbook/basic-types.html).

# Aclaración sobre `let`

let es similar a var, ya que sirve para declarar variables, pero actúa dentro de un bloque de codigo. Ojo compatibilidad.

Se puede ver mejor en la declaración de variables, aunque [aquí les dejo una web](https://basarat.gitbooks.io/typescript/content/docs/let.html) donde lo explican bastante bien.

# Tipos conocidos en JavaScript

```ts
// Number: decimal, hex, binary, etc.
var listNum: number[] = [1,2,3];
// o tambien
var listNumAlternative:Array<number> = [1,2,3];

// Boolean
let isDone: boolean = false
// etc...
```

# Tuple

Al contratio que otros lenguajes, como python, las tuplas pueden cambiar pero en este caso deben mantener el tipo.

[Documentación](https://www.typescriptlang.org/docs/handbook/basic-types.html#tuple).

```ts
// Declaramos la tupla
let x: [string, number];
// Inicializamos
x = ["hello", 10]; // OK
// Error en el tipo, 10 no es string
x = [10, "hello"]; // Error
```

Al acceder a un indice fuera de rango se hace una union:

```ts
x[3] = "world"; // OK, 'string' puede usarse con el tipo 'string | number'

console.log(x[5].toString()); // OK, 'string' and 'number' ambos tienen 'toString'

x[6] = true; // Error, 'boolean' no es 'string | number'
```

El tipo Union se verá en un futuro capítulo.

# Enum

El enum funciona al contrario que un Array, en lugar de acceder a "Green" mediante la posicion 1, accedemos al 1 mediante "Green".

[Documentación](https://www.typescriptlang.org/Handbook#basic-types-enum).

```ts
enum Color {Red, Green, Blue};
let c: Color = Color.Green;
```

Tenemos tres alternativas:

1.- Cambiamos la ordenacion, por ejemplo, empezando por 1 en lugar de 0.

```ts
enum Color {Red = 1, Green, Blue};
let c: Color = Color.Green;
```

2.- También podemos alterar el orden de cada elemento

```ts
enum Color {Red = 1, Green = 2, Blue = 4};
let c: Color = Color.Green;
```

3.- Accedemos mediante posicion en lugar de su nombre.

```ts
enum Color {Red = 1, Green, Blue};
let colorName: string = Color[2];

alert(colorName);
```

# Any

El any equivaldría a la declaración actual de variables de JavaScript. Puede ser cualquier tipo.

```ts
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitivamente un boolean
```

# Void

El tipo void es el contrario a any, no devuelve nada. Para variables solo admite undefined y null.
Suele verse en funciones que no tienen return.

```ts
function warnUser(): void {
    alert("This is my warning message");
}
```

# Type assertions

Lo usa solamente el compilador y es una forma de especificar "se lo que hago". Actua como un typecast concretando el valor de una variable.
Podemos declararlos de dos maneras:

1.- Tipo "ángulos" (mayor y menor que):

```ts
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```

2.- Mediante `as`:

```ts
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

Cuando se usa TypeScript con JSX solamente se admite el segundo tipo `as`.
