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
