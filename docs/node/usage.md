# Usage

InertiaNode is a server-side adapter for Inertia.js. It allows you to use Inertia.js with Node.js frameworks like Hono, Express, and Koa.

It essentially allows you to use a front-end framework (React, Vue, Svelte) as if it were a server-rendered template, but maintain the SPA experience. It dramatically simplifies the process of building a modern web application with a modern front-end framework paired with a server-side framework.

## Routes

To pass data to a page component, use `Inertia()`.

```ts
// framework: hono
app.get("/posts", async (c) => {
  const posts = await db.posts.findMany();

  return await c.Inertia("Posts", {
    posts,
  });
});
```

```ts
// framework: express
app.get("/posts", async (req, res) => {
  const posts = await db.posts.findMany();

  await res.Inertia("Posts", {
    posts,
  });
});
```

```ts
// framework: nestjs
import { Controller, Get } from "@nestjs/common";
import { Inert, type Inertia } from "@inertianode/nestjs";

@Controller()
export class PostsController {
  @Get("/posts")
  async index(@Inert() inertia: Inertia) {
    const posts = await db.posts.findMany();

    await inertia("Posts", {
      posts,
    });
  }
}
```

```ts
// framework: koa
router.get("/posts", async (ctx) => {
  const posts = await db.posts.findMany();

  await ctx.Inertia("Posts", {
    posts,
  });
});
```

To make a form endpoint, remember that the request data is passed using JSON.

```ts
// framework: hono
app.post("/posts", async (c) => {
  const post = await c.req.json();

  // Validation errors are passed automatically
  const errors = validatePost(post);
  if (errors) {
    return await c.Inertia("Posts/Create", {
      errors,
      post,
    });
  }

  await db.posts.create(post);
  return c.redirect("/posts");
});
```

```ts
// framework: express
app.post("/posts", async (req, res) => {
  const post = req.body;

  // Validation errors are passed automatically
  const errors = validatePost(post);
  if (errors) {
    return await res.Inertia("Posts/Create", {
      errors,
      post,
    });
  }

  await db.posts.create(post);
  res.redirect("/posts");
});
```

```ts
// framework: nestjs
import { Controller, Post, Body } from "@nestjs/common";
import { Inert, type Inertia } from "@inertianode/nestjs";

@Controller()
export class PostsController {
  @Post("/posts")
  async create(@Body() post: any, @Inert() inertia: Inertia) {
    // Validation errors are passed automatically
    const errors = validatePost(post);
    if (errors) {
      return await inertia("Posts/Create", {
        errors,
        post,
      });
    }

    await db.posts.create(post);
    await inertia.redirect("/posts");
  }
}
```

```ts
// framework: koa
router.post("/posts", async (ctx) => {
  const post = ctx.request.body;

  // Validation errors are passed automatically
  const errors = validatePost(post);
  if (errors) {
    return await ctx.Inertia("Posts/Create", {
      errors,
      post,
    });
  }

  await db.posts.create(post);
  ctx.redirect("/posts");
});
```

# Features

## Shared data

You can add some shared data to your views using middleware:

```ts
// framework: hono
app.use("*", async (c, next) => {
  const userId = c.get("userId"); // From session middleware

  c.Inertia.share("auth", {
    userId,
  });

  // Or using an object
  c.Inertia.share({
    auth: {
      userId,
    },
  });

  await next();
});
```

```ts
// framework: express
app.use((req, res, next) => {
  const userId = req.session.userId;

  res.Inertia.share("auth", {
    userId,
  });

  // Or using an object
  res.Inertia.share({
    auth: {
      userId,
    },
  });

  next();
});
```

```ts
// framework: nestjs
import { Injectable, NestMiddleware } from "@nestjs/common";
import { Request, Response, NextFunction } from "express";

@Injectable()
export class InertiaSharedDataMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const userId = (req as any).session.userId;

    res.Inertia.share("auth", {
      userId,
    });

    // Or using an object
    res.Inertia.share({
      auth: {
        userId,
      },
    });

    next();
  }
}
```

```ts
// framework: koa
app.use(async (ctx, next) => {
  const userId = ctx.session.userId;

  ctx.Inertia.share("auth", {
    userId,
  });

  // Or using an object
  ctx.Inertia.share({
    auth: {
      userId,
    },
  });

  await next();
});
```

## Lazy Props

You can use lazy props to load data asynchronously. This is useful for loading data that is not needed for the initial render of the page.

```ts
// framework: hono
app.get("/posts", async (c) => {
  return await c.Inertia("Posts", {
    posts: async () => await db.posts.findMany(),
  });
});
```

```ts
// framework: express
app.get("/posts", async (req, res) => {
  await res.Inertia("Posts", {
    posts: async () => await db.posts.findMany(),
  });
});
```

```ts
// framework: nestjs
import { Controller, Get } from "@nestjs/common";
import { Inert, type Inertia } from "@inertianode/nestjs";

@Controller()
export class PostsController {
  @Get("/posts")
  async index(@Inert() inertia: Inertia) {
    await inertia("Posts", {
      posts: async () => await db.posts.findMany(),
    });
  }
}
```

```ts
// framework: koa
router.get("/posts", async (ctx) => {
  await ctx.Inertia("Posts", {
    posts: async () => await db.posts.findMany(),
  });
});
```
