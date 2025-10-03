# Quick Start

Use the following steps to get up and running with a fresh project using [InertiaNode](https://github.com/inertianode). If you would like to install InertiaNode into an existing project, please refer to the [installation instructions](/server-side-setup.md).

If you are unfamiliar with InertiaNode, please refer to the [Introduction](/index.md) page.

## Prerequisites

- Node.js 18.0 or higher

## New Project

You can use one of our starter kits to get started quickly.

```bash
npm create inertianode@latest
```

You will be asked to enter a project name and to select the server framework (Hono, Express, or Koa) and front-end library (React, Vue, or Svelte) you want to use.

This will create a new project with the name you provide.

### Run the project

Once you have created the project, you can run it using the following command:

```bash
npm run dev
```

This will start both the development server and the Vite build process. You can access the application at `http://localhost:3000` (the exact port may vary depending on your configuration).

## Existing Project

If you are looking to install InertiaNode into an existing project, please refer to the [installation instructions](/server-side-setup.md).
