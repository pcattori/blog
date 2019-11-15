---
title: "Composable Behavior with React Hooks"
date: 2019-11-14T21:07:19-05:00
tags: [React]
---

You don't have to spend much time developing [React](https://reactjs.org/) code to come across the old ["controlled vs uncontrolled components"](https://reactjs.org/docs/uncontrolled-components.html) debate. :atom:

And I think understanding composable behavior may be the Holy Grail. :trophy:

## Team #uncontrolled vs Team #controlled

The quintessential example is the humble `<input/>`.

An *uncontrolled* `<input/>` does the obvious thing out-of-the-box. Namely, as a user types, the characters they are typing appear in the text box.

```jsx
// UncontrolledInput.jsx
import React, { useState } from 'react'

const UncontrolledInput = () => {
  const [value, setValue] = useState('')
  return (
    <input
      type="text"
      value={value}
      onChange={e => setValue(e.target.value)}
    />
  )
}

export default ControlledInput

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

An *controlled* `<input/>` needs to be wired up, but gives you the flexibility to have the `<input/>` react to other events. For example, if the `<input/>` is being used as a search box, you could wire up your app so that clicking a "Clear filters" button also wipes out the text in the `<input/>`.

```jsx
// ControlledInput.jsx
import React from 'react'

const ControlledInput = ({ value, onChange }) => {
  return (
    <input
      type="text"
      value={value}
      onChange={onChange}
    />
  )
}

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

The difference boils down to the control given to the user.
Uncontrolled components say "Don't worry, I've got this for you." whereas controlled components say "You say jump, I say how high!".
Ease of use **vs** flexibility. Choose one.

**But this is a false dichotomy.**
I want both.
And now that we have React Hooks, there's a way...

## Composable behavior

Behold!

```jsx
// Input.jsx
import React, { useState } from 'react'

// basic behavior that users can easily opt-in to
export function useOnChange(startingText='') {
  const [value, setValue] = useState(startingText)
  const onChange = e => setValue(e.target.value)
  // returns a POJO in the shape of the component props
  return { value, onChange }
}

// Note: this is the same implementation as ControlledInput
const Input = ({ value, onChange }) => {
  return (
    <input
      type="text"
      value={value}
      onChange={onChange}
    />
  )
}

// App.jsx
import React, { useState } from 'react'

import Input, { useOnChange } from './Input'

const App = () => {
  // basic behavior
  const defaultInputProps = useOnChange()

  // complex custom behavior
  // only allow numbers written to the input
  // also, allow button to clear this input
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
      <Input {...defaultInputProps} />

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

This is the power of composable behavior in React.
Like the power to [compose components](https://twitter.com/dan_abramov/status/1021850499618955272)[^1], this pattern let's you skip tedious wiring logic for both the controlled and uncontrolled case.[^2]
[^1]: See [this article](https://varun.ca/flattening-deep-hierarchies-of-components/) for more on composing components
[^2]: *Technically*, I suppose `const defaultInputProps = useOnChange()` and `{...defaultInputProps}` **is** wiring logic, but c'mon its such a small price to pay to get the best of both worlds.

## FP?

This isn't a new idea.
Composing behavior is the central tenet of [Functional Programming](https://en.wikipedia.org/wiki/Functional_programming), though in that context we'd say ["Functions can be passed as arguments to other functions"](https://en.wikipedia.org/wiki/Functional_programming#First-class_and_higher-order_functions).

It makes me happy to see [React borrowing from FP](https://overreacted.io/algebraic-effects-for-the-rest-of-us/#a-note-on-purity) more and more where it makes sense.
No need to reinvent the wheel for composable behavior.

Ship functions (as hooks) alongside stateless, controlled components and let the user compose their symphony. :rocket:
