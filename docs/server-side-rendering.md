# Server-side Rendering (SSR)

Server-side rendering pre-renders your JavaScript pages on the server, allowing your visitors to receive fully rendered HTML when they visit your application. Since fully rendered HTML is served by your application, it's also easier for search engines to index your site.

Server-side rendering uses Node.js to render your pages in a background process; therefore, Node must be available on your server for server-side rendering to function properly.

## Configuration

InertiaNode SSR is configured through middleware options:

```ts
// framework: hono
import { inertiaHonoAdapter } from "@inertianode/hono";

app.use(
  inertiaHonoAdapter({
    ssr: {
      enabled: true,
      url: "http://127.0.0.1:13714/render", // default
    },
  })
);
```

```ts
// framework: express
import { inertiaExpressAdapter } from "@inertianode/express";

app.use(
  inertiaExpressAdapter({
    ssr: {
      enabled: true,
      url: "http://127.0.0.1:13714/render", // default
    },
  })
);
```

```ts
// framework: koa
import { inertiaKoaAdapter } from "@inertianode/koa";

app.use(
  inertiaKoaAdapter({
    ssr: {
      enabled: true,
      url: "http://127.0.0.1:13714/render", // default
    },
  })
);
```

Remember to include the Inertia head content in your root template. When SSR is enabled, the SSR response will be passed via the `viewData` parameter:

```ts
import type { Page } from '@inertianode/core';
import { viteAssets, inertiaHead, inertiaBody } from '@inertianode/core';

export function rootTemplate(page: Page, viewData: any = {}): string {
  const ssr = viewData.ssr; // SSR response when enabled

  return `<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My App</title>
    ${viteAssets('src/app.tsx')}
    ${inertiaHead(ssr?.head)}
  </head>
  <body>
    ${inertiaBody(page, ssr?.body)}
  </body>
</html>`;
}
```

The `inertiaHead()` helper will render the SSR head content when available, and `inertiaBody()` will use the pre-rendered HTML from the SSR server. When SSR is not enabled or fails, these helpers gracefully fall back to client-side rendering.

## Install dependencies

If you are not using an InertiaNode template and would like to manually configure SSR, we'll first install the additional dependencies required for server-side rendering. This is only necessary for the Vue adapters, so you can skip this step if you're using React or Svelte.

Vue:

```bash
npm install @vue/server-renderer
```

```js
// framework: react
// No additional dependencies required
```

```js
// framework: svelte
// No additional dependencies required
```

## Add server entry-point

Next, we'll create a `src/ssr.js` file within our project that will serve as our SSR entry point.

```bash
touch src/ssr.js
```

This file is going to look very similar to your `src/app.js` file, except it's not going to run in the browser, but rather in Node.js. Here's a complete example.

```js
// framework: vue
import { createInertiaApp } from '@inertiajs/vue3'
import createServer from '@inertiajs/vue3/server'
import { renderToString } from '@vue/server-renderer'
import { createSSRApp, h } from 'vue'
createServer(page =>
  createInertiaApp({
    page,
    render: renderToString,
    resolve: name => {
      const pages = import.meta.glob('./Pages/**/*.vue', { eager: true })
      return pages[\`./Pages/\${name}.vue\`]
    },
    setup({ App, props, plugin }) {
      return createSSRApp({
        render: () => h(App, props),
      }).use(plugin)
    },
  }),
)
```

```jsx
// framework: react
import { createInertiaApp } from '@inertiajs/react'
import createServer from '@inertiajs/react/server'
import ReactDOMServer from 'react-dom/server'
createServer(page =>
  createInertiaApp({
    page,
    render: ReactDOMServer.renderToString,
    resolve: name => {
      const pages = import.meta.glob('./Pages/**/*.jsx', { eager: true })
      return pages[\`./Pages/\${name}.jsx\`]
    },
    setup: ({ App, props }) => <App {...props} />,
  }),
)
```

```js
// framework: svelte4
import { createInertiaApp } from '@inertiajs/svelte'
import createServer from '@inertiajs/svelte/server'
createServer(page =>
  createInertiaApp({
    page,
    resolve: name => {
      const pages = import.meta.glob('./Pages/**/*.svelte', { eager: true })
      return pages[\`./Pages/\${name}.svelte\`]
    },
    setup({ App, props }) {
      return App.render(props)
    },
  }),
)
```

```js
// framework: svelte5
import { createInertiaApp } from '@inertiajs/svelte'
import createServer from '@inertiajs/svelte/server'
import { render } from 'svelte/server'
createServer(page =>
  createInertiaApp({
    page,
    resolve: name => {
      const pages = import.meta.glob('./Pages/**/*.svelte', { eager: true })
      return pages[\`./Pages/\${name}.svelte\`]
    },
    setup({ App, props }) {
      return render(App, { props })
    },
  }),
)
```

When creating this file, be sure to add anything that's missing from your `app.js` file that makes sense to run in SSR mode, such as plugins or custom components.

### Clustering

By default, the SSR server will run on a single thread. Clustering starts multiple Node servers on the same port, requests are then handled by each thread in a round-robin way.

You can enable clustering by passing a second argument of options to `createServer`.

```js
// framework: vue
import { createInertiaApp } from "@inertiajs/vue3";
import createServer from "@inertiajs/vue3/server";
import { renderToString } from "@vue/server-renderer";
import { createSSRApp, h } from "vue";
createServer(
  (page) =>
    createInertiaApp({
      // ...
    }),
  { cluster: true }
);
```

```jsx
// framework: react
import { createInertiaApp } from "@inertiajs/react";
import createServer from "@inertiajs/react/server";
import ReactDOMServer from "react-dom/server";
createServer(
  (page) =>
    createInertiaApp({
      // ...
    }),
  { cluster: true }
);
```

```js
// framework: svelte4
import { createInertiaApp } from "@inertiajs/svelte";
import createServer from "@inertiajs/svelte/server";
createServer(
  (page) =>
    createInertiaApp({
      // ...
    }),
  { cluster: true }
);
```

```js
// framework: svelte5
import { createInertiaApp } from "@inertiajs/svelte";
import createServer from "@inertiajs/svelte/server";
import { render } from "svelte/server";
createServer(
  (page) =>
    createInertiaApp({
      // ...
    }),
  { cluster: true }
);
```

## Setup Vite

Next, we need to update our Vite configuration to build our new `ssr.js` file. We can do this by adding a `ssr` property to your Vite configuration in your `vite.config.js` file.

```js
import laravel from "laravel-vite-plugin";

export default defineConfig({
  plugins: [
    // Your existing plugins
    laravel({
      input: ["src/app.ts"],
      ssr: "src/ssr.ts",
      refresh: true,
    }),
  ],
});
```

## Update npm script

Next, let's update the `build` script in our `package.json` file to also build our new `ssr.js` file.

```diff
"scripts": {
  "dev": "vite",
\-   "build": "vite build"
\+   "build": "vite build && vite build --ssr"
},
```

Now you can build both your client-side and server-side bundles.

```bash
npm run build
```

## Running the SSR server

Now that you have built both your client-side and server-side bundles, you should be able run the Node-based Inertia SSR server using Node.js directly or through your preferred process manager.

```bash
node wwwroot/build/ssr.js
```

You may also use other JavaScript runtimes like Bun if preferred.

```bash
bun wwwroot/build/ssr.js
```

With the server running, you should be able to access your app within the browser with server-side rendering enabled. In fact, you should be able to disable JavaScript entirely and still navigate around your application.

## Client side hydration

Since your website is now being server-side rendered, you can instruct <vue>Vue</vue><react>React</react><svelte>Svelte</svelte> to "hydrate" the static markup and make it interactive instead of re-rendering all the HTML that we just generated.

<vue>To enable client-side hydration in a Vue app, update your `ssr.js` file to use `createSSRApp` instead of `createApp`.

</vue><react>To enable client-side hydration in a React app, update your `ssr.js` file to use `hydrateRoot` instead of `createRoot`.

</react><svelte4>To enable client-side hydration in a Svelte 4 app, set the `hydrate` option to `true` in your `ssr.js` file.

</svelte4><svelte5>To enable client-side hydration in a Svelte 5 app, update your `ssr.js` file to use `hydrate` instead of `mount` when server rendering.

</svelte5>

```diff
// framework: vue
\- import { createApp, h } from 'vue'
\+ import { createSSRApp, h } from 'vue'
import { createInertiaApp } from '@inertiajs/vue3'
createInertiaApp({
resolve: name => {
const pages = import.meta.glob('./Pages/**/*.vue', { eager: true })
return pages[\`./Pages/\${name}.vue\`]
},
setup({ el, App, props, plugin }) {
\-     createApp({ render: () => h(App, props) })
\+     createSSRApp({ render: () => h(App, props) })
.use(plugin)
.mount(el)
},
})
```

```diff
// framework: react
import { createInertiaApp } from '@inertiajs/react'
\- import { createRoot } from 'react-dom/client'
\+ import { hydrateRoot } from 'react-dom/client'
createInertiaApp({
resolve: name => {
const pages = import.meta.glob('./Pages/**/*.jsx', { eager: true })
return pages[\`./Pages/\${name}.jsx\`]
},
setup({ el, App, props }) {
\-     createRoot(el).render(<App {...props} />)
\+     hydrateRoot(el, <App {...props} />)
},
})
```

```diff
// framework: svelte4
import { createInertiaApp } from '@inertiajs/svelte'
createInertiaApp({
resolve: name => {
const pages = import.meta.glob('./Pages/**/*.svelte', { eager: true })
return pages[\`./Pages/\${name}.svelte\`]
},
setup({ el, App, props }) {
\-     new App({ target: el, props })
\+     new App({ target: el, props, hydrate: true })
},
})
```

```diff
// framework: svelte5
import { createInertiaApp } from '@inertiajs/svelte'
\-  import { mount } from 'svelte'
\+  import { hydrate, mount } from 'svelte'
createInertiaApp({
resolve: name => {
const pages = import.meta.glob('./Pages/**/*.svelte', { eager: true })
return pages[\`./Pages/\${name}.svelte\`]
},
setup({ el, App, props }) {
\-      mount(App, { target: el, props })
\+      if (el.dataset.serverRendered === 'true') {
+
hydrate(App, { target: el, props })
\+      } else {
+
mount(App, { target: el, props })
\+      }
},
})
```

<svelte4>You will also need to set the `hydratable` compiler option to `true` in your `vite.config.js` file:

```diff
// framework: svelte4
import { svelte } from '@sveltejs/vite-plugin-svelte'
import laravel from 'laravel-vite-plugin'
import { defineConfig } from 'vite'
export default defineConfig({
  plugins: [
    laravel({
      input: ['src/app.css', 'src/app.js'],
      ssr: 'src/ssr.js',
      refresh: true,
    }),
\-     svelte(),
\+     svelte({
+
compilerOptions: {
+
hydratable: true,
+
},
\+     }),
],
})
```

</svelte4>

<!-- TODO: Double Check this -->
<!--
## Deployment

When deploying your SSR enabled app to production, you'll need to build both the client-side (`app.js`) and server-side bundles (`ssr.js`), and then run the SSR server as a background process, typically using a process monitoring tool such as PM2 or systemd.

```bash
node public/build/ssr.js
```

To stop the SSR server, you can use your process manager's stop commands. Your process monitor should be responsible for automatically restarting the SSR server after it has stopped.

You can verify that the SSR server is running by making HTTP requests to it or checking the process status. This can be helpful after deployment and works well as a health check to ensure the server is responding as expected.

InertiaNode can be configured to check if the server-side bundle exists before dispatching requests to the SSR server through the appropriate configuration options.

### Using PM2

PM2 is a popular process manager for Node.js applications. You can use it to run both your main application and SSR server:

```bash
# Start your main application
pm2 start npm --name "app" -- start

# Start the SSR server
pm2 start public/build/ssr.js --name "ssr-server"

# Save the process list
pm2 save

# Generate startup script
pm2 startup
```

### Docker

To run the SSR server in Docker, create a Dockerfile that builds both your application and SSR bundle:

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source files
COPY . .

# Build the application and SSR bundle
RUN npm run build

FROM node:18-alpine

WORKDIR /app

# Copy built files from builder
COPY --from=builder /app/public ./public
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package*.json ./

# Expose ports for main app and SSR server
EXPOSE 3000 13714

# Start both processes (you may want to use a process manager like PM2)
CMD ["sh", "-c", "node dist/server.js & node public/build/ssr.js"]
```

### Other hosting providers

For other hosting providers:
1. Ensure Node.js runtime is available
2. Build your SSR bundle using `npm run build`
3. Configure your process manager to run the SSR server alongside your main Node.js application
4. Ensure the SSR server port (default 13714) is accessible to your main application -->
