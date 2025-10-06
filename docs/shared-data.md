# Shared data

Sometimes you need to access specific pieces of data on numerous pages within your application. For example, you may need to display the current user in the site header. Passing this data manually in each response across your entire application is cumbersome. Thankfully, there is a better option: shared data.

## Sharing data

Inertia's server-side adapters all provide a method of making shared data available for every request. This is typically done in middleware. Shared data will be automatically merged with the page props provided in your routes.

```ts
// framework: hono
import { Hono } from "hono";

const app = new Hono();

app.use("*", async (c, next) => {
  // Use the request-specific Inertia instance
  // Synchronously...
  c.Inertia.share("appName", process.env.APP_NAME);

  // Lazily...
  c.Inertia.share("auth", () => {
    return c.get("user")
      ? {
          user: {
            id: c.get("user").id,
            name: c.get("user").name,
            email: c.get("user").email,
          },
        }
      : null;
  });

  await next();
});
```

```ts
// framework: express
import express from "express";

const app = express();

app.use((req, res, next) => {
  // Use the request-specific Inertia instance
  // Synchronously...
  res.Inertia.share("appName", process.env.APP_NAME);

  // Lazily...
  res.Inertia.share("auth", () => {
    return req.user
      ? {
          user: {
            id: req.user.id,
            name: req.user.name,
            email: req.user.email,
          },
        }
      : null;
  });

  next();
});
```

```ts
// framework: nestjs
import { Injectable, NestMiddleware } from "@nestjs/common";
import { Request, Response, NextFunction } from "express";

@Injectable()
export class ShareDataMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Use the request-specific Inertia instance
    // Synchronously...
    res.Inertia.share("appName", process.env.APP_NAME);

    // Lazily...
    res.Inertia.share("auth", () => {
      return req.user
        ? {
            user: {
              id: req.user.id,
              name: req.user.name,
              email: req.user.email,
            },
          }
        : null;
    });

    next();
  }
}

// Register in AppModule:
// consumer.apply(ShareDataMiddleware).forRoutes('*');
```

```ts
// framework: koa
import Koa from "koa";

const app = new Koa();

app.use(async (ctx, next) => {
  // Use the request-specific Inertia instance
  // Synchronously...
  ctx.Inertia.share("appName", process.env.APP_NAME);

  // Lazily...
  ctx.Inertia.share("auth", () => {
    return ctx.state.user
      ? {
          user: {
            id: ctx.state.user.id,
            name: ctx.state.user.name,
            email: ctx.state.user.email,
          },
        }
      : null;
  });

  await next();
});
```

Shared data should be used sparingly as all shared data is included with every response.

Page props and shared data are merged together, so be sure to namespace your shared data appropriately to avoid collisions.

## Accessing shared data

Once you have shared the data server-side, you will be able to access it within any of your pages or components. Here's an example of how to access shared data in a layout component.

```vue
<!-- framework: vue -->
<script setup>
import { computed } from "vue";
import { usePage } from "@inertiajs/vue3";
const page = usePage();
const user = computed(() => page.props.auth.user);
</script>
<template>
  <main>
    <header>You are logged in as: {{ user.name }}</header>
    <article>
      <slot />
    </article>
  </main>
</template>
```

```jsx
// framework: react
import { usePage } from "@inertiajs/react";
export default function Layout({ children }) {
  const { auth } = usePage().props;
  return (
    <main>
      <header>You are logged in as: {auth.user.name}</header>
      <article>{children}</article>
    </main>
  );
}
```

```svelte
<!-- framework: svelte -->
<script>
  import { page } from "@inertiajs/svelte";
</script>
<main>
  <header>You are logged in as: {$page.props.auth.user.name}</header>
  <article>
    <slot />
  </article>
</main>
```

## Flash messages

Another great use-case for shared data is flash messages. These are messages stored in the session only for the next request. For example, it's common to set a flash message after completing a task and before redirecting to a different page.

Here's a simple way to implement flash messages in your Inertia applications using session middleware. First, install a session package:

```bash
# For Express
npm install express-session

# For Koa
npm install koa-session
```

Then share the flash message on each request:

```ts
// framework: hono (using a simple session implementation)
import { Hono } from "hono";

const app = new Hono();

app.use("*", async (c, next) => {
  // Use request-specific Inertia instance
  c.Inertia.share("flash", () => {
    const flash = c.get("flash") || {};
    c.set("flash", {}); // Clear flash after reading
    return flash;
  });

  await next();
});

// In your route:
app.post("/users", async (c) => {
  // Create user...
  c.set("flash", { message: "User created successfully!" });
  return c.redirect("/users");
});
```

```ts
// framework: express
import express from "express";
import session from "express-session";

const app = express();

app.use(
  session({ secret: "your-secret", resave: false, saveUninitialized: false })
);

app.use((req, res, next) => {
  // Use request-specific Inertia instance
  res.Inertia.share("flash", () => {
    const flash = req.session.flash || {};
    delete req.session.flash; // Clear flash after reading
    return flash;
  });

  next();
});

// In your route:
app.post("/users", async (req, res) => {
  // Create user...
  req.session.flash = { message: "User created successfully!" };
  res.redirect("/users");
});
```

```ts
// framework: nestjs
import {
  Module,
  NestModule,
  MiddlewareConsumer,
  Injectable,
  NestMiddleware,
  Controller,
  Post,
  Body,
} from "@nestjs/common";
import { Request, Response, NextFunction } from "express";
import * as session from "express-session";
import { Inert, type Inertia } from "@inertianode/nestjs";

@Injectable()
export class FlashMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Use request-specific Inertia instance
    res.Inertia.share("flash", () => {
      const flash = (req.session as any).flash || {};
      delete (req.session as any).flash; // Clear flash after reading
      return flash;
    });

    next();
  }
}

@Module({
  // ...
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(
        session({
          secret: "your-secret",
          resave: false,
          saveUninitialized: false,
        }),
        FlashMiddleware
      )
      .forRoutes("*");
  }
}

@Controller("users")
export class UsersController {
  @Post()
  async store(@Inert() inertia: Inertia) {
    // Create user...
    // Note: Flash messages are typically set via request.session in middleware
    // This redirect will pick up flash messages from the session
    await inertia.redirect("/users");
  }
}
```

```ts
// framework: koa
import Koa from "koa";
import session from "koa-session";

const app = new Koa();
app.keys = ["your-secret"];
app.use(session(app));

app.use(async (ctx, next) => {
  // Use request-specific Inertia instance
  ctx.Inertia.share("flash", () => {
    const flash = ctx.session.flash || {};
    delete ctx.session.flash; // Clear flash after reading
    return flash;
  });

  await next();
});

// In your route:
router.post("/users", async (ctx) => {
  // Create user...
  ctx.session.flash = { message: "User created successfully!" };
  ctx.redirect("/users");
});
```

Next, display the flash message in a frontend component, such as the site layout.

```vue
<!-- framework: vue -->
<template>
  <main>
    <header></header>
    <article>
      <div v-if="$page.props.flash.message" class="alert">
        {{ $page.props.flash.message }}
      </div>
      <slot />
    </article>
    <footer></footer>
  </main>
</template>
```

```jsx
// framework: react
import { usePage } from "@inertiajs/react";
export default function Layout({ children }) {
  const { flash } = usePage().props;
  return (
    <main>
      <header></header>
      <article>
        {flash.message && <div class="alert">{flash.message}</div>}
        {children}
      </article>
      <footer></footer>
    </main>
  );
}
```

```svelte
<!-- framework: svelte -->
<script>
  import { page } from "@inertiajs/svelte";
</script>
<main>
  <header></header>
  <article>
    {#if $page.props.flash.message}
    <div class="alert">{$page.props.flash.message}</div>
    {/if}
    <slot />
  </article>
  <footer></footer>
</main>
```
