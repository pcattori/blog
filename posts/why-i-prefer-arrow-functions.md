# Why I prefer arrow functions

tl;dr: 
- `=>` connotes "this transforms inputs to output"
- assigning to variable creates an affordance for treating functions like any other value
- Typescript function types can't be used with named `function`s
- I never need hoisting, and [I don't think you do either](#hoisting)
- `this` is a footgun that is present in `function`s, but not in arrow functions

## Designing your own function syntax

Imagine that you were in charge of designing the syntax for function in Javascript.
You can do whatever you want.

The only things that exist so far are:
- primitive values like `number`s, `boolean`s, `string`s, etc..
- objects and arrays
- a way to assign values to variables
- standard math operators like `+`, `-`, `>`, etc..

```ts
let a = 1
let b = 'hello world'
let c = [a, b]
let d = { e: 2, f: 'blah' }
let g = d.e > a
```

This is _my_ thought experiment so let's pretend that there's no trailing semicolons and that `var` never existed, so we use `let`.

Ok, now it's up to you!
How would you design function syntax?

## Functions are transformations

Well you didn't invent the _concept_ of functions.
So if you want to design from first principles, it's a good idea to remember that functions come from math.

In math, functions take inputs and produce and output.
You might sketch that in your notebook with an arrow, like:

```txt
Function: input --> output
```

Unlike in math, programming functions can have side-effects and can also blow up with errors, so you'll need to deal with that.[^programming-function]

[^programming-function]: There are some technical names for these things. Functions without side-effects are called _pure_ functions. Programs that produce a valid output for every input are called _total_ functions. In programming, our functions can be _impure_ and _partial_.

Though modeling side-effects[^side-effects] and error handling[^error-handling] are super interesting challenges, let's not innovate on those for this and just say that:

- Yes, functions can throw errors, but we're not going to model that in our function syntax
- Functions can have side-effects, but we're not going to model that in syntax either.

[^side-effects]: If you've ever heard of monads, they are all about explicitly modeling side-effects to keep functions pure.

[^error-handling]: Java _does_ model checked errors in its function signature e.g. `public String Hello(string world) throws Exception`.

## Arrow syntax

Looking at the notebook, the arrow `->` is what you used to connote "transforms".
Perfect, let's use that!

Now if there were [good reasons not to use the `->` characters](https://softwareengineering.stackexchange.com/a/306103), you can fallback to `=>` to represent an arrow.

## Input syntax

Functions can take one or multiple inputs.
So you'll need to pick a delimiter between the inputs.
The simple `,` works well:[^space-delimiter]

```txt
// our syntax so far:
input1, input2, input3 => <output>
```

[^space-delimiter]: Haskell uses a space (` `) to delimit its function inputs. E.g. `add x y = x + y` is the equivalent to `let add = (x,y) => x + y`.

Functions can also take _other functions_ as inputs.
Hmm... you'll need some way to disambiguate cases like:

```txt
animal, bear => cat, dog, eagle => lion => zebra

// Is it?
((animal, bear => cat, dog, eagle) => lion) => zebra

// Or?
(animal, bear => cat, (dog, eagle => lion)) => zebra
```

Well, your intuition already used parens `()` to disambiguate them in our notebook, so let's use those!

```txt
(input1, input2, ...) => <output>
```

## Output syntax

If the input can be transformed in a single expression, why not use that expression as the "output"?

```ts
(world) => "Hello " + world
```

And if you need multiple statements to compute the output, you'd need syntax for a scope.
Let's borrow curly braces `{}` from almost every other programming language.
Those work well and will be familiar to users.
And you can also use `return` within a function scope to denote what the output finally turns out to be:

```ts
(name) => {
  let sanitized = sanitizeUserInput(name)
  let goodWords = censorBadWords(sanitized)
  return goodWords
}
```

## Naming functions

Nice!
You've designed anonymous functions!

Now you just need a way to assign names to functions that need to be reused or referenced elsewhere.

What if you added the name of a function just before the parens?

```ts
hello(world) => 'Hello ' + world
```

That would work.
But do you want that?
What [affordances](https://en.wikipedia.org/wiki/Affordance) does that create for usage?

Ideally, you want something that encourages users to define anonymous functions when needed, but makes it clear that names can be assigned.
And that variables that point to functions can be passed around just like any other value.

And _that's_ the key insight: "just like any other value".

Instead of inventing a new convention for naming _functions_, you could reuse the existing syntax for naming _any_ value.
That way users get a sense that "yes, functions are another type of value".
Namely, it should feel as natural to use functions, say as arguments to another function, as it was to pass in strings or numbers.

```ts
let hello = (world) => 'Hello ' + world
```

## Typescript function types

Ok so you've designed a nice function syntax.
But how does it interop with Typescript?

If you had to design Typescript types for your function syntax, how would you do it?

Leaning on our brain's capacity for pattern matching, why not mirror the function syntax in the syntax for function types?

```ts
// pattern: <function name> = (<inputs>) => <output>
type Hello = (world: string) => string
let hello = (world) => 'Hello ' + world
```

As a benefit of using standard assignment for naming functions, you can reuse how types are bound to variable assignments:

```ts
type Hello = (world: string) => string
let hello: Hello = (world) => 'Hello ' + world
```

## Comparison to `function`

If instead, you designed function syntax like so:

```ts
function hello(name) {
  return `Hello ${name}`
}
```

then, you'd lose out on most of the affordances and design wins we've discussed so far.

## When _not_ to use arrow functions

I hope this little story conveyed my love of arrow functions to you. ❤️

But there are a couple cases where you should use `function`s instead of arrow functions:
- If you're in a rare situation where you actually need `this`
- If you need to write a [generator function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) [^generator-arrow-function]

[^generator-arrow-function]: There's a [TC39 proposal for generator arrow functions](https://github.com/tc39/proposal-generator-arrow-functions). If that's accepted, I'd probably use arrow functions for generators too.

### Hoisting?

Proponents of `function`s tend to use "hoisting" as one of the main reasons not to use arrow functions:

```ts
// this will fail since `hello` is called before it is defined
hello("friends!")
const hello = (world: string) => 'Hello ' + world

// Javascript will "hoist" the function declaration
// as if `function salutations` was on the first line
// so this will run just fine
salutations("pals!")
function salutations = (earth: string) => 'Salutations ' + earth
```

But I never run into this.
I don't expect to be able to use variables before they are assigned, whether the value is a number, string, or function.

Just like any other value, if I want to use it on a line above where it's defined I use scopes:

```ts
export const doingStuff = () => {
  hello("friends!")
}
const hello = (world: string) => 'Hello ' + world
```

I don't conciously think to do this, it just happens because I don't like using the outermost scope of a module so that I can avoid having side-effects for importing my modules.

And for files that are simple scripts that I want to run, I just wrap statements in a `main` function and call `main` at the end:

```ts
const main = () => {
  hello("friends!")
}
const hello = (world: string) => 'Hello ' + world

main()
```

In fact, [`function` hoisting can make things harder to understand](https://twitter.com/wesbos/status/1601312464485892097).

For me, it's extremely rare to run into hoisting issues and when I do they're easily solved by the strategies above.
Definitely worth it to keep using the beautiful affordances of arrow functions.
