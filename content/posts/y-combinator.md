---
title: "Inventing the Y-Combinator"
date: 2020-04-28T13:59:48-04:00
tags: [Functional Programming, Lambda Calculus]
---

In this post we will derive the [Y-combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator#Y_combinator) from first principles.

In other words, we will invent a reusable way to use recursion in [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus).

**Step 1:**

Start with a function {{< katex inline >}}f{{< /katex >}} whose implementation is self-referencing.
Then the body of {{< katex inline >}}f{{< /katex >}} must look like:

{{< katex >}}
f = ...f...f...
{{< /katex >}}

Here we use {{< katex inline >}}...{{< /katex >}} to denote any lambda calculus expression.

**Step 2:**

Instead of hardcoding {{< katex inline >}}f{{< /katex >}} in the body, let's factor out {{< katex inline >}}f{{< /katex >}}:

{{< katex >}}
f = (\lambda r ...r...r...)f = Mf
{{< /katex >}}

where we define {{< katex inline >}}M := \lambda r ...r...r...{{< /katex >}}

Logically, {{< katex inline >}}M{{< /katex >}} encodes how {{< katex inline >}}f{{< /katex >}} should be reused within the implementation of {{< katex inline >}}f{{< /katex >}}.

We still have {{< katex inline >}}f{{< /katex >}} on the right-hand side (RHS)...

**Step 3:**

If we squint, {{< katex inline >}}M \approx f{{< /katex >}}, so let's try {{< katex inline >}}f \stackrel{?}{=} MM{{< /katex >}} to remove {{< katex inline >}}f{{< /katex >}} from the RHS and see how close we get:

{{< katex >}}
f \stackrel{?}{=} (\lambda r ...r...r...)(\lambda r ...r...r...) = MM
{{< /katex >}}

Well, we don't depend on {{< katex inline >}}f{{< /katex >}} on the RHS anymore, but we know {{< katex inline >}}f = Mf \ne MM {{< /katex >}}. Close, but no cigar.

**Step 4:**

This current attempt wants both {{< katex inline >}}M = f{{< /katex >}} and {{< katex inline >}}f = MM{{< /katex >}}

Hmm, what if M didn't take f as a variable, but expected itself as a variable.
So we define {{< katex inline >}}M'{{< /katex >}} such that {{< katex inline >}}f = M'M'{{< /katex >}}
Then, to create {{< katex inline >}}f{{< /katex >}}, we can just apply that variable in {{< katex inline >}}M'{{< /katex >}} to itself!

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
