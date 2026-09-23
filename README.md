# TypeScript Object-Oriented Cheat Sheet

_All of TypeScript's object-oriented keywords and terminology in one place._

> **Legend:** ❌ marks a compile error, followed by TypeScript's message. 💥 marks a runtime error. Examples assume `"strict": true` and `"noImplicitOverride": true`.

## Table of Contents

- [Overview](#overview)
- [Inheritance](#inheritance)
  - [`extends`](#extends)
  - [`super`](#super)
  - [`override`](#override)
- [Encapsulation](#encapsulation)
  - [`public`](#public)
  - [`private`, `get` & `set`](#private-get--set)
  - [`#private`: truly private fields](#private-truly-private-fields)
  - [`protected`](#protected)
- [Polymorphism](#polymorphism)
  - [Through subclasses](#through-subclasses)
  - [`interface` & `implements`](#interface--implements)
- [Abstract Classes](#abstract-classes)
  - [`abstract`](#abstract)
- [Other Modifiers](#other-modifiers)
  - [`static`](#static)
  - [`readonly`](#readonly)
- [Quick Reference](#quick-reference)
- [Conclusion](#conclusion)

## Overview

### Classes under the hood

<details>
<summary>Explanation</summary>

You declare a class with the `class` keyword, which arrived in ES2015 (ES6). Calling it with `new` runs the `constructor` and returns a new object, called an **instance** of the class.

A class is mostly a cleaner way to write JavaScript's older constructor-function and prototype pattern. Methods live on the class's `prototype`, and every instance shares them. The two forms below behave almost the same way. The class version is stricter in a few places: calling it without `new` throws, its body always runs in strict mode, and its methods are non-enumerable.

</details>

`spot` is an instance of `Dog`:

```ts
class Dog {
  age: number;
  breed: string;

  constructor(age: number, breed: string) {
    this.age = age;
    this.breed = breed;
  }

  getRelativeAge(): number {
    return this.age * 7;
  }
}

const spot = new Dog(2, "Labrador");
spot.getRelativeAge(); // => 14
```

The same thing as an ES5 constructor function:

```js
function Dog(age, breed) {
  this.age = age;
  this.breed = breed;
}

Dog.prototype.getRelativeAge = function () {
  return this.age * 7;
};

var spot = new Dog(2, "Labrador");
```

### What TypeScript adds

<details>
<summary>Explanation</summary>

TypeScript builds on JavaScript classes. It adds type annotations along with `public`, `private`, `protected`, `readonly`, `abstract`, `implements`, `override`, and parameter properties.

Nearly all of these features exist **only at compile time**. The compiler checks them and then erases them, so none of them appear in the JavaScript it emits. That matters. A `private` field is still an ordinary property at runtime, and an `interface` leaves nothing behind. The [Quick Reference](#quick-reference) at the end shows which features survive to runtime.

If you'd like to dig into how JavaScript constructor functions and prototypes work (I highly recommend it), see my Medium article, [_The JavaScript Object Paradigm and Prototypes Explained Simply_](https://levelup.gitconnected.com/the-javascript-object-paradigm-and-prototypes-explained-simply-e9cb9eaa49aa).

</details>

### When to use a class

<details>
<summary>Explanation</summary>

##### A class fits when...

- **The object owns mutable state** and its methods have to keep that state valid over time (a connection pool, a cache, a parser).
- **You'll create many instances.** Every instance shares its methods through the prototype instead of carrying its own copies.
- **You need `instanceof`**, or you're extending a class you don't control, such as `Error` or `HTMLElement`.

##### Reach for something simpler when...

- **You only need a namespace.** Export functions from a module, or use an object literal.
- **You're building a configured object with no real lifecycle.** A factory function avoids `this` entirely, and its closure gives you private state the language actually enforces:

  ```ts
  function createCounter(start = 0) {
    let count = start; // only reachable through the closure

    return {
      increment(): number {
        return ++count;
      },
      get value(): number {
        return count;
      },
    };
  }

  type Counter = ReturnType<typeof createCounter>;

  const counter: Counter = createCounter();
  counter.increment(); // => 1
  ```

- **The data crosses a boundary** (JSON, network, storage). `JSON.stringify` keeps only an object's own data, and `JSON.parse` returns plain objects. So the methods and the prototype are lost, and TypeScript can't tell:

  ```ts
  const json = JSON.stringify(new Dog(2, "Labrador")); // '{"age":2,"breed":"Labrador"}'
  const revived = JSON.parse(json) as Dog; // compiles, but it's a lie
  revived instanceof Dog; // => false
  revived.getRelativeAge(); // 💥 TypeError: revived.getRelativeAge is not a function
  ```

  For this kind of data, describe the shape with a `type` or `interface`, validate it where it enters your program, and keep behavior in plain functions.

</details>

<p align="center">· · ·</p>

## Inheritance

### `extends`

<details>
<summary>Explanation</summary>

`extends` makes one class inherit from another. The child class gets all of the parent's properties and methods. So instead of redeclaring `age` and `breed` for every new animal in the pet store, each animal extends a shared `Animal` class.

A class can extend only **one** parent. Chains are allowed (`Puppy extends Dog extends Animal`), but a deep hierarchy is hard to change later, so keep them shallow.

</details>

### `super`

<details>
<summary>Explanation</summary>

`super` has two jobs:

1. **`super(...)` calls the parent's constructor.** In a child constructor, you must call it before you use `this`.
2. **`super.method()` calls the parent's version of a method.** This is mostly useful inside an override, when you want to extend the parent's behavior instead of replacing it. `super` points at the parent's _prototype_, so it can reach methods but not instance properties like `age`. Those live on `this`.

If you haven't overridden a method, just call it with `this.method()`.

</details>

### `override`

<details>
<summary>Explanation</summary>

`override` (TypeScript 4.3+) marks a method as intentionally replacing a parent method. With `noImplicitOverride` turned on, the modifier is required. If someone later renames or removes the parent method, every override stops compiling instead of silently becoming an orphaned method that nothing calls.

</details>

### Example

`Animal` is the parent class of `Dog` and `Cat`:

```ts
class Animal {
  age: number;
  breed: string;

  constructor(age: number, breed: string) {
    this.age = age;
    this.breed = breed;
  }

  describe(): string {
    return `${this.age}-year-old ${this.breed}`;
  }

  makeSound(): string {
    return "...";
  }
}

class Dog extends Animal {
  playsFetch: boolean;

  constructor(age: number, breed: string, playsFetch: boolean) {
    super(age, breed); // must run before `this` is used
    this.playsFetch = playsFetch;
  }

  override makeSound(): string {
    return "Woof!";
  }

  override describe(): string {
    // Extend the parent's behavior instead of replacing it
    return `${super.describe()} who ${this.playsFetch ? "loves" : "ignores"} fetch`;
  }

  getAgeInHumanYears(): number {
    return this.age * 7; // `this.age`, never `super.age`
  }
}

class Cat extends Animal {
  // No constructor needed: Cat inherits Animal's as-is.

  override makeSound(): string {
    return "Meow!";
  }
}

const rex = new Dog(3, "Beagle", true);
rex.describe(); // => '3-year-old Beagle who loves fetch'
rex.makeSound(); // => 'Woof!'
```

> Everything in this section except `override` is plain JavaScript. Declaring fields in the class body (outside the constructor) wasn't possible in ES2015; JavaScript added class fields in ES2022, along with truly private `#fields`.

<p align="center">· · ·</p>

## Encapsulation

### `public`

<details>
<summary>Explanation</summary>

Suppose the pet store has a `PetStore` class that needs to read each dog's name. Any code in the program can use a `public` member. Members are public by default, so writing `public` is optional.

</details>

```ts
class Dog {
  public name: string; // same as writing just `name: string;`

  constructor(name: string) {
    this.name = name;
  }
}

class PetStore {
  dogs: Dog[] = [new Dog("Fido"), new Dog("Rex")];

  printAllDogNames(): void {
    for (const dog of this.dogs) {
      console.log(dog.name);
    }
  }
}
```

### `private`, `get` & `set`

<details>
<summary>Explanation</summary>

Letting other code write straight to an object's state isn't usually a good idea. A dog's name shouldn't be empty, and a realistic one is never longer than 20 characters. If anyone can assign `dog.name`, nothing stops a bad value from getting in.

The fix is to mark the stored value `private`, so it's only accessible inside the class, and expose it through **accessors**. A `get` accessor runs when the property is read, and a `set` accessor runs when it's assigned. From the outside, `dog.name` still looks like an ordinary property.

For an invariant to hold, _every_ path that writes the value has to enforce it. That includes the constructor, not just the setter. That's why both paths below go through a single validator.

</details>

```ts
function validateName(name: string): string {
  const trimmed = name.trim();
  if (trimmed.length === 0 || trimmed.length > 20) {
    throw new RangeError(`Invalid dog name: "${name}"`);
  }
  return trimmed;
}

class Dog {
  private _name: string; // a leading underscore is the usual convention

  constructor(name: string) {
    this._name = validateName(name);
  }

  get name(): string {
    return this._name;
  }

  set name(value: string) {
    this._name = validateName(value);
  }
}

const fido = new Dog("Fido");
fido.name; // => 'Fido' (calls `get`)
fido.name = "Sir Fetchalot the Magnificent"; // 💥 RangeError (calls `set`)
fido._name = ""; // ❌ Property '_name' is private and only accessible within class 'Dog'.
```

> **`private` is a compile-time promise only.** The emitted JavaScript has an ordinary `_name` property. It appears in `Object.keys()`, `JSON.stringify()`, and `console.log()`, and TypeScript even allows the escape hatch `fido["_name"] = ""`.

A getter with no matching setter makes a property read-only, and unlike [`readonly`](#readonly), that holds at runtime too.

### `#private`: truly private fields

<details>
<summary>Explanation</summary>

`#` fields are a **JavaScript** feature (ES2022), not a TypeScript one. The engine enforces them at runtime. Code outside the class can't read, write, or even detect them, and no escape hatch exists. They also enable a reliable "brand check": `#field in obj` returns `true` only for objects this class actually created.

</details>

```ts
class Dog {
  #name: string; // no underscore needed; `#` is part of the name

  constructor(name: string) {
    this.#name = validateName(name);
  }

  get name(): string {
    return this.#name;
  }

  set name(value: string) {
    this.#name = validateName(value);
  }

  static isDog(value: unknown): value is Dog {
    return typeof value === "object" && value !== null && #name in value;
  }
}

const fido = new Dog("Fido");
fido.#name; // ❌ Property '#name' is not accessible outside class 'Dog' because it has a private identifier.
Dog.isDog(fido); // => true
Dog.isDog({ name: "Fido" }); // => false: it has the right shape but wasn't built by Dog
```

|                                            | `private`          | `#private`                                                 |
| ------------------------------------------ | ------------------ | ---------------------------------------------------------- |
| Enforced by                                | The compiler only  | The JavaScript engine                                      |
| Escape hatch                               | `obj["field"]`     | None                                                       |
| Shows up in `JSON.stringify`/`Object.keys` | Yes                | No                                                         |
| Same name in a subclass                    | Compile error      | Fine (each class gets its own slot)                        |
| Works through a `Proxy`                    | Yes                | No (accessing it through a proxy throws a `TypeError`)     |
| Target requirements                        | None               | ES2022, or compiled down to `WeakMap`s for ES2015+ targets |

### `protected`

<details>
<summary>Explanation</summary>

`protected` sits between `public` and `private`. The class itself **and its subclasses** can use a protected member, but outside code can't. It fits helpers that are part of the contract between a parent and its children but not part of the public API.

</details>

```ts
class Animal {
  protected repeatSound(sound: string, times = 3): string {
    return Array(times).fill(sound).join(" ");
  }
}

class Dog extends Animal {
  bark(): string {
    return this.repeatSound("woof"); // subclasses can call protected members
  }
}

const dog = new Dog();
dog.bark(); // => 'woof woof woof'
dog.repeatSound("meow"); // ❌ Property 'repeatSound' is protected and only accessible within class 'Animal' and its subclasses.
```

> JavaScript has no runtime equivalent of `protected`. `#` fields aren't visible to subclasses at all.

### Parameter properties

<details>
<summary>Explanation</summary>

Adding a modifier to a constructor parameter (`public`, `private`, `protected`, or [`readonly`](#readonly)) declares the property and assigns it in a single step. The two classes below are equivalent:

</details>

```ts
class Verbose {
  readonly id: string;
  private owner: string;

  constructor(id: string, owner: string) {
    this.id = id;
    this.owner = owner;
  }
}

class Concise {
  constructor(
    readonly id: string,
    private owner: string,
  ) {}
}
```

> [!IMPORTANT]
> Parameter properties are one of the few TypeScript features that **generate** JavaScript code. As a result, they're rejected under `--erasableSyntaxOnly` (TypeScript 5.8+) and by Node's built-in type stripping. If you target either one, write the long form.

<p align="center">· · ·</p>

## Polymorphism

<details>
<summary>Explanation</summary>

_Polymorphism_ means "many forms." Code written against one type works with any object that fits that type, and each object brings its own behavior. TypeScript supports it in two ways: through **subclasses** (overriding a parent's method) and through **interfaces** (matching a shape).

</details>

### Through subclasses

Using `Animal`, `Dog`, and `Cat` from the [Inheritance example](#example), the loop below never checks which animal it has:

```ts
const animals: Animal[] = [new Dog(3, "Beagle", true), new Cat(5, "Siamese")];

for (const animal of animals) {
  console.log(animal.makeSound()); // => 'Woof!', then 'Meow!'
}
```

### `interface` & `implements`

<details>
<summary>Explanation</summary>

An interface describes a **shape**: the properties and methods an object must have. It has no implementation and is erased at compile time.

One situation where interfaces shine is testing. Suppose `Dog.getPedigree()` makes a network call, and you want to unit test a function that uses it without hitting the network. If the function depends on an interface rather than on `Dog`, a test can pass in anything with the right shape.

`implements` asks the compiler to check that a class matches an interface. A class can extend only one parent but can implement any number of interfaces.

</details>

```ts
interface Pedigree {
  sire: string;
  dam: string;
}

interface PedigreeSource {
  getPedigree(): Promise<Pedigree>;
}

class Dog implements PedigreeSource {
  #registryId: string;

  constructor(registryId: string) {
    this.#registryId = registryId;
  }

  async getPedigree(): Promise<Pedigree> {
    const response = await fetch(`https://registry.example.com/dogs/${this.#registryId}`);
    return (await response.json()) as Pedigree;
  }
}

class FakeDog implements PedigreeSource {
  async getPedigree(): Promise<Pedigree> {
    return { sire: "Rex", dam: "Lassie" };
  }
}

async function describeLineage(source: PedigreeSource): Promise<string> {
  const { sire, dam } = await source.getPedigree();
  return `by ${sire} out of ${dam}`;
}

// Production
await describeLineage(new Dog("AKC-123"));

// In tests: no network involved
await describeLineage(new FakeDog());

// Structural typing: any object with the right shape works, with no class required
await describeLineage({ getPedigree: async () => ({ sire: "Rex", dam: "Lassie" }) });
```

A few things worth knowing:

- **TypeScript is structurally typed.** An object fits a type if it has the right shape; it doesn't have to declare that it does. `implements` is optional. It just moves the error from the call site to the class declaration.
- **`implements` checks the class but doesn't change it.** It adds no code and doesn't inform type inference, so you still have to annotate the parameters of the class's methods.
- **Don't use `Function` as a type.** It accepts any callable and throws away the parameter and return types. Write the full signature instead, like `getPedigree(): Promise<Pedigree>`.
- **Skip the `I` prefix** (`IDog`). Common TypeScript convention names an interface after what it describes.

For more, see the TypeScript Handbook's [Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html) and [Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html) pages.

<p align="center">· · ·</p>

## Abstract Classes

### `abstract`

<details>
<summary>Explanation</summary>

An abstract class is a cross between a regular parent class and an interface. Like an interface, it declares members that subclasses must provide. Like a class, it can also contain real implementation code, constructors, and fields. A member without an implementation is marked `abstract`, and any class with an abstract member must itself be `abstract`. You can't instantiate an abstract class with `new`; you can only extend it.

Cats and dogs both have an age, and we want to express it in human years, but the conversion differs by animal. So the shared parent declares `getRelativeAge()` without implementing it, and each subclass supplies its own version.

</details>

```ts
abstract class Animal {
  abstract readonly species: string;
  protected readonly age: number;

  constructor(age: number) {
    this.age = age;
  }

  abstract getRelativeAge(): number;

  describe(): string {
    return `${this.species}, ${this.getRelativeAge()} in human years`;
  }
}

class Dog extends Animal {
  readonly species = "Dog";

  getRelativeAge(): number {
    return this.age * 7;
  }
}

class Cat extends Animal {
  readonly species = "Cat";

  getRelativeAge(): number {
    return this.age * 6;
  }
}

new Dog(3).describe(); // => 'Dog, 21 in human years'
new Animal(3); // ❌ Cannot create an instance of an abstract class.
```

> **Note:** This isn't an accurate way to calculate a cat's or a dog's age.

> **Pitfall:** Don't call abstract members from the parent's constructor. The parent's constructor finishes before the subclass initializes its fields, so calling `this.describe()` inside `Animal`'s constructor would print `undefined` for `species`.

|                             | Abstract class           | Interface                 |
| --------------------------- | ------------------------ | ------------------------- |
| Exists at runtime           | Yes (`instanceof` works) | No, it's erased           |
| Can contain implementation  | Yes                      | No                        |
| Constructors & field values | Yes                      | No                        |
| How many per class          | One (`extends`)          | Any number (`implements`) |

<p align="center">· · ·</p>

## Other Modifiers

### `static`

<details>
<summary>Explanation</summary>

A `static` member belongs to the **class itself** rather than to its instances, so you use it without calling `new`. It fits values and helpers that don't depend on any particular instance. For example, every dog is the same species.

</details>

```ts
class Dog {
  static species = "Canis familiaris";
  static count = 0;

  age: number;

  constructor(age: number) {
    this.age = age;
    Dog.count++;
  }
}

Dog.species; // => 'Canis familiaris'
Dog.age; // ❌ Property 'age' does not exist on type 'typeof Dog'.
new Dog(4).species; // ❌ Property 'species' does not exist on type 'Dog'. Did you mean to access the static member 'Dog.species' instead?
```

### `readonly`

<details>
<summary>Explanation</summary>

A `readonly` property can be assigned where it's declared or in the constructor, and nowhere else. Keep two things in mind:

- **It's shallow.** You can't reassign a `readonly` property, but you can still mutate the object it holds. For arrays, also use the `readonly string[]` type.
- **It's compile-time only.** For a guarantee at runtime, use `Object.freeze()` or a getter with no setter.

</details>

```ts
class Dog {
  static readonly species = "Canis familiaris";
  readonly birthDate: Date;
  readonly tricks: readonly string[];

  constructor(birthDate: Date, tricks: string[]) {
    this.birthDate = birthDate; // allowed in the constructor
    this.tricks = tricks;
  }
}

const dog = new Dog(new Date(2022, 0, 1), ["sit"]);

Dog.species = "Felis catus"; // ❌ Cannot assign to 'species' because it is a read-only property.
dog.birthDate = new Date(); // ❌ Cannot assign to 'birthDate' because it is a read-only property.
dog.birthDate.setFullYear(1999); // compiles: `readonly` is shallow
dog.tricks.push("roll over"); // ❌ Property 'push' does not exist on type 'readonly string[]'.
```

<p align="center">· · ·</p>

## Quick Reference

| Keyword              | What it does                                               | Enforced at                   |
| -------------------- | ---------------------------------------------------------- | ----------------------------- |
| `class` / `new`      | Declares a class / creates an instance                     | Runtime (JS)                  |
| `constructor`        | Initializes a new instance                                 | Runtime (JS)                  |
| `extends`            | Inherits from one parent class                             | Runtime (JS)                  |
| `super`              | Calls the parent's constructor or methods                  | Runtime (JS)                  |
| `override`           | Marks a method as replacing a parent method                | Compile time                  |
| `public`             | Accessible everywhere (the default)                        | Compile time                  |
| `private`            | Accessible only in the class                               | Compile time                  |
| `#field`             | Accessible only in the class, guaranteed                   | Runtime (JS)                  |
| `get` / `set`        | Run code when a property is read or assigned               | Runtime (JS)                  |
| `protected`          | Accessible in the class and its subclasses                 | Compile time                  |
| Parameter properties | Declare and assign a property from a constructor parameter | Compile time (generates code) |
| `interface`          | Describes a shape                                          | Compile time                  |
| `implements`         | Checks that a class matches an interface                   | Compile time                  |
| `abstract`           | Must be implemented by a subclass; can't be instantiated   | Compile time                  |
| `static`             | Belongs to the class, not its instances                    | Runtime (JS)                  |
| `readonly`           | Can't be reassigned after construction (shallow)           | Compile time                  |
