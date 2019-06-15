[![Travis](https://img.shields.io/travis/skiptirengu/tsyringex.svg)](https://travis-ci.org/skiptirengu/tsyringex/)

# TSyringex - NOT PUBLISHED YET

A dependency injection container for TypeScript/JavaScript.

This is a fork of the original [tsyringe](https://github.com/microsoft/tsyringe) project,
and aims add some useful features like scoped dependency lifetime and multi injection.
Keep in mind that this version can, and probably will, be incompatible with the original tsyringe at some point.

- [Installation](#installation)
- [API](#api)
  - [injectable()](#injectable)
  - [singleton()](#singleton)
  - [scoped()](#scoped)
  - [autoInjectable()](#autoinjectable)
  - [inject()](#inject)
  - [injectAll()](#injectAll)
- [Scoped container](#scoped-container)
- [Full Examples](#full-examples)

## Installation

Install with `npm`

```sh
npm install --save tsyringex
```

**or** install with `yarn`

```sh
yarn add tsyringex
```

Modify your `tsconfig.json` to include the following settings

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

Add a polyfill for the Reflect API (examples below use reflect-metadata). You can use:

- [reflect-metadata](https://www.npmjs.com/package/reflect-metadata)
- [core-js (core-js/es7/reflect)](https://www.npmjs.com/package/core-js)
- [reflection](https://www.npmjs.com/package/@abraham/reflection)

The Reflect polyfill import should only be added once, and before DI is used:

```typescript
// main.ts
import "reflect-metadata";

// Your code here...
```

## API

### injectable()

Class decorator factory that allows the class' dependencies to be injected at
runtime.

#### Usage

```typescript
import {injectable} from "tsyringex";

@injectable()
class Foo {
  constructor(private database: Database) {}
}

// some other file
import "reflect-metadata";
import {container} from "tsyringex";
import {Foo} from "./foo";

const instance = container.resolve(Foo);
```

### singleton()

Class decorator factory that registers the class as a singleton within the
global container.

#### Usage

```typescript
import {singleton} from "tsyringex";

@singleton()
class Foo {
  constructor() {}
}

// some other file
import "reflect-metadata";
import {container} from "tsyringex";
import {Foo} from "./foo";

const instance = container.resolve(Foo);
```

### scoped()

Class decorator factory that registers the class as a scoped dependency within the
global container.

#### Usage

```typescript
import {scoped} from "tsyringex";

@scoped()
class Foo {
  constructor() {}
}

// some other file
import "reflect-metadata";
import {container} from "tsyringex";
import {Foo} from "./foo";

const instance = container.createScope().resolve(Foo);
```

### autoInjectable()

Class decorator factory that replaces the decorated class constructor with
a parameterless constructor that has dependencies auto-resolved.

**Note** Resolution is performed using the global container

#### Usage

```typescript
import {autoInjectable} from "tsyringex";

@autoInjectable()
class Foo {
  constructor(private database?: Database) {}
}

// some other file
import {Foo} from "./foo";

const instance = new Foo();
```

Notice that in order to allow the use of the empty constructor `new Foo()`, we
need to make the parameters optional, e.g. `database?: Database`

### inject()

Parameter decorator factory that allows for interface and other non-class
information to be stored in the constructor's metadata

#### Usage

```typescript
import {injectable, inject} from "tsyringex";

interface Database {
  // ...
}

@injectable()
class Foo {
  constructor(@inject("Database") private database?: Database) {}
}
```

### injectAll()

Parameter decorator factory that allows for interface and other non-class
information to be stored in the constructor's metadata, when injecting 
multiple dependencies.

#### Usage

```typescript
import {injectable, injectAll} from "tsyringex";

interface Bar {
  // ...
}

@injectable()
class Foo {
  constructor(@injectAll("Bar") private bar?: Bar[]) {}
}
```
## Scoped container

The container will treat scoped dependencies as following:
  + Resolving the dependency from the global container or a child container:
    + The dependency will be treated as transient (a new instance will be created every time the `resolve()` method is invoked).
  + Resolving the dependency from a scoped container created from `createScope()` method:
    + The dependency will be treated as singleton (one instance will be created and cached for the container's lifetime).

```typescript
// SuperService.ts
export class ScopedService {
  // ...
}
```

```typescript
// main.ts
import "reflect-metadata";
import {ScopedService} from "./ScopedService";
import {container} from "tsyringex";

container.registerScoped(ScopedService, ScopedService);

const scope = container.createScope();

const globalService = container.resolve(ScopedService);
const scopedService = scope.resolve(ScopedService);

// Outputs false, as the two "ScopedService" 
// instances are not the same object
console.log(globalService === scopedService);
// Outputs true, since the "ScopedService" instance
// is cached within the scoped container
console.log(scopedService === scope.resolve(ScopedService));
```

## Full examples

### Example without interfaces

Since classes have type information at runtime, we can resolve them without any
extra information.

```typescript
// Foo.ts
export class Foo {}
```

```typescript
// Bar.ts
import {Foo} from "./Foo";
import {injectable} from "tsyringex";

@injectable()
export class Bar {
  constructor(public myFoo: Foo) {}
}
```

```typescript
// main.ts
import "reflect-metadata";
import {container} from "tsyringex";
import {Bar} from "./Bar";

const myBar = container.resolve(Bar);
// myBar.myFoo => An instance of Foo
```

### Example with interfaces

Interfaces don't have type information at runtime, so we need to decorate them
with `@inject(...)` so the container knows how to resolve them.

```typescript
// SuperService.ts
export interface SuperService {
  // ...
}
```

```typescript
// TestService.ts
import {SuperService} from "./SuperService";
export class TestService implements SuperService {
  //...
}
```

```typescript
// Client.ts
import {injectable, inject} from "tsyringex";

@injectable()
export class Client {
  constructor(@inject("SuperService") private service: SuperService) {}
}
```

```typescript
// main.ts
import "reflect-metadata";
import {Client} from "./Client";
import {TestService} from "./TestService";
import {container} from "tsyringex";

container.register("SuperService", {
  useClass: TestService
});

const client = container.resolve(Client);
// client's dependencies will have been resolved
```
