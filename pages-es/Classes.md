# Introducción

El JavaScript tradicional emplea funciones y propotipos (prototype) para construir componentes reusable, pero difiere de la implemtación que vemos en otros lenguajes como C# o Java.
A partir de ECMAScript 2015, también conocido como ECMAScript 6 (ES6), existe la opción de crear aplicaciones orientadas a objetos (Programación Orientada a Objetos o por sus siglas `POO`) con un estilo basado en clases.
En TypeScript se permiten estas técinas compilando en un JavaScript soportado por la mayoría de navegadores y plataformas sin que tengas que esperar a nuevas versiones de JavaScript.

# Classes

Veamos un ejemplo básico basado en clases:

```ts
class Bienvenida {
    saludo: string;
    constructor(message: string) {
        this.saludo = message;
    }
    saludar() {
        return "Hello, " + this.saludo;
    }
}

let bienvenida = new Bienvenida("world");
```

Declaramos una clase `Bienvenida`.
Esta clase tiene tres miembros: una propiedad llamada `saludo`, un constructor y un método `saludar`.

Dentro de la clase se habrá dado cuenta de que cuando nos referimos a un miembro de esta usamos `this`.
Implica un acceso a un miembro.

En la última línea creamos una instancia de la clase `Bienvenida` usando `new`.
Esto llama al constructor de la clase `Bienvenida` y la inicializa.

# Herencia

Una de los fundamentos de la POO basada en clases es la posiblidad de extender las clases existentes para crear nuevas usando herencia.
Veamos un ejemplo:

```ts
class Animal {
    nombre: string;
    constructor(elNombre: string) { this.nombre = elNombre; }
    mover(distanciaEnMetros: number = 0) {
        console.log(`${this.nombre} movió ${distanciaEnMetros}m.`);
    }
}

class Serpiente extends Animal {
    constructor(nombre: string) { super(nombre); }
    mover(distanciaEnMetros = 5) {
        console.log("Deslizándose...");
        super.mover(distanciaEnMetros);
    }
}

class Caballo extends Animal {
    constructor(nombre: string) { super(nombre); }
    mover(distanciaEnMetros = 45) {
        console.log("Galopando...");
        super.mover(distanciaEnMetros);
    }
}

let sam = new Serpiente("Sammy la Pitón");
let tom: Animal = new Caballo("Tommy el Palomino");

sam.mover();
tom.mover(34);
```

Este ejemplo cubre bastantes de las características de la herencia en TypeScript que son comunes a otros lenguajes.
Vemos la palabra clave `extends` usada para crear la subclase.
De esta forma las subclases `Caballo` y `Serpiente` obtienen acceso a las características de la clase `Animal`.

Las clases derivadas deben contener un constructor que llame a `super()` el que ejecutará el constructor de la clase padre.
Derived classes that contain constructor functions must call  which will execute the constructor function on the base class.

El ejemplo también muestra como sobreescribir métodos de la clase padre con otros más especializados de la subclase.
Tanto `Serpiente` como `Caballo` sobreescriben el método `mover` de `Animal` dándole una funcionalidad específica para cada clase.
Tenga en cuenta que a pesar de que `tom` es declarado como tipo `Animal` , su valor es un `Caballo`.
Cuando `tom.mover(34)` llama al método sobreescrito en `Caballo`.

El resultado:

```Text
Deslizándose...
Sammy la Pitón movió 5m.
Galopando...
Tommy el Palomino movió 34m.
```

# Modificadores público, protegido y privado (*public*, *protected*, *private*)

## Public por defecto

En TypeScript los miembros son públicos por defecto, en otros lenguajes como C# hay que marcarlos como `public` para que sean visibles.
Podríamos haber escrito la clase `Animal` de la sección anterior de la siguiente manera:

```ts
class Animal {
    public nombre: string;
    public constructor(elNombre: string) { this.nombre = elNombre; }
    public mover(distanciaEnMetros: number) {
        console.log(`${this.nombre} movió ${distanciaEnMetros}m.`);
    }
}
```

## Entendiendo `private`

Cuando un miembro es marcado como `private`, no puede ser accedido desde fuera de la clase que lo contiene.
Por ejemplo:

```ts
class Animal {
    private nombre: string;
    constructor(elNombre: string) { this.nombre = elNombre; }
}

new Animal("Gato").nombre; // Error: 'nombre' is private;
```

TypeScript es un sistema de tipo estructural.
Cuando comparamos dos tipos diferentes, independientemente de donde vengan, si los tipos de todos los miembros de son compatibles, entonces podemos decir que son compatibles entre sí.

Sin embargo, cuando comparamos miembros de tipo `private` y `protected`, los tratamos de forma diferente.
Para que consideremos a los los tipos compatibles, ambos deben tener un miembro `private` originado en la misma declaración.
Lo mismo pasa con los mirmbros `protected`.

Veamos un ejemplo para aclarar estos conceptos:

```ts
class Animal {
    private nombre: string;
    constructor(elNombre: string) { this.nombre = elNombre; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Empleado {
    private nombre: string;
    constructor(elNombre: string) { this.nombre = elNombre; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let empleado = new Empleado("Bob");

animal = rhino;
animal = empleado; // Error: 'Animal' and 'Empleado' are not compatible
```

En este ejemplo tenemos un `Rhino` que es una subclase de `Animal`.
También tenemos una nueva clase `Empleado` que tiene la misma forma que `Animal`.
Creamos varias instancias de estas clases y las asignamos unas a otras.
Como `Animal` y `Rhino` comparten su forma `private` en la misma declaración (`private nombre: string` en `Animal`), son compatibles.
Sin embargo, cuando tratamos de asignar `Empleado` a `Animal` obtemos un error de incompatibilidad debido a que `Empleado` también tiene un miembro `private` llamado `nombre`, pero este no es el mismo declarado en `Animal`. Forma parte de una clase diferente.

## Entendiendo `protected`

Los `protected` actúan de manera similar a `private` con la diferencia de que a los miembros declarados con `protected` se puede acceder mediante instancias de las clases derivadas. Por ejemeplo:

```ts
class Persona {
    protected nombre: string;
    constructor(nombre: string) { this.nombre = nombre; }
}

class Empleado extends Persona {
    private departamento: string;

    constructor(nombre: string, departamento: string) {
        super(nombre);
        this.departamento = departamento;
    }

    // Elevator pitch es un discurso rapido enfocado a
    // vender una idea, producto o a ti mismo en poco tiempo.
    // Literalmente: Dicurso o charla de ascensor.
    public getElevatorPitch() {
        return `Hola, mi nombre es ${this.nombre} y trabajo en ${this.departamento}.`;
    }
}

let howard = new Empleado("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.nombre); // error
```

Nótese que aunque no podemos usar `nombre` desde fuera de `Persona`, podemos seguir usándolo desde un método de instancia de `Empleado` porque `Empleado` deriva de `Persona`.

## Propiedades de parámetros

En el último ejemplo, declaramos un miembro privado `nombre` y un parámetro en el constructor `elNombre`.
Entonces inmediatamente establecimos el `nombre` al valor recibido de `elNombre`.
Esta es una práctica común.
*Propiedades de parámetros* te permite crear e inicializar un miembro en un sólo lugar.
Esta es una revisicion complementaria a la clase anterior `Animal` usando una propiedad de parámetro:

```ts
class Animal {
    constructor(private nombre: string) { }
    mover(distanciaEnMetros: number) {
        console.log(`${this.nombre} movió ${distanciaEnMetros}m.`);
    }
}
```

Fíjese que hemos desechado `elNombre` y usamos el parámetro acortado `private nombre: string` en el constructor que crea e inicializa el miembro `nombre`.
Hemos fusionado la declaración y la asignación en un mismo lugar.

# Descriptores de acceso

TypeScript soporta getters/setters (obtenedores y asignadores *aunque usaremos sus nombres en inglés*) como una manera para interceptar el acceso a un miembro de un objeto.
Esto le da un control más preciso sobre cómo se accede a un miembro de cada objeto.

Convirtamos una clase simple para que use `get` y `set`.
Primero un ejemplo sin getters ni setters.

```ts
class Empleado {
    nombreCompleto: string;
}

let empleado = new Empleado();
empleado.nombreCompleto = "Bob Smith";
if (empleado.nombreCompleto) {
    console.log(empleado.nombreCompleto);
}
```

Aunque permitir a la gente cambiar a su antojo `nombreCompleto` es muy práctico, esto nos puede meter en problemas.

En esta versión, comprobamos que el usuario tiene un código de acceso secreto antes de modificar `empleado`.
Conseguimos esto reemplazando el acceso directo a `nombreCompleto` con un `set` que comprueba el código de acceso.
Agregamos el correspondiente `get` para permitir que funcione de forma similar al ejemplo anterior.

```ts
let codigoAcceso = "codigo de acceso secreto";

class Empleado {
    private _nombreCompleto: string;

    get nombreCompleto(): string {
        return this._nombreCompleto;
    }

    set nombreCompleto(nuevoNombre: string) {
        if (codigoAcceso && codigoAcceso == "codigo de acceso secreto") {
            this._nombreCompleto = nuevoNombre;
        }
        else {
            console.log("Error: Actualizacion no autorizada de empleado!");
        }
    }
}

let empleado = new Empleado();
empleado.nombreCompleto = "Bob Smith";
if (empleado.nombreCompleto) {
    console.log(empleado.nombreCompleto);
}
```

Para comprobar que nuestros descriptores de acceso funcionan, solamente tenemos que modificar `codigoAcceso` y recibiremos el mensaje de error por consola.

**Importante**: Los descriptores de acceso requieren que el compilador devuelva ECMAScript 5 o mayor.

# Propiedades estáticas (*static*)

Hasta este punto solo hemos hablado de miembros de *instancias* de la clase, esos que se muestran al instanciar un objeto.
También podemos crear miembros estáticos de la clase, aquellos que son visibles en la propia clase, sin instancias.

En este ejemplo usamos `static` en *origen* porque es un valor general para tadas las cuadrículas.
Cada instancia accede a este valor anteponiendo el nombre de la clase.
Es similar a anteponer `this.` en frente del acceso a la instancia.
En este ejemplo anteponemos `Cuadricula.` en frente del acceso estático.

```ts
class Cuadricula {
    static origen = {x: 0, y: 0};
    calcularDistanciaDesdeOrigen(point: {x: number; y: number;}) {
        let xDist = (point.x - Cuadricula.origen.x);
        let yDist = (point.y - Cuadricula.origen.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Cuadricula(1.0);  // 1x scale
let grid2 = new Cuadricula(5.0);  // 5x scale

console.log(grid1.calcularDistanciaDesdeOrigen({x: 10, y: 10}));
console.log(grid2.calcularDistanciaDesdeOrigen({x: 10, y: 10}));
```

En el ejemplo también usamos un método estático de clase `Math` llamado `sqrt` para calcular la raíz cuadrada de un número.

# Clases abstractas

Las clases abstractas sirven de base para otras clases derivadas, pero no se pueden instanciar directamente.
Como diferencia con las interfaces las clases abstractas pueden contener detalles sobre la implementación de sus miembros.
La palabra clave `abstract` es usada para definir tanto la clase abstracta como sus métodos abstractos.
Veamos un ejemplo sencillo:

```ts
abstract class Animal {
    abstract makeSound(): void;
    mover(): void {
        console.log("roaming the earth...");
    }
}
```

Los métodos dentor de una clase abstracta marcados como abstractos no contienen una implementación y deben ser implementados en la clase derivada.
Los métodos abstractos muestran una sintaxis similar a los métodos de interfaz.
Ambos definen la firma de un método sin incluir el cuerpo del método.
Sin embargo, los métodos abstractos deben incluir la palabra clave `abstract` y, optionalmente, modificadores de acceso.

```ts
abstract class Department {

    constructor(public nombre: string) {
    }

    printName(): void {
        console.log("Department nombre: " + this.nombre);
    }

    abstract printMeeting(): void; // must be implemented in derived classes
}

class AccountingDepartment extends Department {

    constructor() {
        super("Accounting and Auditing"); // constructors in derived classes must call super()
    }

    printMeeting(): void {
        console.log("The Accounting Department meets each Monday at 10am.");
    }

    generateReports(): void {
        console.log("Generating accounting reports...");
    }
}

let departamento: Department; // ok to create a reference to an abstract type
departamento = new Department(); // error: cannot create an instance of an abstract class
departamento = new AccountingDepartment(); // ok to create and assign a non-abstract subclass
departamento.printName();
departamento.printMeeting();
departamento.generateReports(); // error: method doesn't exist on declared abstract type
```

# Técnicas avanzadas

## Funciones constructoras

Cuando declaras una clase en TypeScript, estás creando múltiples declaraciones al mismo tiempoi.
La primera es el tipo de *instancia* de la clase.

```ts
class Bienvenida {
    saludo: string;
    constructor(message: string) {
        this.saludo = message;
    }
    saludar() {
        return "Hello, " + this.saludo;
    }
}

let bienvenida: Bienvenida;
bienvenida = new Bienvenida("world");
console.log(bienvenida.saludar());
```

Cuando decimos `let bienvenida: Bienvenida`, estamos usando `Bienvenida` como el tipo de instancia de la clase `Bienvenida`.

También estamos creando otro valor que llamamos *funciones constructoras*.
Esta es la función a la que llamamos cuando hacemos un `new` para crear una nueva instancia de una clase.
Para verlo en la práctica, echemos un vistazo al JavaScript creado en el ejemplo siguiente:

```ts
let Bienvenida = (function () {
    function Bienvenida(message) {
        this.saludo = message;
    }
    Bienvenida.prototype.saludar = function () {
        return "Hello, " + this.saludo;
    };
    return Bienvenida;
})();

let bienvenida;
bienvenida = new Bienvenida("world");
console.log(bienvenida.saludar());
```

Aquí, `let Bienvenida` va a ser asignada a la función constructora.
Cuando llamamos a `new` y ejecutamos esta función, obtenemos una instancia de la clase.
La función constructora también contiene todos los miembros estáticos de la clase.
Otra forma de pensar para cada clase es que existe un lado *instance*  y otro lado *static*.

Modifiquemos un poco el ejemplo para mostrar estas diferencias:

```ts
class Bienvenida {
    static standardGreeting = "Hello, there";
    saludo: string;
    saludar() {
        if (this.saludo) {
            return "Hello, " + this.saludo;
        }
        else {
            return Bienvenida.standardGreeting;
        }
    }
}

let greeter1: Bienvenida;
greeter1 = new Bienvenida();
console.log(greeter1.saludar());

let greeterMaker: typeof Bienvenida = Bienvenida;
greeterMaker.standardGreeting = "Hey there!";

let greeter2: Bienvenida = new greeterMaker();
console.log(greeter2.saludar());
```

En este ejemplo, `greeter1` funciona de manera similar al ejemplo anterior.
Instanciamos la clase `Bienvenida` y usamos este objeto.
Esto lo hemos visto antes.

A continuación, usamos la clase directamente.
Creamos una nueva variable llamada `greeterMaker`.
Esta variable mantendrá la propia clase en si, o dicho de otra forma es la función constructora.

Usamos `typeof Bienvenida`, que quiere decir "dame el tipo de la clase `Bienvenida` en sí mismo" en lugar el tipo instancia.
O, más precisamente, "dame el tipo del símbolo llamado `Bienvenida`", el que es el tipo de la función constructora.
Este tipo contendrá todos los miembros estáticos de la clase Bienvenida junto con el constructor que crea la isntancia de la clase `Bienvenida`.
Mostramos esto usando `new` en `greeterMaker`, creando una nueva instancia de `Bienvenida` e invocándola antes.

## Usando una clase como una interfaz

Una declaración de clase crea dos cosas: un tipo que representa la instancia de la clase y una función constructora.
Dado que las clases crean tipos, pueda usarlos de la misma manera que usas las interfaces.

```ts
class Punto {
    x: number;
    y: number;
}

interface Punto3d extends Punto {
    z: number;
}

let point3d: Punto3d = {x: 1, y: 2, z: 3};
```
