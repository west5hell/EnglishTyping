# useEffect

`useEffect` is a React Hook that lets you synchronize a component with an external system.

```js
useEffect(setup, dependencies?)
```

## Reference

`useEffect(setup, dependencies?)`

Call `useEffect` at the top level of your component to declare an Effect:

```js
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

### Parameters

- `setup`: The function with your Effect's logic. Your setup function may also optionally return a _cleanup_ function. When your component is added to the DOM, React will run your setup function. After every re-render with changed dependencies, React will first run the cleanup function (if you proviede it) with the old values, and then run your setup function with the new values. After your component is removed from the DOM, React will run your cleanup function.
- **optional** `dependencies`: The list of all reactive values referenced inside of the `setup` code. Reactive values include props, state, and all the variables and functions declared directly inside your component body. If your linter is [configured for React](https://react.dev/learn/editor-setup#linting), it will verify that every reactive value is correctly specified as a dependency. The list of dependencies must have a constant number of items and be written inline like `[dep1, dep2, dep3]`. React will compare each dependency with its previous value using the `Object.is` comparison. If you omit this argument, your Effect will re-run after every re-render of component. [See the difference between passing an array of dependencies, an empty array, and no dependencies at all.](https://react.dev/reference/react/useEffect#examples-dependencies)

### Returns

`useEffect` returns `undefined`.

### Caveats

- `useEffect` is a Hook, so you can only call it **at the top level of your component** or your own Hooks. You cant't call it inside loops or conditions. If you need that, extract a new component and move the state into it.
- If you're **not trying to synchronize with some external system**, [you probably don't need an Effect](https://react.dev/learn/you-might-not-need-an-effect).
- When Strict Mode is on, React will **run one extra development-only setup+cleanup cycle** before the first read setup. This is a stress-test that ensures that your cleanup logic "mirrors" your setup logic and that is stops or undos whatever the setup is doing. If this causes a problem, [implement the cleanup function](https://react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development).
- If some of your dependencies are objects or functions defined inside the component, there is a risk that they will **cause the Effect to re-run more often that needed**. To fix this, remove unnecessary object and function dependencies. You can also extract state updates and non-reactive logic outside of your Effect.
- If your Effect wasn't caused by an interaction (like a click), React will generally let the browser **paint the updated screen first before running your Effect**. If your Effect is doing something visual (for example, positioning a tooltip), and the delay is noticeable (for example, it flickers), replace `useEffect` with `useLayoutEffect`.
- If your Effect is caused by an interaction (like a click), **React may run your Effect before the browser paints the updated screen**. This ensures that the result of the Effect can be observed by the event system. Usually, this works as expected. However, if you must defer the work until after paint, such as an `alert()`, you can use `setTimeout`. See [reactwg/react-18/128](https://github.com/reactwg/react-18/discussions/128) for more information.
- Even if your Effect was caused by an interaction (line a click), **React may allow the browser to repaint the screen before processing the state updates inside your Effect**. Usually, this works as expected. However, if you must block the browser from repainting the screen, you need to replace `useEffect` with `useLayoutEffect`.
- Effects **only run on the client**. They don't run during server rendering.

## Usage

### Connecting to an external system

Some components need to stay connected to the network, some browser API, or a third-party library, while they are displayed on the page. These systems aren't controlled by React, so they are called _external_.

To [connect your component to some external system](https://react.dev/learn/synchronizing-with-effects), call `useEffect` at the top level of your component:

```js
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
}
```

You need to pass two arguments to `useEffect`:

1. A _setup function_ with setup code that connects to that system.
   - It should return a _cleanup function_ with cleanup code that disconnects from that system.
2. A list of dependencies including every value from your component used inside of those functions.

**React calls your setup and cleanup functions whenever it's necessary, which may happen multiple times:**

1. Your setup code runs when your component is added to the page (mounts).
2. After every re-render of your component where the dependencies have changed:
   - First, your cleanup code runs with the old props and state.
   - Then, your setup code runs with the new props and state.
3. Your cleanup code runs one final time after your component is removed from the page (unmounts).

**Let's illustrate this sequence for the example above.**

when the `ChatRoom` component above gets added to the page, it will connect to the chat room with the initial `serverUrl` and `roomId`. If either `serverUrl` or `roomId` change as a result of a re-render (say, if the user picks a different chat room in a dropdown), your Effect will _disconnect from the previous room, and connect to the next one_. When the `ChatRoom` component is removed from the page, your Effect will disconnect one last time.

**To help you find bugs, in development React runs setup and cleanup one extra time before the setup.** This is a stress-test that verifies your Effect's logic is implemented correctly. If this causes visible issues, your cleanup function is missing some logic. The cleanup function should stop or undo whatever the setup function was doing. The rule of thumb is that the user shouldn't be able to distinguish between the setup being called once (as in production) and a setup → cleanup → setup sequence (as in development). [See common solutions.](https://react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

**Try to write every Effect as an independent process and think about a single setup/cleanup cycle at a time.** It shouldn't matter whetehr your component is mounting, updating, or unmounting. When your cleanup logic correctly "mirrors" the setup logic, your Effect is resilient to running setup and cleanup as often as needed.

> ## Note
>
> An Effect lets you keep your component synchronized with some external system (like a chat service). Here, _external system_ means any piece of code that's not controlled by React, such as:
>
> - A timer managed with `setInterval()` and `clearInterval()`.
> - An event subscription using `window.addEventListener()` and `window.removeEventListener()`.
> - A third-party animation library with an API like `animation.start()` and `animation.reset()`.
>
> **If you're not connecting to any external system, you probably don't need an Effect.**

### Wrapping Effects in custom Hooks

Effects are an ["escape hatch"](https://react.dev/learn/escape-hatches): you use them when you need to "step outside React" and when there is no better built-in solution for your use case. If you find yourself often needing to manually write Effects, it's usually a sign that you need to extract some [custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) for common behaviors your components rely on.

For example, this `useChatRoom` custom Hook "hides" the logic of your Effect behind a more declarative API:

```js
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

Then you can use it from any component like this:

```js
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
  });
  // ...
}
```

There are also many excellent custom Hooks for every purpose available in the React ecosystem.

[Learn more about wrapping Effects in custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks).

### Controlling a non-React widget

Sometimes, you want to keep an external system synchronized to some prop or state of your component.

For example, if you have a third-party map widget or a video player component written without React, you can use an Effect to call methods on it that make its state match the current state of your React component. This Effect creates an instance of a `MapWidget` class defined in `map-widget.js`. When you change the `zoomLevel` prop of the `Map` component, the Effect calls the `setZoom()` on the class instance to keep it synchronized:

```js
export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }

    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);

  return <div style={{ width: 200, height: 200 }} ref={containerRef} />;
}
```

In this example, a cleanup function is not needed because the `MapWidget` class manages only the DOM node that was passed to it. After the `Map` React component is removed from the tree, both the DOM node and the `MapWidget` class instance will automatically garbage-collected by the browser JavaScript engine.

### Fetching data with Effects

You can use an Effect to fetch data for your component. Note that [if you use a framework](https://react.dev/learn/start-a-new-react-project#production-grade-react-frameworks), using your framework's data fetching mechanism will be a lot more efficient than writing Effects manually.

If you want to fetch data from an Effect manually, your code might look like this:

```js
export default function Page() {
  const [person, setPerson] = useState("Alice");
  const [bio, setBio] = useState(null);

  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then((result) => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    };
  }, [person]);

  // ...
}
```

Note the `ignore` variable which is initialized to `false`, and is set to `true` during cleanup. This ensures your code doesn't suffer from "race conditions": network responses may arrive in a different order than you sent them.

You can also rewrite using the `async` / `await` syntax, but you still need to provide a cleanup function:

```js
export default function Page() {
  const [person, setPerson] = useState("Alice");
  const [bio, setBio] = useState(null);
  useEffect(() => {
    async function startFetching() {
      setBio(null);
      const result = await fetchBio(person);
      if (!ignore) {
        setBio(result);
      }
    }

    let ignore = false;
    startFetching();
    return () => {
      ignore = true;
    };
  }, [person]);
}
```

Writing data fetching directly in Effects gets reprtitve and makes it difficult to add optimizations like caching and server rendering later. It's easier to use a custom Hook--either your own or mantained by the community.

> DEEP DIVE
>
> ### What are good alternatives to data fetching in Effects?
>
> Writing `fetch` calls inside Effects is a popular way to fetch data, expecially in fully client-side apps. This is, however, a very manual approach and it has significant downsides:
>
> - **Effects don't run on the server.** This means that the intial server-rendered HTML will only include a loading state with no data. The client computer will have to download all JavaScript and render your app only to discover that now it needs to load the data. This is not very efficient.
> - **Fetching directly in Effects makes it easy to create "network waterfalls".** You render the parent component, it fetches some data, renders the child components, and then they start fetching their data. If the network is not very fast, this is significantly slower than fetching all data in parallel.
> - **Fetching directly in Effects usually means you don't preload or cache data.** For example, if the component unmounts and then mounts again, it would have to fetch the data again.
> - **It's not very ergonomic.** There's quite a bit of boilerplate code involved when writing `fetch` calls in a way that doesn't suffer from bugs like race conditions.
>
> This list of downsides is not specific to React. It applies to fetching data on mount with any library. Like with routing, data fetching is nor trivial to do well, so we recommend the following approaches:
>
> - **If you use a framework, use its built-in data fetching mechanism.** Modern React frameworks have integrated data fetching mechanisms that are efficient and don't suffer from the above pitfalls.
> - **Otherwise, consider using or building a client-size cache.** Popular open source solutions include React Query, useSWR, and React Rounter 6.4+. You can build your own solution too, in which case you would use Effects under the hood but also add logic for deduplicating requests, caching responses, and avoiding network waterfalls (by preloading data or hoisting data requiremnents to routes).
>
> you can continue fetching data directly in Effects if neither of these approaches suit you.

### Specifying reactive dependencies
