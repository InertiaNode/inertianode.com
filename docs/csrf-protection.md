# CSRF protection

## Making requests

For Node.js applications, CSRF protection can be implemented using middleware packages like `csurf` (for Express) or `koa-csrf` (for Koa). We recommend setting the token header name to `X-XSRF-TOKEN` to match the default header name used by Axios.

```ts
// framework: express
import express from 'express';
import csrf from 'csurf';
import cookieParser from 'cookie-parser';

const app = express();

// Setup cookie parser and CSRF protection
app.use(cookieParser());
const csrfProtection = csrf({ cookie: true });

// Share CSRF token with all Inertia requests
app.use((req, res, next) => {
  if (req.csrfToken) {
    res.Inertia.share('csrf_token', req.csrfToken());
  }
  next();
});

// Apply CSRF protection to routes
app.post('/users', csrfProtection, async (req, res) => {
  // Create user...
  res.redirect('/users');
});
```

```ts
// framework: koa
import Koa from 'koa';
import csrf from 'koa-csrf';
import session from 'koa-session';

const app = new Koa();

// Setup session and CSRF protection
app.keys = ['your-secret'];
app.use(session(app));
app.use(new csrf());

// Share CSRF token with all Inertia requests
app.use(async (ctx, next) => {
  ctx.Inertia.share('csrf_token', ctx.csrf);
  await next();
});

// CSRF is automatically validated for non-GET requests
router.post('/users', async (ctx) => {
  // Create user...
  ctx.redirect('/users');
});
```

```ts
// framework: hono
import { Hono } from 'hono';
import { csrf } from 'hono/csrf';

const app = new Hono();

// Apply CSRF middleware
app.use('*', csrf());

// Share CSRF token with all Inertia requests
app.use('*', async (c, next) => {
  const token = c.req.header('X-CSRF-Token') || '';
  c.Inertia.share('csrf_token', token);
  await next();
});

// CSRF is automatically validated for non-GET requests
app.post('/users', async (c) => {
  // Create user...
  return c.redirect('/users');
});
```

This ensures each Inertia request includes the necessary CSRF token for `POST`, `PUT`, `PATCH`, and `DELETE` requests.

However, if you want to handle anti-forgery protection manually, one approach is to include the anti-forgery token as a prop on every response. You can then use the token when making Inertia requests.

```js
// framework: vue
import { router, usePage } from "@inertiajs/vue3";
const page = usePage();
router.post("/users", {
  _token: page.props.csrf_token,
  name: "John Doe",
  email: "john.doe@example.com",
});
```

```js
// framework: react
import { router, usePage } from "@inertiajs/react";
const props = usePage().props;
router.post("/users", {
  _token: props.csrf_token,
  name: "John Doe",
  email: "john.doe@example.com",
});
```

```js
// framework: svelte
import { page, router } from "@inertiajs/svelte";
router.post("/users", {
  _token: $page.props.csrf_token,
  name: "John Doe",
  email: "john.doe@example.com",
});
```

You can even use Inertia's [shared data](/shared-data) functionality to automatically include the `csrf_token` with each response.

However, a better approach is to use the anti-forgery functionality already built into [axios](https://github.com/axios/axios) for this. Axios is the HTTP library that Inertia uses under the hood.

Axios automatically checks for the existence of an `XSRF-TOKEN` cookie. If it's present, it will then include the token in an `X-XSRF-TOKEN` header for any requests it makes.

The easiest way to implement this is to configure your CSRF middleware to include the `XSRF-TOKEN` cookie on each response, and then verify the token using the `X-XSRF-TOKEN` header sent in the requests from axios.

## Handling mismatches

When an anti-forgery token mismatch occurs, your server-side framework will likely throw an exception that results in an error response. For example, CSRF middleware typically returns a `403` or `400` error page. Since that isn't a valid Inertia response, the error is shown in a modal.

<video controls="" style="max-width: 100%; margin: 30px auto;"><source src="https://inertiajs.com/mp4/csrf-mismatch-modal.mp4" type="video/mp4"></source></video>Obviously, this isn't a great user experience. A better way to handle these errors is to return a redirect back to the previous page, along with a flash message that the page expired. This will result in a valid Inertia response with the flash message available as a prop which you can then display to the user. Of course, you'll need to share your [flash messages](/shared-data#flash-messages) with Inertia for this to work.

You can implement custom error handling in your CSRF middleware to redirect the user back to the page they were previously on with an appropriate message. Here's an example:

```ts
// framework: hono
app.use('*', async (c, next) => {
  try {
    await csrf()(c, next);
  } catch (error) {
    // CSRF token mismatch - redirect back with error
    c.Inertia.share('error', 'The page expired, please try again.');
    return c.Inertia.back();
  }
});
```

```ts
// framework: express
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    // CSRF token mismatch - redirect back with error
    res.Inertia.share('error', 'The page expired, please try again.');
    return res.Inertia.back();
  }
  next(err);
});
```

```ts
// framework: koa
app.use(async (ctx, next) => {
  try {
    await next();
  } catch (err) {
    if (err.status === 403) {
      // CSRF token mismatch - redirect back with error
      ctx.Inertia.share('error', 'The page expired, please try again.');
      return ctx.Inertia.back();
    }
    throw err;
  }
});
```

The end result is a much better experience for your users. Instead of seeing the error modal, the user is instead presented with a message that the page "expired" and are asked to try again.

<video controls="" style="max-width: 100%; margin: 30px auto;"><source src="https://inertiajs.com/mp4/csrf-mismatch-warning.mp4" type="video/mp4"></source></video>
