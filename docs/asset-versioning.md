# Asset versioning

One common challenge when building single-page apps is refreshing site assets when they've been changed. Thankfully, Inertia makes this easy by optionally tracking the current version of your site assets. When an asset changes, Inertia will automatically make a full page visit instead of a XHR visit on the next request.

## Configuration

To enable automatic asset refreshing, you need to tell Inertia the current version of your assets. This can be any arbitrary string (letters, numbers, or a file hash), as long as it changes when your assets have been updated.

The asset version can be provided using the `Inertia.version()` method in your middleware.

```ts
// framework: hono
import { Hono } from "hono";
import { inertiaHonoAdapter } from "@inertianode/hono";

const app = new Hono();

// Set version in adapter options
app.use(
  "*",
  inertiaHonoAdapter({
    version: () => "1234567890",
  })
);
```

```ts
// framework: express
import express from "express";
import { inertiaExpressAdapter } from "@inertianode/express";

const app = express();

// Set version in adapter options
app.use(inertiaExpressAdapter({
  version: () => "1234567890"
}));
```

```ts
// framework: koa
import Koa from "koa";
import { inertiaKoaAdapter } from "@inertianode/koa";

const app = new Koa();

// Set version in adapter options
app.use(inertiaKoaAdapter({
  version: () => "1234567890"
}));
```

> **Note:** You can also set the version per-request using `res.Inertia.setVersion()` (Express) or `ctx.Inertia.setVersion()` (Koa), but setting it in the adapter options is recommended for global configuration.

## Cache busting

Asset refreshing in Inertia works on the assumption that a hard page visit will trigger your assets to reload. However, Inertia doesn't actually do anything to force this. Typically this is done with some form of cache busting. For example, appending a version query parameter to the end of your asset URLs.

With modern build tools like Vite, asset versioning is often done automatically. You can also implement cache busting by including version hashes in your asset filenames or by appending version query parameters.

## Manual refreshing

If you want to take asset refreshing into your control, you can return a fixed value from the `version` method. This disables InertiaNode's automatic asset versioning.

For example, if you want to notify users when a new version of your frontend is available, you can still expose the actual asset version to the frontend by including it as [shared data](/shared-data).

```ts
// framework: hono
import { Hono } from "hono";
import { inertiaHonoAdapter } from "@inertianode/hono";

const app = new Hono();

// Disable automatic versioning
app.use(
  "*",
  inertiaHonoAdapter({
    version: null,
  })
);

// Share version as data
app.use("*", async (c, next) => {
  c.Inertia.share("version", () => "1234567890");
  await next();
});
```

```ts
// framework: express
import express from "express";
import { inertiaExpressAdapter } from "@inertianode/express";

const app = express();

// Disable automatic versioning
app.use(inertiaExpressAdapter({
  version: null
}));

// Share version as data
app.use((req, res, next) => {
  res.Inertia.share("version", () => "1234567890");
  next();
});
```

```ts
// framework: koa
import Koa from "koa";
import { inertiaKoaAdapter } from "@inertianode/koa";

const app = new Koa();

// Disable automatic versioning
app.use(inertiaKoaAdapter({
  version: null
}));

// Share version as data
app.use(async (ctx, next) => {
  ctx.Inertia.share("version", () => "1234567890");
  await next();
});
```

On the frontend, you can watch the `version` property and show a notification when a new version is detected.
