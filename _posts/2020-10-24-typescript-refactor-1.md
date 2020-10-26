---
layout: post
title: Refactoring a TypeScript function, part 1.
categories: [TypeScript]
---
<!-- 
Let's say I'm working on an Angular project that is integrating a web component. This component is a navigation bar that extend its behaviour by adding custom options in a dropdown. Usually these options are links to navigate to external pages. However these options can also fire events that can be handled on the Angular application and run any code we want.

This is how one of these events would look like in the console: -->

En un proyecto tuve que utilizar un componente web que tenía la capacidad de comunicarse con el resto de la aplicación mediante eventos. Este componente era cerrado y no podía modificarlo, pero era configurable mediante parámetros de entrada que permitían modificar el detalle de los eventos que emitían.

```
CustomEvent: {
  ...
  type: 'optionClick',
  detail: {
    code: 'showDialog',
    selected: true
  }
}
```

Dependiendo del código del detalle tendremos que ejecutar una determinada función en la aplicación Angular, de modo que en el handler del evento añadiremos un bloque if para asegurarnos de que ejecutamos la función que queremos para el código showDialog. En este caso, mostraremos un diálogo.

<!-- ```html
<web-component-navbar
  [languageOptions]="languageOptions"
  [mainOptions]="mainOptions"
  [customOptions]="customOptions">
<web-component-navbar>
``` -->
<!-- 
So firstly I register the event in my Angular application. As I already know the properties of the event detail, it would be nice to create a type with TypeScript.

In the event callback we add an expression to filter the code.  -->

```typescript
document.addEventListener("optionClick", event => {
  if (event.detail.code === "showDialog") {
    this.showDialog();
  }
});
```

No obstante, es posible que a lo largo del ciclo de vida de esta aplicación tengamos que extender la funcionalidad del web component con nuevos códigos de evento. A partir del código existente podríamos extender el bloque if añadiendo una sentencia else por cada nuevo código que haya que gestionar.

```typescript
document.addEventListener("optionClick", event => {
  if (event.detail.code === "showDialog") {
    this.showDialog();
  } else if (event.detail.code === "anotherEvent") {
    this.anotherFunction();
  } else if (event.detail.code === "anotherEventMore") {
    this.evenAnotherFunction();
  } // ... more to come?
});
```

El problema aquí es que la cantidad la cantidad de funciones y eventos nuevos no está definida. Podría quedarse en tres, o podrían ser veinte, por lo que veinte bloques if anidados podría no ser la solución más elegante. Una alternativa a esto podría ser la sentencia switch, pero se repetiría el problema de que es un bloque de código muy largo. ¿Cómo podríamos simplificar la gestión de pares código-función?

```typescript
document.addEventListener("optionClick", event => {
  switch (event.detail.helpSelected) {
    case "showDialog":
      this.showReleaseDialog();
      break;
    case 'anotherEvent':
      this.anotherFunction();
      break;
    case 'evenAnotherEvent':
      this.evenAnotherFunction();
      break;
    // ... more to come?
  }
});
```

### Factoría de métodos como solución
<!-- TODO Explicar cómo es el patrón factoría -->
Mediante la variable eventFactory podemos indexar los códigos del detalle de los eventos emitidos y asociarlos a los métodos que queremos ejecutar.

```typescript
const eventFactory = {
  showDialog: this.showDialog,
  anotherEvent: this.anotherFunction,
  evenAnotherEvent: this.evenAnotherFunction,
};
```

Podemos añadir un tipo a esta factoría mediante los tipos indexados de TypeScript. De esta forma definimos que los índices, que en este caso son los códigos de los eventos, deben ser de tipo string, mientras que la respuesta es una función sin signature ni respuesta.

```typescript
export type EventHandlerFactory = {
  [K in string]: () => void;
};

const eventFactory: EventHandlerFactory = {
  showDialog: this.showDialog,
  anotherEvent: this.anotherFunction,
  evenAnotherEvent: this.evenAnotherFunction,
};
```

De esta forma podemos acceder a un método a partir de su correspondiente código y ejecutarlo sin necesidad de usar sentencias de control.

```typescript
document.addEventListener("optionClick", (event: CustomEvent<OptionClick>) => {
  eventFactory[event.detail.code](); // Recuerda añadir ()
});
```

### Añadiendo restricciones a las propiedades con TypeScript

Es posible que mientras declaremos la variable eventFactory cometamos un error tipográfico en la escritura de una de las propiedades.

```typescript
const eventFactory: EventHandlerFactory = {
  showDialgo: this.showDialog, // should be 'showDialog' instead of 'showDialgo'
}
```

Cuando el handler reciba el evento con el código showDialog intentará ejecutarlo a través de eventFactoy y no encontrará ninguna función asociada a ese nombre, por lo que devolverá el error `Uncaught TypeError: eventFactory.showDialog is not a function`.

¿Cómo podemos evitar esto? Una forma de darnos cuenta de que cometemos un error tipográfico es que nuestro editor de código sea capaz de avisarnos. Para ello modificaremos la definición del tipo EventHandlerFactory haciendo uso de las características de TypeScript.

Definiremos el tipo AllowedCode como un conjunto de literales correspondientes con los códigos de evento que esperamos gestionar.

```typescript
type AllowedCode = "showDialog" | "anotherEvent" | "evenAnotherEvent";
```

Y después modificamos la definición de EventHandlerFactory tal como vemos en el extracto de código de debajo. Anteriormente la definición `[K in string]` establecía que las propiedades debían ser string, pero ahora con la definición `[K in AllowedCode]` las propiedades deben formar parte del conjunto AllowedCode.

```typescript
export type EventHandlerFactory = {
  [K in AllowedCode]: () => void;
};
```

De esta forma, si cometemos un error tipográfico el analizador estático de nuestro editor de código nos avisará de que hemos cometido un error en nuestro código.

```typescript
const eventFactory: EventHandlerFactory = {
  showDialgo: this.showDialog, // Type '{ showDialgo: () => void; }' is not assignable to type 'EventHandlerFactory'.
}
```

#### Bonus: accediendo a los valores de códigos permitidos en tiempo de ejecución.

Posiblemente se de la situación de que tengamos que leer el valor del conjunto AllowedCode en tiempo de ejecución. Por ejemplo, si estamos recibiendo eventos distintos a los que esperábamos y queremos filtrarlos para no tener errores al intentar ejecutarlos.

El problema que nos encontramos es que AllowedCode es un tipo y los tipos no existen en tiempo de ejecución. ¿Cómo podemos solucionar esto? Simplemente utilizando un enumerable en lugar de un tipo.

```typescript
enum AllowedCodes {
  showDialog = 'showDialog',
  anotherEvent = 'anotherEvent',
  evenAnotherEvent = 'evenAnotherEvent'
}
```

<!-- TODO explicar por qué no puedo acceder a los tipos en tiempo de ejecución -->
De esta forma sí que podemos acceder a esta lista de valores durante el tiempo de ejecución, tal y como podemos ver en el código de debajo. Y no sólo eso, sino que la funcionalidad del tipo EventHandlerFactory se mantiene igual, ya que está permitido que los índices de los tipos indexados pertenezcan a un enumerable.

```typescript
export type EventHandlerFactory = {
  [K in AllowedCodes]: () => void; // Funciona incluso si AllowedCodes es enum
};
```

```typescript
document.addEventListener("optionClick", (event: CustomEvent<OptionClick>) => {
  const eventCode = event.detail.code;
  if (eventCode in AllowedCodes) { // Como es un enum, podemos leerlo en tiempo de ejecucion y usarlo para filtrar
    this.helpMenuActionFactory[eventCode]();
  }
});
```

Como habéis podéis comprobar TypeScript es una herramienta muy potente que puede aportar mucha calidad a tus proyectos mediante la comprobación de tipos, evitándonos errores en tiempo de ejecución.

En las próximas entradas profundizaré más en algunas funcionalidades de TypeScript interesantes, ¡hasta la próxima!
