# History encryption

Imagine a scenario where your user is authenticated, browses privileged information on your site, then logs out. If they press the back button, they can still see the privileged information that is stored in the window's history state. This is a security risk. To prevent this, Inertia.js provides a history encryption feature.

## How it works

When you instruct Inertia to encrypt your app's history, it uses the browser's built-in [`crypto` api ](https://developer.mozilla.org/en-US/docs/Web/API/Crypto)to encrypt the current page's data before pushing it to the history state. We store the corresponding key in the browser's session storage. When the user navigates back to a page, we decrypt the data using the key stored in the session storage.

Once you instruct Inertia to clear your history state, we simply clear the existing key from session storage roll a new one. If we attempt to decrypt the history state with the new key, it will fail and Inertia will make a fresh request back to your server for the page data.

History encryption relies on `window.crypto.subtle` which is only available in secure environments (sites with SSL enabled).

## Opting in

History encryption is an opt-in feature. There are several methods for enabling it:

### Global encryption

If you'd like to enable history encryption globally, set the `Inertia.` config value to `true`.

```ts
// framework: hono
app.use("*", async (c, next) => {
  c.Inertia.encryptHistory();
  await next();
});
```

```ts
// framework: express
app.use((req, res, next) => {
  res.Inertia.encryptHistory();
  next();
});
```

```ts
// framework: nestjs
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class EncryptHistoryMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    res.Inertia.encryptHistory();
    next();
  }
}
```

```ts
// framework: koa
app.use(async (ctx, next) => {
  ctx.Inertia.encryptHistory();
  await next();
});
```

You are able to opt out of encryption on specific pages by calling the `Inertia.encryptHistory()` method before returning the response.

```ts
// framework: hono
app.get("/page", async (c) => {
  c.Inertia.encryptHistory(false);

  return await c.Inertia("Page", {});
});
```

```ts
// framework: express
app.get("/page", async (req, res) => {
  res.Inertia.encryptHistory(false);

  await res.Inertia("Page", {});
});
```

```ts
// framework: nestjs
import { Controller, Get, Req, Res } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller()
export class PageController {
  @Get('/page')
  async show(@Req() req: Request, @Res() res: Response) {
    res.Inertia.encryptHistory(false);

    await res.Inertia.render('Page', {});
  }
}
```

```ts
// framework: koa
router.get("/page", async (ctx) => {
  ctx.Inertia.encryptHistory(false);

  await ctx.Inertia("Page", {});
});
```

### Per-request encryption

To encrypt the history of an individual request, simply call the `Inertia.encryptHistory()` method before returning the response.

```ts
// framework: hono
app.post("/secure-action", async (c) => {
  c.Inertia.encryptHistory();

  return await c.Inertia("SecurePage", {});
});
```

```ts
// framework: express
app.post("/secure-action", async (req, res) => {
  res.Inertia.encryptHistory();

  await res.Inertia("SecurePage", {});
});
```

```ts
// framework: nestjs
import { Controller, Post, Req, Res } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller()
export class SecureController {
  @Post('/secure-action')
  async secureAction(@Req() req: Request, @Res() res: Response) {
    res.Inertia.encryptHistory();

    await res.Inertia.render('SecurePage', {});
  }
}
```

```ts
// framework: koa
router.post("/secure-action", async (ctx) => {
  ctx.Inertia.encryptHistory();

  await ctx.Inertia("SecurePage", {});
});
```

### Encrypt middleware

You can also use middleware to control history encryption.

```ts
// framework: hono
import { Hono } from "hono";

const app = new Hono();

app.use("*", async (c, next) => {
  c.Inertia.encryptHistory(false);
  await next();
});
```

```ts
// framework: express
import express from "express";

const app = express();

app.use((req, res, next) => {
  res.Inertia.encryptHistory(false);
  next();
});
```

```ts
// framework: nestjs
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class EncryptHistoryConfigMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    res.Inertia.encryptHistory(false);
    next();
  }
}
```

```ts
// framework: koa
import Koa from "koa";

const app = new Koa();

app.use(async (ctx, next) => {
  ctx.Inertia.encryptHistory(false);
  await next();
});
```

## Clearing history

To clear the history state, you can call the `Inertia.clearHistory()` method before returning the response.

```ts
// framework: hono
app.post("/logout", async (c) => {
  c.Inertia.clearHistory();

  return await c.Inertia("Login", {});
});
```

```ts
// framework: express
app.post("/logout", async (req, res) => {
  res.Inertia.clearHistory();

  await res.Inertia("Login", {});
});
```

```ts
// framework: nestjs
import { Controller, Post, Req, Res } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller()
export class AuthController {
  @Post('/logout')
  async logout(@Req() req: Request, @Res() res: Response) {
    res.Inertia.clearHistory();

    await res.Inertia.render('Login', {});
  }
}
```

```ts
// framework: koa
router.post("/logout", async (ctx) => {
  ctx.Inertia.clearHistory();

  await ctx.Inertia("Login", {});
});
```

Once the response has rendered on the client, the encryption key will be rotated, rendering the previous history state unreadable.

You can also clear history on the client site by calling `router.clearHistory()`.
