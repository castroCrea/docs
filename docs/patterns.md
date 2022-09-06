# Profiling Examples

Palette's profiler was designed with flexibility in mind. You can start and stop the profiler with Palette's profiler API:

```ts
import { profiler } from "@palette.dev/browser";

profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 });
someLongRunningFunction();
profiler.stop();
```

This flexibility lets you profile in whichever way you choose. Below are some patterns for profiling common scenarios.

## Infrequent Events

Pages are usually slow during events like startup time, routing changes, loading 3rd party scripts, and more. Knowing these scenarios are prone to performance regressions, it's a good idea to profile them.

In the example below, we profile the page load event:

```ts
// Start the profiler when the page loads
// Run this in the top-level of your app's entrypoint file
profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 });

addEventListener("load", () => {
  profiler.stop();
});
```

## Labeling

Profiles alone often don't provide enough context on what the user did. Labels provide context on what the user was doing when the profile was taken. They're similar to tracing in DataDog and Sentry.

The `label` API is a tiny wrapper around the [`performance.mark()`](https://developer.mozilla.org/en-US/docs/Web/API/Performance/mark) and [`performance.measure`](https://developer.mozilla.org/en-US/docs/Web/API/Performance/measure) web APIs.

Here's an example of `label` which profiles and labels the initial react render:

```ts
import { label, profiler } from "@palette.dev/browser";

label.start("react.rendering");
profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 }); // start profiling before loading the script
render(<App />, document.getElementById("root"));
profiler.stop(); // stop profiling after loading the script
label.end("react.rendering");
```

## Frequent Events

This pattern works well for user interactions like typing, dragging, and clicking. Generally, a user action that may be repeated multiple times works well with this pattern.

```ts
// A utility for profiling and label frequent events
const debounce = (start, stop, opts = { timeout: 1_000 }) => {
  let timeoutId;
  return () => {
    if (timeoutId == undefined) {
      start();
    } else {
      clearTimeout(timeoutId);
    }
    // Debounce marking the end of the label
    timeoutId = setTimeout(() => {
      stop();
      timeoutId = undefined;
    }, timeout);
  };
};
```

#### Examples of `debounce`

```tsx
import { label, profiler } from "@palette.dev/browser";

const onChangeHandler = debounce(
  () => {
    label.start("ui.textarea.typing");
    profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 });
  },
  () => {
    profiler.stop();
    label.end("ui.textarea.typing");
  }
);

const Hello = () => (
  <textarea
    placeholder="type something"
    onChange={onChangeHandler}
    style={{ width: "100%", height: "100%", padding: 10 }}
  />
);
```
