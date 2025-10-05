# Vite Helper

InertiaNode automatically integrates with Vite through the `laravel-vite-plugin` npm package. This helper automatically loads your generated styles and scripts, and enables HMR (Hot Module Replacement) during development.

## Configuration

Configure Vite options in your Inertia setup:

```ts
import { Inertia } from "@inertianode/core";

Inertia.setViteOptions({
  publicDirectory: "public",
  buildDirectory: "build",
  hotFile: "hot",
  manifestFilename: "manifest.json",
  reactRefresh: true, // Enable React Fast Refresh in development
});
```

Each adapter has a different way of setting the Vite options:

```ts
// framework: hono
import { inertiaHonoAdapter, Inertia } from "@inertianode/hono";

const app = new Hono();

app.use(
  inertiaHonoAdapter({
    vite: {
      publicDirectory: "public",
      buildDirectory: "build",
      hotFile: "hot",
      manifestFilename: "manifest.json",
      reactRefresh: true, // Enable React Fast Refresh
    },
  })
);
```

```ts
// framework: express
import { inertiaExpressAdapter, Inertia } from "@inertianode/express";

const app = express();

app.use(
  inertiaExpressAdapter({
    vite: {
      publicDirectory: "public",
      buildDirectory: "build",
      hotFile: "hot",
      manifestFilename: "manifest.json",
      reactRefresh: true, // Enable React Fast Refresh
    },
  })
);
```

```ts
// framework: koa
import { inertiaKoaAdapter, Inertia } from "@inertianode/koa";

const app = new Koa();

app.use(
  inertiaKoaAdapter({
    vite: {
      publicDirectory: "public",
      buildDirectory: "build",
      hotFile: "hot",
      manifestFilename: "manifest.json",
      reactRefresh: true, // Enable React Fast Refresh
    },
  })
);
```

The Vite helper will automatically detect if you're in development mode (by checking for the `hot` file) and load the appropriate assets.

## Root Template Integration

The root template function should include Vite asset references. InertiaNode provides a `viteAssets()` helper that automatically handles both development and production asset loading:

```ts
import type { Page } from "@inertianode/core";
import { viteAssets, inertiaBody } from "@inertianode/core";

export function rootTemplate(page: Page): string {
  return `<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    ${viteAssets("src/app.tsx")}
  </head>
  <body>
    ${inertiaBody(page)}
  </body>
</html>`;
}
```

The `viteAssets()` helper:
- Automatically detects development vs production mode
- Loads assets from Vite dev server (with HMR) in development
- Loads hashed assets from the manifest in production
- Supports React Fast Refresh when `reactRefresh: true` is set in Vite options
- Supports multiple entrypoints

### Multiple Entrypoints

You can pass multiple entrypoints to load several assets:

```ts
${viteAssets(["src/app.tsx", "src/admin.tsx"])}
```

### Custom Vite Options

If your Vite configuration differs from the defaults, you can pass custom options:

```ts
import { viteAssets, inertiaBody } from "@inertianode/core";

export function rootTemplate(page: Page): string {
  return `<!DOCTYPE html>
<html>
  <head>
    ${viteAssets("src/app.tsx", {
      publicDirectory: "dist",
      buildDirectory: "assets",
      hotFile: ".vite-hot",
    })}
  </head>
  <body>
    ${inertiaBody(page)}
  </body>
</html>`;
}
```

### Using the Vite Class (Advanced)

For more control, you can use the `Vite` class directly:

```ts
import type { Page } from "@inertianode/core";
import { Vite, inertiaBody } from "@inertianode/core";

export function rootTemplate(page: Page): string {
  const isDev = Vite.isRunningHot();
  const hotUrl = Vite.hotUrl() || "http://localhost:5173";
  const manifest = Vite.manifest();

  let assetTags = "";

  if (isDev) {
    assetTags = `
    <script type="module" src="${hotUrl}/@vite/client"></script>
    <script type="module" src="${hotUrl}/src/app.tsx"></script>`;
  } else if (manifest && manifest["src/app.tsx"]) {
    const asset = manifest["src/app.tsx"];
    assetTags = `
    <link rel="stylesheet" href="/build/${asset.css?.[0]}" />
    <script type="module" src="/build/${asset.file}"></script>`;
  }

  return `<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    ${assetTags}
  </head>
  <body>
    ${inertiaBody(page)}
  </body>
</html>`;
}
```

The `Vite` class provides these methods:

- `Vite.isRunningHot(options?)` - Check if Vite dev server is running
- `Vite.hotUrl(options?)` - Get the Vite dev server URL from the hot file
- `Vite.manifest(options?)` - Load the Vite manifest (cached)
- `Vite.asset(entrypoint, options?)` - Get asset info for a specific entrypoint
- `Vite.makeTag(entrypoints, options?)` - Generate complete asset tags
- `Vite.reactRefresh(options?)` - Generate React Fast Refresh script tag (for manual usage)
- `Vite.clearManifestCache()` - Clear the manifest cache (useful for testing)
- `Vite.defaultOptions` - Access or modify the default Vite options

## Vite Configuration

### React Example

Here's a complete `vite.config.ts` for a React app:

```ts
// framework: react
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import laravel from "laravel-vite-plugin";

export default defineConfig({
  plugins: [
    laravel({
      input: ["src/app.tsx"],
      publicDirectory: "public",
      buildDirectory: "build",
    }),
    react(),
  ],
  server: {
    host: "localhost",
    port: 5173,
  },
  build: {
    manifest: true,
    outDir: "public/build",
    emptyOutDir: true,
  },
});
```

### Vue Example

Here's a complete `vite.config.ts` for a Vue app with Hot Reload:

```ts
// framework: vue
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import laravel from "laravel-vite-plugin";

export default defineConfig({
  plugins: [
    laravel({
      input: ["src/app.ts"],
      publicDirectory: "public",
      buildDirectory: "build",
      refresh: true,
    }),
    vue({
      template: {
        transformAssetUrls: {
          base: null,
          includeAbsolute: false,
        },
      },
    }),
  ],
  server: {
    host: "localhost",
    port: 5173,
  },
  build: {
    manifest: true,
    outDir: "public/build",
    emptyOutDir: true,
  },
});
```

### Svelte Example

Here's a complete `vite.config.ts` for a Svelte app:

```ts
// framework: svelte
import { defineConfig } from "vite";
import { svelte } from "@sveltejs/vite-plugin-svelte";
import laravel from "laravel-vite-plugin";

export default defineConfig({
  plugins: [
    laravel({
      input: ["src/app.ts"],
      publicDirectory: "public",
      buildDirectory: "build",
      refresh: true,
    }),
    svelte(),
  ],
  server: {
    host: "localhost",
    port: 5173,
  },
  build: {
    manifest: true,
    outDir: "public/build",
    emptyOutDir: true,
  },
});
```

## Development vs Production

During development, Vite runs a development server (typically on `http://localhost:5173`) that provides Hot Module Replacement (HMR). Your root template should detect this by checking for the presence of the `hot` file and load assets from the Vite dev server.

In production, Vite builds your assets with content hashes for cache busting and generates a `manifest.json` file that maps your source files to their built counterparts. Your root template should read this manifest and load the hashed assets.

The `laravel-vite-plugin` handles much of this complexity automatically, including:

- Detecting development vs production mode
- Generating the manifest file
- Handling hot reload
- Processing CSS imports
