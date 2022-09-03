# Sampling Patterns

## Frequent Events

This pattern works well for user interactions like typing, dragging, and clicking. Generally, an user action that may be repeated multiple times works well with this pattern.

```ts
import { profiler, label } from "@palette.dev/electron/renderer";

/**
 * A utility for profiling and label frequent events
 */
export const debounceLabel = (
  fn: () => void | Promise<void>,
  start: () => void,
  stop: () => void,
  _opts?: { name?: string; timeout?: number }
) => {
  const { name, timeout } = {
    name: fn.name ?? "anonymous",
    timeout: 1_000,
    ..._opts,
  };

  let timeoutId: ReturnType<typeof global.setTimeout> | undefined;

  return async () => {
    if (timeoutId === undefined || timeoutId === null) {
      label.start(name); // Mark the start of the label
      start();
    } else {
      clearTimeout(timeoutId);
    }

    await fn(); // Invoke the function to be profiled

    // Debounce marking the end of the label
    timeoutId = setTimeout(() => {
      stop();
      label.end(name); // Mark the end of the label
      timeoutId = undefined;
    }, timeout);
  };
};
```

#### Examples of `debounceMeasure`

```tsx
const handleInputAndLabel = debounceLabel(
  handleInput,
  () => profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 }),
  () => profiler.stop(),
  {
    name: "handle-input",
    timeout: 1_000,
  }
);

const Hello = () => (
  <textarea
    placeholder="type something"
    onChange={handleInputAndLabel}
    style={{ width: "100%", height: "100%", padding: 10 }}
  />
);
```

Here's an example of how `debounceMeasure` would handle `keypress` events:

| Event    | Time | Debounce Timeout | Action                                                 |
| -------- | ---- | ---------------- | ------------------------------------------------------ |
| keypress | 0s   | 1s               | calls `_handleInput` and marks _start_ of label        |
| keypress | 1s   | 1s               | calls `_handleInput` and marks _end_ of label          |
| keypress | 3s   | 1s               | calls `_handleInput`, marks _start_ and _end_ of label |

## Infrequent Events

These events tend to be one-off events

- App startup
- Routing changes
- Loading third-party scripts
- Animations

```ts
export const labelFn = async (
  fn: () => void | Promise<void>,
  start: () => void,
  stop: () => void,
  opts?: { name?: string }
) => {
  const { name } = { name: fn.name ?? "anonymous", ...opts };
  label.start(name);
  profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 });
  await fn();
  profiler.stop();
  label.end(name);
};
```

#### Examples of `measureFn`:

```ts
// Profile and create a label named "electron-when-ready"
labelFn(
  () => loadScript("https://cdn.example.com/my-scripts.js"), // load a script
  () => profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 }), // start profiling before loading the script
  () => profiler.stop(), // stop profiling after loading the script
  {
    name: "load-script", // the name of the label
  }
);
```
