---
title: "Composable Behavior with React Hooks"
date: 2019-11-14T21:07:19-05:00
lastmod: 2020-02-15T12:45:00-00:00
tags: [React]
---

When I first learned [React](https://reactjs.org/) :atom:, it took me less than a day to run into the ol' ["controlled vs uncontrolled components"](https://reactjs.org/docs/uncontrolled-components.html) dichotomy.
At the time, I didn't know the terms "controlled" and "uncontrolled", but was still struggling to fit my designs into these two buckets.

The tension was between 2 competing goals:

1. I want to make my components as **reusable** as possible
2. My use-cases were *similar*, but **not exactly the same**...

Here's a checkbox component that takes care of styling and toggling when clicked.
It used it as-is in a couple places, but then kept running into cases where I wanted more fine-grained control.
E.g. I was using some checkboxes as filters and I wanted my *Clear filters* button to uncheck all the filter checkboxes.

```jsx
import React, { Component } from 'react'

const styles = {...}

class Checkbox extends Component {
  state = { checked: false }

  toggle = () => this.setState({ checked: !this.state.checked })

  render() {
    return (
      <input
        type='checkbox'
        checked={this.state.checked}
        onChange={this.toggle}
        style={styles}
      />
    )
  }
}
```

The tension becomes apparent as I kept adding more and more props to my components to configure them to deal with the variety of use-cases.
It looked ugly and quickly became unmaintainable.

Maybe writing *truly* reusable UI components is a fool's errand.
I gave up on finding a better way and settled into writing mostly controlled components wired to Redux.
But every time I wrote Redux code just to enable obvious, default behavior (e.g. "toggling a checkbox", "writing characters into a text input"), I longed again for a better way.

My persistence to find a better way was tied up with my ego as a developer.
I took pride in writing clean, reusable code in scripts and backend code.
But here I was wrestling with a humble checkbox...
Surely, any great software engineer should be able to write reusable UI components, no?

---

A couple weeks ago I had 2 epiphanies on this subject:

1. From a design perspective, making everything reusable means you **don't understand the problem you are trying to solve**.
2. The "controlled" vs "uncontrolled" is a **false dichotomy**.

I realized (1) after reading the first couple chapters from [Eric Evan's "Domain-Driven Design"](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215).[^1]
[^1]: I highly recommend the first couple chapters, but then watch [Domain-Driven Design Made Functional](https://www.youtube.com/watch?v=Up7LcbGZFuo)

In the rest of this post, I want to focus on how understanding and applying "composable behavior" frees us from the shackles of (2). It is not only possible to make reusable components that can adapt to many use-cases, it's *a joy* to write and use them.

## Case study: reusable input

First things first: I'm going to use [React Hooks](https://reactjs.org/docs/hooks-intro.html) from here on out instead of `state` and `setState`.
Hooks will keep our code focused on solving our problem rather than writing boilerplate code.

For brevity, the examples will use [inline styles](https://reactjs.org/docs/dom-elements.html#style), but I'd recommend using CSS-in-JS solutions like [emotion](https://emotion.sh/docs/introduction) instead.

With that out of the way, let's look at making a reusable `Input.jsx` component.

---

An *uncontrolled* `<input/>` does the obvious thing out-of-the-box. Namely, as a user types, the characters they are typing appear in the text box.

```jsx
// UncontrolledInput.jsx
import React, { useState } from 'react'

styles = {...}

const UncontrolledInput = () => {
  const [value, setValue] = useState('')
  return (
    <input
      type="text"
      value={value}
      onChange={e => setValue(e.target.value)}
      style={styles}
    />
  )
}

export default UncontrolledInput

// App.jsx
import React from 'react'

import UncontrolledInput from './UncontrolledInput'

const App = () => {
  return (
    <div>
      <h1>My App</h1>
      <UncontrolledInput />
    </div>
  )
}
```

---

A *controlled* `<input/>` needs to be wired up, but gets flexibility to react to other events. For example, if the `<input/>` is used as a search box, you could wire up your app so that clicking a **Clear filters** button also wipes out the text in the `<input/>`.

```jsx
// ControlledInput.jsx
import React from 'react'

styles = {...}

const ControlledInput = ({ value, onChange }) => {
  return (
    <input
      type="text"
      value={value}
      onChange={onChange}
      style={styles}
    />
  )
}

export default ControlledInput

// App.jsx
import React, { useState } from 'react'

import ControlledInput from './ControlledInput'

const App = () => {
  const [value, setValue] = useState('')
  return (
    <div>
      <h1>My App</h1>
      <ControlledInput
        value={value}
        onChange={e => setValue(e.target.value)}
      />
      <button onClick={() => setValue('')}>Clear</button>
    </div>
  )
}
```

---

To unify `UncontrolledInput.jsx` and `ControlledInput.jsx`, we need to let the user tells us when to do the default behavior and when to do something custom.
The key insight will be for the user to **explicitly opt-in to default behavior**.

## Composable behavior

Let's decouple our rendering and behavior.

```jsx
// Rendering
import React from 'react'

const Input = ({ value, onChange }) => {
  return (
    <input
      type="text"
      value={value}
      onChange={onChange}
    />
  )
}

export default Input
```

You'll notice that our rendering code is now the same as `ControlledInput.jsx`.
This is great!
Writing our components as pure functions keeps everything ultra configurable and adaptable.
Now the only challenge is shipping some default behavior that the user can easily, ergonomically opt-in to.

```jsx
// Behavior
import { useState } from 'react'

export function useDefaultBehavior(startingText='') {
  const [value, setValue] = useState(startingText)
  const onChange = e => setValue(e.target.value)
  return { value, onChange }
}
```

The design here is to create a hook that returns a JS object in the same shape as our component props and delivers the default behavior for our component.
(Note that React hooks must have [names starting with "use"](https://reactjs.org/docs/hooks-custom.html#extracting-a-custom-hook).)

Now we can just combine these 2 into 1 file:

```jsx
// Input.jsx
import React, { useState } from 'react'

export function useDefaultBehavior(startingText='') {
  const [value, setValue] = useState(startingText)
  const onChange = e => setValue(e.target.value)
  return { value, onChange }
}

const Input = ({ value, onChange }) => {
  return (
    <input
      type="text"
      value={value}
      onChange={onChange}
    />
  )
}

export default Input
```

And with that, usage is as simple as:

```jsx
// App.jsx
import React, { useState } from 'react'

import Input, { useDefaultBehavior } from './Input'

const App = () => {
  const inputDefaults = useDefaultBehavior()

  return (
    <div>
      <h1>My App</h1>
      {/* opt-in to basic behavior by spreading the props */}
      <Input {...inputDefaults} />
    </div>
  )
}
```

We let the user **compose** the behavior with the component themselves.
This tiny bit of wiring is still ergonomic and is a tiny price to pay for adaptability.

Let's see what that adaptability looks like by trying to create another `Input` with custom behavior right next to our current, default-behavior `Input`:

```jsx
// App.jsx
import React, { useState } from 'react'

import Input, { useDefaultBehavior } from './Input'

const App = () => {
  const inputDefaults = useDefaultBehavior()

  // only allow numbers written to the second input
  // also, allow button to clear the second input
  const [value, setValue] = useState('')
  const onlyNumbers = value => {
    if (value === '' || isNaN(value)) {
      // value could not be parsed as a number. ignore
      return
    }
    setValue(value)
  }

  return (
    <div>
      <h1>My App</h1>
      {/* opt-in to basic behavior by spreading the props */}
      <Input {...inputDefaults} />

      {/*
        wire up to complex behavior
        by treating <Input/> like controlled component
      */}
      <Input
        value={value}
        onChange={e => onlyNumbers(e.target.value)}
      />
      <button onClick={() => setValue('')}>Clear</button>
    </div>
  )
}
```

Behold the power of composable behavior!

Like the power to [compose components](https://twitter.com/dan_abramov/status/1021850499618955272)[^2], this pattern let's you ergonomically choose between built-in behaviors and custom ones to fit the needs of every use-case in your app.
[^2]: See [this article](https://varun.ca/flattening-deep-hierarchies-of-components/) for more on composing components

**Nota Bene**: we've been asserting that there is **1** obvious, default behavior, but notice that this pattern scales gracefully.
There is nothing preventing us from writing multiple hooks for multiple pre-packaged behaviors.

## Off the hook

Finally, we can decouple our behavior from hooks (and any 1 particular state management e.g. Redux, React Context, etc...) by shipping behavior as pure functions too!

I recommend writing a function that accepts the current state and produces the next state.[^3]
[^3]: Similar to a Redux reducer, but not *quite*. Reducers accept `(state, action)` whereas here we accept only `(state)` since the `action` is encoded as the particular function (e.g. `toggle`)


```jsx
// Checkbox.js

// pure function! state -> state
export function toggle(state) {
  return !state
}

// as a hook for convenience
export function useToggle(start=False) => {
    const [value, setValue] = useState(start)
    const onChange = () => setValue(toggle(value))
    return { value, onChange }
  }
}
```

For example, now we can use this behavior in Redux:

```javascript
// reducer.js
import { toggle } from './Checkbox'

function reducer(state, action) {
  switch(action.type) {
    case TOGGLE:
      return toggle(state)
    // other actions omitted
  }
}

export default reducer
```

If this example seems contrived, it is.
`toggle` is too simple a behavior to warrant a shareable function, but imagine cases where computing the next state is not trivial.


## Coupling under your control

You might argue:
> But if I know the default behavior for something, why not ship with an uncontrolled component that includes that behavior?
> I don't want to rewire the same, default behavior over and over.
> That's too tedious...

If you find yourself constantly wiring the same behavior to the same component, you've discovered something about your use-case!
Maybe that coupling *should* be promoted to be a concept unto itself, and then yes, you can pre-couple that behavior and rendering into a stateful component for reuse.

But the point is that *you* are in control of that, its not a pre-coupling that happened somewhere that you can't reach.
As a user of a library, its much easier to create a new coupling (via composition) than it is to detangle part of the library (if you've ever had to wrangle `ref`s and callbacks you know what I mean).

## FP?

This isn't a new idea.
Composing behavior is the central tenet of [Functional Programming](https://en.wikipedia.org/wiki/Functional_programming), though in that context we'd say ["Functions can be passed as arguments to other functions"](https://en.wikipedia.org/wiki/Functional_programming#First-class_and_higher-order_functions).

It makes me happy to see [React borrowing from FP](https://overreacted.io/algebraic-effects-for-the-rest-of-us/#a-note-on-purity) more and more where it makes sense.
No need to reinvent the wheel for composable behavior.

Ship behavior (as functions/hooks) alongside stateless, controlled components and let the user compose their symphony. :rocket:
