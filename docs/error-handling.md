# Error handling

> [!NOTE]
> We have a another example of how to handle errors in the [Recommended middleware](/core/recommended-middleware.md) documentation, which includes 404 and 500 error handling.

## Development

One of the advantages to working with a robust server-side framework is the built-in exception handling you get for free. For example, Node.js frameworks like Express, Koa, and Hono provide error handling mechanisms that display nicely formatted stack traces and detailed exception information in local development.

The challenge is, if you're making an XHR request (which Inertia does) and you hit a server-side error, you're typically left digging through the network tab in your browser's devtools to diagnose the problem.

Inertia solves this issue by showing all non-Inertia responses in a modal. This means you get the same beautiful error-reporting you're accustomed to, even though you've made that request over XHR.

## Production

In production you will want to return a proper Inertia error response instead of relying on the modal-driven error reporting that is present during development. To accomplish this, you'll need to update your framework's default exception handler to return a custom error page.

When building Node.js applications, you can accomplish this by configuring a custom exception handling middleware in your application's startup configuration.

```ts
// framework: hono
import { Hono } from "hono";
// Hono uses per-request Inertia instance (c.Inertia);

const app = new Hono();

app.onError(async (err, c) => {
  const isDevelopment = process.env.NODE_ENV === "development";

  if (isDevelopment) {
    throw err; // Show detailed error in development
  }

  const statusCode = err.status || 500;
  c.status(statusCode);

  return await c.Inertia("ErrorPage", {
    status: statusCode,
  });
});
```

```ts
// framework: express
import express from "express";

const app = express();

// Error handling middleware (must be last)
app.use(async (err, req, res, next) => {
  const isDevelopment = process.env.NODE_ENV === "development";

  if (isDevelopment) {
    return next(err); // Show detailed error in development
  }

  const statusCode = err.status || 500;
  res.status(statusCode);

  await res.Inertia("ErrorPage", {
    status: statusCode,
  });
});
```

```ts
// framework: nestjs
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()
export class InertiaExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const isDevelopment = process.env.NODE_ENV === 'development';

    if (isDevelopment) {
      // Re-throw in development for detailed error display
      throw exception;
    }

    const statusCode = exception instanceof HttpException
      ? exception.getStatus()
      : 500;

    response.status(statusCode);
    (request as any).Inertia.render('ErrorPage', {
      status: statusCode,
    });
  }
}

// Register in main.ts:
// app.useGlobalFilters(new InertiaExceptionFilter());
```

```ts
// framework: koa
import Koa from "koa";

const app = new Koa();

app.use(async (ctx, next) => {
  try {
    await next();
  } catch (err) {
    const isDevelopment = process.env.NODE_ENV === "development";

    if (isDevelopment) {
      throw err; // Show detailed error in development
    }

    const statusCode = err.status || 500;
    ctx.status = statusCode;

    await ctx.Inertia("ErrorPage", {
      status: statusCode,
    });
  }
});
```

You may have noticed we're returning an `ErrorPage` page component in the example above. You'll need to actually create this component, which will serve as the generic error page for your application. Here's an example error component you can use as a starting point.

```vue
// framework: vue
<script setup>
import { computed } from "vue";
const props = defineProps({ status: Number });
const title = computed(() => {
  return {
    503: "503: Service Unavailable",
    500: "500: Server Error",
    404: "404: Page Not Found",
    403: "403: Forbidden",
  }[props.status];
});
const description = computed(() => {
  return {
    503: "Sorry, we are doing some maintenance. Please check back soon.",
    500: "Whoops, something went wrong on our servers.",
    404: "Sorry, the page you are looking for could not be found.",
    403: "Sorry, you are forbidden from accessing this page.",
  }[props.status];
});
</script>
<template>
  <div>
    <h1>{{ title }}</h1>
    <div>{{ description }}</div>
  </div>
</template>
```

```jsx
// framework: react
export default function ErrorPage({ status }) {
  const title = {
    503: "503: Service Unavailable",
    500: "500: Server Error",
    404: "404: Page Not Found",
    403: "403: Forbidden",
  }[status];
  const description = {
    503: "Sorry, we are doing some maintenance. Please check back soon.",
    500: "Whoops, something went wrong on our servers.",
    404: "Sorry, the page you are looking for could not be found.",
    403: "Sorry, you are forbidden from accessing this page.",
  }[status];
  return (
    <div>
      <H1>{title}</H1>
      <div>{description}</div>
    </div>
  );
}
```

```svelte
// framework: svelte4
<script>
export let status
$: title = {
  503: '503: Service Unavailable',
  500: '500: Server Error',
  404: '404: Page Not Found',
  403: '403: Forbidden',
}[status]
$: description = {
  503: 'Sorry, we are doing some maintenance. Please check back soon.',
  500: 'Whoops, something went wrong on our servers.',
  404: 'Sorry, the page you are looking for could not be found.',
  403: 'Sorry, you are forbidden from accessing this page.',
}[status]
</script>
<div>
<h1>{title}</h1>
<div>{description}</div>
</div>
```

```svelte
// framework: svelte5
<script>
let { status } = $props()
const title = {
  503: '503: Service Unavailable',
  500: '500: Server Error',
  404: '404: Page Not Found',
  403: '403: Forbidden',
}
const description = {
  503: 'Sorry, we are doing some maintenance. Please check back soon.',
  500: 'Whoops, something went wrong on our servers.',
  404: 'Sorry, the page you are looking for could not be found.',
  403: 'Sorry, you are forbidden from accessing this page.',
}
</script>
<div>
<h1>{title[status]}</h1>
<div>{description[status]}</div>
</div>
```
