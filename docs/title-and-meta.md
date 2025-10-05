# Title &amp; meta

Since Inertia powered JavaScript apps are rendered within the document `<body>`, they are unable to render markup to the document `<head>`, as it's outside of their scope. To help with this, Inertia ships with a `<Head>` component which can be used to set the page `<title>`, `<meta>` tags, and other `<head>` elements.

> [!WARNING] The `<Head>` component will only replace elements that are not in your server-side root template.

> [!WARNING] The `<Head>` component is not available in the Svelte adapter, as Svelte already ships with its own `<svelte:head>` component.

## Head component

To add `<head>` elements to your page, use the `<Head>` component. Within this component, you can include the elements that you wish to add to the document `<head>`.

```jsx
// framework: vue
import { Head } from '@inertiajs/vue3'
<Head>
  <title>Your page title</title>
  <meta name="description" content="Your page description">
</Head>
```

```jsx
// framework: react
import { Head } from "@inertiajs/react";
<Head>
  <title>Your page title</title>
  <meta name="description" content="Your page description" />
</Head>;
```

```svelte
// framework: svelte
<svelte:head>
  <title>Your page title</title>
  <meta name="description" content="Your page description" />
</svelte:head>
```

## Title shorthand

If you only need to add a `<title>` to the document `<head>`, you may simply pass the title as a prop to the `<Head>` component.

```jsx
// framework: vue
import { Head } from "@inertiajs/vue3";
<Head title="Your page title" />;
```

```jsx
// framework: react
import { Head } from "@inertiajs/react";
<Head title="Your page title" />;
```

```js
// framework: svelte
// Not supported
```

## Title callback

You can globally modify the page `<title>` using the `title` callback in the `createInertiaApp` setup method. Typically, this method is invoked in your application's main JavaScript file. A common use case for the title callback is automatically adding an app name before or after each page title.

```js
createInertiaApp({
  title: (title) => `${title} - My App`,
  // ...
});
```

After defining the `title` callback, the callback will automatically be invoked when you set a title using the `<Head>` component.

```jsx
// framework: vue
import { Head } from '@inertiajs/vue3'
<Head title="Home">
```

```jsx
// framework: react
import { Head } from '@inertiajs/react'
<Head title="Home">
```

```js
// framework: svelte
// Not supported
```

Which, in this example, will result in the following `<title>` tag.

```html
<title>Home - My App</title>
```

The `title` callback will also be invoked when you set the title using a `<title>` tag within your `<Head>` component.

```jsx
// framework: vue
import { Head } from "@inertiajs/vue3";
<Head>
  <title>Home</title>
</Head>;
```

```jsx
// framework: react
import { Head } from "@inertiajs/react";
<Head>
  <title>Home</title>
</Head>;
```

```js
// framework: svelte
// Not supported
```

## Multiple Head instances

It's possible to have multiple instances of the `<Head>` component throughout your application. For example, your layout can set some default `<Head>` elements, and then your individual pages can override those defaults.

Vue:

```jsx
// Layout.vue
import { Head } from "@inertiajs/vue3";
<Head>
  <title>My app</title>
  <meta
    head-key="description"
    name="description"
    content="This is the default description"
  />
  <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
</Head>;
// About.vue
import { Head } from "@inertiajs/vue3";
<Head>
  <title>About - My app</title>
  <meta
    head-key="description"
    name="description"
    content="This is a page specific description"
  />
</Head>;
```

```jsx
// framework: react
// Layout.js
import { Head } from "@inertiajs/react";
<Head>
  <title>My app</title>
  <meta
    head-key="description"
    name="description"
    content="This is the default description"
  />
  <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
</Head>;
// About.js
import { Head } from "@inertiajs/react";
<Head>
  <title>About - My app</title>
  <meta
    head-key="description"
    name="description"
    content="This is a page specific description"
  />
</Head>;
```

```js
// framework: svelte
// Not supported
```

Inertia will only ever render one `<title>` tag; however, all other tags will be stacked since it's valid to have multiple instances of them. To avoid duplicate tags in your `<head>`, you can use the `head-key` property, which will make sure the tag is only rendered once. This is illustrated in the example above for the `<meta name="description">` tag.

The code example above will render the following HTML.

```html
<head>
  <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
  <title>About - My app</title>
  <meta name="description" content="This is a page specific description" />
</head>
```

## Head extension

When building a real application, it can sometimes be helpful to create a custom head component that extends Inertia's `<Head>` component. This gives you a place to set app-wide defaults, such as appending the app name to the page title.

```vue
// framework: vue
<!-- AppHead.vue -->

<script setup>
import { Head } from "@inertiajs/vue3";

defineProps({ title: String });
</script>

<template>
  <Head :title="title ? `${title} - My App` : 'My App'">
    <slot />
  </Head>
</template>
```

```jsx
// framework: react
// AppHead.js

import { Head } from "@inertiajs/react";

const Site = ({ title, children }) => {
  return (
    <Head>
      <title>{title ? `${title} - My App` : "My App"}</title>
      {children}
    </Head>
  );
};

export default Site;
```

```js
// framework: svelte
// Not supported
```

Once you have created the custom component, you can just start using it in your pages.

```jsx
// framework: vue
import AppHead from "./AppHead";
<AppHead title="About" />;
```

```jsx
// framework: react
import AppHead from "./AppHead";
<AppHead title="About" />;
```

```js
// framework: svelte
// Not supported
```
