---
title: CPU
---

# CPU Monitoring

:::caution

**This API is only supported in Electron. It is only supported in renderer processes with `nodeIntegration` enabled**.

:::

### Usage

The CPU plugin collects CPU samples from Electron's renderer and main processes.

```ts {6} title="main.js (main process entrypoint)"
import { init, cpu } from "@palette.dev/electron/main";

init({
  key: "YOUR_CLIENT_KEY",
  plugins: [
    cpu(),
    // ...
  ],
});
```

```ts {6} title="renderer.js (renderer process entrypoint)"
import { init, cpu } from "@palette.dev/electron/renderer";

init({
  key: "YOUR_CLIENT_KEY",
  plugins: [
    cpu(),
    // ...
  ],
});
```

### Sampling Rate

CPU samples are collected in intervals. The default sample rate is `1000 ms`.

```ts {7} title="main.js (main process entrypoint)"
import { init, cpu } from "@palette.dev/electron/main";

init({
  key: "YOUR_CLIENT_KEY",
  plugins: [
    cpu({
      sampleInterval: 1_000,
    }),
  ],
});
```

A smaller sampling rate provides more accurate reporting while a greater one will consume less resources.

### Starting and Stopping Sampling

```ts {4,6} title="main.js (main process entrypoint)"
import { cpu } from "@palette.dev/electron/main";

if (userIsIdle) {
  cpu.stop();
} else {
  cpu.start();
}
```
