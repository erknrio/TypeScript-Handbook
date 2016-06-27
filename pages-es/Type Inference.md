# Introducción

En esta sección, cubriremos la inferencia de tipos en TypeScript. Discutiremos dónde y cómo se infieren los tipos.

# Básicos

En TypeScript, hay varios lugares donde se usa la inferencia de tipos para proporcionar información del tipo cuando no hay una anotación explícita. Por ejemplo, este código:

```ts
let x = 3;
```

El tipo de la variable `x` se infiere a `number`.
Este tipo de inferencia se da cuando se inicializan variables y miembros, se configuran los valores de por defecto de los parámetros y determinando el valor de retorno de una función.

En la mayoría de los casos, la inferencia de tipos es directa.
En las siguiente secciones, exploraremos los matices de como los tipos son inferidos.

# Mejor tipo común

Cuando la inferencia de tipos se hace desde múltiples expresiones, los tipos de esas expresiones son usados para calcular el "mejor tipo común". Por ejemplo:

```ts
let x = [0, 1, null];
```

Para inferir el tipo de `x`, debemos considerar el tipo de cada elemento.
Estamos dando dos opciones para el tipo del array: `number` y `null`.
El algoritmo del mejor tipo común considera cada posible tipo y escoge el que es compatible con todos ellos.
En el ejemplo anterior sería `number[]`.

Debido a que el mejor tipo común tiene que elegir de entre los tipos proporcionados, hay algunos casos donde los tipos comparten una estructura común pero no un super tipo para todos los candidatos. Por ejemplo:

```ts
let zoo = [new Rino(), new Elefante(), new Serpiente()];
```

Idealmente, podemos querer que `zoo` sea inferido como un `Animal[]`, pero como no hay objetos del tipo `Animal` en el array, no hacemos inferencia sobre el tipo de elementos del array.
Para corregirlo, en su lugar, provee el tipo expecífico cuando ninguno de los tipos es un super tipo de los otros candidatos:

```ts
let zoo: Animal[] = [new Rino(), new Elefante(), new Serpiente()];
```

Cuando no se encuentra un mejor tipo común, el resultado de la inferencia es el tipo objeto vacío, `{}`.
Como este tipo no tiene miembros, al tratat de utilizar cualquier propiedad de estas causaría un error.
Este resultado le permite seguir usando el objeto de una manera independiente al tipo, mientras que provee seguridad en los tipos en los casos donde el tipo del objeto no pueda ser determinado implícitamente.

# Tipo Contextual

La inferencia de tipos en algunos casos en TypeScript también funciona en "la otra dirección".
Esto es conocido como "tipificación contextual".
Esto ocurre cuando el tipo está implícito a su localización. Por ejemplo:

```ts
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.buton);  //<- Error
};
```

Para que el código anterior de el error de tipo, el comprobador de tipos de TypeScript usó el tipo de la función `Window.onmousedown` para inferir el tipo de la función del lado derecho de la asignación.
Cuando hace esto, es capaz de inferir el tipo del parámetro `mouseEvent`.
En la expresión de esta función no estamos en en una posición contextualmente tipada, el parámetro `mouseEvent` debería tener el tipo `any` y ningún error deberá ser emitido.

Si el tipado contextual contiene explícitamente información sobre el tipo, el tipo contextual es ignorado.
Corrigamos el ejemplo anterior:

```ts
window.onmousedown = function(mouseEvent: any) {
    console.log(mouseEvent.buton);  //<- Ahora, no recibimos ningun error
};
```

La expresión de la función con una anotación explícita sobre el tipo en el parámetro sobreescribe el tipo contextual.

La tipificación contextual se aplica en muchos casos.
Los casos comunes incluyen argumentos en las llamadas de una función, el lado derecho de asignaciones,type assertions, los miembros de objetos y arrays literales y declaraciones de retorno.
El tipo contextual también actúa como un candidato al mejor tipo común. Por ejemplo:

```ts
function crearZoo(): Animal[] {
    return [new Rino(), new Elefante(), new Serpiente()];
}
```

En este ejemplo, el mejor tipo común tiene un conjunto de cuatro candidatos: `Animal`, `Rino`, `Elefante` y `Serpiente`.
De estos, `Animal` puede ser elegido por el algoritmo del mejor tipo común.
