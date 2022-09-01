# Vitals

Palette's vitals plugin captures [web vitals](https://web.dev/vitals/) including FCP, LCP, FID, TTI, TTFB, and CLS.

```ts
import { init, vitals } from "@palette.dev/electron/renderer";

init({
  key: "YOUR_CLIENT_KEY",
  plugins: [vitals()],
});
```
