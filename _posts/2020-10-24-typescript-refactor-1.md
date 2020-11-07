---
layout: post
title: Refactoring a TypeScript function, part 1.
categories: [TypeScript]
---

In a previous project I had to use a web component that communicated with the rest of an Angular application through events. This component was closed and I could not modify it, but it was configurable through input parameters that allowed to modify the detail of the events that were issued.

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

Depending on the code of the detail we will have to execute a certain function in the Angular application, so in the event handler we will add an if block to make sure that we execute the function we want for the showDialog code. In this case we will display a dialog on screen.

```typescript
document.addEventListener("optionClick", event => {
  if (event.detail.code === "showDialog") {
    this.showDialog();
  }
});
```

However, it is possible that throughout the life cycle of this application we will have to extend the functionality of the web component with new event codes. From the existing code we could extend the if block by adding a else statement for each new code to be managed.

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

The problem here is that the number of new functions and events is not final. It could be three, or it could be twenty, so twenty nested if blocks might not be the most elegant solution. An alternative to this could be the switch statement, but it would repeat the problem that it is a very long code block. How could we simplify the management of code-function pairs?

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

### Factory method as a solution
Using the variable eventFactory we can index the codes inside the detail of the issued events and associate them to the methods that we want to execute.

```typescript
const eventFactory = {
  showDialog: this.showDialog,
  anotherEvent: this.anotherFunction,
  evenAnotherEvent: this.evenAnotherFunction,
};
```

We can add a type to this factory using TypeScript's indexed types. This way we define that the indexes, which in this case are the event codes, must be of type string, while the response is a function with no signature or response.

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

This way we can access a method from its corresponding code and execute it without using control sentences.

```typescript
document.addEventListener("optionClick", (event: CustomEvent<OptionClick>) => {
  eventFactory[event.detail.code](); // Remember to add ()
});
```

### Adding restrictions to properties with TypeScript

It is possible that while declaring the variable eventFactory we have a typo while writing one of the properties.

```typescript
const eventFactory: EventHandlerFactory = {
  showDialgo: this.showDialog, // should be 'showDialog' instead of 'showDialgo'
}
```

When the handler receives the event with the code showDialog it will try to run it through eventFactoy and will not find any function associated with that name, so it will return the error `Uncaught TypeError: eventFactory.showDialog is not a function`.

How can we avoid this? One way to do this is to allow the code editor to show an error when we make a typo in the property name. To do this, we will modify the definition of the EventHandlerFactory type by using some of the TypeScript features.

We will define the AllowedCode type as a set of corresponding literals with the event codes that we expect to manage.

```typescript
type AllowedCode = "showDialog" | "anotherEvent" | "evenAnotherEvent";
```

And then we modify the definition of EventHandlerFactory as we see in the code extract below. Previously the definition `[K in string]` established that the properties should be string, but now with the definition `[K in AllowedCode]` the properties should be part of the AllowedCode set.

```typescript
export type EventHandlerFactory = {
  [K in AllowedCode]: () => void;
};
```

This way, if we make a typo the static analyzer of our code editor will warn us that we have an error.

```typescript
const eventFactory: EventHandlerFactory = {
  showDialgo: this.showDialog, // Type '{ showDialgo: () => void; }' is not assignable to type 'EventHandlerFactory'.
}
```

### Bonus: reading allowed codes in runtime.

We may have to read the value of the AllowedCode set at runtime. For example, if we are receiving different events from the ones we expected and we want to filter them in order not to have errors when trying to execute them.

The problem that we found is that AllowedCode is a type and types do not exist at runtime. How can we solve this? Simply by using an enum instead of a type.

```typescript
enum AllowedCodes {
  showDialog = 'showDialog',
  anotherEvent = 'anotherEvent',
  evenAnotherEvent = 'evenAnotherEvent'
}
```

This way we can access the list of values during runtime, as we can see in the code below. Also the functionality of the type EventHandlerFactory remains the same, since indexes can be part of an enum.

```typescript
export type EventHandlerFactory = {
  [K in AllowedCodes]: () => void; // Still works if AllowedCodes is an enum
};
```

```typescript
document.addEventListener("optionClick", (event: CustomEvent<OptionClick>) => {
  const eventCode = event.detail.code;
  if (eventCode in AllowedCodes) { // We can read it in runtime and use it to control the logic flow
    this.helpMenuActionFactory[eventCode]();
  }
});
```

As you can see, TypeScript is a very powerful tool that can bring a lot of quality to your projects by checking types, avoiding errors at runtime.

In future posts I will go deeper into some interesting TypeScript features.
