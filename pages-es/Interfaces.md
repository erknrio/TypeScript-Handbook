# Introducción

En TypeScript el chequeo del tipo tiene que ver con la *forma*.
A esto se le llama "duck typing" or "structural subtyping". Pondré un ejemplo con y sin duck typing:

* Sin:
A una función la llamamos con un objeto Pato e invocamos a los métodos caminar y graznar.

* Con:
A una función la llamamos con cualquier objeto e invocamos a los métodos caminar y graznar.
Si este objeto no dispone de dichos métodos se manda un error.

De estos ejemplos determinamos que importa la forma no el contenedor en sí.
No importa el objeto que se mande, solamente que contenga los métodos / propiedades requeridos.

# La interfaz más simple

Pongamos un ejemplo sin usar interfaz:

```ts
function imprimirEtiqueta(etiquetaObj: { etiqueta: string }) {
    console.log(etiquetaObj.etiqueta);
}

let myObj = {size: 10, etiqueta: "Size 10 Object"};
imprimirEtiqueta(myObj);
```

Como veremos también en el ejemplo a continuación en el que usaremos una interfaz, la función `imprimirEtiqueta` recibe un objeto con la propiedad `etiqueta` de tipo `string`.
No importa el orden de las propiedades del objeto recibido ni si existen más propiedades en dicho objeto, solamente importa se comprueba que la propiedad requerida `etiqueta` se encuentre en el objeto recibido y sea del tipo establecido.

```ts
interface valorEtiquetado {
    etiqueta: string;
}

function imprimirEtiqueta(etiquetaObj: valorEtiquetado) {
    console.log(etiquetaObj.etiqueta);
}

let myObj = {size: 10, etiqueta: "Size 10 Object"};
imprimirEtiqueta(myObj);
```

# Optional Properties

No todas las propiedades de una interfaz son requeridos.
Pueden darse situaciones en las que sólo se necesiten algunas propiedades.
Pongamos un ejemplo:

```ts
interface CuadradoConfig {
    color?: string;
    width?: number;
}

function crearCuadrado(config: CuadradoConfig): {color: string; area: number} {
    let nuevoCuadrado = {color: "white", area: 100};
    if (config.color) {
        nuevoCuadrado.color = config.color;
    }
    if (config.width) {
        nuevoCuadrado.area = config.width * config.width;
    }
    return nuevoCuadrado;
}

let miCuadrado = crearCuadrado({color: "negro"});
```

Las interfaces con propiedades opcionales se escriben con un `?` al final del nombre de la propiedad, en la declaración.

Una ventaja de las propiedades opcionales es que previene el uso de propiedades que no existen en la interfaz.
Por ejemplo, nos equivocamos al escribir `color` en `crearCuadrado` y obtenemos un mensaje de error:

```ts
interface CuadradoConfig {
    color?: string;
    width?: number;
}

function crearCuadrado(config: CuadradoConfig): { color: string; area: number } {
    let nuevoCuadrado = {color: "white", area: 100};
    if (config.color) {
        // Error: Property 'collor' does not exist on type 'CuadradoConfig'
        // Error: Propiedad 'collor' no existe en el tipo 'CuadradoConfig'
        nuevoCuadrado.color = config.collor;
    }
    if (config.width) {
        nuevoCuadrado.area = config.width * config.width;
    }
    return nuevoCuadrado;
}

let miCuadrado = crearCuadrado({color: "negro"});
```

# Exceso de comprobaciones en las propiedades

Vayamos directos a un ejemplo.

```ts
interface CuadradoConfig {
    color?: string;
    width?: number;
}

function crearCuadrado(config: CuadradoConfig): { color: string; area: number } {
    // ...
}

let miCuadrado = crearCuadrado({ colour: "red", width: 100 });
```

En el siguiente ejemplo vemos como, por error, en `crearCuadrado` hemos introducido la propiedad *`colour`* en lugar de `color`.
En JavaScript no obtendríamos un error ya que la propiedad `width` sí existe en `CuadradoConfig` pero en TypeScript obtendremos un error advirtiendo que podríamos habernos equivocado.

Por lógica, no deberíamos pasar a una función más parámetros de los necesarios, por este motivo se hace un *exceso de comprobaciones en las propiedades* cuando se asignan objetos literales a otras variables o se pasan como argumentos.

```ts
// error: 'colour' not expected in type 'CuadradoConfig'
let miCuadrado = crearCuadrado({ colour: "red", width: 100 });
```

Para evitarlo podemos usar una [type assertion](./Basic Types.md).

```ts
let miCuadrado = crearCuadrado({ width: 100, opacity: 0.5 } as CuadradoConfig);
```

Sin embargo, una forma mejor de actuar sería establecer en la interfaz `CuadradoConfig` que puede haber propiedades extra:

```ts
interface CuadradoConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

Más adelante veremos este de propiedades pero estamos diciendo que `CuadradoConfig` puede tener un *número indeterminado* de propiedades que no sean `color` ni `width` y de cualquier tipo.

Una última aproximación al problema es guardar el objeto dentro de otra variable ya que en este caso no se raliza *exceso de comprobaciones en las propiedades*:

```ts
let opcionesCuadrado = { colour: "red", width: 100 };
let miCuadrado = crearCuadrado(opcionesCuadrado);
```

Tenga cuidado porque aunque estos ejemplos "parchean" el problema no dejan de ser bugs que deberían ser revisados.

# Tipos de funciones

Las interfaces son capaces de describir los tipos de una función.
Esto implica la lista de parámetros y retorna el tipo. Es requerido nombre y tipo para cada parámetro:

```ts
interface BuscarFunc {
    (fuente: string, subString: string): boolean;
}
```

Una vez definido podemos usar esta interfaz para definir los parámetros de una función.

Los parámetros se comprueban uno a uno según su orden y tipo. Podemos usar diferentes nombres para los parámetros de la función que vamos a implementar así como no espeficiar los tipos.
Estos tipos se tomarán directamente de `BuscarFunc` incluído el retorno de la función.

Veamos un ejemplo:

```ts
let mySearch: BuscarFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    if (result == -1) {
        return false;
    }
    else {
        return true;
    }
}
```

Como podemos comprobar tomamos los tipos de los parámetros de `BuscarFunc`, que serán string.
Al realizarse una comprobación por orden los nombres de los parámetros se pueden cambiar mientras coincidan sus tipos (pruebe a cambiar `src` a `src:number` y obtendrá un error).
También tomamos el tipo de retorno de la función de `BuscarFunc` que será de tipo boolean (`true` y `false`).

# Tipos indexables

Podeos describir tipos en los que podemos indexar como `a[10]`, or `mapaEdad["daniel"]`.
Los tipos indexables tienen un *índice de firma (index signature)* que describe los tipos que podemos usar para indexar dentro de un objecto junto con su correspondiente tipo cuando es indexado.
Let's take an example:

```ts
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

En el ejemplo superior, la interfaz `StringArray` tiene un índice de firma.
`StringArray` es indexado con un `number` y devuelve un `string`.

Hay dos tipos de índices soportados: string y number.
Es posible usar ambos índices, pero el tipo retornado desde un indexador numérico debe ser un subtipo del tipo retornado por un indexador de cadena.
Esto pasa porque cuando indexamos con un `number`, JavaScript lo convierte a `string` antes de indexarlo en un objeto.
Eso significa que `100` (un `number`) es lo mismo que indexar con `"100"` (un `string`), estos dos deben ser consistentes.

```ts
class Animal {
    name: string;
}
class Perro extends Animal {
    breed: string;
}

// Error: indexando con un 'string' algunas veces te dara un Perro!
interface NotOkay {
    [x: number]: Animal;
    [x: string]: Perro;
}
```

Mientras que los índices de firma son un método poderoso para describir el patrón de un "diccionario" (objeto), también fuerza a todas las propiedades de este a coincidir en su tipo retornado.
Esto es porque los índices de cadena declaran que `obj.property` está también disponible como `obj["property"]`.
En el siguiente ejemplo `nombre` (string) no coindice con el tipo del índice (number) y el comprobador de tipos devuelve un error:

```ts
interface DiccionarioNumero {
    [index: string]: number;
    length: number;    // ok, length es un number
    nombre: string;      // error, el tipo de 'name' (string) no es un subtipo del indexador (number)
}
```

# Tipos de clase

## Implementando una interfaz

Uno de los usos más comunes de las interfaces en C# y Java, es que fuerzan a la clase a implementar una declaración concreta.
Es posible hacerlo en TypeScript pero solamente con la parte pública de la clase.

```ts
interface RelojInterfaz {
    horaActual: Date;
}

class Reloj implements RelojInterfaz {
    horaActual: Date;
    constructor(h: number, m: number) { }
}
```

También podemos describir métodos en la interfaz que serán implementados en la clase, como `setTime` en el siguiente ejemplo:

```ts
interface RelojInterfaz {
    horaActual: Date;
    setTime(d: Date);
}

class Reloj implements RelojInterfaz {
    horaActual: Date;
    setTime(d: Date) {
        this.horaActual = d;
    }
    constructor(h: number, m: number) { }
}
```

## Diferencias entre la parte estática y la instacia de las clases

No se puede crear directamente una clase que implemente una interfaz con un constructor, devolverá error:

```ts
interface ConstructorReloj {
    new (hour: number, minute: number);
}

class Reloj implements ConstructorReloj {
    horaActual: Date;
    constructor(h: number, m: number) { }
}
```

Este error es causado porque solamente se comprueba la parte de la instacia de una clase, el constructor, al formar parte de la parte estática no se comprueba.

En su lugar tendrá que trabajar con la parte estática directamente.
En este ejemplo definimos dos interfaces: `ConstructorReloj` para el constructor y `RelojInterfaz` para los métodos de la instancia.
Por comodidad definimos una función para el constructor `crearReloj` que crea una instancia del tipo que se le pase.

```ts
interface ConstructorReloj {
    new (hour: number, minute: number): RelojInterfaz;
}
interface RelojInterfaz {
    tick();
}

function crearReloj(ctor: ConstructorReloj, hour: number, minute: number): RelojInterfaz {
    return new ctor(hour, minute);
}

class RelojDigital implements RelojInterfaz {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class RelojAnalog implements RelojInterfaz {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

let digital = crearReloj(RelojDigital, 12, 17);
let analog = crearReloj(RelojAnalog, 7, 32);
```

Debido a que el primer parametro de `crearReloj` es de tipo `ConstructorReloj`, en `crearReloj(RelojAnalog, 7, 32)`, comprueba que `RelojAnalog` tiene la correcta firma de constructor (constructor signature).

# Extendiendo interfaces

Las interfaces se pueden extender de forma múltiple creando combinaciones.

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

# Tipos híbridos

En JavaScript a veces encontramos objetos que actúan como una combinación de los tipos que hemos visto anteriormente.
Veamos un nuevo ejemplo de un objeto que actúa como tal y como una función con propiedades adicionales:

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

Cuando interactúa con JavaScript de terceras partes, tal vez necesite patrones como el anterior que decriban la forma del tipo.

# Interfaces que extienden clases

Cuando una interfaz extiende un tipo clase, hereda los miembros, incluídos los protegidos y privados de la clase pero no sus implementaciones.
Esto implica que, al tener elementos privados y protegidos, esta interfaz solamente puede ser implementada por clases o subclases de esta.

Es útil cuando tiene largas herencias y quiere especificar que el código trabaja sólo con subclases que tienen ciertas propiedades.
The subclasses don't have to be related besides inheriting from the base class.
For example:

```ts
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control {
    select() { }
}

class TextBox extends Control {
    select() { }
}

class Image extends Control {
}

class Location {
    select() { }
}
```

En el ejemplo superior, `SelectableControl` contiene todos los miembros de `Control`incluyendo la propiedad privada `state`.

Como `state` es un elemento privado, solamente los descendiente de `Control` pueden implementar `SelectableControl`.
Esto es debido a que los descendientes de `Control` serán los únicos que posean el elemento privado  `state`.

Dentro de la clase `Control` es posible acceder al elemento privado `state` mediante una instancia de `SelectableControl`.
`SelectableControl` actúa como `Control` que es conocido que tiene el método `select`.

Las clases `Button` y `TextBox` son subtipos de `SelectableControl` (porque ambos heredan de `Control` y tienen un método `select`), pero las clases `Image` y `Location`no.
