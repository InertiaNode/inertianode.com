# Authorization

When using Inertia, authorization is best handled server-side in your application's authorization policies. However, you may be wondering how to perform checks against your authorization policies from within your Inertia page components since you won't have access to your framework's server-side helpers.

The simplest approach to solving this problem is to pass the results of your authorization checks as props to your page components.

```ts
// framework: hono
import { Hono } from 'hono';
// Hono uses per-request Inertia instance (c.Inertia);

const app = new Hono();

app.get('/users', async (c) => {
  const currentUser = c.get('user'); // Assumes auth middleware sets user
  const canCreateUser = await authService.can(currentUser, 'createUser');
  const users = await userService.getAllUsers();

  const userViewModels = await Promise.all(
    users.map(async (user) => {
      const canEditUser = await authService.can(currentUser, 'editUser', user);
      return {
        firstName: user.firstName,
        lastName: user.lastName,
        email: user.email,
        can: {
          editUser: canEditUser
        }
      };
    })
  );

  return await c.Inertia('Users/Index', {
    can: {
      createUser: canCreateUser
    },
    users: userViewModels
  });
});
```

```ts
// framework: express
import express from 'express';

const app = express();

app.get('/users', async (req, res) => {
  const currentUser = req.user; // Assumes auth middleware sets user
  const canCreateUser = await authService.can(currentUser, 'createUser');
  const users = await userService.getAllUsers();

  const userViewModels = await Promise.all(
    users.map(async (user) => {
      const canEditUser = await authService.can(currentUser, 'editUser', user);
      return {
        firstName: user.firstName,
        lastName: user.lastName,
        email: user.email,
        can: {
          editUser: canEditUser
        }
      };
    })
  );

  await res.Inertia('Users/Index', {
    can: {
      createUser: canCreateUser
    },
    users: userViewModels
  });
});
```

```ts
// framework: koa
import Koa from 'koa';
import Router from '@koa/router';

const app = new Koa();
const router = new Router();

router.get('/users', async (ctx) => {
  const currentUser = ctx.state.user; // Assumes auth middleware sets user
  const canCreateUser = await authService.can(currentUser, 'createUser');
  const users = await userService.getAllUsers();

  const userViewModels = await Promise.all(
    users.map(async (user) => {
      const canEditUser = await authService.can(currentUser, 'editUser', user);
      return {
        firstName: user.firstName,
        lastName: user.lastName,
        email: user.email,
        can: {
          editUser: canEditUser
        }
      };
    })
  );

  await ctx.Inertia('Users/Index', {
    can: {
      createUser: canCreateUser
    },
    users: userViewModels
  });
});

app.use(router.routes());
```
