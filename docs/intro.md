---
slug: /
---

# Getting Started

## Installation

<Tabs>
<TabItem value="electron" label="Electron">

```npm2yarn
npm install @palette.dev/electron
```

</TabItem>
<TabItem value="browser" label="Browser">

```npm2yarn
npm install @palette.dev/browser
```

</TabItem>
</Tabs>

## Usage

Import `init` and pass your API key and the plugins you want to use. Import palette **before all other imports** in your app's entrypoint file.

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<Tabs>
<TabItem value="electron" label="Electron">

#### Main Process

```ts title="main.js"
import { init, events, measure, profiler } from "@palette.dev/electron/main";

init({
  key: "YOUR_API_KEY",
  // Collect click, performance events, and profiles
  plugins: [events(), measure(), profiler()],
});
```

#### Renderer Process

```ts title="renderer.js"
import {
  init,
  events,
  vitals,
  measure,
  network,
  profiler,
} from "@palette.dev/electron/renderer";

init({
  key: "YOUR_API_KEY",
  // Collect click, web vitals, network, performance events, and profiles
  plugins: [events(), vitals(), network(), measure(), profiler()],
});
```

#### Preload Script (optional)

If you have a preload script you need to call `init` from `@palette.dev/electron/preload`. Skip this if you don't have a preload script.

```ts title="preload.js"
import { init } from "@palette.dev/electron/preload";

init();
```

</TabItem>
<TabItem value="browser" label="Browser">

```ts title="index.js"
import {
  init,
  events,
  vitals,
  measure,
  network,
  profiler,
} from "@palette.dev/browser";

init({
  key: "YOUR_API_KEY",
  // Collect click, web vitals, network, performance events, and profiles
  plugins: [events(), vitals(), network(), measure(), profiler],
});
```

</TabItem>
</Tabs>

## Tagging

```ts
import { tag } from "@palette.dev/electron/renderer";

tag("userId", "u-123");
```

## Custom Metrics

Capture custom metrics with the built-in [`performance.mark()`](https://developer.mozilla.org/en-US/docs/Web/API/Performance/mark) and [`performance.measure`](https://developer.mozilla.org/en-US/docs/Web/API/Performance/measure) web APIs. Palette's `measure` plugin records all events recorded by `mark` and `measure`.

```ts
// Mark events at a specific action or event
performance.mark("myApp.someUserEvent", {
  detail: "some event details",
});
performance.mark("myApp.stateChange", {
  detail: {
    from: "stateA",
    to: "stateB",
  },
});

// Measuring time durations
performance.mark("userAction.start"); // marks starting point
performance.measure("userAction.duration", "userAction.start"); // measures from starting point
```

## Labeling

A label marks a critical interaction in Palette's timeline.

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

Labeling is supported in `electron/main`, `electron/renderer`, and `browser` clients.

## Uploading Source Maps

`@palette.dev/webpack-plugin` is a webpack plugin that uploads source maps to Palette. Webpack is the only bundler supported at the moment.

### Installation

```bash
npm install @palette.dev/webpack-plugin --save-dev
```

### Usage

1. Find your **asset key** at `https://palette.dev/[your-username]/[your-project]/settings`.
2. Add your **asset key** to your webpack config.

```ts title="webpack.config.js"
import PalettePlugin from "@palette.dev/webpack-plugin";

export default {
  // ...
  plugins: [new PalettePlugin({ key: "YOUR_ASSET_KEY" })],
};
```
