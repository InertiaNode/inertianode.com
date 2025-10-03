# Merging props

Inertia overwrites props with the same name when reloading a page. However, you may need to merge new data with existing data instead. For example, when implementing a "load more" button for paginated results.

Prop merging only works during [partial reloads](/partial-reloads). Full page visits will always replace props entirely, even if you've marked them for merging.

## Server-side configuration

To merge a prop instead of overwriting it, use the `merge()` helper when returning your response:

```ts
// framework: hono
import { merge } from '@inertianode/hono';

app.get('/posts', async (c) => {
  const posts = await postService.getPaginated();

  return await c.Inertia('Posts/Index', {
    posts: merge(posts)
  });
});
```

```ts
// framework: express
import { merge } from '@inertianode/express';

app.get('/posts', async (req, res) => {
  const posts = await postService.getPaginated();

  await res.Inertia('Posts/Index', {
    posts: merge(posts)
  });
});
```

```ts
// framework: koa
import { merge } from '@inertianode/koa';

router.get('/posts', async (ctx) => {
  const posts = await postService.getPaginated();

  await ctx.Inertia('Posts/Index', {
    posts: merge(posts)
  });
});
```

### Deep merging

For nested objects, you can use `deepMerge()` to recursively merge the entire structure:

```ts
// framework: hono
import { deepMerge } from '@inertianode/hono';

return await c.Inertia('Dashboard', {
  analytics: deepMerge(analyticsData)
});
```

```ts
// framework: express
import { deepMerge } from '@inertianode/express';

await res.Inertia('Dashboard', {
  analytics: deepMerge(analyticsData)
});
```

```ts
// framework: koa
import { deepMerge } from '@inertianode/koa';

await ctx.Inertia('Dashboard', {
  analytics: deepMerge(analyticsData)
});
```

### Prepending vs appending

By default, new items are appended to arrays. You can change this behavior:

```ts
// framework: hono
import { merge } from '@inertianode/hono';

// Prepend new items
return await c.Inertia('Posts/Index', {
  posts: merge(posts).prepend()
});
```

```ts
// framework: express
import { merge } from '@inertianode/express';

await res.Inertia('Posts/Index', {
  posts: merge(posts).prepend()
});
```

```ts
// framework: koa
import { merge } from '@inertianode/koa';

await ctx.Inertia('Posts/Index', {
  posts: merge(posts).prepend()
});
```

## Matching items

When merging arrays, you can match existing items by a specific field and update them instead of appending new ones:

```ts
// framework: hono
import { merge } from '@inertianode/hono';

// Match items by 'id' field
return await c.Inertia('Users/Index', {
  users: merge(users).matchOn('id')
});
```

```ts
// framework: express
import { merge } from '@inertianode/express';

await res.Inertia('Users/Index', {
  users: merge(users).matchOn('id')
});
```

```ts
// framework: koa
import { merge } from '@inertianode/koa';

await ctx.Inertia('Users/Index', {
  users: merge(users).matchOn('id')
});
```

## Client side

On the client side, Inertia detects that this prop should be merged. If the prop returns an array, it will append the response to the current prop value. If it's an object, it will merge the response with the current prop value. If you have opted to `DeepMerge`, Inertia ensures a deep merge of the entire structure.

## Client side visits

You can also merge props directly on the client side without making a server request using [client side visits](/manual-visits#client-side-visits). Inertia provides [prop helper methods](/manual-visits#prop-helpers) that allow you to append, prepend, or replace prop
values.

## Combining with deferred props

You can also combine [deferred props](/deferred-props) with mergeable props to defer the loading of the prop and ultimately mark it as mergeable once it's loaded.

## Resetting props

On the client side, you can indicate to the server that you would like to reset the prop. This is useful when you want to clear the prop value before merging new data, such as when the user enters a new search query on a paginated list.

The `reset` request option accepts an array of the props keys you would like to reset.

```js
// framework: vue
router.reload({
  reset: ["results"],
  // ...
});
```

```js
// framework: react
router.reload({
  reset: ["results"],
  // ...
});
```

```js
// framework: svelte
router.reload({
  reset: ["results"],
  // ...
});
```
