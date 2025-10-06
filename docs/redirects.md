# Redirects

When making a non-GET Inertia request manually or via a `<Link>` element, you should ensure that you always respond with a proper Inertia redirect response.

For example, if your controller is creating a new user, your "store" endpoint should return a redirect back to a standard `GET` endpoint, such as your user "index" page. Inertia will automatically follow this redirect and update the page accordingly.

```ts
// framework: hono
import { Hono } from 'hono';
// Hono uses per-request Inertia instance (c.Inertia);

const app = new Hono();

app.get('/users', async (c) => {
  const users = await userService.getAllUsers();
  return await c.Inertia('Users/Index', {
    users
  });
});

app.post('/users', async (c) => {
  const { name, email } = await c.req.json();

  // Validate request
  if (!name || !email) {
    return c.json({ error: 'Invalid request' }, 400);
  }

  await userService.createUser({ name, email });

  // Redirect back to previous page, or to /users as fallback
  return c.redirect(c.req.header('Referer') || '/users', 303);
});
```

```ts
// framework: express
import express from 'express';

const app = express();

app.get('/users', async (req, res) => {
  const users = await userService.getAllUsers();
  await res.Inertia('Users/Index', {
    users
  });
});

app.post('/users', async (req, res) => {
  const { name, email } = req.body;

  // Validate request
  if (!name || !email) {
    return res.status(400).json({ error: 'Invalid request' });
  }

  await userService.createUser({ name, email });

  // Redirect back to previous page
  res.Inertia.back('/users');
});
```

```ts
// framework: nestjs
import { Controller, Get, Post, Body, Req, Res } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller('users')
export class UsersController {
  @Get()
  async index(@Req() req: Request, @Res() res: Response) {
    const users = await userService.getAllUsers();
    await res.Inertia.render('Users/Index', {
      users
    });
  }

  @Post()
  async store(@Body() body: { name: string; email: string }, @Req() req: Request, @Res() res: Response) {
    const { name, email } = body;

    // Validate request
    if (!name || !email) {
      return res.status(400).json({ error: 'Invalid request' });
    }

    await userService.createUser({ name, email });

    // Redirect back to previous page
    res.Inertia.back('/users');
  }
}
```

```ts
// framework: koa
import Koa from 'koa';
import Router from '@koa/router';

const app = new Koa();
const router = new Router();

router.get('/users', async (ctx) => {
  const users = await userService.getAllUsers();
  await ctx.Inertia('Users/Index', {
    users
  });
});

router.post('/users', async (ctx) => {
  const { name, email } = ctx.request.body;

  // Validate request
  if (!name || !email) {
    ctx.status = 400;
    ctx.body = { error: 'Invalid request' };
    return;
  }

  await userService.createUser({ name, email });

  // Redirect back to previous page
  ctx.Inertia.back('/users');
});

app.use(router.routes());
```

## 303 response code

When redirecting after a `PUT`, `PATCH`, or `DELETE` request, you must use a `303` response code, otherwise the subsequent request will not be treated as a `GET` request. A `303` redirect is very similar to a `302` redirect; however, the follow-up request is explicitly changed to a `GET` request.

The `back()` method automatically uses a `303` status code. For manual redirects, you can specify the status:

```ts
// framework: hono
return c.redirect('/users', 303);
```

```ts
// framework: express
res.redirect(303, '/users');
// Or use back() which handles it automatically
res.Inertia.back('/users');
```

```ts
// framework: nestjs
import { Controller, Put, Req, Res } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller('users')
export class UsersController {
  @Put(':id')
  async update(@Req() req: Request, @Res() res: Response) {
    // Update user logic...

    // Use 303 status for redirect after PUT/PATCH/DELETE
    res.redirect(303, '/users');
    // Or use back() which handles it automatically
    res.Inertia.back('/users');
  }
}
```

```ts
// framework: koa
ctx.status = 303;
ctx.redirect('/users');
// Or use back() which handles it automatically
ctx.Inertia.back('/users');
```

## External redirects

Sometimes it's necessary to redirect to an external website, or even another non-Inertia endpoint in your app while handling an Inertia request. This can be accomplished using a server-side initiated `window.location` visit via the `Inertia.location()` method.

```ts
// framework: hono
app.get('/external-redirect', async (c) => {
  return c.Inertia.location('https://example.com');
});
```

```ts
// framework: express
return res.Inertia.location('https://example.com');
```

```ts
// framework: nestjs
import { Controller, Get, Req, Res } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller()
export class AppController {
  @Get('/external-redirect')
  async externalRedirect(@Req() req: Request, @Res() res: Response) {
    return res.Inertia.location('https://example.com');
  }
}
```

```ts
// framework: koa
return ctx.Inertia.location('https://example.com');
```

The `location()` method will generate a `409 Conflict` response and include the destination URL in the `X-Inertia-Location` header. When this response is received client-side, Inertia will automatically perform a `window.location = url` visit.
