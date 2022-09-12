# Labels

A label marks a critical interaction in Palette's timeline.

<Tabs>
<TabItem value="browser" label="Browser">

```ts
import { label } from "@palette.dev/browser";

// Label and profile specific interactions or events
const labelFn = (name, fn) => {
  label.start(name);
  fn();
  label.end(name);
};

// Profile initial react render
labelFn("react.render", () => {
  render(<MyApp />, document.getElementById("root"));
});
```

</TabItem>

<TabItem value="electron" label="Electron">

```ts
import { label } from "@palette.dev/electron/renderer";

// Label and profile specific interactions or events
const labelFn = (name, fn) => {
  label.start(name);
  fn();
  label.end(name);
};

// Profile initial react render
labelFn("react.render", () => {
  render(<MyApp />, document.getElementById("root"));
});
```

</TabItem>
</Tabs>

Labeling is supported in `electron/main`, `electron/renderer`, and `browser` clients.
