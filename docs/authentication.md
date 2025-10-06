# Authentication

One of the benefits of using Inertia is that you don't need a special authentication system such as OAuth to connect to your data provider (API). Also, since your data is provided via your controllers, and housed on the same domain as your JavaScript components, you don't have to worry about setting up CORS.

Rather, when using Inertia, you can simply use whatever authentication system your server-side framework ships with. Typically, this will be a session-based authentication system.

## Session-based authentication

The most common approach to authentication with Inertia is to use session-based authentication. This works by storing a session identifier in an HTTP-only cookie which is sent with every request.

### Hono

Hono provides flexible session management through middleware. Here's how to set up authentication with Hono:

```ts
// framework: hono
import { Hono } from "hono";
import { getCookie, setCookie } from "hono/cookie";
import { sign, verify } from "hono/jwt";

const app = new Hono();

// Authentication middleware
app.use("*", async (c, next) => {
  const token = getCookie(c, "auth_token");

  if (token) {
    try {
      const payload = await verify(token, process.env.JWT_SECRET!);
      // Store user in context
      c.set("user", payload);
    } catch (err) {
      // Invalid token
    }
  }

  await next();
});

// Share authenticated user with all Inertia requests
app.use("*", async (c, next) => {
  const user = c.get("user");
  c.Inertia.share("auth", {
    user: user || null,
  });
  await next();
});

// Login route
app.post("/login", async (c) => {
  const { email, password } = await c.req.json();

  // Validate credentials (example)
  const user = await validateCredentials(email, password);

  if (!user) {
    return c.json({ error: "Invalid credentials" }, 401);
  }

  // Create JWT token
  const token = await sign(
    { id: user.id, email: user.email },
    process.env.JWT_SECRET!
  );

  // Set HTTP-only cookie
  setCookie(c, "auth_token", token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "Lax",
    maxAge: 60 * 60 * 24 * 7, // 7 days
  });

  return c.Inertia.back();
});

// Logout route
app.post("/logout", async (c) => {
  setCookie(c, "auth_token", "", {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "Lax",
    maxAge: 0,
  });

  return c.redirect("/");
});

// Protected route
app.get("/dashboard", async (c) => {
  const user = c.get("user");

  if (!user) {
    return c.redirect("/login");
  }

  return c.Inertia("Dashboard", {
    user,
  });
});
```

### Express

Express has a rich ecosystem of authentication middleware. Here's an example using `express-session` and `passport`:

```ts
// framework: express
import express from "express";
import session from "express-session";
import passport from "passport";
import { Strategy as LocalStrategy } from "passport-local";

const app = express();

// Session configuration
app.use(
  session({
    secret: process.env.SESSION_SECRET!,
    resave: false,
    saveUninitialized: false,
    cookie: {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      maxAge: 1000 * 60 * 60 * 24 * 7, // 7 days
    },
  })
);

// Passport configuration
passport.use(
  new LocalStrategy(async (username, password, done) => {
    const user = await validateCredentials(username, password);
    if (!user) {
      return done(null, false, { message: "Invalid credentials" });
    }
    return done(null, user);
  })
);

passport.serializeUser((user: any, done) => {
  done(null, user.id);
});

passport.deserializeUser(async (id: number, done) => {
  const user = await findUserById(id);
  done(null, user);
});

app.use(passport.initialize());
app.use(passport.session());

// Share authenticated user with all Inertia requests
app.use((req, res, next) => {
  res.Inertia.share("auth", {
    user: req.user || null,
  });
  next();
});

// Login route
app.post("/login", passport.authenticate("local"), (req, res) => {
  res.Inertia.back();
});

// Logout route
app.post("/logout", (req, res) => {
  req.logout((err) => {
    if (err) {
      return res.status(500).json({ error: "Logout failed" });
    }
    res.redirect("/");
  });
});

// Protected route
app.get("/dashboard", (req, res) => {
  if (!req.isAuthenticated()) {
    return res.redirect("/login");
  }

  res.Inertia.render("Dashboard", {
    user: req.user,
  });
});
```

### NestJS

NestJS provides a flexible authentication system through Guards and middleware. Here's an example using `@nestjs/passport` and session-based auth:

```ts
// framework: nestjs
import {
  Module,
  Injectable,
  CanActivate,
  ExecutionContext,
  Controller,
  Get,
  Post,
  Body,
  Req,
  Res,
  UseGuards,
  NestMiddleware,
} from "@nestjs/common";
import { Request, Response, NextFunction } from "express";
import * as session from "express-session";
import * as passport from "passport";
import { Strategy as LocalStrategy } from "passport-local";

// Passport local strategy
@Injectable()
export class LocalAuthStrategy extends LocalStrategy {
  constructor() {
    super({
      usernameField: "email",
      passwordField: "password",
    });
  }

  async validate(email: string, password: string): Promise<any> {
    const user = await validateCredentials(email, password);
    if (!user) {
      throw new UnauthorizedException("Invalid credentials");
    }
    return user;
  }
}

// Authentication guard
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return request.isAuthenticated();
  }
}

// Middleware to share auth data with Inertia
@Injectable()
export class ShareAuthMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    res.Inertia.share("auth", {
      user: req.user || null,
    });
    next();
  }
}

// Auth controller
@Controller()
export class AuthController {
  @Post("/login")
  @UseGuards(passport.authenticate("local"))
  async login(@Inert() inertia: Inertia) {
    await inertia.back();
  }

  @Post("/logout")
  async logout(@Inert() inertia: Inertia) {
    inertia.req?.logout((err) => {
      if (err) {
        throw new InternalServerErrorException("Logout failed");
      }
    });
    await inertia.redirect("/");
  }

  @Get("/dashboard")
  @UseGuards(AuthGuard)
  async dashboard(@Inert() inertia: Inertia) {
    await inertia("Dashboard", {
      user: inertia.req?.user,
    });
  }
}

// App module configuration
@Module({
  imports: [],
  controllers: [AuthController],
  providers: [LocalAuthStrategy, AuthGuard, ShareAuthMiddleware],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // Session configuration
    consumer
      .apply(
        session({
          secret: process.env.SESSION_SECRET!,
          resave: false,
          saveUninitialized: false,
          cookie: {
            httpOnly: true,
            secure: process.env.NODE_ENV === "production",
            maxAge: 1000 * 60 * 60 * 24 * 7, // 7 days
          },
        }),
        passport.initialize(),
        passport.session(),
        ShareAuthMiddleware
      )
      .forRoutes("*");
  }
}
```

### Koa

Koa provides session support through middleware. Here's an example using `koa-session`:

```ts
// framework: koa
import Koa from "koa";
import session from "koa-session";

const app = new Koa();

// Session configuration
app.keys = [process.env.SESSION_SECRET!];

app.use(
  session(
    {
      key: "koa:sess",
      maxAge: 86400000, // 1 day
      httpOnly: true,
      signed: true,
      secure: process.env.NODE_ENV === "production",
    },
    app
  )
);

// Share authenticated user with all Inertia requests
app.use(async (ctx, next) => {
  ctx.Inertia.share("auth", {
    user: ctx.session?.user || null,
  });
  await next();
});

// Login route
app.use(async (ctx, next) => {
  if (ctx.path === "/login" && ctx.method === "POST") {
    const { email, password } = ctx.request.body;

    const user = await validateCredentials(email, password);

    if (!user) {
      ctx.status = 401;
      ctx.body = { error: "Invalid credentials" };
      return;
    }

    // Store user in session
    ctx.session!.user = user;

    return ctx.Inertia.back();
  }

  await next();
});

// Logout route
app.use(async (ctx, next) => {
  if (ctx.path === "/logout" && ctx.method === "POST") {
    ctx.session = null;
    ctx.redirect("/");
    return;
  }

  await next();
});

// Protected route
app.use(async (ctx, next) => {
  if (ctx.path === "/dashboard") {
    if (!ctx.session?.user) {
      ctx.redirect("/login");
      return;
    }

    await ctx.Inertia.render("Dashboard", {
      user: ctx.session.user,
    });
    return;
  }

  await next();
});
```

## Sharing authenticated user data

A common requirement is to access the authenticated user in your JavaScript page components. The best way to do this is to use shared data.

```ts
// framework: hono
app.use("*", async (c, next) => {
  c.Inertia.share("auth", {
    user: c.get("user") || null,
  });
  await next();
});
```

```ts
// framework: express
app.use((req, res, next) => {
  res.Inertia.share("auth", {
    user: req.user || null,
  });
  next();
});
```

```ts
// framework: nestjs
import { Injectable, NestMiddleware } from "@nestjs/common";
import { Request, Response, NextFunction } from "express";

@Injectable()
export class ShareAuthMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    res.Inertia.share("auth", {
      user: req.user || null,
    });
    next();
  }
}
```

```ts
// framework: koa
app.use(async (ctx, next) => {
  ctx.Inertia.share("auth", {
    user: ctx.session?.user || null,
  });
  await next();
});
```

Then you can access the authenticated user in your frontend components:

```vue
// framework: vue
<script setup>
import { usePage } from "@inertiajs/vue3";
import { computed } from "vue";

const page = usePage();
const user = computed(() => page.props.auth.user);
</script>

<template>
  <div v-if="user">Welcome back, {{ user.name }}!</div>
  <div v-else>Please log in.</div>
</template>
```

```jsx
// framework: react
import { usePage } from "@inertiajs/react";

export default function Dashboard() {
  const { auth } = usePage().props;

  return (
    <div>
      {auth.user ? (
        <div>Welcome back, {auth.user.name}!</div>
      ) : (
        <div>Please log in.</div>
      )}
    </div>
  );
}
```

```svelte
// framework: svelte4
<script>
import { page } from '@inertiajs/svelte'

$: user = $page.props.auth.user
</script>

{#if user}
  <div>Welcome back, {user.name}!</div>
{:else}
  <div>Please log in.</div>
{/if}
```

```svelte
// framework: svelte5
<script>
import { page } from '@inertiajs/svelte'

const user = $derived(page.props.auth.user)
</script>

{#if user}
  <div>Welcome back, {user.name}!</div>
{:else}
  <div>Please log in.</div>
{/if}
```

## Protecting routes

You can protect routes by checking if the user is authenticated before rendering the Inertia response. If the user is not authenticated, redirect them to the login page.

```ts
// framework: hono
app.get("/dashboard", async (c) => {
  if (!c.get("user")) {
    return c.redirect("/login");
  }

  return c.Inertia("Dashboard");
});
```

```ts
// framework: express
app.get("/dashboard", (req, res) => {
  if (!req.isAuthenticated()) {
    return res.redirect("/login");
  }

  res.Inertia.render("Dashboard");
});
```

```ts
// framework: nestjs
import { Controller, Get, UseGuards } from "@nestjs/common";
import { Inert, type Inertia } from "@inertianode/nestjs";

@Controller()
export class AppController {
  @Get("/dashboard")
  @UseGuards(AuthGuard)
  async dashboard(@Inert() inertia: Inertia) {
    await inertia("Dashboard");
  }
}
```

```ts
// framework: koa
app.use(async (ctx, next) => {
  if (ctx.path === "/dashboard") {
    if (!ctx.session?.user) {
      ctx.redirect("/login");
      return;
    }

    await ctx.Inertia.render("Dashboard");
    return;
  }

  await next();
});
```

You can also create authentication middleware for reusability:

```ts
// framework: hono
const requireAuth = async (c, next) => {
  if (!c.get("user")) {
    return c.redirect("/login");
  }
  await next();
};

app.get("/dashboard", requireAuth, async (c) => {
  return c.Inertia("Dashboard");
});
```

```ts
// framework: express
const requireAuth = (req, res, next) => {
  if (!req.isAuthenticated()) {
    return res.redirect("/login");
  }
  next();
};

app.get("/dashboard", requireAuth, (req, res) => {
  res.Inertia.render("Dashboard");
});
```

```ts
// framework: nestjs
import {
  Injectable,
  CanActivate,
  ExecutionContext,
  Controller,
  Get,
  Req,
  Res,
  UseGuards,
} from "@nestjs/common";
import { Request, Response } from "express";

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    if (!request.isAuthenticated()) {
      const response = context.switchToHttp().getResponse();
      response.redirect("/login");
      return false;
    }
    return true;
  }
}

@Controller()
export class AppController {
  @Get("/dashboard")
  @UseGuards(AuthGuard)
  async dashboard(@Inert() inertia: Inertia) {
    await inertia("Dashboard");
  }
}
```

```ts
// framework: koa
const requireAuth = async (ctx, next) => {
  if (!ctx.session?.user) {
    ctx.redirect("/login");
    return;
  }
  await next();
};

app.use(async (ctx, next) => {
  if (ctx.path === "/dashboard") {
    await requireAuth(ctx, async () => {
      await ctx.Inertia.render("Dashboard");
    });
    return;
  }
  await next();
});
```

## OAuth and third-party providers

You can also use OAuth providers like Google, GitHub, or Facebook for authentication. Most frameworks have libraries to help with this.

### Hono with OAuth

```ts
// framework: hono
import { Hono } from "hono";
import { OAuth2Client } from "google-auth-library";

const app = new Hono();
const oauth2Client = new OAuth2Client(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  `${process.env.APP_URL}/auth/google/callback`
);

app.get("/auth/google", (c) => {
  const url = oauth2Client.generateAuthUrl({
    access_type: "offline",
    scope: ["profile", "email"],
  });
  return c.redirect(url);
});

app.get("/auth/google/callback", async (c) => {
  const code = c.req.query("code");
  const { tokens } = await oauth2Client.getToken(code);
  oauth2Client.setCredentials(tokens);

  const ticket = await oauth2Client.verifyIdToken({
    idToken: tokens.id_token!,
    audience: process.env.GOOGLE_CLIENT_ID,
  });

  const payload = ticket.getPayload();

  // Find or create user
  const user = await findOrCreateUser({
    email: payload?.email,
    name: payload?.name,
    provider: "google",
  });

  // Create session or JWT
  // ...

  return c.redirect("/dashboard");
});
```

### Express with Passport OAuth

```ts
// framework: express
import passport from "passport";
import { Strategy as GoogleStrategy } from "passport-google-oauth20";

passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      callbackURL: `${process.env.APP_URL}/auth/google/callback`,
    },
    async (accessToken, refreshToken, profile, done) => {
      const user = await findOrCreateUser({
        email: profile.emails?.[0].value,
        name: profile.displayName,
        provider: "google",
      });
      return done(null, user);
    }
  )
);

app.get(
  "/auth/google",
  passport.authenticate("google", { scope: ["profile", "email"] })
);

app.get(
  "/auth/google/callback",
  passport.authenticate("google", { failureRedirect: "/login" }),
  (req, res) => {
    res.redirect("/dashboard");
  }
);
```

### NestJS with Passport OAuth

```ts
// framework: nestjs
import {
  Controller,
  Get,
  Req,
  Res,
  UseGuards,
  Injectable,
} from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { AuthGuard } from "@nestjs/passport";
import { Strategy as GoogleStrategy } from "passport-google-oauth20";
import { Request, Response } from "express";

@Injectable()
export class GoogleAuthStrategy extends PassportStrategy(
  GoogleStrategy,
  "google"
) {
  constructor() {
    super({
      clientID: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      callbackURL: `${process.env.APP_URL}/auth/google/callback`,
      scope: ["profile", "email"],
    });
  }

  async validate(
    accessToken: string,
    refreshToken: string,
    profile: any
  ): Promise<any> {
    const user = await findOrCreateUser({
      email: profile.emails?.[0].value,
      name: profile.displayName,
      provider: "google",
    });
    return user;
  }
}

@Controller("auth")
export class AuthController {
  @Get("google")
  @UseGuards(AuthGuard("google"))
  async googleAuth() {
    // Initiates the Google OAuth2 flow
  }

  @Get("google/callback")
  @UseGuards(AuthGuard("google"))
  async googleAuthCallback(@Inert() inertia: Inertia) {
    await inertia.redirect("/dashboard");
  }
}
```

### Koa with OAuth

```ts
// framework: koa
import Router from "@koa/router";
import { OAuth2Client } from "google-auth-library";

const router = new Router();
const oauth2Client = new OAuth2Client(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  `${process.env.APP_URL}/auth/google/callback`
);

router.get("/auth/google", (ctx) => {
  const url = oauth2Client.generateAuthUrl({
    access_type: "offline",
    scope: ["profile", "email"],
  });
  ctx.redirect(url);
});

router.get("/auth/google/callback", async (ctx) => {
  const code = ctx.query.code;
  const { tokens } = await oauth2Client.getToken(code as string);
  oauth2Client.setCredentials(tokens);

  const ticket = await oauth2Client.verifyIdToken({
    idToken: tokens.id_token!,
    audience: process.env.GOOGLE_CLIENT_ID,
  });

  const payload = ticket.getPayload();

  // Find or create user
  const user = await findOrCreateUser({
    email: payload?.email,
    name: payload?.name,
    provider: "google",
  });

  // Store in session
  ctx.session!.user = user;

  ctx.redirect("/dashboard");
});

app.use(router.routes());
```

### Other Authentication Strategies

The above examples are limited to show how to use session-based authentication with Inertia. You can use any authentication strategy that your framework supports. The following auth packages would likely also work:

- [Passport.js](https://www.passportjs.org)
- [BetterAuth](https://www.better-auth.com)
- [Magic](https://magic.link)
- [Auth0](https://auth0.com)
- [Okta](https://www.okta.com)
- [Clerk](https://clerk.com)
- [Supabase](https://supabase.com)
- [Firebase](https://firebase.google.com)
