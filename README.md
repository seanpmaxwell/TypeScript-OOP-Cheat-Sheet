# TypeScript Object-Oriented Cheat Sheet

_All the major parts of TypeScript object-oriented keywords/terminology in one place_

## Table of Contents

- [Overview](#overview)
- [TypeScript Objects vs. JavaScript Objects](#typescript-objects-vs-javascript-objects)
- [Objects and Classes Overview](#object-overview)
- [Inheritance](#inheritance)
- [Encapsulation](#Encapsulation)
- [Other Modifiers](#other-modifiers)
- [Polymorphism](#polymorphism)
  - [Interfaces](#interfaces)
- [Abstract Classes and Methods](#abstract-classes-and-methods)
- [Conclusion](#conclusion)

<p align="center">· · ·</p>

## Overview

<details>
<summary>Classes vs Factory functions</summary>

Classes are implemented using the `class` keyword (introduced in ES6). To create an object, we call the class with the `new` keyword, which triggers the constructor and returns an instance-object, just like in regular JavaScript. Since objects in JavaScript technically include functions, `null`, and classes too (anything where `typeof "..." === 'object'` or `"..." instanceof Object === true`), I like to use the term _instance-object_ to refer to objects returned by the `new` keyword.

`Spot` is an instance-object of `Dog`:

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

let Spot = new Dog(2, "Labrador");
```

The equivalent using function-objects in ES5:

```js
function Dog(age, breed) {
  this.age = age;
  this.breed = breed;
}

Dog.prototype.getRelativeAge = function () {
  return this.age * 7;
};

var Spot = new Dog(2, "Labrador");
```
</details>


<details>
<summary>TypeScript Classes vs. JavaScript Classes</summary>
  
TypeScript objects are just syntactic sugar for JavaScript function-objects. There's a lot of repetitive code involved in using function-objects as classes in JavaScript, which is why `class` was introduced in ES6. TypeScript takes ES6 classes to a higher plane of reality by adding not only types but also object features such as `public`, `private`, `abstract`, etc. If you're interested in learning more about the quirks of JavaScript function-objects (which I highly recommend), please check out my Medium article [here](https://levelup.gitconnected.com/the-javascript-object-paradigm-and-prototypes-explained-simply-e9cb9eaa49aa).

</details>

<p align="center">· · ·</p>

## Inheritance

Now that you know how to make objects and can see how they work under the hood in JavaScript, let's start learning about TypeScript inheritance. In our PetStore program, we're selling dogs and cats, but there could be different breeds of dogs and cats, right? Dogs and cats might also share some of the same attributes, like `age` and `weight`. Superclasses (a.k.a. parent classes) allow related objects to be grouped together so that they can inherit similar attributes. To inherit from a parent class, we use the `extends` keyword. When you extend a class, all of its attributes and methods are passed down. Instead of creating `age` and `weight` each time we add a new animal to our pet store, we can now just expand upon a parent `Animal` class. Multi-level inheritance is also possible by extending child classes.

The `super` keyword serves two roles in inheritance. First, it acts as a function that calls the parent class's constructor. It has to be called before `this` is used in the child class's constructor. Second, it allows us to access methods (but NOT attributes) of the parent class.

`Animal` is the parent class of `Dog`:

```ts
class Animal {
  age: number;
  breed: string;

  constructor(age: number, breed: string) {
    this.age = age;
    this.breed = breed;
  }

  makeSound_(sound: string): void {
    console.log(sound);
    console.log(sound);
    console.log(sound);
  }
}
```

Basic inheritance using `super`:

```ts
class Dog extends Animal {
  playsFetch: boolean;

  constructor(age: number, breed: string, playsFetch: boolean) {
    super(age, breed); // call parent constructor
    this.playsFetch = playsFetch;
  }

  makeSound(): void {
    super.makeSound_("woof woof");
  }

  getAgeInHumanYears(): number {
    return this.age * 7; // super.age will throw an error
  }
}

class Cat extends Animal {
  constructor(age: number, breed: string) {
    super(age, breed);
  }

  makeSound(): void {
    super.makeSound_("meow meow");
  }
}
```

JavaScript inheritance works the same way, but TypeScript adds access-control modifiers when working with parent classes. Declaring class-level variables outside of methods (class fields) wasn't possible in ES6, but modern JavaScript supports it as of ES2022, along with truly private `#fields`.

<p align="center">· · ·</p>

## Encapsulation

### `public`

Suppose our PetStore program has a class named `PetStore`. If this class wants to call methods on our `Dog` objects, then those methods will need to be marked `public`. When a method or variable is public, it can be accessed by other parts of our program. Leaving off a modifier on a variable or method is the same as marking it `public`.

```ts
class Dog {
  public name: string; // leaving out 'public' would work too
}

class PetStore {
  dogs: Array<Dog>;

  printAllDogNames(): void {
    this.dogs.forEach((dog) => {
      console.log(dog.name);
    });
  }
}
```

### `private`

Allowing other coders to directly access an object's attributes generally isn't a good idea, though. It's better to use getters and setters to access and modify class properties, so we can run some logic when setting a value and prevent errors. For example, a dog's name shouldn't be falsy, and it should be under a certain length; a realistic dog name would never be more than 10–20 characters. To make a class variable or method accessible only within that class, we mark it `private`. TypeScript classes have built-in `get` and `set` accessors, which trigger our getter and setter whenever the property is accessed or assigned.

```ts
class Dog {
  private _name: string; // a leading underscore is the convention

  get name(): string {
    return this._name;
  }

  set name(name: string) {
    if (!name || name.length > 20) {
      throw new Error("Name invalid");
    } else {
      this._name = name;
    }
  }
}

class PetStore {
  private dogs: Array<Dog>; // we changed this to private too

  constructor() {
    this.dogs = [new Dog()];
    this.dogs[0].name = "Fido"; // will call 'set'
  }

  printAllDogNames(): void {
    this.dogs.forEach((dog) => {
      console.log(dog.name); // will call 'get'
    });
  }
}
```

### `#` a new alternative to `private`

Unlike `private` this provides both compile-time AND runtime safety. 

```ts
class PetStore {
  #dogs: Array<Dog>; // we changed this to private too

  constructor() {
    this.#dogs = [new Dog()];
    this.#dogs[0].name = "Fido"; // will call 'set'
  }

  printAllDogNames(): void {
    this.#dogs.forEach((dog) => {
      console.log(dog.name); // will call 'get'
    });
  }
}
```

### `protected`

Lastly, let's look at the `protected` keyword. Protected means that a variable or method can only be accessed within the class itself and its child classes. Remember the `makeSound_` method of the `Animal` parent class? We shouldn't be able to call that method externally on `Animal` or any class that inherits from it, because not all animals make sounds. I like to append a trailing underscore to protected members, although it's not a convention.

```ts
class Animal {
  protected makeSound_(sound: string): void {
    console.log(sound);
    console.log(sound);
    console.log(sound);
  }
}

class Dog extends Animal {
  makeSound(): void {
    super.makeSound_("woof woof");
  }
}

class PetStore {
  makeSomeSounds(): void {
    let dog = new Dog();
    dog.makeSound(); // => 'woof woof' 'woof woof' 'woof woof'

    let animal = new Animal();
    animal.makeSound_("..."); // => NOT ALLOWED
  }
}
```

<p align="center">· · ·</p>

## Other Modifiers

### `static`

There are two other modifiers that are important to mention when talking about TypeScript classes: `static` and `readonly`. If we want to access a property on a class without going through the trouble of creating an instance-object (calling the class with `new`), we can mark it `static`, and it will be set on the class (function-object) itself. This is useful for methods and class variables that don't depend on any dynamic property. For example, a dog will always be the same species.

```ts
class Dog {
  static species = "Canis familiaris";
  age = 10;
}

class PetStore {
  printSpecies(): void {
    console.log(Dog.species); // => 'Canis familiaris'
    console.log(Dog.age); // => NOT ALLOWED (undefined in plain JS)
  }
}
```

### `readonly`

The `readonly` keyword is pretty self-explanatory. It's used for class-level variables and means that the value cannot be reassigned. Values that are initialized when the class is created and that you know will never change should be `readonly`. Our `Dog` class's `species` property is a good example: no matter what attributes we assign to a dog, it will always be the same species.

```ts
class Dog {
  static readonly species = "Canis familiaris";
}

class PetStore {
  printSpecies(): void {
    console.log(Dog.species); // => 'Canis familiaris'
    Dog.species = "Terdus maximus"; // => NOT ALLOWED
  }
}
```

<p align="center">· · ·</p>

## Polymorphism

### `interface`

Whenever we want to say that an object being passed around has a specific set of attributes, we can use an interface. Interfaces are nifty little tools that come in handy in several situations.

The most immediate one that comes to mind is testing. Suppose we have a method in our `Dog` class that makes an I/O call, and we want to unit test a method in our `PetStore` class that calls it. We don't want to fire an I/O call every time a unit test runs, but we still need an object that satisfies the `Dog` type. Let's create an `IDog` interface that specifies a method for both the real class and the mock class we create for our unit test.

```ts
interface IDog {
  getPedigree: Function;
}

class Dog implements IDog {
  getPedigree(): Promise<Pedigree> {
    return someThirdPartyIoCall("...");
  }
}

class MockDog implements IDog {
  getPedigree(): Promise<Pedigree> {
    return Promise.resolve(new DummyPedigreeObject());
  }
}

async function methodToBeTested(dog: IDog): Promise<void> {
  try {
    let pedigree = await dog.getPedigree();
    // do assertions here
  } catch (err) {
    console.log(err);
  }
}

// Real world
methodToBeTested(new Dog());

// During testing
methodToBeTested(new MockDog());
```

This is only one small example of using interfaces; there are plenty more uses. I recommend checking out the TypeScript docs [here](https://www.typescriptlang.org/docs/handbook/interfaces.html) for more information.

## Abstract Classes and Methods

## `abstract`

Think of abstract classes as a cross between regular parent classes and interfaces. Like interfaces, abstract classes define attributes for other classes, but unlike interfaces, some of their methods may contain an implementation. A method without an implementation must be marked `abstract`, and so must its containing class. Abstract classes cannot be instantiated (you can't use `new` on them) and are useful when you know you'll never need the parent class directly.

Both cats and dogs have an `age` property, and we want to know each one's age in human years. The way to calculate this differs depending on the animal, though, so let's use an abstract class.

```ts
abstract class Animal {
  protected age_: number;

  abstract getRelativeAge(): number;
}

class Dog extends Animal {
  getRelativeAge(): number {
    return this.age_ * 7;
  }
}

class Cat extends Animal {
  getRelativeAge(): number {
    return this.age_ * 6;
  }
}
```

> **Note:** This is not meant to be an accurate representation of how to calculate a cat's or a dog's age.
