---
slug: /
---

# Getting Started

## 1. Setup Client {#client}

#### Install

<Tabs>
<TabItem value="browser" label="Browser">

```bash
npm install @palette.dev/browser
```

</TabItem>
<TabItem value="electron" label="Electron">

```bash
npm install @palette.dev/electron
```

</TabItem>
</Tabs>

#### Get your Client Key

:::info
Get your **client key** at `https://palette.dev/[your-username]/[your-project]/settings`
:::

Import palette **before all other imports** in your app's entrypoint file (eg. main.js or index.js):

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<Tabs>
<TabItem value="browser" label="Browser">

```ts title="index.js"
// Import palette before all other imports
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

// Your other imports...
```

</TabItem>

<TabItem value="electron" label="Electron">

#### Main Process

```ts title="main.js"
// Import palette before all other imports
import { init, events, measure, profiler } from "@palette.dev/electron/main";

init({
  key: "YOUR_CLIENT_KEY",
  // Collect click, performance events, and profiles
  plugins: [events(), measure(), profiler()],
});

// Your other imports...
```

#### Renderer Process

```ts title="renderer.js"
// Import palette before all other imports
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

// Your other imports...
```

#### Preload Script

Skip this step if you have [`nodeIntegration`](https://www.electronjs.org/docs/latest/api/browser-window#new-browserwindowoptions) enabled.

If `nodeIntegration` is disabled and you don't have a preload script, you'll need to [create one](https://www.electronjs.org/docs/latest/tutorial/tutorial-preload).

Call `init` from `@palette.dev/electron/preload` in your preload script:

```ts title="preload.js"
import { init } from "@palette.dev/electron/preload";

init();
```

</TabItem>
</Tabs>

## 2. Upload Source Maps {#assets}

:::info
This step is required if you are using a framework or a bundler (like next.js, webpack, vite, and parcel).
:::

### Prerequisite

1. Get your **asset key** at `https://palette.dev/[your-username]/[your-project]/settings`.
2. In your CI, set an env variable named `PALETTE_ASSET_KEY` to your **asset key**. This is necessary since source maps are only uploaded in CI.

:::tip

Set the `PALETTE_ASSET_KEY` env variable from the CLI.

<Tabs>
<TabItem value="vercel" label="Vercel">

```bash
npx vercel env add
```

See the [Vercel CLI docs](https://vercel.com/docs/cli/env#usage) for more info.

</TabItem>

<TabItem value="netlify" label="Netlify">

```bash
npx netlify env:set PALETTE_ASSET_KEY "YOUR_ASSET_KEY"
```

See the [Netlify CLI docs](https://www.netlify.com/blog/2021/12/10/more-tips-for-environment-variables-and-netlify-cli/) for more info.

</TabItem>

<TabItem value="github" label="GitHub Workflows">

```bash
gh secret set PALETTE_ASSET_KEY
```

See the [GitHub CLI docs](https://cli.github.com/manual/gh_secret_set) for more info.

</TabItem>
</Tabs>

:::

### Configure Your Bundler

<Tabs>
<TabItem value="next" label="Next.js">

#### Install

```bash
npm install @palette.dev/webpack-plugin --save-dev
```

#### Usage

```ts title="next.config.js"
const PalettePlugin = require("@palette.dev/webpack-plugin");

module.exports = {
  // ...
  webpack(config) {
    if (config.mode === "production") {
      config.plugins.push(
        new PalettePlugin({
          key: process.env.PALETTE_ASSET_KEY,
          include: [".next/static"],
        })
      );
    }
    return config;
  },
};
```

</TabItem>

<TabItem value="cra" label="Create React App (ejected)">

#### Install

```bash
npm install @palette.dev/webpack-plugin --save-dev
```

#### Usage

```ts title="config/webpack.config.js"
const PalettePlugin = require("@palette.dev/webpack-plugin");

module.exports = {
  // ...
  plugins: [
    isEnvProduction &&
      new PalettePlugin({
        key: process.env.PALETTE_ASSET_KEY,
        include: ["build/static/js"],
      }),
  ].filter(Boolean),
};
```

</TabItem>

<TabItem value="webpack" label="Webpack">

#### Install

```bash
npm install @palette.dev/webpack-plugin --save-dev
```

#### Usage

```ts title="webpack.config.js"
const PalettePlugin = require("@palette.dev/webpack-plugin");

module.exports = {
  // ...
  plugins: [
    new PalettePlugin({
      key: process.env.PALETTE_ASSET_KEY,
    }),
  ],
};
```

</TabItem>

<TabItem value="vite" label="Vite">

#### Install

```bash
npm install @palette.dev/plugin-vite --save-dev
```

#### Usage

Update your vite config

```ts title="vite.config.ts"
import path from "path";
import { defineConfig } from "vite";
import palette from "@palette.dev/plugin-vite";

export default defineConfig({
  plugins: [
    // Add palette plugin
    palette({
      key: process.env.PALETTE_ASSET_KEY,
      outputPath: "dist/assets",
    }),
  ],
  build: {
    // Output source maps
    sourcemap: true,
    // Set source maps paths relative to the current working directory
    rollupOptions: {
      output: {
        sourcemapPathTransform: (relativeSourcePath, sourcemapPath) =>
          path.relative(
            process.cwd(),
            path.resolve(path.dirname(sourcemapPath), relativeSourcePath)
          ),
      },
    },
  },
});
```

Import `virtual:@palette.dev/plugin-vite/init` in your app's entrypoint (e.g. main.js):

```ts title="main.ts"
import "virtual:@palette.dev/plugin-vite/init"; // import before other imports
import { init } from "@palette.dev/...";

init({
  // ...palette opts
});
```

</TabItem>

<TabItem value="other" label="Other">

#### Install

```bash
npm install @palette.dev/cli --save-dev
```

#### Usage

Run the CLI from your project's root directory. Asset paths are relative to the project root.

```bash
palette upload path/to/assets
```

The asset key will be read from the `PALETTE_ASSET_KEY` env variable.

#### Example

Suppose you have the following directory structure:

<!-- Directory tree generated with https://tree.nathanfriend.io -->

```
my-project/
├── dist/
│   └── main.js
└── package.json
```

Define an upload script in your `package.json`:

```json
{
  "scripts": {
    "upload": "palette upload dist"
  }
}
```

</TabItem>

</Tabs>

## 3. Add Headers {#headers}

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

<TabItem value="vercel" label="Vercel">

Use the following config if you're deploying a **non-next.js framework** to Vercel:

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

<TabItem value="vercel-legacy" label="Vercel (legacy config)">

Use the following config if you're using Vercel's [legacy `routes` config](https://vercel.com/docs/project-configuration#legacy/routes):

```json title="vercel.json"
{
  "routes": [
    {
      "src": "/[^.]+",
      "dest": "/",
      "status": 200,
      "headers": {
        "Document-Policy": "js-profiling"
      }
    }
  ]
}
```

See the [vercel docs](https://vercel.com/docs/project-configuration#legacy/routes) for more info.

</TabItem>

<TabItem value="netlify" label="Netlify">

```toml title="_headers"
/*
  Document-Policy: js-profiling
```

See the [netlify docs](https://docs.netlify.com/routing/headers/#syntax-for-the-headers-file) for more info.

</TabItem>

<TabItem value="electron" label="Electron">

This step was already done in [step 1](#client).

The `profiler` plugin from `@palette.dev/electron/main` adds the corresponding headers.

```ts title="main.js"
import { init, profiler } from "@palette.dev/electron/main";

init({
  key: "YOUR_CLIENT_KEY",
  plugins: [profiler()],
});
```

</TabItem>

</Tabs>

## 4. Start the Profiler {#profiler}

<Tabs>
<TabItem value="browser" label="Browser">

Include this at the **top-level** of your app's entrypoint:

```ts title="index.js"
import { profiler, label } from "@palette.dev/browser";

// Profile page load
profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 });
window.addEventListener("load", () => profiler.stop());

// A utility for profiling and label frequent events
let timeoutId;
const debounce = (start, stop, opts = { timeout: 1_000 }) => {
  return () => {
    if (typeof timeoutId === "number") {
      clearTimeout(timeoutId);
    } else {
      start();
    }
    // Debounce marking the end of the label
    timeoutId = setTimeout(() => {
      stop();
      timeoutId = undefined;
    }, opts.timeout);
  };
};

// Debounce starting the profiler
const debounceProfiler = debounce(
  () => {
    label.start("ui.interaction");
    profiler.start({
      sampleInterval: 10,
      maxBufferSize: 10_000,
    });
  },
  () => {
    label.end("ui.interaction");
    return profiler.stop();
  }
);

// Profile scroll, mousemove, and click events
window.addEventListener("wheel", debounceProfiler);
window.addEventListener("mousemove", debounceProfiler);
window.addEventListener("click", debounceProfiler);
window.addEventListener("keypress", debounceProfiler);
```

:::warning
Palette only emits metrics in production. You'll need to build your app to see metrics in your dashboard.
:::

</TabItem>

<TabItem value="electron" label="Electron">

Include this at the **top-level** of your app's **renderer process**:

```ts title="renderer.js"
import { profiler, label } from "@palette.dev/electron/renderer";

// Profile page load
profiler.start({ sampleInterval: 10, maxBufferSize: 10_000 });
window.addEventListener("load", () => profiler.stop());

// A utility for profiling and label frequent events
let timeoutId;
const debounce = (start, stop, opts = { timeout: 1_000 }) => {
  return () => {
    if (typeof timeoutId === "number") {
      clearTimeout(timeoutId);
    } else {
      start();
    }
    // Debounce marking the end of the label
    timeoutId = setTimeout(() => {
      stop();
      timeoutId = undefined;
    }, opts.timeout);
  };
};

// Debounce starting the profiler
const debounceProfiler = debounce(
  () => {
    label.start("ui.interaction");
    profiler.start({
      sampleInterval: 10,
      maxBufferSize: 10_000,
    });
  },
  () => {
    label.end("ui.interaction");
    return profiler.stop();
  }
);

// Profile scroll, mousemove, and click events
window.addEventListener("wheel", debounceProfiler);
window.addEventListener("mousemove", debounceProfiler);
window.addEventListener("click", debounceProfiler);
window.addEventListener("keypress", debounceProfiler);
```

:::warning
Palette only emits metrics in production. You'll need to package your electron app to see metrics in your dashboard.
:::

</TabItem>

</Tabs>

For more examples of profiling, see the [profiling patterns](/patterns).

## You're all set! 🎉

Go to your project's page to see your metrics.
