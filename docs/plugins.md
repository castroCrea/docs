# Using Plugins

Palette allows you to import only what you need.

## Electron Main

```ts
import { init, cpu } from "@palette.dev/electron/main";

init({
  key: "YOUR_CLIENT_KEY",
  plugins: [cpu()],
});
```

## Electron Renderer

```ts
import { init, cpu, events } from "@palette.dev/electron/renderer";

init({
  key: "YOUR_CLIENT_KEY",
  plugins: [cpu(), events()],
});
```
