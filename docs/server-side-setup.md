# Server-side setup

The first step when installing Inertia is to configure your server-side framework. InertiaNode provides server-side adapters for [Hono](https://hono.dev/), [Express](https://expressjs.com/), [NestJS](https://nestjs.com/), and [Koa](https://koajs.com/). For other frameworks, please see the [community adapters](https://inertiajs.com/community-adapters).

The documentation examples on this website utilize Hono, Express, NestJS, and Koa. For examples of using Inertia with other server-side frameworks, please refer to the framework specific documentation maintained by that adapter.

## Project templates

If you want a batteries-included experience, we recommend using one of the project templates from the [quick start guide](/core/quick-start.md).

InertiaNode project templates provide out-of-the-box scaffolding for new Inertia applications. These templates are the absolute fastest way to start building a new Inertia project using Node.js with Vue, React, or other frontend frameworks. However, if you would like to manually install Inertia into your application, please consult the documentation below.

## Install dependencies

First, install the Inertia server-side adapter for your framework of choice:

```bash
# Hono
npm install @inertianode/hono @inertianode/core hono

# Express
npm install @inertianode/express @inertianode/core express

# NestJS
npm install @inertianode/nestjs @inertianode/core @nestjs/common @nestjs/core @nestjs/platform-express

# Koa
npm install @inertianode/koa @inertianode/core koa @koa/router
```

## Root template

Next, setup the root template that will be loaded on the first page visit to your application. This template should include your site's CSS and JavaScript assets, along with the Inertia mounting point. InertiaNode uses a templating function to generate the HTML.

Create a root template function (typically in a separate file):

```ts
import type { Page } from '@inertianode/core';
import { viteAssets, inertiaHead, inertiaBody } from '@inertianode/core';

export function rootTemplate(page: Page, viewData: any = {}): string {
  const ssr = viewData.ssr; // SSR response if enabled

  return `<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    ${viteAssets('src/app.tsx')}
    ${inertiaHead(ssr?.head)}
  </head>
  <body>
    ${inertiaBody(page, ssr?.body)}
  </body>
</html>`;
}
```

### Helper Functions

InertiaNode provides several helper functions to simplify template creation:

- **`viteAssets(entrypoints)`** - Automatically detects development or production mode and loads the appropriate assets. Handles the Vite dev server in development and loads hashed assets from the manifest in production.

- **`inertiaHead(ssrHead?)`** - Renders SSR head content when server-side rendering is enabled. For non-SSR applications, this returns an empty string and head content is managed client-side using the `<Head>` component.

- **`inertiaBody(page, ssrBody?, appId?)`** - Renders the Inertia app mount point. When SSR is enabled, it renders the pre-rendered HTML. Otherwise, it renders a div with serialized page data. The `appId` parameter is optional (defaults to `'app'`).

## App Setup

Next we need to setup the Inertia middleware. The setup varies slightly between frameworks:

```ts
// framework: hono
import { Hono } from 'hono';
import { inertiaHonoAdapter } from '@inertianode/hono';
import { serveStatic } from '@hono/node-server/serve-static';
import { rootTemplate } from './templates/root';

const app = new Hono();

// Add Inertia middleware with configuration
app.use('*', inertiaHonoAdapter({
  html: (page) => rootTemplate(page),
  vite: {
    reactRefresh: true, // Enable React Fast Refresh (for React apps)
  }
}));

// Serve static files
app.use('*', serveStatic({ root: './public' }));

// Your routes go here
app.get('/', async (c) => {
  return await c.Inertia('Index', {
    title: 'Welcome'
  });
});
```

```ts
// framework: express
import express from 'express';
import { inertiaExpressAdapter } from '@inertianode/express';
import { rootTemplate } from './templates/root';

const app = express();

// Parse JSON
app.use(express.json());

// Add Inertia middleware with configuration
app.use(inertiaExpressAdapter({
  html: (page) => rootTemplate(page),
  vite: {
    reactRefresh: true, // Enable React Fast Refresh (for React apps)
  }
}));

// Serve static files
app.use(express.static('public'));

// Your routes go here
app.get('/', async (req, res) => {
  await res.Inertia.render('Index', {
    title: 'Welcome'
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

```ts
// framework: nestjs
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { inertiaNestJSAdapter } from '@inertianode/nestjs';
import { ServeStaticModule } from '@nestjs/serve-static';
import { rootTemplate } from './templates/root';
import { join } from 'path';

@Module({
  imports: [
    ServeStaticModule.forRoot({
      rootPath: join(__dirname, '..', 'public'),
    }),
  ],
  controllers: [AppController],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(
        inertiaNestJSAdapter({
          html: (page) => rootTemplate(page),
          vite: {
            reactRefresh: true, // Enable React Fast Refresh (for React apps)
          }
        })
      )
      .forRoutes('*');
  }
}

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
  console.log('Server running on port 3000');
}

bootstrap();
```

```ts
// framework: koa
import Koa from 'koa';
import Router from '@koa/router';
import { inertiaKoaAdapter } from '@inertianode/koa';
import serve from 'koa-static';
import { rootTemplate } from './templates/root';

const app = new Koa();
const router = new Router();

// Add Inertia middleware with configuration
app.use(inertiaKoaAdapter({
  html: (page) => rootTemplate(page),
  vite: {
    reactRefresh: true, // Enable React Fast Refresh (for React apps)
  }
}));

// Serve static files
app.use(serve('./public'));

// Your routes go here
router.get('/', async (ctx) => {
  await ctx.Inertia.render('Index', {
    title: 'Welcome'
  });
});

app.use(router.routes());

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

The Inertia middleware provides configuration options for setting your [asset version](/asset-versioning), as well as methods for defining [shared data](/shared-data).

For the best possible experience, you may also want to visit the recommended middleware setup section in the [Middleware Recommendations](/core/recommended-middleware.md) documentation.

## Creating responses

That's it, you're all ready to go server-side! Now you're ready to start creating Inertia [pages](/pages) and rendering them via [responses](/responses).

```ts
// framework: hono
app.get('/events/:id', async (c) => {
  const eventItem = await db.events.find(c.req.param('id'));

  // Using the callable syntax
  return await c.Inertia('Event/Show', {
    event: {
      id: eventItem.id,
      title: eventItem.title,
      startDate: eventItem.startDate,
      description: eventItem.description
    }
  });
});
```

```ts
// framework: express
app.get('/events/:id', async (req, res) => {
  const eventItem = await db.events.find(req.params.id);

  // Using the render method
  await res.Inertia.render('Event/Show', {
    event: {
      id: eventItem.id,
      title: eventItem.title,
      startDate: eventItem.startDate,
      description: eventItem.description
    }
  });
});
```

```ts
// framework: nestjs
import { Controller, Get, Param, Req, Res } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller('events')
export class EventsController {
  @Get(':id')
  async show(@Param('id') id: string, @Req() req: Request, @Res() res: Response) {
    const eventItem = await db.events.find(id);

    // Using the render method
    await res.Inertia.render('Event/Show', {
      event: {
        id: eventItem.id,
        title: eventItem.title,
        startDate: eventItem.startDate,
        description: eventItem.description
      }
    });
  }
}
```

```ts
// framework: koa
router.get('/events/:id', async (ctx) => {
  const eventItem = await db.events.find(ctx.params.id);

  // Using the render method
  await ctx.Inertia.render('Event/Show', {
    event: {
      id: eventItem.id,
      title: eventItem.title,
      startDate: eventItem.startDate,
      description: eventItem.description
    }
  });
});
```
