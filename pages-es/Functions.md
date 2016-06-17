# Introducción

En TypeScript, mientras que hay clases, espacios de nombre y módulos, las funciones siguen jugando un rol fundamental describiendo el como *hacer* de las cosas.
TypeScript también añade algunas nuevas capacidades a las funciones de JavaScript para hacer más fácil el trabajar con ellas.

# Funciones

Para empezar, como en JavaScript, las funciones de TypeScript se pueden crear tanto como una función con nombre como anónima.

Repasemos como hacer esto JavaScript:

```ts
// Funcion con nombre
function agregar(x, y) {
    return x + y;
}

// Funcion anonima
let miAgregar = function(x, y) { return x+y; };
```

Como en JavaScript, las funciones pueden referir a variables que estan fuera de ellas.
Cuando esto sucede decimos que `capturamos` estas variables.
La explicación sobre su funcionamiento y el ámbito de actuación (scope) están fuera de este artículo, pero son clave para trabajar con JavaScript and TypeScript.

```ts
let z = 100;

function addToZ(x, y) {
    return x + y + z;
}
```

# Tipos de funciones

## Al escribir la función

Vamos a agregarle tipos a nuestro sencillo ejemplo anterior:

```ts
function agregar(x: number, y: number): number {
    return x + y;
}

let miAgregar = function(x: number, y: number): number { return x+y; };
```

Podemos agregar tipos a cada parámetro, y a la propia función para establecer un tipo de valor de retorno.
TypeScript puede suponer el tipo de valor de retorno buscando en las declaraciones devueltas por lo que podemos dejar esto como algo opcional.

## Escribiendo el tipo de función

Ahora que hemos escrito la función vamos a escribir el tipo completo fijándonos en cada parte del tipo de función.

El tipo de una función tiene las mismas dos partes: el tipo de los argumentos y el de retorno.
Ambas partes son requeridas.
Escribimos el tipo de los parámetros como una lista de estos dando a cada parámetro un nombre y tipo.
Este nombre es por legibilidad, podemos escribir algo como lo siguiente:

```ts
let miAgregar: (baseValue:number, increment:number) => number =
    function(x: number, y: number): number { return x + y; };
```

Desde que los tipos coinciden se considera válido independientemente de si los nombres de los parámetros coinciden.

La segunda parte el tipo de retorno, se especifica con una flecha gruesa (`=>`) entre los parámetros y el tipo de retorno.
Al ser requerido, si la función no devuelve un valor entonce se debe especificar el tipo de retorno como `void`.

Es de destacar que solamente los parámetros y el tipo de retorno constituyen el tipo de función.
Las variables capturadas no se reflejan en el tipo.
De hecho, las variables capturadas son parte del "estado oculto" de cualquier función y no constituyen su API.

## Inferir en los tipos

El compilador de TypeScript puede compiler puede averiguar el tipo de un lado de la ecuación si los tienes en el otro:

```ts
// miAgregar tiene el tipo de funcion completo
let miAgregar = function(x: number, y: number): number { return  x + y; };

// Los parámetros 'x' e 'y' tienen el tipo number
let miAgregar: (baseValue:number, increment:number) => number =
    function(x, y) { return x + y; };
```

Esto es llamado "tipificación contextual", una forma de inferencia de tipos.
Esto ayuda a reducir el esfuerzo de mantener su programa tipado.

# Parámetros por Defecto y Opcionales

En TypeScript, cada parámetro se asume como requerido por la función, independientemente de si su valor puede ser `null` o `undefined`.
O dicho de otra forma, el número de parámetros recibidos por la función debe ser igual al de los esperados por esta.

```ts
function construirNombre(nombre: string, apellido: string) {
    return nombre + " " + apellido;
}

let result1 = construirNombre("Bob");                  // error, pocos parametros
let result2 = construirNombre("Bob", "Adams", "Sr.");  // error, demasiados parámetros
let result3 = construirNombre("Bob", "Adams");         // ah, correcto
```

En JavaScript cada parámetro es opcional y los usuarios pueden obviarlo si lo consideran conveniente.
Cuando lo hacen su valor es `undefined`.
Podemos obtener esta funcionalidad en TypeScript agregando un `?` al final del parámetro opcional.
Por ejemplo, imaginemos que el parámetro apellido es opcional:

```ts
function construirNombre(nombre: string, apellido?: string) {
    if (apellido)
        return nombre + " " + apellido;
    else
        return nombre;
}

let result1 = construirNombre("Bob");                  // ahora funciona correctamente
let result2 = construirNombre("Bob", "Adams", "Sr.");  // error, demasiado parametros
let result3 = construirNombre("Bob", "Adams");         // ah, correcto
```

Cualquier parámetro opcional debe estar a continuación de los requeridos.
Los parámetros requeridos siempre van antes que los opcionales.
Si quisiéramos que `nombre` fuese el parámetro opcional tendríamos que cambiar el oden.

En TypeScript, podemos establecer un valor por defecto en el caso de que el usuario no pase ninguno o este sea `undefined`.
Tenga cuidado porque solamente funciona con `undefined`, si recibe `null` asumirá que ese es el apellido.

Usemos el ejemplo previo y establezcamos el valor por defecto de apellido a `"Smith"`.

```ts
function construirNombre(nombre: string, apellido = "Smith") {
    return nombre + " " + apellido;
}

let result1 = construirNombre("Bob");                  // funciona correctamente, retorna "Bob Smith"
let result2 = construirNombre("Bob", undefined);       // todavía funciona, también retorna "Bob Smith"
let result3 = construirNombre("Bob", "Adams", "Sr.");  // error, demasiado parametros
let result4 = construirNombre("Bob", "Adams");         // ah, correcto
```

Los parámetros inicializados que vienen después de todos los parámetros requeridos son tratados como si fuesen opcionales, pueden ser omitidos al llamar a la función:

```ts
function construirNombre(nombre: string, apellido?: string) {
    // ...
}
```

y

```ts
function construirNombre(nombre: string, apellido = "Smith") {
    // ...
}
```

Al contrario que los parámetros opciones, los parámetros incializados por defecto no *neceistan* situarse después de los requerido.
La diferencia es que al llamar a la función tendremos que pasarle `undefined` para que use el valor por defecto.
Por ejemplo, podemos escribir nuestor último ejemplo incializando solamente `nombre`:

```ts
function construirNombre(nombre = "Will", apellido: string) {
    return nombre + " " + apellido;
}

let result1 = construirNombre("Bob");                  // error, pocos parametros
let result2 = construirNombre("Bob", "Adams", "Sr.");  // error, demasiado parametros
let result3 = construirNombre("Bob", "Adams");         // okay y retorna "Bob Adams"
let result4 = construirNombre(undefined, "Adams");     // okay y retorna "Will Adams"
```

# Resto de Parámetros

Los parámetros requerido, opcional y por defecto tienen algo en común: todos hablan de un sólo parámetro al mismo tiempo.
Algunas veces quieres trabajar con múltiples parámetros agrupados o no sabes cuando parámetros recibirá la función.
En JavaScript puedes trabajar con múltiples parámetros usando la variable `arguments` la cual es visible dentro de la función.

En TypeScript puedes reunir todos estos parámetros en una variable:

```ts
function construirNombre(nombre: string, ...restoDeNombre: string[]) {
    return nombre + " " + restoDeNombre.join(" ");
}

let nombreEmpleado = construirNombre("Joseph", "Samuel", "Lucas", "MacKinzie");
```

El *resto de parámetros* son tratados como un número infinito de estos.
El compilador construirá un array con los argumentos recibidos dándole el nombre que hay después de `...`, permitiéndote utilizarlo como una variable de tu función.
Es usado también en el tipo de la función:

```ts
function construirNombre(nombre: string, ...restoDeNombre: string[]) {
    return nombre + " " + restoDeNombre.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = construirNombre;
```

# Lambdas y usando `this`

Como funciona `this` en las funciones de JavaScript es un tema principal para los programadores JavaScript.
Como TypeScript es un superconjunto de JavaScript, los desarrolladores de TypeScript también necesitan aprender como usar `this` correctamente.
Es un tema extenso pero aquí cubriremos algunos apartados básicos.

En JavaScript, `this` es una variable que se asigna cuando la función es llamada.
Es una característica muy poderosa y flexible pero al coste de que debemos saber en todo momento el contexto desde el que se ejecuta la función.
Esto puede ser muy confuso cuando, por ejemplo, la función es usada como callback.

Veamos un ejemplo:

```ts
let baraja = {
    palos: ["corazones", "espadas", "treboles", "diamantes"],
    cartas: Array(52),
    crearSelectorCarta: function() {
        return function() {
            let cartaElegida = Math.floor(Math.random() * 52);
            let paloElegido = Math.floor(cartaElegida / 13);

            return {palo: this.palos[paloElegido], carta: cartaElegida % 13};
        }
    }
}

let selectorCarta = baraja.crearSelectorCarta();
let cartaElegida = selectorCarta();

alert("carta: " + cartaElegida.carta + " of " + cartaElegida.palo);
```

Si tratamos de ejecutar el ejemplo devolverá un error.
Esto sucede porque `this` se utiliza en la función creada por `crearSelectorCarta`.
Será asignado a `window` en lugar de `baraja`.
Es el resultado de llamar a `selectorCarta()`. Aquí no hay unión dinaḿica para el `this` diferente a Window.
*Tenga en cuenta* que: baja el strict mode, será undefined en lugar de Window.

Podemos solucionarlo asegurándonos que la función apunta al `this` correcto antes de devolverla.
De esta manera seguirá veidno el objeto `baraja` original.

Para solucionarlo cambiamos la expresión JavaScript function para usar la sintaxis de flecha (`() => {}`).
Esto capturará atuomáticamente el `this` disponbile cuando la función es creada en lugar de invocada.

```ts
let baraja = {
    palos: ["corazones", "espadas", "treboles", "diamantes"],
    cartas: Array(52),
    crearSelectorCarta: function() {
        // Notice: la línea siguiente es ahora una lambda, permitiéndonos capturar el 'this' antes
        return () => {
            let cartaElegida = Math.floor(Math.random() * 52);
            let paloElegido = Math.floor(cartaElegida / 13);

            return {palo: this.palos[paloElegido], carta: cartaElegida % 13};
        }
    }
}

let selectorCarta = baraja.crearSelectorCarta();
let cartaElegida = selectorCarta();

alert("carta: " + cartaElegida.carta + " of " + cartaElegida.palo);
```

Para más información sobre el uso de `this` puede leer de Yehuda Katz's [Understanding JavaScript Function Invocation and "this"](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/).

# Sobrecargas

JavaScript es inherentemente un lenguaje muy dinámico.
No es poco común en una sóla función de JavaScript devolver varios tipos de objetos basados en la forma de los argumentos que se le pasan.

```ts
let palos = ["corazones", "espadas", "treboles", "diamantes"];

function escogerCarta(x): any {
    // Comprobamos si estamos trabajando con un objeto/array
    // de ser correcto, ellos nos dieron la baraja y nosotros elegiremos la carta
    if (typeof x == "object") {
        let cartaElegida = Math.floor(Math.random() * x.length);
        return cartaElegida;
    }
    // De lo contrario les dejamos escoger una carta
    else if (typeof x == "number") {
        let paloElegido = Math.floor(x / 13);
        return { palo: palos[paloElegido], carta: x % 13 };
    }
}

let myDeck = [{ palo: "diamantes", carta: 2 }, { palo: "espadas", carta: 10 }, { palo: "corazones", carta: 4 }];
let cartaEscogida1 = myDeck[escogerCarta(myDeck)];
alert("carta: " + cartaEscogida1.carta + " of " + cartaEscogida1.palo);

let cartaEscogida2 = escogerCarta(15);
alert("carta: " + cartaEscogida2.carta + " of " + cartaEscogida2.palo);
```

La función `escogerCarta` devolverá dos cosas diferentes basadas en lo que el usuario a pasado.
Si el usuario pasa un objeto que reppresenta la baraja, la función elige una carta.
Si el usuario elige una carta, nostros le decimos que carta ha elegido.
¿Pero cómo describimos este tipo de sistema?

La respuesta es suministrar múltiples tipos de función a la misma función como una lista de Sobrecargas.
Esta lista es lo que usará el compilador para resolver las llamadas a la función.
Vamos a crear una lista de Sobrecargas que describan lo que nuestra `escogerCarta` acepta y devuelve.

```ts
let palos = ["corazones", "espadas", "treboles", "diamantes"];

function escogerCarta(x: {palo: string; carta: number; }[]): number;
function escogerCarta(x: number): {palo: string; carta: number; };
function escogerCarta(x): any {
    // Comprobamos si estamos trabajando con un objeto/array
    // de ser correcto, ellos nos dieron la baraja y nosotros elegiremos la carta
    if (typeof x == "object") {
        let cartaElegida = Math.floor(Math.random() * x.length);
        return cartaElegida;
    }
    // De lo contrario les dejamos escoger una carta
    else if (typeof x == "number") {
        let paloElegido = Math.floor(x / 13);
        return { palo: palos[paloElegido], carta: x % 13 };
    }
}

let myDeck = [{ palo: "diamantes", carta: 2 }, { palo: "espadas", carta: 10 }, { palo: "corazones", carta: 4 }];
let cartaEscogida1 = myDeck[escogerCarta(myDeck)];
alert("carta: " + cartaEscogida1.carta + " of " + cartaEscogida1.palo);

let cartaEscogida2 = escogerCarta(15);
alert("carta: " + cartaEscogida2.carta + " of " + cartaEscogida2.palo);
```

Con este cambio, las sobrecargas nos dan la comprobación de tipo para las llamadas a la función `escogerCarta`.

El compilador busca en las sobrecargas y usa la primera coincidencia como la correcta.
Por este motivo es importante establecer el orden de las sobrecargas de la más específica a la que menos order

Fíjese que el código `function escogerCarta(x): any` no es parte de la lista de sobrecarga, solamente tiene dos sobrecargas: una que obtiene un objecto y otra que obtiene un número.
Llamando a `escogerCarta` con cualquier otro tipo de parámetro with provocará un error.
