# Introducción

La compatibilidad de tipos en TypeScript esta basado en el suptipo estructural.
La tipificación estructural  es una manera de relacionar los tipos basados solamente en sus miembros.
Esto está en contraposición a la tipificación nominal
Considere el siguiente código:

```ts
interface Named {
    nombre: string;
}

class Persona {
    nombre: string;
}

let p: Named;
// OK, debido al tipificado estructural
p = new Persona();
```

En lenguajes nominalmente tipados como C# o Java, el código equivalente resultaría en un error porque la clase `Persona`no se describe a si misma como un implementador de la interfaz `Named`.

La tipificación estructural de TypeScript fue diseñada sobre como es escrito normalmente el código JavaScript.

Debido a que JavaScript usa ampliamente objetos anónimos como expresiones de función y objetos literales es más natural representar el tipo de relaciones que se encuentran en las librerías de JavaScript con un sistema de tipo estructural en lugar de uno nominal.

## Una nota sobre la solidez

El sistema de tipos de TypeScript permite ciertas operaciones que no pueden ser conocidas durante el tiempo de compilación para ser seguras.
Cuando un tipo de sistema tiene esta propiedad se dice que no son "sonoros" (*"sound"*).
Los lugares donde TypeScript permite un compotamiento erróneo fueron considerados cuidadosamente y a lo largo de este documento explicaremos donde ocurre y los ecenarios que los motivaron.

# Empezando

La regla básica del sistema de tipos estructural para TypeScript es que `x` es compatible con `y` si `y` tiene al menos los mismos elementos que `x`. Por ejemplo:

```ts
interface Named {
    nombre: string;
}

let x: Named;
// y's inferred type is { nombre: string; localizacion: string; }
let y = { nombre: "Alice", localizacion: "Seattle" };
x = y;
```

Para comprobar que `y` puede ser asignado a `x`, el compilador comprueba cada propiedad de `x` para encontrar una propiedad compatible `y`.
En este caso, `y` debe tener un elemento llamado `nombre` que sea una cadena. Si lo tiene, la asignacioń es permitida.

La misma regla de asignación es usada para comprobar los parámetro de llamada a una función:

```ts
function greet(n: Named) {
    alert("Hello, " + n.nombre);
}
greet(y); // OK
```

Fíjese que  `y` tiene una propiedad extra `localizacion`, pero ésta no genera un error.
Solamente elemento del tipo objetivo (`Named` en este caso) son considerados cuando se comprueba la compatibilidad.

Este proceso de comparación se produce recursivamente, explorando el tipo de cada elemento y sub-elemento.

# Comparando dos funciones

Mientras que comparar tipos primitivos y tipos de objetos es relativamente directo, la cuestión de que tipo de funciones deberían considerar compatibles es un poco más complicado.
Empecemos con un ejemplo básico de dos funciones que difieren solamente en la lista de parámetros:

```ts
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```

Para comprobar si `x` es asinable a `y`, primero buscamos en la lista de parámetros.
Cada parámettro en `x` debe tener un parámetro correspondiente en `y` con un tipo compatible.
En este caso se cumple.

El segundo asignamiento es un error porque `y` tiene un segundo parámetro que `x` no tiene, entonces no está permitido.

Debes estar sorprendido de porque permitimos 'descarta' parámetros como en el ejemplo `y = x`.
La razón es que ignorar parámetros extra de en la función es algo común actualmente en JavaScript.
Por ejemplo, `Array#forEach` provee tres parámetro a la función de retorno: el array de elementos, sus índice y el contenido del array.
Sin embargo, es muy común proporcionar una función de retorno  que solamente use el primer parámetro:

```ts
let elementos = [1, 2, 3];

// No fuerce estos parametros extra
elementos.forEach((elemento, index, array) => console.log(elemento));

// Deberia estar OK!
elementos.forEach(elemento => console.log(elemento));
```

Ahora veamos como los tipos devueltos son tratado usando dos funciones que solamente difieren por su valor devuelto:

```ts
let x = () => ({nombre: "Alice"});
let y = () => ({nombre: "Alice", localizacion: "Seattle"});

x = y; // OK
y = x; // Error porque x() carece de la propiedad localizacion
```

El sistema de tipos fuerza a que el valor devuelto de la función fuente sea un subtipo del valor devuelto del destino.

## Function Parameter Bivariance

Cuando se está comparando los tipos de función de los parámetros, los asignamientos tienen éxito si incluso el parámetro de fuente es asignable al target parámetro, o al revés.  
Esto es unsonund pq un llamador debe terminar siendo dado una función que tenga un tipo más especializado,pero invoca la función con un tipo menos especializado.
En la práctica,  este tipo de error es raro y permitiendo esto deshabilita muchos patterns comunes de Javascript. Un buen ejemplo

```ts
enum EventType { Mouse, Keyboard }

interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// Unsound, but useful and common
listenEvent(EventType.Mouse, (e: MouseEvent) => console.log(e.x + "," + e.y));

// Undesirable alternatives in presence of soundness
listenEvent(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + "," + (<MouseEvent>e).y));
listenEvent(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + "," + e.y)));

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
listenEvent(EventType.Mouse, (e: number) => console.log(e));
```

## Parámetros opcionles y rest

Cuando comparamos funciones por compatibilidad, los parámetros opcionales y requeridos son intercambiables.
Los parámetros opcionales extra del tipo fuente no son un error y los parámetros opcionales del tipo destino sin parámetros correspondientes en el tipo destino no son un error.

Cuando una función tiene un parámetro rest, es tratada como si fuera una serie infinita de parámetros opcionales.

Esto es poco sólido desde una perspectiva del sistema de tipos, pero desde un punto de vista de tiempo de ejecución la idea de un parámetro opcional generalmente no está bien aplicada, ya que pasando `undefined` en esa posición es equivalente para la mayoría de las funciones.

El ejemplo motivador es el patrón común de una función que toma una devolución de llamada y la invoca con un número de parámetros un poco predecible (al programador) pero desconocido (para el sistema de tipos):

```ts
function invokeLater(args: any[], callback: (...args: any[]) => void) {
    /* ... Invoke callback with 'args' ... */
}

// Unsound - invokeLater "might" provide any number of arguments
invokeLater([1, 2], (x, y) => console.log(x + ", " + y));

// Confusing (x and y are actually required) and undiscoverable
invokeLater([1, 2], (x?, y?) => console.log(x + ", " + y));
```

## Sobrecarga de funciones

Cuando una función tiene sobrecargas, cada sobrecarga en el tipo de origen debe concordar con una firma compatible en el tipo de destino.
Esto asegura que la función de destino puede ser llamada en todas las mismas situaciones que la función de origen.

# Enumeraciones

Las enumeraciones son compatibles con números y los números son compatibles con enumeraciones.
Los valores enumerados de diferentes tipos se consideran incompatibles. Por ejemplo:

```ts
enum Estado { Preparado, Esperando };
enum Color { Rojo, Azul, Verde };

let estado = Estado.Preparado;
estado = Color.Verde;  //error
```

# Clases

Las clases funcionan similarmente a tipos de objetos literales e interfaces con una excepción: ambos tienen tipos estáticos y de instancia.
Cuando comparamos dos objetos del tipo clase, solamente se comparan los miembros de la instancia.
Miembros estáticos y constructores no afectan a la compatibilidad.

```ts
class Animal {
    feet: number;
    constructor(nombre: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

let a: Animal;
let s: Size;

a = s;  //OK
s = a;  //OK
```

## Miembros privados y protegidos en las clases

Los miembros privados y protegidos en una clases afectan su compatibilidad.
Cuando una instancia de una clase es comprobada por compatibilidad, si la instancia contiene un miembro privado, entonces el tipo destino debe contener también un mimebro privado originado desde la misma clase.
Lo mismo se aplica a instancias con miembros protegidos.
Esto permite que una clase sea compatible con la asignación de su super clase, pero *no* con clases de diferente herencia que tengan la misma forma.

# Genéricos

Debido a que TypeScript es una sistema de tipificación estructural, los tipos de parámetros solo afectan el tipo resultante cuando se consuman como parte del tipo d eun miembro. Por ejemplo,

```ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // okay, y matches structure of x
```

En el ejemplo anterior, `x` e `y` son comaptibles porque sus estructuras no usan el argumento de tipo de una forma diferenciadora.
Veamos como funciona este ejemplo al añadir un miembro a `Empty<T>`:

```ts
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // error, x e y no son compatibles
```

De esta manera, un tipo genérico que tiene sus tipos de argumentos actúa como un tipo no genérico.

Para tipos genéricos que no tienen sus tipos de argumentos especificados, la compatibilidad es comprobada al especificar `any` en lugar de todos los tipos de argumentos no especificados.
Los tipos resultantes se comprueban por compatibilidad, al igual que en el caso no genérico.

Por ejemplo:

```ts
let identity = function<T>(x: T): T {
    // ...
}

let reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // Okay because (x: any)=>any matches (y: any)=>any
```

# Temas avanzados

## Subtipo vs Asignación

Hasta ahora, hemos utilizado "compatible", que no es un término definido en la especificación del lenguaje.
En TypeScript, hay dos tipos de compatibilidad: el subtipo y la asignación.
Estos sólo se diferencian en que la asignación extiende la compatibilidad del subtipo con reglas para permitir la asignación desde y hacia `any` y desde una enumeración con los valores numéricos correspondientes.

Diferentes lugares en la lengua utilizan uno de los dos mecanismos de compatibilidad, dependiendo de la situación.
A efectos prácticos, la compatibilidad tipo está dictada por la asignación de la compatibilidad incluso en los casos de las claúsulas `implements` y `extends`.
Para más información, consulte [TypeScript spec](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md).
