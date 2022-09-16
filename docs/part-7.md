---
id: part-7
title: Part 7. Functional Programming Introduction
---

# Functional Programming

Contributors:

- Sergi Dote

## Introduction

Functional programming is a paradigm that puts the attention on functions. But this definition can be a little confusing since developers are coding using functions and methods and this doesn't mean they are programming following the FP paradigm.

FP talks about abstract control flows, data mapping, operations, immutability, and other concepts we are reviewing in the next steps.

Change our mind to a FP perspective takes a while.

## What really FP is?

Functional programming comes from mathematical logic, and λ-calculus. The λ-calculus helps us to describe programs as data transformations using function applications.

FP teachs us to decompose a problem into simple functions and chain them to get a final result. Thinking like this, we can break the complexity of a program, and achieve any problem as a set of easy steps.

In my opinion, a perfect description is what Michael Feathers said:

*“OO makes code understandable by encapsulating moving parts. FP makes code understandable by minimizing moving parts”.*

### FP is declarative

There are two programming paradigms:

**Imperative:**
It determines HOW TO DO things, describing step by step what we need to do to get our result. Each instruction changes the state.

**Declarative:**
It determines WHAT TO DO, not HOW.

Under this paradigm we define which operations are involved but we don't specify how to do in the low-level (loops, conditions, assignments, ...).

Functional programming is declarative, it defines operations and data flows without any strict control flow definition.

## FP concepts

We will list the main features below:

### Immutability

Immutable data cannot be modified after its creation. When mutating data we are producing side effects, and we cannot ensure the program correctness.

Immutability is a core concept of Functional Programming, without it, you cannot ensure the data flow is working properly, getting a buggy program.

### Pure functions and side effects

With FP our code becomes immutable, using pure functions. Pure functions don’t manipulate outer variables or input parameters: **Same inputs produce same outputs.**

Pure functions don’t have side effects, getting a simpler code.

This is a pure function (val parameter is never mutated):

```ts
const increment = (val: number) => val + 1;
```

When using impure functions, we are running into side effects. In this case we cannot predict code alterations, and that produces buggy code. Side effects modify the state of the program or system, resulting an uncontrollable and unreliable code.

This is an impure function:

```ts
function increment(val: number): number {
    val += 1;
    return val;
}
```

### Referential transparency

A function that with the same inputs always produces the same output, is a referentially transparent function.

That means we can replace this kind of functions by their expression without changing the program behavior. This statement is an important pillar providing us correctness and algorithm simplicity and allowing lazy evaluation and memorization.

### Partial application and Currying

Partial application means fixing a number of arguments to a function, producing another function of smaller arity.

The currying concept (created Haskell Curry) is the technique of converting a function that takes multiple arguments into a sequence of functions that each takes a single argument.

For instance, this function: 

```ts
function add(a: number, b: number): number {
    return a + b;
}
```

Can be curryfied like this:

```ts
const add = (a: number) => (b: number) => a + b;
```

Usage:

```ts
const ten = add(5)(5);
```

With currying, we can create new functions from others:

```ts
const add10 = add(10);

const result = add10(5);
```

Defining `add10` function we have a new function that will be executed when all parameters are provided (a second parameter is required).

### Function composition

Pure functions can be composed to get our expected result. Using the function composition technique, any problem can be decomposed into simpler functions with only one responsibility, and be solved chaining these pieces.

With the folloeing example let's see, given a problem, we solve it with imperative code, and after that with FP and declarativelly, using function composition.

Having this interface:

```ts
enum Gender {
  Female = 'Female',
  Male = 'Male',
  Other = 'Other'
}

interface Student {
  name: string;
  lastName: string;
  gender: Gender;
  university: string;
}
```
We want to find all women studing in 'Union College'.

Under an imperative perspective, can be solve like this:

```ts
function getWomenFromUnionCollege(students: Array<Student>) {
  const women = [];

  for (const s of students) {
    if (s.gender === Gender.Female && s.university === 'Union College') {
      women.push(s);
    }
  }

  return women;
}
```

This function has a single responsibility, but is not reusable, and we are controlling what are the program doing to achieve the final result.

One solution following a FP approach, could be this:

```ts
const select =
  <T, K extends keyof T & string = any>(key: keyof T, value: T[K]) =>
  (obj: T) =>
    obj[key] === value;

const isWoman = select<Student>('gender', Gender.Female);

const getWomen = (students: Array<Student>) => students.filter(isWoman);

const fromUnionCollege = select<Student>('university', 'Union College');

const womenFromUnionCollege = getWomen(students).filter(fromUnionCollege);
```

In this case we are telling what we need, but not how to do this:

- We have a generic function (select), that can be used to create another functions.
- Combining functions we get the desired result.

#### Function chaining

Talking about function composition, there is a related concept: function chaining. It is another way to combine function in order to get a final result. An example is RxJS pipe function that allows as to chain operators, creating a data flow that will give us the final value.

Let's create our pipe function for chaining our pure functions: 

```ts
const pipe = (fns) => (x) => fns.reduce((v, f) => f(v), x);
```

Using this previous definition, we can chain functions: 

```ts
const oddNumbers = (numbers: Array<number>) => numbers.filter(_ => _ % 2 !== 0);

const power = (pow: number) => (numbers: Array<number>) => numbers.map(_ => _ ** pow);

 pipe(
    oddNumbers,
    power(2),
  )([1, 2, 3, 4, 5, 6, 7, 8, 9]) // output: [1, 9, 25, 49, 81]
```

Because all involved functions are pure, we can ensure the result has never any side effect.

### High Order Functions

In Functional Programming, a function is a first-class citizen of the language, that means that a function is just another value.

A good example is the Javascript's `filter` function:  

```ts
filter(predicate: (value: T, index: number, array: T[]) => unknown, thisArg?: any): T[];
```

A function that will apply a function given as a parameter.

This is so powerful. Let's see one example: 

```ts
const repeat = (times, funct, initial) =>
  times > 1 ? repeat(times - 1, funct, funct(initial)) : funct(initial);
```

This function will repeat given function N times from an initial value. The good thing is having this function we can do other functions, only passing the function we want to work with:

```ts
const rateMovie = (stars) => repeat(stars, (_) => _ + '*', '');

console.log(rateMovie(5)); // output: *****
```

or:

```ts
const increment = (value) => (times) => repeat(times, (_) => _ + 1, value);

console.log(increment(2)(1)); // output: 3
```