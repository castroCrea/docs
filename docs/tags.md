# Tags

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
