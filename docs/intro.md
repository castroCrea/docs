---
slug: /
---

# Getting Started

## 1. Setup Client

#### Installation

<Tabs>
<TabItem value="browser" label="Browser">

```npm2yarn
npm install @palette.dev/browser
```

</TabItem>
<TabItem value="electron" label="Electron">

```npm2yarn
npm install @palette.dev/electron
```

</TabItem>
</Tabs>

#### Find your Client Key

:::info
Find your **client key** at `https://palette.dev/[your-username]/[your-project]/settings` and pass it to `init` along with the plugins you want to use. Import palette **before all other imports** in your app's entrypoint file.
:::

Then pass your client key to your Palette's `init` function:

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<Tabs>
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
  key: "YOUR_CLIENT_KEY",
  // Collect click, web vitals, network, performance events, and profiles
  plugins: [events(), vitals(), network(), measure(), profiler()],
});
```

</TabItem>

<TabItem value="electron" label="Electron">

#### Main Process

```ts title="main.js"
import { init, events, measure, profiler } from "@palette.dev/electron/main";

init({
  key: "YOUR_CLIENT_KEY",
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
  key: "YOUR_CLIENT_KEY",
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
</Tabs>

## 2. Upload Source Maps

:::danger
This step is required if you are using a bundler (like webpack, esbuild, and parcel).
:::

`@palette.dev/webpack-plugin` is a webpack plugin that uploads source maps to Palette. Webpack is the only bundler supported at the moment.

#### Installation

```bash
npm install @palette.dev/webpack-plugin
```

#### Usage

1. Find your **asset key** at `https://palette.dev/[your-username]/[your-project]/settings`.
2. Add your **asset key** to your webpack config.

```ts title="webpack.config.js"
import PalettePlugin from "@palette.dev/webpack-plugin";

export default {
  // ...
  plugins: [new PalettePlugin({ key: "YOUR_ASSET_KEY" })],
};
```

## 3. Add Headers

To enable profiling, you need to add the following headers to your server responses.

```
"Document-Policy": "js-profiling"
```

<Tabs>
<TabItem value="next.js" label="Next.js">

```js title="next.config.js"
module.exports = {
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: [
          {
            key: "Document-Policy",
            value: "js-profiling",
          },
        ],
      },
    ];
  },
};
```

See the [next.js docs](https://nextjs.org/docs/api-reference/next.config.js/headers) for more info.

</TabItem>

<TabItem value="vercel" label="Vercel (no next.js)">

```json title="vercel.json"
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "Document-Policy",
          "value": "js-profiling"
        }
      ]
    }
  ]
}
```

See the [vercel docs](https://vercel.com/docs/project-configuration#project-configuration/headers) for more info.

</TabItem>

<TabItem value="cra" label="Create React App">

```js title="src/setupProxy.js"
module.exports = function (app) {
  app.use(function (req, res, next) {
    res.setHeader("Document-Policy", "js-profiling");
    next();
  });
};
```

See the [create-react-app docs](https://create-react-app.dev/docs/proxying-api-requests-in-development/) for more info.

</TabItem>

<TabItem value="netlify" label="Netlify">

```toml title="_headers"
/*
  Document-Policy: js-profiling
```

See the [netlify docs](https://docs.netlify.com/routing/headers/#syntax-for-the-headers-file) for more info.

</TabItem>

</Tabs>

## 4. Start the Profiler

<Tabs>
<TabItem value="browser" label="Browser">

```ts
import { profiler } from "@palette.dev/browser";

// Profile page load
profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 });
addEventListener("load", () => {
  profiler.stop();
});
```

</TabItem>

<TabItem value="electron" label="Electron">

```ts
import { profiler } from "@palette.dev/electron/renderer";

// Profile page load
profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 });
addEventListener("load", () => {
  profiler.stop();
});
```

</TabItem>
</Tabs>

For more examples of profiling, see the [profiling patterns](https://docs.palette.dev/patterns).

## Tagging

Tags allow you to provide additional context about a user's session that might be useful later.

Often you'll want to tag a session id and a user id with a tag to identify users.

<Tabs>
<TabItem value="browser" label="Browser">

```ts
import { tag } from "@palette.dev/browser";

tag("palette.userId", "user-id-123");
```

</TabItem>

<TabItem value="electron" label="Electron">

```ts
import { tag } from "@palette.dev/electron/main";

tag("palette.userId", "user-id-123");
```

</TabItem>
</Tabs>

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
