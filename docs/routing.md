# Routing

## Defining routes

When using Inertia, all of your application's routes are defined server-side. This means that you don't need Vue Router or React Router. Instead, you can simply define Node.js routes and return [Inertia responses](/responses) from those routes.

## Shorthand routes

If you have a [page](/pages) that doesn't need a corresponding controller method, like an "FAQ" or "about" page, you can route directly to a component using minimal controller actions with attribute routing.

```ts
// framework: hono
import { Hono } from "hono";

const app = new Hono();

app.get("/about", async (c) => {
  return await c.Inertia("About");
});

app.get("/faq", async (c) => {
  return await c.Inertia("FAQ");
});
```

```ts
// framework: express
import express from "express";

const app = express();

app.get("/about", async (req, res) => {
  await res.Inertia("About");
});

app.get("/faq", async (req, res) => {
  await res.Inertia("FAQ");
});
```

```ts
// framework: nestjs
import { Controller, Get, Req, Res } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller()
export class PagesController {
  @Get('/about')
  async about(@Req() req: Request, @Res() res: Response) {
    await res.Inertia.render('About');
  }

  @Get('/faq')
  async faq(@Req() req: Request, @Res() res: Response) {
    await res.Inertia.render('FAQ');
  }
}
```

```ts
// framework: koa
import Koa from "koa";
import Router from "@koa/router";

const app = new Koa();
const router = new Router();

router.get("/about", async (ctx) => {
  await ctx.Inertia("About");
});

router.get("/faq", async (ctx) => {
  await ctx.Inertia("FAQ");
});

app.use(router.routes());
```

<!-- TODO: Build a route generator for TypeScript and Node.js -->

## Customizing the Page URL

The [page object](/the-protocol#the-page-object) includes a `url` that represents the current page's URL. By default, InertiaNode resolves this using the request path and query string, returning a relative URL.

If you need to customize how the URL is resolved, you can configure it in middleware:

```ts
// framework: hono
app.use("*", async (c, next) => {
  c.Inertia.resolveUrlUsing(() => {
    return (
      c.req.path +
      (c.req.query
        ? "?" +
          new URLSearchParams(c.req.query as Record<string, string>).toString()
        : "")
    );
  });
  await next();
});
```

```ts
// framework: express
import { Inertia } from "@inertianode/core";

app.use((req, res, next) => {
  Inertia.resolveUrlUsing(() => {
    return (
      req.path +
      (req.query && Object.keys(req.query).length
        ? "?" +
          new URLSearchParams(req.query as Record<string, string>).toString()
        : "")
    );
  });
  next();
});
```

```ts
// framework: nestjs
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { Inertia } from '@inertianode/core';

@Injectable()
export class ResolveUrlMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    Inertia.resolveUrlUsing(() => {
      return (
        req.path +
        (req.query && Object.keys(req.query).length
          ? '?' +
            new URLSearchParams(req.query as Record<string, string>).toString()
          : '')
      );
    });
    next();
  }
}
```

```ts
// framework: koa
import { Inertia } from "@inertianode/core";

app.use(async (ctx, next) => {
  Inertia.resolveUrlUsing(() => {
    return ctx.path + (ctx.querystring ? "?" + ctx.querystring : "");
  });
  await next();
});
```
