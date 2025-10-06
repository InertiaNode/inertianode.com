# Responses

## Creating responses

Creating an Inertia response is simple. To get started, invoke `Inertia()` as a function within your controller or route, providing both the name of the [JavaScript page component](/pages) that you wish to render, as well as any properties (data) for the page.

In the example below, we will pass a single property (`event`) which contains four attributes ( `id`, `title`, `start_date` and `description`) to the `Event/Show` page component.

```ts
// framework: hono
import { Hono } from "hono";

const app = new Hono();

app.get("/events/:id", async (c) => {
  const id = c.req.param("id");
  const eventItem = await eventService.getEvent(id);

  return await c.Inertia("Event/Show", {
    event: {
      id: eventItem.id,
      title: eventItem.title,
      startDate: eventItem.startDate,
      description: eventItem.description,
    },
  });
});
```

```ts
// framework: express
import express from "express";

const app = express();

app.get("/events/:id", async (req, res) => {
  const eventItem = await eventService.getEvent(req.params.id);

  await res.Inertia("Event/Show", {
    event: {
      id: eventItem.id,
      title: eventItem.title,
      startDate: eventItem.startDate,
      description: eventItem.description,
    },
  });
});
```

```ts
// framework: nestjs
import { Controller, Get, Param } from "@nestjs/common";
import { Inert, type Inertia } from "@inertianode/nestjs";

@Controller()
export class EventsController {
  @Get("/events/:id")
  async show(@Param("id") id: string, @Inert() inertia: Inertia) {
    const eventItem = await eventService.getEvent(id);

    await inertia("Event/Show", {
      event: {
        id: eventItem.id,
        title: eventItem.title,
        startDate: eventItem.startDate,
        description: eventItem.description,
      },
    });
  }
}
```

```ts
// framework: koa
import Koa from "koa";
import Router from "@koa/router";

const app = new Koa();
const router = new Router();

router.get("/events/:id", async (ctx) => {
  const eventItem = await eventService.getEvent(ctx.params.id);

  await ctx.Inertia("Event/Show", {
    event: {
      id: eventItem.id,
      title: eventItem.title,
      startDate: eventItem.startDate,
      description: eventItem.description,
    },
  });
});

app.use(router.routes());
```

> **Note:** You can also use `Inertia()` as a shorthand for `Inertia.render()`. Both syntaxes work identically - `Inertia()` is simply more concise.

To ensure that pages load quickly, only return the minimum data required for the page. Also, be aware that all data returned from the controllers will be visible client-side, so be sure to omit sensitive information.

## Properties

To pass data from the server to your page components, you can use properties. You can pass various types of values as props, including primitive types, arrays, objects, and functions:

```ts
// framework: hono
app.get("/dashboard", async (c) => {
  return await c.Inertia("Dashboard", {
    // Primitive values
    title: "Dashboard",
    count: 42,
    active: true,
    // Objects and arrays
    settings: { theme: "dark", notifications: true },
    // Database models
    user: await userService.getCurrentUser(),
    users: await userService.getAllUsers(),
    // DTOs and custom objects
    profile: new UserProfileDto(await userService.getCurrentUser()),
    // Plain objects
    data: { key: "value" },
    // Computed properties using functions
    timestamp: () => Math.floor(Date.now() / 1000),
  });
});
```

```ts
// framework: express
app.get("/dashboard", async (req, res) => {
  await res.Inertia("Dashboard", {
    // Primitive values
    title: "Dashboard",
    count: 42,
    active: true,
    // Objects and arrays
    settings: { theme: "dark", notifications: true },
    // Database models
    user: await userService.getCurrentUser(),
    users: await userService.getAllUsers(),
    // DTOs and custom objects
    profile: new UserProfileDto(await userService.getCurrentUser()),
    // Plain objects
    data: { key: "value" },
    // Computed properties using functions
    timestamp: () => Math.floor(Date.now() / 1000),
  });
});
```

```ts
// framework: nestjs
import { Controller, Get } from "@nestjs/common";
import { Inert, type Inertia } from "@inertianode/nestjs";

@Controller()
export class DashboardController {
  @Get("/dashboard")
  async index(@Inert() inertia: Inertia) {
    await inertia("Dashboard", {
      // Primitive values
      title: "Dashboard",
      count: 42,
      active: true,
      // Objects and arrays
      settings: { theme: "dark", notifications: true },
      // Database models
      user: await userService.getCurrentUser(),
      users: await userService.getAllUsers(),
      // DTOs and custom objects
      profile: new UserProfileDto(await userService.getCurrentUser()),
      // Plain objects
      data: { key: "value" },
      // Computed properties using functions
      timestamp: () => Math.floor(Date.now() / 1000),
    });
  }
}
```

```ts
// framework: koa
router.get("/dashboard", async (ctx) => {
  await ctx.Inertia("Dashboard", {
    // Primitive values
    title: "Dashboard",
    count: 42,
    active: true,
    // Objects and arrays
    settings: { theme: "dark", notifications: true },
    // Database models
    user: await userService.getCurrentUser(),
    users: await userService.getAllUsers(),
    // DTOs and custom objects
    profile: new UserProfileDto(await userService.getCurrentUser()),
    // Plain objects
    data: { key: "value" },
    // Computed properties using functions
    timestamp: () => Math.floor(Date.now() / 1000),
  });
});
```

<!-- TODO: Verify this -->
<!-- TODO: Enable MergeProps and MergeStrategies -->
<!-- Objects are automatically serialized to JSON using JavaScript's built-in serialization.

## Custom prop transformers

When passing props to your components, you may want to create custom classes that can transform themselves into the appropriate data format. You can implement this pattern using functions or classes.

Here's an example of a custom prop transformer for user avatars:

```ts
interface User {
  avatar?: string;
  name: string;
}

class UserAvatar {
  constructor(private user: User, private size: number = 64) {}

  toJSON() {
    return this.user.avatar
      ? `/storage/${this.user.avatar}`
      : `https://ui-avatars.com/api/?name=${encodeURIComponent(this.user.name)}&size=${this.size}`;
  }
}
```

Once defined, you can use this class directly as a prop value.

```ts
// framework: express
app.get('/profile', async (req, res) => {
  const user = await userService.getCurrentUser();

  await res.Inertia('Profile', {
    user: user,
    avatar: new UserAvatar(user, 128)
  });
});
```

```ts
// framework: nestjs
import { Controller, Get } from '@nestjs/common';
import { Inert, type Inertia } from '@inertianode/nestjs';

@Controller()
export class ProfileController {
  @Get('/profile')
  async show(@Inert() inertia: Inertia) {
    const user = await userService.getCurrentUser();

    await inertia('Profile', {
      user: user,
      avatar: new UserAvatar(user, 128)
    });
  }
}
```

You can also create patterns for merging with shared data:

```ts
import { Inertia } from '@inertianode/core';

class MergeWithShared {
  constructor(private key: string, private items: string[]) {}

  toJSON() {
    // Access shared data
    const shared = Inertia.getShared(this.key) || [];
    // Merge with new items
    return [...shared, ...this.items];
  }
}

// Usage
Inertia.share('notifications', ['Welcome back!']);

app.get('/dashboard', async (req, res) => {
  await res.Inertia('Dashboard', {
    notifications: new MergeWithShared('notifications', ['New message received'])
    // Result: ["Welcome back!", "New message received"]
  });
});
```

## Grouped props

In some situations you may want to group related props together for reusability across different pages. You can accomplish this using functions or classes.

```ts
interface User {
  role: string;
}

class UserPermissions {
  constructor(private user: User) {}

  async toJSON() {
    return {
      canEdit: await this.checkPermission('edit'),
      canDelete: await this.checkPermission('delete'),
      canPublish: await this.checkPermission('publish'),
      isAdmin: this.user.role === 'admin'
    };
  }

  private async checkPermission(permission: string): Promise<boolean> {
    // Your permission checking logic
    return true;
  }
}
```

You can use these prop classes directly in your render calls:

```ts
// framework: express
app.get('/profile', async (req, res) => {
  const user = await userService.getCurrentUser();
  const permissions = new UserPermissions(user);

  await res.Inertia('UserProfile', {
    permissions: await permissions.toJSON()
  });
});
```

```ts
// framework: nestjs
import { Controller, Get } from '@nestjs/common';
import { Inert, type Inertia } from '@inertianode/nestjs';

@Controller()
export class ProfileController {
  @Get('profile')
  async show(@Inert() inertia: Inertia) {
    const user = await userService.getCurrentUser();
    const permissions = new UserPermissions(user);

    await inertia('UserProfile', {
      permissions: await permissions.toJSON()
    });
  }
}
```

You can also combine multiple prop objects with other props:

```ts
// framework: express
app.get('/profile', async (req, res) => {
  const user = await userService.getCurrentUser();
  const permissions = new UserPermissions(user);

  await res.Inertia('UserProfile', {
    user: user,
    permissions: await permissions.toJSON()
  });
});
```

```ts
// framework: nestjs
import { Controller, Get } from '@nestjs/common';
import { Inert, type Inertia } from '@inertianode/nestjs';

@Controller()
export class ProfileController {
  @Get('profile')
  async show(@Inert() inertia: Inertia) {
    const user = await userService.getCurrentUser();
    const permissions = new UserPermissions(user);

    await inertia('UserProfile', {
      user: user,
      permissions: await permissions.toJSON()
    });
  }
}
```

## Root template data

There are situations where you may want to access your prop data in your application's root template. For example, you may want to add a meta description tag, Twitter card meta tags, or Facebook Open Graph meta tags. You can access this data through the page object in your template function.

```ts
import type { Page } from '@inertianode/core';

export function rootTemplate(page: Page): string {
  const eventTitle = page.props.event?.title || 'Default Title';

  return `<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="twitter:title" content="${eventTitle}" />
    ${page.head.join('\n    ')}
  </head>
  <body>
    <div id="app" data-page='${JSON.stringify(page)}'></div>
    <script type="module" src="/src/app.tsx"></script>
  </body>
</html>`;
}
```

Sometimes you may even want to provide data to the root template that will not be sent to your JavaScript page component. This feature is currently under development for InertiaNode. -->

## Maximum response size

To enable client-side history navigation, all Inertia server responses are stored in the browser's history state. However, keep in mind that some browsers impose a size limit on how much data can be saved within the history state.

For example, [Firefox](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) has a size limit of 16 MiB and throws a `NS_ERROR_ILLEGAL_VALUE` error if you exceed this limit. Typically, this is much more data than you'll ever practically need when building applications.
