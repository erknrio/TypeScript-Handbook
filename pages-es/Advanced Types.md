# Tipos de Unión

Ocasionalmente, se encontrará con una librería que espera un parámetro de tipo `number` o `string`.
Por ejemplo, tomemos la siguiente función:

```ts
/**
 * Tomamo una cadena y agregamos "padding" a la izquierda.
 * Si 'padding' es un string, entonces 'padding' es agregado al lado izquierdo.
 * Si 'padding' es un number, entonces ese numero de espacios es agregado al lado izquierdo.
 */
function padIzquierda(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

padIzquierda("Hello world", 4); // returns "    Hello world"
```

El problema con `padIzquierda` es que su parámetro `padding` está tipado como `any`.
Esto quiere decir que podemos llamarlo con un argumento que no sea `number` ni `string`.

```ts
let indentedString = padIzquierda("Hello world", true); // pasa en tiempo de compilacion pero falla en tiempo de ejecucion.
```

En el código tradicional orientado a objetos, podríamos resumir los dos tipos mediante la creación de una jerarquía de tipos.
Mientras que esto es mucho más explícito, también es un poco exagerado.
Una de las cosas buenas de la versión original de `padIzquierda` era que nos permitía pasar tipos primitivos de uso fácil.
Este nuevo enfoque no sería de ayuda si sólo estuviesemos tratando de usar una función que existiese en otro lado.

En lugar de `any`, podemos usar un tipo de union (*union type*) para el parámetro `padding`:

```ts
/**
 * Tomamos una cadena y agregamos "padding" a la izquierda.
 * Si 'padding' es un string, entonces agregamos 'padding' al lado izquierdo.
 * Si 'padding' es un number, entonces ese numero de espacios son agregados al lado izquierdo.
 */
function padIzquierda(value: string, padding: string | number) {
    // ...
}

let indentedString = padIzquierda("Hello world", true); // errors durante la compilacion
```

Un tipo unión describe un valor que puede ser uno de mucho tipos.
Usamos la barra vertical (`|`) para separar cada tipo, así que si `number | string | boolean` es el tipo de un valor, este puede ser `number`, `string` o `boolean`.

Si tenemos un valor que es un tipo unión, sólo podemos acceder a los miembros que son comunes a todos los tipos de la union.

```ts
interface Pajaro {
    volar();
    poneHuevos();
}

interface Pez {
    nadar();
    poneHuevos();
}

function getSmallPet(): Pez | Pajaro {
    // ...
}

let mascota = getSmallPet();
mascota.poneHuevos(); // okay
mascota.nadar();    // errores
```

Los tipos de unión pueden ser un poco problemáticos aquí pero sólo se necesita un poco de intuición para acostumbrarse.
Si un tipo tiene `A | B`, solamente conocemos con *certeza* que tiene miembros en ambos `A` *y* `B`.
En este ejemplo `Pajaro` tiene un miembro llamado `volar`.
No podemos estar seguros de si una variable de tipo `Pajaro | Pez` tiene un método `volar`.
Si la variable es realmente un `Pez` en tiempo de ejecución, llamando a `mascota.volar()` fallará.

# Los Guardas de Tipo y la Diferenciación de Tipos

Los tipos de unión son útiles para modelar situaciones cuando los valores se pueden solapar en los tipos que pueden tomar.

Union types are useful for modeling situations when values can overlap in the types they can take on.
¿Qué ocurre cuando necesitamos saber concretamente si tenemos un `Pez`?
Un idioma común en JavaScript para diferenciar entre dos posibles valores es comprobar la presencia de un miembro.
Como mencionamos, solamente puedes acceder a miembros que garantizan estar en todas los componentes de la unión.

```ts
let mascota = getSmallPet();

// Cada uno de los accesos a las propiedades provocara un error
if (mascota.nadar) {
    mascota.nadar();
}
else if (mascota.volar) {
    mascota.volar();
}
```
Para conseguir que funcione el mismo código, necesitamos un type assertion:

```ts
let mascota = getSmallPet();

if ((<Pez>mascota).nadar) {
    (<Pez>mascota).nadar();
}
else {
    (<Pajaro>mascota).volar();
}
```

## Guardas de Tipo Definidos por el Usuario

Fíjese que tiene que usar type assertions muchas veces.
Sería mucho mejor si una vez realizada la comprobación pudiesemos saber el tipo de `mascota` dentro de cada rama.

Lo que ocurre es que TypeScript tiene algo llamado protectores de tipos (*type guard*).
Un protector de tipos es una expresión que tiene lugar durante el tiempo de ejecución y garantiza el tipo en un ámbito (*scope*)
Para deifinir un protector de tipo, simplemente definimos una función cuyo tipo devuelto es un tipo predicado (*type predicate*):

```ts
function esPez(mascota: Pez | Pajaro): mascota is Pez {
    return (<Pez>mascota).nadar !== undefined;
}
```

`mascota is Pez` es nuestro tipo predicado en este ejemplo.
Un predicado toma la forma `nombreParametro is Tipo`, donde `nombreParametro` debe ser el nombre del parámetro de la firma de la función actual.

Cada vez que `esPez` es llamado con alguna variable, TypeScript *acotará* esa variable a un tipo específico si el tipo original es compatible.

```ts
// Ambas llamadas a 'nadar' y 'volar' ahora son correctas.

if (esPez(mascota)) {
    mascota.nadar();
}
else {
    mascota.volar();
}
```

Note que TypeScript no solo sabe que `mascota` es un `Pez` en el `if` sino que también sabe que en el `else`, *no* tienes un `Pez`, debes tener un `Pajaro`.

## `typeof` de los protectores de tipo

No discutimos la implementación de la versión de `padIzquierda` que usaba tipos de unión.
Podemos escribirla con predicados de la siguiente manera:

```ts
function esNumero(x: any): x is number {
    return typeof x === "number";
}

function esCadena(x: any): x is string {
    return typeof x === "string";
}

function padIzquierda(value: string, padding: string | number) {
    if (esNumero(padding)) {
        return Array(padding + 1).join(" ") + value;
    }
    if (esCadena(padding)) {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

Sin embargo, tener que definir una función para averiguar si un tipo es primitivo es un dolor.
Por fortuna, no tenemos que abstraer `typeof x === "number"` en su propia función porque TypeScript lo reconocerá como un protector de tipo.
Eso quiero decir que podríamos escribir estas comprobaciones en la misma línea.

```ts
function padIzquierda(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

Este *`typeof` protectores de tipo* son reconocidos de dos formas diferentes: `typeof v === "nombretipo"` y `typeof v !== "nombretipo"`, donde `"nombretipo"` debe ser `"number"`, `"string"`, `"boolean"`, o `"symbol"`.
Mientras TypeScript no prohibirá la comparación con otras cadenas o cambiar los dos lados de la comparación, el lenguaje no reconocerá estas formas como protectores de tipo.

## `instanceof` protectores de tipo

Si ha leído sobre `typeof` protectores de tipo y esta familiarizado con el operador `instanceof` en JavaScript, probablemente tiene una idea sobre de que va esta sección.

*`instanceof` protectores de tipo* son una forma de reducción de tipos usando su función constructora.
Por ejemplo, vamos a tomar prestado nuestro ejemplo de *string-padder* industrial de antes:

```ts
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces + 1).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

function getRandomPadder() {
    return Math.random() < 0.5 ?
        new SpaceRepeatingPadder(4) :
        new StringPadder("  ");
}

// Type is 'SpaceRepeatingPadder | StringPadder'
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // type narrowed to 'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
    padder; // type narrowed to 'StringPadder'
}
```

El lugar correcto de `instanceof` necesita ser la función constructora y TypeScript los reducirá a:

1. El tipo de propiedad `prototype` de la funciónsi su tipo no es `any`.
2. Los tipos de unión devueltos por las firmas de los tipos del constructor.

En ese orden.

# Tipos de intersección

Los tipos de intersección están estrechamente relacionados con los tipos de unión, pero son usados de manera muy diferente.
Un tipo de intersección `Persona & Serializable & Loggable`, por ejemplo, es una `Persona` *y* `Serializable` *y* `Loggable`.
Esto quiere decir que un objeto te este tipo tendrá todos los miembros de los tres tipos.
En la práctica mayoritariamente sólo verá el tipo intersección usado para mixins.
Este es un ejemplo simple de mixin:

```ts
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U>{};
    for (let id in first) {
        (<any>result)[id] = (<any>first)[id];
    }
    for (let id in second) {
        if (!result.hasOwnProperty(id)) {
            (<any>result)[id] = (<any>second)[id];
        }
    }
    return result;
}

class Persona {
    constructor(public name: string) { }
}
interface Loggable {
    log(): void;
}
class ConsoleLogger implements Loggable {
    log() {
        // ...
    }
}
var jim = extend(new Persona("Jim"), new ConsoleLogger());
var n = jim.name;
jim.log();
```

# Tipo alias

Los tipos alias crea un nuevo nombre para un tipo.
Algunas veces son similares a las interfaces pero pueden nombrar tipos primitivos, uniones, tuplas y otros tipos que de lo contrario tendría que escribir a mano.

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === "string") {
        return n;
    }
    else {
        return n();
    }
}
```

Los alias no crean un nuevo tipo, crean un nuevo *nombre* que referencia a ese tipo.
Los alias de tipos primitivos no son muy útiles, aunque pueden ser usados como documentación.

Como las interfaces, los alias pueden ser genéricos, podems agregar el tipo de parámetros y usarlos en el lado correcto de la declaración del alias:

```ts
type Container<T> = { value: T };
```

También podemos tener un alias refiriendose a sí mismo en una propiedad:

```ts
type Arbol<T> = {
    value: T;
    left: Arbol<T>;
    right: Arbol<T>;
}
```

Junto con los tipos de intersección, podemos hacer algunos tipos alucinantemente bonitos:

```ts
type LinkedList<T> = T & { next: LinkedList<T> };

interface Persona {
    name: string;
}

var people: LinkedList<Persona>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
```

Sin embargo, no es posible para el tipo alias aparecer en cualquier lugar en el lado derecho de la declaración:

```ts
type Yikes = Array<Yikes>; // error
```

## Tipo Alias vs Interfaces

Como mencionamos, los alias pueden actuar como interfaces.
Sin embargo, hay algunas sutiles diferencias.

Una diferencia importante es que el tipo alias no puede ser extendido o implementado (tampoco puede extender o implementar de otros tipos).
Porque [una propiedad ideal del software es ser abierto a la extensión](https://en.wikipedia.org/wiki/Open/closed_principle).
Debería usar siempre una interfaz en lugar de un alias si es posible.

Por otro lado, si no puede expresar una forma con una interfaz y necesita una unión o tupla, el tipo alias es lo usado normalmente.

# Tipos de Cadena Literales

Los tipos de cadena literales le permiten especificar el valor exacto que una cadena debe tener.
En la práctica una cadena literal combina muy bien con los tipos de union, protectores y alias.
Puede usar estas características en conjunto para obtener con cadenas un comportamiento similar a las enumeraciones.

```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        }
        else if (easing === "ease-out") {
        }
        else if (easing === "ease-in-out") {
        }
        else {
            // error! should not pass null or undefined.
        }
    }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // error: "uneasy" is not allowed here
```

Puede pasar cualquiera de las tres cadenas permitidas, pero cualquier otra cadena dará un error.

```text
Argumento del tipo '"uneasy"' no es assignable al parametro de tipo '"ease-in" | "ease-out" | "ease-in-out"'
```

Los tipos de cadena literales pueden ser usados de la misma manera para distinguir sobrecargas:

```ts
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
    // ... code goes here ...
}
```

# Tipos `this` polimórficos

Un tipo `this` polimórfico representa a un tipo que es un *subtipp* de la clase o interfaz contenedora.
Esto es llamado como polimorfirsmo delimitado-*F*.

Esto hace las jerarquías de interfaces más fluidas y fáciles de expresar, por ejemplo.
Cogamos una calculadora simple que retorne `this` después de cada operación:

```ts
class BasicCalculator {
    public constructor(protected value: number = 0) { }
    public currentValue(): number {
        return this.value;
    }
    public add(operand: number): this {
        this.value += operand;
        return this;
    }
    public multiply(operand: number): this {
        this.value *= operand;
        return this;
    }
    // ... other operations go here ...
}

let v = new BasicCalculator(2)
            .multiply(5)
            .add(1)
            .currentValue();
```

Puesto que la clase usa los tipos `this`, puede extenderlo y la nueva clase puede usar los viejos métodos sin cambios.

```ts
class ScientificCalculator extends BasicCalculator {
    public constructor(value = 0) {
        super(value);
    }
    public sin() {
        this.value = Math.sin(this.value);
        return this;
    }
    // ... other operations go here ...
}

let v = new ScientificCalculator(2)
        .multiply(5)
        .sin()
        .add(1)
        .currentValue();
```

Sin el tipo `this`, `ScientificCalculator` no le sería posible extender `BasicCalculator` y mantener el flujo de la interfaz.
`multiply` habría retornado `BasicCalculator`, el que no tine el método `sin`.
Sin embargo, con los tipos `this`, `multiply` devuelve `this`, el cual es `ScientificCalculator` en este caso.
