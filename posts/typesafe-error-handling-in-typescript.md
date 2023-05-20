# Typesafe error handling in Typescript

When using `try`/`catch`, Typescript will infer the thrown type to be `unknown`:

```ts
try {
  // do some stuff
} catch (thrown) {
  //     ^? unknown
}
```

Normally, you'd expect an `Error` to be thrown, but Javascript lets you throw _anything_:

```ts
throw { type: "not an error" };
```

Inferring as `unknown` is a nice reminder of that.
Thanks Typescript!

But handling `unknown` is annoying.
You can't access things like `message` or `stack` that you'd expect from an `Error` since they might not be there.

```ts
try {
  // do some stuff
} catch (thrown) {
  console.error(thrown.message);
  //                   ^^^^^^^
  // Typecheck error: `unknown` does not have property `message`
}
```

Luckily, I stumbled on a [post by Kent C. Dodds for typesafe error messages][kcd-error-message].
But, I didn't want just the error message.
I wanted the error _itself_.
That way I could format the stacktrace or filter the error based on its name.

Here's my way of getting typesafety while handling errors.
First define the `asError` utility:

```ts
let asError = (thrown: unknown): Error => {
  if (thrown instanceof Error) return thrown;
  try {
    return new Error(JSON.stringify(thrown));
  } catch {
    // fallback in case there's an error stringifying.
    // for example, due to circular references.
    return new Error(String(thrown));
  }
};
```

Then use `asError` to narrow the `thrown` type to `Error`:

```ts
try {
  // do some stuff
} catch (thrown) {
  //     ^? unknown
  let error = asError(thrown);
  //  ^? Error

  // access `name`, `message`, and `stack`
  console.error(`${error.name}: ${error.message}`);
  if (error.stack) console.error(error.stack);
}
```

This coercion of `unknown` to `Error` is blunt, so if you need to handle non-`Error`s more delicately, feel free to modify `asError`.
I stick to using `Error`s whenever I'm throwing, but need a small patch like `asError` to handle the rare non-`Error` being thrown.

## Custom errors

If you have a custom error type that is more specific than `Error`, you can further narrow the type with a typeguard:

```ts
class CustomError extends Error {
  data: Record<string, unknown>;
  constructor(message: string, data: Record<string, unknown>) {
    super(message);
    this.data = data;
  }
}

// typeguard for CustomError
let isCustomError = (error: Error) error is CustomError => {
  return "data" in error && typeof error.data === 'object'
}

try {
  // do some stuff
} catch (thrown) {

  let error = asError(thrown)
  if (isCustomError(error)) {
    console.error(error.data);
    //            ^? CustomError
    return
  }

  throw thrown
}
```

[kcd-error-message]: https://kentcdodds.com/blog/get-a-catch-block-error-message-with-typescript
