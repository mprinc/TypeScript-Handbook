# Introduction

A major part of software engineering is building components that not only have well-defined and consistent APIs, but are also <span class='definition'>reusable</span>.
Components that are capable of working on the data of today as well as the data of tomorrow will give you the most flexible capabilities for building up large software systems.

In languages like C# and Java, one of the main tools in the toolbox for creating reusable components is <span class='definition'>*generics*</span>, that is, being able to create a component that <span class='important'>can work over a variety of types rather than a single one</span>.
This allows users to consume these components and use their own types.

# Hello World of Generics

To start off, let's do the "hello world" of generics: the identity function.
The identity function is a function that will return back whatever is passed in.
You can think of this in a similar way to the `echo` command.

Without generics, we would either have to give the identity function a specific type:

```ts
function identity(arg: number): number {
    return arg;
}
```

Or, we could describe the identity function using the `any` type:

```ts
function identity(arg: any): any {
    return arg;
}
```

While <span class='important'>using `any` is certainly generic in that it will cause the function to accept any and all types for the type of `arg`, we actually are losing the information about what that type was when the function returns</span>.
If we passed in a number, the only information we have is that any type could be returned.

Instead, we need a way of capturing the type of the argument in such a way that we can also use it to denote what is being returned.
Here, we will use a <span class='definition'>*type variable*</span>, a special kind of variable that <span class='important'>works on types rather than values</span>.

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

We've now added a type variable `T` to the identity function.
This `T` allows us to capture the type the user provides (e.g. `number`), so that we can use that information later.
Here, we use `T` again as the return type. On inspection, we can now see the same type is used for the argument and the return type.
This allows us to traffic that type information in one side of the function and out the other.

We say that this version of the `identity` function is generic, as it works over a range of types.
Unlike using `any`, it's also just as precise (ie, it doesn't lose any information) as the first `identity` function that used numbers for the argument and return type.

Once we've written the generic identity function, we can call it in one of two ways.
The first way is to <span class='definition'>pass all of the arguments, including the type argument</span>, to the function:

```ts
let output = identity<string>("myString");  // type of output will be 'string'
```

Here we explicitly set `T` to be `string` as one of the arguments to the function call, denoted using the `<>` around the arguments rather than `()`.
<span class='comment' data-comment='This helps compiler to know that the `output` variable is of the type `string` and not of the type `any`'></span>

The second way is also perhaps the most common. Here we use <span class='definition'>*type argument inference*</span> -- that is, we want <span class='important'>the compiler to set the value of `T` for us automatically</span> based on the type of the argument we pass in:

```ts
let output = identity("myString");  // type of output will be 'string'
```

Notice that we didn't have to explicitly pass the type in the angle brackets (`<>`); the compiler just looked at the value `"myString"`, and set `T` to its type.
While type argument inference can be a helpful tool to keep code shorter and more readable, you may need to explicitly pass in the type arguments as we did in the previous example when the compiler fails to infer the type, as may happen in more complex examples.

# Working with Generic Type Variables

When you begin to use generics, you'll notice that when you create generic functions like `identity`, the compiler will enforce that you use any generically typed parameters in the body of the function <span class='important'>correctly</span>.
That is, that you actually treat these parameters as if they could be any and all types.

Let's take our `identity` function from earlier:

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

What if we want to also log the length of the argument `arg` to the console with each call?
We might be tempted to write this:

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

When we do, the compiler will give us an error that we're using the `.length` member of `arg`, but <span class='important'>nowhere have we said that `arg` has this member</span>.
Remember, we said earlier that <span class='definition'>these type variables stand in for any and all types</span>, so someone using this function <span class='important'>could have passed in a `number` instead, which does not have a `.length` member</span>.

Let's say that we've actually intended this function to work on <span class='important'>arrays of `T` rather than `T` directly</span>. Since we're working with arrays, the `.length` member should be available.
We can describe this just like we would create arrays of other types:

```ts
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

You can read the type of `loggingIdentity` as "the generic function `loggingIdentity` takes a type parameter `T`, and an argument `arg` which is an array of `T`s, and returns an array of `T`s."
If we passed in an array of numbers, we'd get an array of numbers back out, as `T` would bind to `number`.
This allows us to use our generic type variable `T` as part of the types we're working with, rather than the whole type, giving us greater flexibility.

We can alternatively write the sample example this way:

```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

You may already be familiar with this style of type from other languages.
In the next section, we'll cover how you can create your own generic types like `Array<T>`.

# Generic Types

In previous sections, we created generic identity functions that worked over a range of types.
In this section, we'll explore the type of the functions themselves and how to create <span class='definition'>generic interfaces</span>.

The <span class='definition'>type of generic functions</span> is just like those of non-generic functions, with the type parameters listed first, similarly to function declarations:

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```

We could also have used a different name for the generic type parameter in the type, so long as the number of type variables and how the type variables are used line up.

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <U>(arg: U) => U = identity;
```

We can also write the generic type as a <span class='definition'>call signature of an object literal type</span>:

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: {<T>(arg: T): T} = identity;
```

Which leads us to writing our first <span class='definition'>generic interface</span>.
Let's take the object literal from the previous example and move it to an interface:

```ts
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

In a similar example, we may want to <span class='definition'>move the generic parameter to be a parameter of the whole interface</span>.
This lets us see what type(s) we're generic over (e.g. `Dictionary<string>` rather than just `Dictionary`).
This makes <span class='important'>the type parameter visible to all the other members of the interface</span>.

```ts
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

Notice that our example has changed to be something slightly different.
<span class='important'>Instead of describing a generic function, we now have a non-generic function signature that is a part of a generic type</span>.
When we use `GenericIdentityFn`, we now will also <span class='important'>need to specify the corresponding type argument</span> (here: `number`), effectively locking in what the underlying call signature will use.
Understanding when to put the type parameter directly on the call signature and when to put it on the interface itself will be helpful in describing what aspects of a type are generic.

In addition to generic interfaces, we can also create generic classes.
Note that it is <span class='definition'>not possible to create generic enums and namespaces</span>.

# Generic Classes

A generic class has a similar shape to a generic interface.
Generic classes have a <span class='definition'>generic type parameter list</span> in angle brackets (`<>`) following the name of the class.

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

This is a pretty literal use of the `GenericNumber` class, but you may have noticed that nothing is restricting it to only use the `number` type.
We could have instead used `string` or even more complex objects.

```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

Just as with interface, putting the type parameter on the class itself lets us make sure all of the properties of the class are working with the same type.

As we covered in [our section on classes](./Classes.md), a class has two sides to its type: the <span class='definition'>static side</span> and the <span class='definition'>instance side</span>.
Generic classes are only generic over their instance side rather than their static side, so when working with classes, <span class='definition'>static members can not use the class's type parameter</span>.

# Generic Constraints

If you remember from an earlier example, you may sometimes want to write a generic function that works on a set of types where you have some knowledge about what capabilities that set of types will have.
In our `loggingIdentity` example, we wanted to be able to access the `.length` property of `arg`, but the compiler could not prove that every type had a `.length` property, so it warns us that we can't make this assumption.

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

Instead of working with any and all types, we'd like to constrain this function to work with any and all types that also have the `.length` property.
As long as the type has this member, we'll allow it, but it's required to have at least this member.
To do so, we must <span class='definition'>list our requirement as a constraint on what T can be</span>.

To do so, we'll <span class='important'>create an interface that describes our constraint</span>.
Here, we'll create an interface that has a single `.length` property and then we'll use this interface and the `extends` keyword to denote our constraint:

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

Because the <span class='important'>generic function is now constrained</span>, it will no longer work over any and all types:

```ts
loggingIdentity(3);  // Error, number doesn't have a .length property
```

Instead, we need to pass in values whose type has all the required properties:

```ts
loggingIdentity({length: 10, value: 3});
```

## Using Type Parameters in Generic Constraints

You can declare a type parameter that is constrained by another type parameter.
For example, here we'd like to get a property from an object given its name.
We'd like to ensure that we're not accidentally grabbing a <span class='definition'>property that does not exist on the `obj`</span>, so we'll place a constraint between the two types:
```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // okay
getProperty(x, "m"); // error: Argument of type 'm' isn't assignable to 'a' | 'b' | 'c' | 'd'.
```

<span class='comment' data-comment='`K extends keyof T` tells that the value of type K has to be an existing key in the object of the type T'></span>

## Using Class Types in Generics

When <span class='definition'>creating factories in TypeScript using generics</span>, it is necessary to <span class='important'>refer to <span class='definition'>class types</span> by their constructor functions</span>. For example,

```ts
function create<T>(c: {new(): T; }): T {
    return new c();
}
```
<span class='comment' data-comment='`c: {new(): T; }` says that c is a parametar of a constructor type that constructs an object of the type T'></span>
A more advanced example uses the <span class='definition'>prototype property</span> to <span class='comment' data-comment='meaning that the instance A that the createInstance function returns is of the type Animal but not just that, but THE SAME type as the type passed as an parameter to the method, and therefore the compiler can guess what type of keeper we are referring to'>infer and constrain relationships between the constructor function and the instance side of class types</span>.

```ts
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function createInstance<A extends Animal>(c: new () => A): A {
    return new c();
}

createInstance(Lion).keeper.nametag;  // typechecks!
createInstance(Bee).keeper.hasMask;   // typechecks!
```
