# Early `return`s in Typescript type definitions

Early returns are great.
They're perfect for modeling preconditions and they let us quickly narrow the context of a function.

```ts
function letterGrade(grade: number): string {
  if (grade > 100) throw Error(`Grades must be <= 100, but got: ${grade}`)
  if (grade < 0) throw Error(`Grades must be >= 0, but got: ${grade}`)

  if (grade >= 90) return "A"
  if (grade >= 80) return "B"
  if (grade >= 70) return "C"
  if (grade >= 60) return "D"
  return "F"
}
```

But when defining types in Typescript, there is no `if`/`else`.
[To do conditionals, you have to use ternaries.](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)

So early returns for types in TS aren't possible, right?
Well, actually there is a way!

## Example: JSON serialization

Let's say we want to add types for for [JSON serialization/deserialization](https://www.json.org/json-en.html).
Specifically, let say we have something like:

```ts
function jsonify(input) {
  // Model server->browser data serialization over the network
  let serialized = JSON.stringify(input)
  return JSON.parse(serialized)
}

let input = /* ... */
let deserialized = jsonify(input)
```

and we want to create a type utility that can infer the type for `deserialized` from the type for `input`.

```ts
type Jsonify<T> =
  // if T is an unserializable JS primitive, `jsonify` will throw an error, which we model with `never`
  T extends undefined | Function | symbol ? never :
  // if T is a JSON primitive, then the output should be of the same type
  T extends number | string | boolean | null ? T :
  // if T has a `toJson()` method, whatever that method returns should be the type for the output
  T extends { toJSON(): infer U } ? U :
  // TODO: recursively handle arrays
  // TODO: recursively handle objects
  // anything else, we just say won't ever happen
  never; // equivent to `throw Error("this should never happen")`
```

I left out handling for arrays and objects since there are some [complications](https://github.com/remix-run/remix/blob/b380d5e40806390362d0210ff59dd5bb06ab21ba/packages/remix-server-runtime/serialize.ts#L26-L29) there that are irrelevant for our mission of modeling early returns in Typescript.

But look at that!
Early returns in our Typescript type definitions!

## "Prettier" Formatting

You'll notice that I formatted the ternaries in a peculiar way.
I started each ternary on its own line and included the `:` at the end of that line:

```ts
T extends Thing ? Return :
```

That makes each line more independent.
Just like with [trailing commas](https://blog.logrocket.com/best-practices-using-trailing-commas-javascript/), you can add, remove, and reorder lines without modifying the others.
It's also great for reducing `git` noise when modifying code paths.

But all of those benefits are just bonuses.
The _real_ benefit is the readability. ✨

Compare our manually formatted code to what [prettier](https://prettier.io/) formats it as:

```ts
// manually formatted
type Jsonify1<T> =
  T extends undefined | Function | symbol ? never :
  T extends number | string | boolean | null ? T :
  T extends { toJSON(): infer U } ? U :
  // TODO: recursively handle arrays
  // TODO: recursively handle objects
  never;

// auto formatted by `prettier`
type Jsonify2<T> = T extends undefined | Function | symbol
  ? never
  : T extends number | string | boolean | null
  ? T
  : T extends { toJSON(): infer U }
  ? U
  // TODO recursively handle arrays
  // TODO recursively handle objects
  : never;
```

For the prettier-formatted one, try commenting out the condition that handles JSON primitives in both example.
Then try uncommenting that and commenting the condition that handles JS primitives.
Yuck.

Listen, I _love_ prettier ♥️.
But this is an exceedlingly rare edge case where I think its worth formatting manually:

```ts
// prettier-ignore
type Jsonify<T> =
  T extends undefined | Function | symbol ? never :
  T extends number | string | boolean | null ? T :
  T extends { toJSON(): infer U } ? U :
  // TODO: recursively handle arrays
  // TODO: recursively handle objects
  never;
```

## No such thing as `not extends`

This pattern works by using the `true` result of the ternary as the early return.
With normal `if` statements, if you instead wanted to return the `false` result, you can simply negate the condition with `!`:

```ts
if (!condition) {
  return falseConditionResult
}
```

Unfortunately, to my knowledge, there's no way to negate an `extends` condition in TS. ☹️