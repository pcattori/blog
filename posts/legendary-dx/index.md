# Talk: Legendary DX

Keeping developers in the flow is critical.
Techniques like live reload and Hot Module Replacement were invented for this purpose.
In this talk, I introduce the next step in web DX:

🔥 **Hot Data Revalidation** 🔥

[![Me standing on the stage at Remix Conf 2023](./images/thumbnail.webp)](https://www.youtube.com/watch?v=79M4vYZi-po)

You can also [check out the slides](https://slides.com/pcattori/legendary-dx).

## Clarifications

1. I think my acting might have been too good 😅
   I am purposefully misleading the crowd into thinking that `loader` changes are impossible to handle gracefully.
   I _think_ most people got the joke, but I could have hammered it home a bit more.

2. I forget to add a `key` to the `<input />` element, causing `whatsup` to be erased during HDR.
   With the `key`, React can track the elements and persist React state more reliably.
   Remix uses the same method as [Vite](https://vitejs.dev/) for handling React HMR updates: [React Refresh](https://github.com/facebook/react/issues/16604#issuecomment-528663101)
   So you can see for yourself that omitting the `key` will cause `<input />` state to be lost with Remix, Vite, or any other tool that uses React Refresh.

3. If hooks are added or removed from a component, React considers this to be a _different_ component.
   That means React Refresh will lose state for that component when processing updates with hook changes.
   Again, you can try this out for yourself with Remix or Vite.

4. HDR detects changes _anywhere_ in your code that affect each `loader`.
   It could be changes to the `loader` function itself, or to any code dependencies of the `loader`.

5. After the HDR demo, I talk about calling the dev tools from the app server.
   I should have first stated that to solve the mismatch problem, we need to (i) ensure the app server is running up-to-date server code, (ii) ask for updated data from the app server, and (iii) only process UI changes via HMR _after_ we have that up-to-date data.
   The trivial way to coordinate code is to run them in the same process and call them in order.
   However, running dev tools in the same Node process as the app server has drawbacks.
   In the presentation, I mention this all in passing and jump straight into the drawbacks.

I cover points (2) and (3) in more detail in a [separate video](https://www.youtube.com/watch?v=S_84Ty2sFfM).

I also did a [livestream with the reputable Kent C. Dodds](https://www.youtube.com/watch?v=IjE18rXpp9Q) where we setup HMR/HDR in some of his projects.
At the time, the API was unstable, so things have probably changed since then, but the concepts remain the same.
