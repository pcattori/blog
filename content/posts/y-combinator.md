---
title: "Inventing the Y-Combinator"
date: 2020-04-28T13:59:48-04:00
lastmod: 2020-07-03T12:00:00-00:00
tags: [Functional Programming, Lambda Calculus]
---

In this post we will derive the [Y-combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator#Y_combinator) from first principles.

In other words, we will invent a reusable way to use recursion in [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus).

To ground our understanding, we will reference the Python implementation of the Fibonacci series as our main example:

```python
# fibonacci
def f(n):
    if n == 0:
        return 1
    if n == 1:
        return 1
    return f(n - 1) + f(n - 2)
```

**Step 1:**

Start with a function {{< katex inline >}}f{{< /katex >}} whose implementation is self-referencing.

If {{< katex inline >}}f{{< /katex >}}'s body references itself once, {{< katex inline >}}f{{< /katex >}} will have a form like:

{{< katex >}}
f_1 = ...f_1...
{{< /katex >}}

For two self-references:

{{< katex >}}
f_2 = ...f_2...f_2...
{{< /katex >}}

Or three:

{{< katex >}}
f_3 = ...f_3...f_3...f_3...
{{< /katex >}}

etc..

Here we use {{< katex inline >}}...{{< /katex >}} to denote any lambda calculus expression.

For our example, we have two self-references (`fib(n - 1)` and `fib(n - 2)`), but our analysis will hold for any number of self-references.

{{< katex >}}
f = ...f...f...
{{< /katex >}}

**Step 2:**

Instead of hardcoding {{< katex inline >}}f{{< /katex >}} in the body, let's factor out {{< katex inline >}}f{{< /katex >}}:

{{< katex >}}
f = (\lambda r ...r...r...)f = Mf
{{< /katex >}}

where we define {{< katex inline >}}M := \lambda r ...r...r...{{< /katex >}}

Logically, {{< katex inline >}}...{{< /katex >}}s encode all the non-recursive logic.

For our Python example, the non-recursive logic look like:

```python
# non-recursive
def M(r, n):
    if n == 0:
        return 1
    if n == 1:
        return 1
    return r(n - 1) + r(n - 2)
```

Notice that this function is no longer recursive; there are no calls to `M` in its body.

We've made progress: {{< katex inline >}}M{{< /katex >}} does not depend on {{< katex inline >}}f{{< /katex >}}!

But we still depend on {{< katex inline >}}f{{< /katex >}} for it's own definition:

{{< katex >}}
f = (\lambda r ...r...r...)f = Mf
{{< /katex >}}

Can we construct an input for {{< katex inline >}}M{{< /katex >}} that does not depend on {{< katex inline >}}f{{< /katex >}}?

**Step 3:**

Let's try using {{< katex inline >}}M{{< /katex >}} in place of {{< katex inline >}}f{{< /katex >}}:

{{< katex >}}
f \stackrel{?}{=} (\lambda r ...r...r...)(\lambda r ...r...r...) = MM
{{< /katex >}}

This may seem like a shot in the dark. And it is!

We don't expect this to work, but exploring this construction will give us insight to what the answer should be.

{{< katex inline >}}f = MM{{< /katex >}} implies {{< katex inline >}}f = M{{< /katex >}} (because {{< katex inline >}}f = Mf{{< /katex >}}).
How far is this from the truth?

Looking at our Python examples, we know that `f` is not the same function as `M`, but there is a lot of overlap!

So {{< katex inline >}}f = Mf \ne MM {{< /katex >}}, but could we tweak {{< katex inline >}}M{{< /katex >}} to make this work?

**Step 4:**

What if {{< katex inline >}}M{{< /katex >}} didn't take {{< katex inline >}}f{{< /katex >}} as a variable, but expected itself as a variable?!

In other words, let's work backward from what we want: define {{< katex inline >}}M'{{< /katex >}} such that {{< katex inline >}}f = M'M'{{< /katex >}} .

{{< katex >}}
f = M'M'
{{< /katex >}}

Then, to create {{< katex inline >}}f{{< /katex >}}, we can just apply that variable in {{< katex inline >}}M'{{< /katex >}} to itself!

{{< katex >}}
f = (\lambda r (rr)) M'
{{< /katex >}}

Can we solve for {{< katex inline >}}M'{{< /katex >}} and use it to define {{< katex inline >}}f{{< /katex >}}? Yes!

{{< katex >}}
f = (\lambda r ...(rr)...(rr)...)(\lambda r ...(rr)...(rr)...) = M'M'
{{< /katex >}}

**Step 5:**

Cleaning up, we can express {{< katex inline >}}M'{{< /katex >}} in terms of {{< katex inline >}}M{{< /katex >}}:

{{< katex >}}
M' = \lambda r ...(rr)...(rr)...
{{< /katex >}}

{{< katex >}}
M = \lambda r ...r...r...
{{< /katex >}}

{{< katex >}}
\therefore M' = \lambda x.M(xx)
{{< /katex >}}

**Step 6:**

Finally, given {{< katex inline >}}M{{< /katex >}} we can define {{< katex inline >}}f{{< /katex >}}:

{{< katex >}}
f = \lambda m.(\lambda x.m(xx))(\lambda x.m(xx))
{{< /katex >}}

Philosophically, the variable {{< katex inline >}}m{{< /katex >}} encodes all interesting parts of {{< katex inline >}}f{{< /katex >}}'s implementation.

The rest of this expression just plumbs through the recursion.
