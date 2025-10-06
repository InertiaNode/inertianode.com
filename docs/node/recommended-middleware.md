# Recommended Middleware

Node.js frameworks provide several middleware options. We can also add a couple of our own to enhance the experience.

InertiaNode recommends using the following middleware in your application:

1. [CSRF Protection (recommended)](#csrf-protection)
2. [Session (recommended)](#session)
3. [Shared Data Middleware (optional)](#shared-data-middleware)
4. [Exception Handling (optional)](#exception-handling)

## CSRF Protection

We strongly recommend using the CSRF protection middleware. Please see the [CSRF Protection](/csrf-protection.md) documentation for more information.

## Session

For the best experience, we recommend using session middleware. Session middleware is used to store state between requests. This is useful for flash messages, authentication state, or other data that you need to persist.


```ts
// framework: express
import express from 'express';
import session from 'express-session';
import { inertiaExpressAdapter } from '@inertianode/express';

const app = express();

// Session middleware
app.use(session({
  name: 'app.session',
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    maxAge: 1800000 // 30 minutes
  }
}));

// Inertia middleware
app.use(inertiaExpressAdapter());

// Static files
app.use(express.static('public'));
app.use(express.json());

// Your routes...
```

```ts
// framework: nestjs
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import * as session from 'express-session';
import { inertiaExpressAdapter } from '@inertianode/express';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Session middleware
  app.use(session({
    name: 'app.session',
    secret: 'your-secret-key',
    resave: false,
    saveUninitialized: false,
    cookie: {
      httpOnly: true,
      maxAge: 1800000 // 30 minutes
    }
  }));

  // Inertia middleware
  app.use(inertiaExpressAdapter());

  await app.listen(3000);
}
bootstrap();
```

```ts
// framework: koa
import Koa from 'koa';
import session from 'koa-session';
import { inertiaKoaAdapter } from '@inertianode/koa';
import serve from 'koa-static';

const app = new Koa();

// Session configuration
app.keys = ['your-secret-key'];
app.use(session({
  key: 'app.session',
  maxAge: 1800000, // 30 minutes
  httpOnly: true
}, app));

// Inertia middleware
app.use(inertiaKoaAdapter());

// Static files
app.use(serve('./public'));

// Your routes...
```


```ts
// framework: hono
import { Hono } from 'hono';
import { inertiaHonoAdapter } from '@inertianode/hono';
import { serveStatic } from '@hono/node-server/serve-static';

const app = new Hono();

// Inertia middleware
app.use(inertiaHonoAdapter());

// Static files
app.use('*', serveStatic({ root: './public' }));

// Your routes...
```

## Shared Data Middleware

We recommend creating custom middleware to handle Inertia shared data. This middleware is used to share data like flash messages, auth data, and other state that you want to include on every response. See the [Shared Data](/shared-data.md) documentation for more information.

> [!NOTE]
> This middleware is not required, but it is recommended for the best experience.
> It's worth noting every Inertia response will include the shared data, so it's important to only include the data that you need. You may want to cache heavy queries or make them lazy/optional to avoid performance issues.


```ts
// framework: express
app.use((req, res, next) => {
  // Share flash messages
  res.Inertia.share('flash', () => {
    const flash = req.session.flash || {};
    delete req.session.flash; // Clear flash after reading
    return flash;
  });

  // Share authentication data
  res.Inertia.share('auth', () => {
    if (req.user) {
      return {
        user: {
          id: req.user.id,
          first_name: req.user.firstName,
          last_name: req.user.lastName,
          email: req.user.email,
          owner: req.user.owner,
          account: {
            id: req.user.accountId,
            name: req.user.account?.name
          }
        }
      };
    }
    return { user: null };
  });

  next();
});
```

```ts
// framework: nestjs
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class InertiaSharedDataMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Share flash messages
    res.Inertia.share('flash', () => {
      const flash = (req as any).session.flash || {};
      delete (req as any).session.flash; // Clear flash after reading
      return flash;
    });

    // Share authentication data
    res.Inertia.share('auth', () => {
      if ((req as any).user) {
        return {
          user: {
            id: (req as any).user.id,
            first_name: (req as any).user.firstName,
            last_name: (req as any).user.lastName,
            email: (req as any).user.email,
            owner: (req as any).user.owner,
            account: {
              id: (req as any).user.accountId,
              name: (req as any).user.account?.name
            }
          }
        };
      }
      return { user: null };
    });

    next();
  }
}
```

```ts
// framework: koa
app.use(async (ctx, next) => {
  // Share flash messages
  ctx.Inertia.share('flash', () => {
    const flash = ctx.session.flash || {};
    delete ctx.session.flash; // Clear flash after reading
    return flash;
  });

  // Share authentication data
  ctx.Inertia.share('auth', () => {
    if (ctx.state.user) {
      return {
        user: {
          id: ctx.state.user.id,
          first_name: ctx.state.user.firstName,
          last_name: ctx.state.user.lastName,
          email: ctx.state.user.email,
          owner: ctx.state.user.owner,
          account: {
            id: ctx.state.user.accountId,
            name: ctx.state.user.account?.name
          }
        }
      };
    }
    return { user: null };
  });

  await next();
});
```


```ts
// framework: hono
app.use('*', async (c, next) => {
  // Share flash messages
  c.Inertia.share('flash', () => {
    const flash = c.get('flash') || {};
    c.set('flash', {}); // Clear flash after reading
    return flash;
  });

  // Share authentication data
  c.Inertia.share('auth', () => {
    const user = c.get('user');
    if (user) {
      return {
        user: {
          id: user.id,
          first_name: user.firstName,
          last_name: user.lastName,
          email: user.email,
          owner: user.owner,
          account: {
            id: user.accountId,
            name: user.account?.name
          }
        }
      };
    }
    return { user: null };
  });

  await next();
});
```

## Exception Handling

We recommend using exception handling middleware to handle 404s and 500-level exceptions. This allows you to return a custom error page for 404s and 500-level exceptions, yielding a better user experience.

> [!NOTE]
> You may wish to disable exception handling middleware during development to let error pages show in modals rendered by your development framework.

```ts
// framework: express
import express from 'express';

const app = express();

// Exception handling middleware (add this after your routes)
app.use((req, res, next) => {
  // Handle 404s
  if (!res.headersSent) {
    res.status(404);
    return res.Inertia.render('Error/NotFound');
  }
  next();
});

app.use((err, req, res, next) => {
  console.error(err);

  if (res.headersSent) {
    return next(err);
  }

  res.status(500);
  return res.Inertia.render('Error/ServerError', {
    message: process.env.NODE_ENV === 'development'
      ? err.message
      : 'An unexpected error occurred. Please try again later.',
    stackTrace: process.env.NODE_ENV === 'development' ? err.stack : null
  });
});
```

```ts
// framework: nestjs
import { ExceptionFilter, Catch, ArgumentsHost, HttpException, HttpStatus, NotFoundException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()
export class InertiaExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    console.error(exception);

    if (status === HttpStatus.NOT_FOUND) {
      response.status(404);
      return request.Inertia.render('Error/NotFound');
    }

    response.status(status);
    return request.Inertia.render('Error/ServerError', {
      message: process.env.NODE_ENV === 'development'
        ? (exception as Error).message
        : 'An unexpected error occurred. Please try again later.',
      stackTrace: process.env.NODE_ENV === 'development' ? (exception as Error).stack : null
    });
  }
}
```

```ts
// framework: koa
import Koa from 'koa';

const app = new Koa();

// Exception handling middleware
app.use(async (ctx, next) => {
  try {
    await next();

    // Handle 404s
    if (ctx.status === 404 && !ctx.body) {
      ctx.status = 404;
      await ctx.Inertia.render('Error/NotFound');
    }
  } catch (err) {
    console.error(err);
    ctx.status = err.status || 500;

    await ctx.Inertia.render('Error/ServerError', {
      message: process.env.NODE_ENV === 'development'
        ? err.message
        : 'An unexpected error occurred. Please try again later.',
      stackTrace: process.env.NODE_ENV === 'development' ? err.stack : null
    });
  }
});
```

```ts
// framework: hono
import { Hono } from 'hono';
// Hono uses per-request Inertia instance (c.Inertia);

const app = new Hono();

// Exception handling middleware
app.onError(async (err, c) => {
  console.error(err);

  return await c.Inertia('Error/ServerError', {
    message: process.env.NODE_ENV === 'development'
      ? err.message
      : 'An unexpected error occurred. Please try again later.',
    stackTrace: process.env.NODE_ENV === 'development' ? err.stack : null
  });
});

// Handle 404s
app.notFound(async (c) => {
  return await c.Inertia('Error/NotFound');
});
```
