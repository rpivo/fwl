# zdf

zdf is a design pattern for zero-dependency frontends.

- [Why?](#why)
- [Practices](#practices)
  - [Dev Dependencies](#dev-dependencies)
    - [Recommended](#recommended)
  - [The Entry File](#the-entry-file)
  - [Using Classes](#using-classes)
  - [Web Components](#web-components)
- [Primary Components](#primary-components)
  - [Base](#base)
  - [Router](#router)
- [Useful Types](#useful-types)
  - [Modules](#modules)
  - [Interfaces](#interfaces)

# Why?

With ES modules now widely available in browsers, creating a zero-dependency frontend is becoming more possible. As new JS/CSS/Web Component specifications become available to users, we can more easily build our own abstractions rather than relying on a framework.

Sometimes you want to quickly scaffold up a project. This is where frameworks and libraries shine. You can set up an enterprise-ready repo in no time. This is great for a lot of people and a lot of projects, but there are drawbacks:

- Most likely there is some magic happening under the hood of which you're not aware.
- You have some control over package size, but a good chunk of that size will likely come from your dependencies.
- Many apps end up being over-engineered (like a blog built on top of React).
- You can only hope that your dependencies will have a long shelf life.
- As newer frameworks/libraries come out, your project begins to feel more stale.
- The peak performance of your project is only as good as your dependencies.

<hr />

# Practices

## Dev Dependencies

With a zero-dependency frontend, dev dependencies are still valid since they are only used for development and don't eventually make it into the build.

Below are a list of recommended dev dependencies which can greatly improve the development experience.

### TypeScript

[TypeScript](https://github.com/microsoft/TypeScript) has been a ubiquitous staple of the JavaScript ecosystem. It does more than just provide types. As a superset of JavaScript, TypeScript provides many other constructs that are used in a lot of zdf code in this repo.

### Vite

[Vite](https://github.com/vitejs/vite) combines ESBuild and Rollup to create an incredibly fast development experience. With a rich CLI and NPM command palette, Vite can bootstrap a project incredibly quickly, greatly reducing the time it takes to get started.

## The Entry File

The entry file is what gets imported into `index.html` in the browser. It's responsible for starting the application.

Generally speaking, to start the app means that you are creating a `Router` that mounts various `pages` or `screens` or `views` to the DOM on a given `mountNode`. These `pages`, `screens` or `views` are typically components themselves, but they don't have to be.

```ts
import Router from 'components/Router';

function startApp() {
  const mountNode = document.querySelector('main');
  new Router(mountNode);
}

document.addEventListener('DOMContentLoaded', startApp);
```

You can also move the app-starting logic to a component called `App`. `App` wouldn't need to be a web component, but it would isolate the router business logic from the `DOMContentLoaded` listener logic.

The `DomContentLoaded` event is used, which indicates when the DOM has completely loaded. It does not wait for other resources to load. The `load` event, on the other hand, waits for these resources. Because `DomContentLoaded` waits only for the DOM to finish loading, it can be used to start the app even while other resources are still loading.

## Using Classes

Web Components are built with classes, and we can use class inheritance to keep our code dry.

The `Base` class is an abstract base class which all web components extend. It provides a few methods and properties that all web components can use.

## Web Components

[Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components) are a native set of APIs which allow us to build reusable components right in the browser. The web component specification continues to evolve and has garnered more and more support over time.

<hr />

# Primary Components

## Base

The `Base` component is an abstract base class which all web components extend. It provides a few methods and properties that all web components can use.

Below is an example `index.ts` file for a `Base` component:

```ts
interface EventListener {
  callback: (event: Event) => void;
  component?: WebComponent;
  event: string;
  selector?: Selector;
}

export default abstract class Base extends HTMLElement {
  constructor(html: string) {
    super();

    const template = document.createElement('template');
    template.innerHTML = html;
    this.attachShadow({ mode: 'open' }).appendChild(
      template.content.cloneNode(true)
    );
  }

  get value() {
    const input = this.shadowRoot?.querySelector('input');

    if (input) {
      return input.value;
    }

    return null;
  }

  public static registerComponents(...elements: Array<WebComponent>): void {
    for (const element of elements) {
      if (!customElements.get(element.displayName)) {
        customElements.define(element.displayName, element);
      }
    }
  }

  addEventListeners({
    callback,
    component,
    event,
    selector = '',
  }: EventListener) {
    const elementToFind = component ? component.displayName : selector;

    const target = this.shadowRoot?.querySelector(elementToFind);

    if (target) {
      target.addEventListener(event, callback);
    }
  }

  get(target: Selector | WebComponent): Nullable<Base> {
    const elementToFind =
      typeof target === 'string' ? target : target.displayName;

    const element = <Nullable<Base>>(
      this.shadowRoot?.querySelector(elementToFind)
    );

    if (element) {
      return element;
    }

    return null;
  }
}
```

### API Reference

### Interfaces

#### `EventListener`

The `EventListener` defines an object which acts as config for the creation of an event listener. This is used as input to the `addEventListeners` method.

```ts
interface EventListener {
  callback: (event: Event) => void;
  component?: WebComponent;
  event: string;
  selector?: Selector;
}
```

### Methods

#### `constructor`

```ts
  constructor(html: string) {
    super();

    const template = document.createElement('template');
    template.innerHTML = html;
    this.attachShadow({ mode: 'open' }).appendChild(
      template.content.cloneNode(true)
    );
  }
```

The `constructor` takes in a string called `html`. This is a template string imported from each web component's corresponding HTML template file.

### Router

<hr />

## Useful Types

Below is an example types file `declarations.d.ts`:

```ts
declare module '*.html' {
  const value: string;
  export default value;
}

interface WebComponent extends CustomElementConstructor {
  displayName: string;
}

type Nullable<T> = T | null;

type Selector = string;
```

### Modules

#### `'*.html'`

```ts
declare module '*.html' {
  const value: string;
  export default value;
}
```

This module declaration is useful when creating template files that are named with the `.html` suffix. TypeScript will interpret these files as ES modules which export a default string. The module bundler should then be used to transform these template files into ES modules.

### Interfaces

#### `WebComponent`

```ts
interface WebComponent extends CustomElementConstructor {
  displayName: string;
}
```

The `WebComponent` interface is used to define a web component. It extends the `CustomElementConstructor`, which defines a custom element.

The `WebComponent` interface also defines a `displayName` property. This is actually a static property defined on each component, although TypeScript currently has no way of enforcing static properties. However, providing the type here does enforce some type checking when attempting to access the `displayName` property.

The `displayName` is used as the name of the element within the DOM. If you create a `Card` element and give this element the `displayName` of `card-element`, then the `Card` element will appear in the DOM as:

```html
<card-element></card-element>
```

Note that Web Component convention dictates that the `displayName` by a hyphenated string.
