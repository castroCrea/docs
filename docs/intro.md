---
slug: /
---

# Getting Started

## 1. Setup Client {#client}

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

#### Get your Client Key

:::info
Get your **client key** at `https://palette.dev/[your-username]/[your-project]/settings` and pass it to `init` along with the plugins you want to use. Import palette **before all other imports** in your app's entrypoint file.
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

## 2. Upload Source Maps {#assets}

:::info
This step is required if you are using a framework or a bundler (like next.js, svelte, webpack, esbuild, and parcel).
:::

#### Prerequisite

1. Get your **asset key** at `https://palette.dev/[your-username]/[your-project]/settings`.
2. Define an environmental variable named `PALETTE_ASSET_KEY` in your CI environment.

<Tabs>
<TabItem value="next" label="Next.js">

#### Installation

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
          include: {
            ext: "js",
          },
        })
      );
    }
    return config;
  },
};
```

</TabItem>

<TabItem value="cra" label="Create React App (ejected)">

#### Installation

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

#### Installation

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

#### Installation

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

<TabItem value="cli" label="CLI">

#### Installation

```bash
npm install @palette.dev/cli --save-dev
```

#### Usage

Run the CLI from your project's root directory. Asset paths are relative to the project root.

```bash
palette upload path/to/assets
```

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

This step was already done in [step 1](#1-setup-client).

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
addEventListener("load", () => profiler.stop());

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
    }, timeoutId);
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
addEventListener("wheel", debounceProfiler);
addEventListener("mousemove", debounceProfiler);
addEventListener("click", debounceProfiler);
addEventListener("keypress", debounceProfiler);
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
addEventListener("load", () => profiler.stop());

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
    }, timeoutId);
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
addEventListener("wheel", debounceProfiler);
addEventListener("mousemove", debounceProfiler);
addEventListener("click", debounceProfiler);
addEventListener("keypress", debounceProfiler);
```

:::warning
Palette only emits metrics in production. You'll need to package your electron app to see metrics in your dashboard.
:::

</TabItem>

</Tabs>

For more examples of profiling, see the [profiling patterns](/patterns).

## You're all set! 🎉

Go to your project's page to see your metrics.
