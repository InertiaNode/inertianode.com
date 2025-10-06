# Infinite scroll

Inertia's infinite scroll feature loads additional pages of content as users scroll, replacing traditional pagination controls. This is great for applications like chat interfaces, social feeds, photo grids, and product listings.

## Server-side

To configure your paginated data for infinite scrolling, you should use the `scroll()` helper when returning your response. This helper automatically configures the proper merge behavior and normalizes pagination metadata for the frontend component.

```ts
// framework: hono
import { Hono } from "hono";
import { scroll } from "@inertianode/hono";

const app = new Hono();

app.get("/users", async (c) => {
  return await c.Inertia("Users/Index", {
    users: scroll(() => userService.getPaginated()),
  });
});
```

```ts
// framework: express
import express from "express";
import { scroll } from "@inertianode/express";

const app = express();

app.get("/users", async (req, res) => {
  await res.Inertia("Users/Index", {
    users: scroll(() => userService.getPaginated()),
  });
});
```

```ts
// framework: nestjs
import { Controller, Get } from "@nestjs/common";
import { Inert, type Inertia } from "@inertianode/nestjs";
import { scroll } from "@inertianode/express";

@Controller()
export class UsersController {
  @Get("/users")
  async index(@Inert() inertia: Inertia) {
    await inertia("Users/Index", {
      users: scroll(() => userService.getPaginated()),
    });
  }
}
```

```ts
// framework: koa
import Koa from "koa";
import Router from "@koa/router";
import { scroll } from "@inertianode/koa";

const app = new Koa();
const router = new Router();

router.get("/users", async (ctx) => {
  await ctx.Inertia("Users/Index", {
    users: scroll(() => userService.getPaginated()),
  });
});

app.use(router.routes());
```

The `scroll()` helper works with various pagination methods. For more details, check out the [scroll() method](#scroll-method) documentation.

## Client-side

On the client side, Inertia provides the `<InfiniteScroll>` component to automatically load additional pages of content. The component accepts a `data` prop that specifies the key of the prop containing your paginated data. The `<InfiniteScroll>` component should wrap the content that depends on the paginated data.

```html
<!-- framework: vue -->
<InfiniteScroll data="users">
  <div v-for="user in users.data" :key="user.id">{{ user.name }}</div>
</InfiniteScroll>
```

```jsx
// framework: react
import { InfiniteScroll } from "@inertiajs/react";
export default function Users({ users }) {
  return (
    <InfiniteScroll data="users">
      {users.data.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </InfiniteScroll>
  );
}
```

```html
<!-- framework: svelte4 -->
<script>
  import { InfiniteScroll } from "@inertiajs/svelte";
  export let users;
</script>
<InfiniteScroll data="users">
  {#each users.data as user (user.id)}
  <div>{user.name}</div>
  {/each}
</InfiniteScroll>
```

The component uses [intersection observers ](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)to detect when users scroll near the end of the content and automatically triggers requests to load the next page. New data is merged with existing content rather than replacing it.

## Loading buffer

You can control how early content begins loading by setting a buffer distance. The buffer specifies how many pixels before the end of the content loading should begin.

```html
<!-- framework: vue -->
<InfiniteScroll data="users" :buffer="500">
  <!-- ... -->
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll data="users" buffer={500}>
  {/* ... */}
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll data="users" buffer="{500}">
  <!-- ... -->
</InfiniteScroll>
```

In the example above, content will start loading 500 pixels before reaching the end of the current content. A larger buffer loads content earlier but potentially loads content that users may never see.

## URL synchronization

The infinite scroll component updates the browser URL's query string (`?page=...`) as users scroll through content. The URL reflects which page has the most visible items on screen, updating in both directions as users scroll up or down. This allows users to bookmark or share links to specific pages. You can disable this behavior to maintain the original page URL.

```html
<!-- framework: vue -->
<InfiniteScroll data="users" preserve-url>
  <!-- ... -->
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll data="users" preserveUrl>
  {/* ... */}
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll data="users" preserve-url>
  <!-- ... -->
</InfiniteScroll>
```

This is useful when infinite scroll is used for secondary content that shouldn't affect the main page URL, such as comments on a blog post or related products on a product page.

## Resetting

When filters or other parameters change, you may need to reset the infinite scroll data to start from the beginning. Without resetting, new results will merge with existing content instead of replacing it.

You can reset data using the `reset` visit option.

```html
<!-- framework: vue -->
<script setup>
  import { router } from "@inertiajs/vue3";
  const show = (role) => {
    router.visit(route("users"), {
      data: { filter: { role } },
      only: ["users"],
      reset: ["users"],
    });
  };
</script>
<template>
  <button @click="show('admin')">Show admins</button>
  <button @click="show('customer')">Show customers</button>
  <InfiniteScroll data="users">
    <div v-for="user in users.data" :key="user.id">{{ user.name }}</div>
  </InfiniteScroll>
</template>
```

```jsx
// framework: react
import { InfiniteScroll, router } from "@inertiajs/react";
export default function Users({ users }) {
  const show = (role) => {
    router.visit(route("users"), {
      data: { filter: { role } },
      only: ["users"],
      reset: ["users"],
    });
  };
  return (
    <>
      <button onClick={() => show("admin")}>Show admins</button>
      <button onClick={() => show("customer")}>Show customers</button>
      <InfiniteScroll data="users">
        {users.data.map((user) => (
          <div key={user.id}>{user.name}</div>
        ))}
      </InfiniteScroll>
    </>
  );
}
```

```html
<!-- framework: svelte4 -->
<script>
import { InfiniteScroll, router } from '@inertiajs/svelte'
export let users
const show = (role) => {
  router.visit(route('users'), {
    data: { filter: { role } },
    only: ['users'],
    reset: ['users'],
  })
}
</script>
<button on:click={() => show('admin')}>Show admins</button>
<button on:click={() => show('customer')">Show customers</button>
<InfiniteScroll data="users">
  {#each users.data as user (user.id)}
    <div>{user.name}</div>
  {/each}
</InfiniteScroll>
```

For more information about the reset option, see the [Resetting props](/merging-props#resetting-props) documentation.

## Loading direction

The infinite scroll component loads content in both directions when you scroll near the start or end. You can control this behavior using the `only-next` and `only-previous` props.

```html
<!-- framework: vue -->
<!-- Only load the next page -->
<InfiniteScroll data="users" only-next>
  <!-- ... -->
</InfiniteScroll>
<!-- Only load the previous page -->
<InfiniteScroll data="messages" only-previous>
  <!-- ... -->
</InfiniteScroll>
<!-- Load in both directions (default) -->
<InfiniteScroll data="posts">
  <!-- ... -->
</InfiniteScroll>
```

```jsx
// framework: react
{
  /* Only load the next page */
}
<InfiniteScroll data="users" onlyNext>
  {/* ... */}
</InfiniteScroll>;
{
  /* Only load the previous page */
}
<InfiniteScroll data="messages" onlyPrevious>
  {/* ... */}
</InfiniteScroll>;
{
  /* Load in both directions (default) */
}
<InfiniteScroll data="posts">{/* ... */}</InfiniteScroll>;
```

```html
<!-- framework: svelte4 -->
<!-- Only load the next page -->
<InfiniteScroll data="users" only-next>
  <!-- ... -->
</InfiniteScroll>
<!-- Only load the previous page -->
<InfiniteScroll data="messages" only-previous>
  <!-- ... -->
</InfiniteScroll>
<!-- Load in both directions (default) -->
<InfiniteScroll data="posts">
  <!-- ... -->
</InfiniteScroll>
```

The default option is particularly useful when users start on a middle page and need to scroll in both directions to access all content.

## Reverse mode

For chat applications, timelines, or interfaces where content is sorted descendingly (newest items at the bottom), you can enable reverse mode. This configures the component to load older content when scrolling upward.

```html
<!-- framework: vue -->
<InfiniteScroll data="messages" reverse>
  <!-- ... -->
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll data="messages" reverse>
  {/* ... */}
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll data="messages" reverse>
  <!-- ... -->
</InfiniteScroll>
```

In reverse mode, the component flips the loading directions so that scrolling up loads the next page (older content) and scrolling down loads the previous page (newer content). The component handles the loading positioning, but you are responsible for reversing your content to display in the correct order.

Reverse mode also enables automatic scrolling to the bottom on initial load, which you can disable with `:auto-scroll="false"`.

```html
<!-- framework: vue -->
<InfiniteScroll data="messages" reverse :auto-scroll="false">
  <!-- ... -->
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll data="messages" reverse autoScroll={false}>
  {/* ... */}
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll data="messages" reverse auto-scroll="{false}">
  <!-- ... -->
</InfiniteScroll>
```

## Manual mode

Manual mode disables automatic loading when scrolling and allows you to control when content loads through the `next` and `previous` slots. For more details about available slot properties and customization options, see the [Slots](#slots) documentation.

```html
<!-- framework: vue -->
<InfiniteScroll data="users" manual>
  <template #previous="{ loading, fetch, hasMore }">
    <button v-if="hasMore" @click="fetch" :disabled="loading">
      {{ loading ? 'Loading...' : 'Load previous' }}
    </button>
  </template>
  <!-- Your content -->
  <template #next="{ loading, fetch, hasMore }">
    <button v-if="hasMore" @click="fetch" :disabled="loading">
      {{ loading ? 'Loading...' : 'Load more' }}
    </button>
  </template>
</InfiniteScroll>
```

```jsx
// framework: react
import { InfiniteScroll } from "@inertiajs/react";
export default ({ users }) => (
  <InfiniteScroll
    data="users"
    manual
    previous={({ loading, fetch, hasMore }) =>
      hasMore && (
        <button onClick={fetch} disabled={loading}>
          {loading ? "Loading..." : "Load previous"}
        </button>
      )
    }
    next={({ loading, fetch, hasMore }) =>
      hasMore && (
        <button onClick={fetch} disabled={loading}>
          {loading ? "Loading..." : "Load more"}
        </button>
      )
    }
  >
    {users.data.map((user) => (
      <div key={user.id}>{user.name}</div>
    ))}
  </InfiniteScroll>
);
```

```html
<!-- framework: svelte4 -->
<script>
  import { InfiniteScroll } from "@inertiajs/svelte";
  export let users;
</script>
<InfiniteScroll data="users" manual>
  <div slot="previous" let:hasMore let:fetch let:loading>
    {#if hasMore}
    <button on:click="{fetch}" disabled="{loading}">
      {loading ? 'Loading...' : 'Load previous'}
    </button>
    {/if}
  </div>
  {#each users.data as user (user.id)}
  <div>{user.name}</div>
  {/each}
  <div slot="next" let:hasMore let:fetch let:loading>
    {#if hasMore}
    <button on:click="{fetch}" disabled="{loading}">
      {loading ? 'Loading...' : 'Load more'}
    </button>
    {/if}
  </div>
</InfiniteScroll>
```

You can also configure the component to automatically switch to manual mode after a certain number of pages using the `manualAfter` prop.

```html
<!-- framework: vue -->
<InfiniteScroll data="users" :manual-after="3">
  <!-- ... -->
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll data="users" manualAfter={3}>
  {/* ... */}
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll data="users" manual-after="{3}">
  <!-- ... -->
</InfiniteScroll>
```

## Slots

The infinite scroll component provides several slots to customize the loading experience. These slots allow you to display custom loading indicators and create manual load controls. Each slot receives properties that provide loading state information and functions to trigger content loading.

### Default slot

The main content area where you render your data items. This slot receives loading state information.

```html
<!-- framework: vue -->
<InfiniteScroll
  data="users"
  #default="{ loading, loadingPrevious, loadingNext }"
>
  <!-- Your content with access to loading states -->
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll data="users">
  {({ loading, loadingPrevious, loadingNext }) => (
    <div>{/* Your content with access to loading states */}</div>
  )}
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll data="users" let:loading let:loadingPrevious let:loadingNext>
  <!-- Your content with access to loading states -->
</InfiniteScroll>
```

### Loading slot

The loading slot is used as a fallback when loading content and no custom `before` or `after` slots are provided. This creates a default loading indicator.

```html
<!-- framework: vue -->
<InfiniteScroll data="users">
  <!-- Your content -->
  <template #loading> Loading more users... </template>
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll data="users" loading={() => "Loading more users..."}>
  {/* Your content */}
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll data="users">
  <!-- Your content -->
  <div slot="loading">Loading more users...</div>
</InfiniteScroll>
```

### Previous and next slots

The `previous` and `next` slots are rendered above and below the main content, typically used for manual load controls. These slots receive several properties including loading states, fetch functions, and mode indicators.

```html
<!-- framework: vue -->
<InfiniteScroll data="users" :manual-after="3">
  <template #previous="{ loading, fetch, hasMore, manualMode }">
    <button v-if="manualMode && hasMore" @click="fetch" :disabled="loading">
      {{ loading ? 'Loading...' : 'Load previous' }}
    </button>
  </template>
  <!-- Your content -->
  <template #next="{ loading, fetch, hasMore, manualMode }">
    <button v-if="manualMode && hasMore" @click="fetch" :disabled="loading">
      {{ loading ? 'Loading...' : 'Load more' }}
    </button>
  </template>
</InfiniteScroll>
```

The `loading`, `previous`, and `next` slots receive the following properties:

- `loading` - Whether the slot is currently loading content
- `loadingPrevious` - Whether previous content is loading
- `loadingNext` - Whether next content is loading
- `fetch` - Function to trigger loading for the slot
- `hasMore` - Whether more content is available for the slot
- `hasPrevious` - Whether more previous content is available
- `hasNext` - Whether more next content is available
- `manualMode` - Whether manual mode is active
- `autoMode` - Whether automatic loading is active

## Custom element

The `InfiniteScroll` component renders as a `<div>` element. You may customize this to use any HTML element using the `as` prop.

```html
<!-- framework: vue -->
<InfiniteScroll data="products" as="ul">
  <li v-for="product in products.data" :key="product.id">{{ product.name }}</li>
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll data="products" as="ul">
  {products.data.map((product) => (
    <li key={product.id}>{product.name}</li>
  ))}
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll data="products" as="ul">
  {#each products.data as product (product.id)}
  <li>{product.name}</li>
  {/each}
</InfiniteScroll>
```

## Element targeting

The infinite scroll component automatically tracks content and assigns page numbers to elements for [URL synchronization](#url-synchronization). When your data items are not direct children of the component's root element, you need to specify which element contains the actual data items using the `itemsElement` prop.

```html
<!-- framework: vue -->
<InfiniteScroll data="users" items-element="#table-body">
  <table>
    <thead>
      <tr>
        <th>Name</th>
      </tr>
    </thead>
    <tbody id="table-body">
      <tr v-for="user in users.data" :key="user.id">
        <td>{{ user.name }}</td>
      </tr>
    </tbody>
  </table>
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll data="users" itemsElement="#table-body">
  <table>
    <thead>
      <tr>
        <th>Name</th>
      </tr>
    </thead>
    <tbody id="table-body">
      {users.data.map((user) => (
        <tr key={user.id}>
          <td>{user.name}</td>
        </tr>
      ))}
    </tbody>
  </table>
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll data="users" items-element="#table-body">
  <table>
    <thead>
      <tr>
        <th>Name</th>
      </tr>
    </thead>
    <tbody id="table-body">
      {#each users.data as user (user.id)}
      <tr>
        <td>{user.name}</td>
      </tr>
      {/each}
    </tbody>
  </table>
</InfiniteScroll>
```

In this example, the component monitors the `#table-body` element and automatically tags each `<tr>` with a page number as new content loads. This enables proper URL updates based on which page's content is most visible in the viewport.

You can also specify custom trigger elements for loading more content using CSS selectors. This prevents the default trigger elements from being rendered and uses intersection observers on your custom elements instead.

```html
<!-- framework: vue -->
<InfiniteScroll
  data="users"
  items-element="#table-body"
  start-element="#table-header"
  end-element="#table-footer"
>
  <table>
    <thead id="table-header">
      <tr>
        <th>Name</th>
      </tr>
    </thead>
    <tbody id="table-body">
      <tr v-for="user in users.data" :key="user.id">
        <td>{{ user.name }}</td>
      </tr>
    </tbody>
    <tfoot id="table-footer">
      <tr>
        <td>Footer</td>
      </tr>
    </tfoot>
  </table>
</InfiniteScroll>
```

```jsx
// framework: react
<InfiniteScroll
  data="users"
  itemsElement="#table-body"
  startElement="#table-header"
  endElement="#table-footer"
>
  <table>
    <thead id="table-header">
      <tr>
        <th>Name</th>
      </tr>
    </thead>
    <tbody id="table-body">
      {users.data.map((user) => (
        <tr key={user.id}>
          <td>{user.name}</td>
        </tr>
      ))}
    </tbody>
    <tfoot id="table-footer">
      <tr>
        <td>Footer</td>
      </tr>
    </tfoot>
  </table>
</InfiniteScroll>
```

```html
<!-- framework: svelte4 -->
<InfiniteScroll
  data="users"
  items-element="#table-body"
  start-element="#table-header"
  end-element="#table-footer"
>
  <table>
    <thead id="table-header">
      <tr>
        <th>Name</th>
      </tr>
    </thead>
    <tbody id="table-body">
      {#each users.data as user (user.id)}
      <tr>
        <td>{user.name}</td>
      </tr>
      {/each}
    </tbody>
    <tfoot id="table-footer">
      <tr>
        <td>Footer</td>
      </tr>
    </tfoot>
  </table>
</InfiniteScroll>
```

Alternatively, you can use template refs instead of CSS selectors. This avoids adding HTML attributes and provides direct element references.

```html
<!-- framework: vue -->
<script setup>
  import { ref } from "vue";
  const tableHeader = ref();
  const tableFooter = ref();
  const tableBody = ref();
</script>
<template>
  <InfiniteScroll
    data="users"
    :items-element="() => tableBody"
    :start-element="() => tableHeader"
    :end-element="() => tableFooter"
  >
    <table>
      <thead ref="tableHeader">
        <tr>
          <th>Name</th>
        </tr>
      </thead>
      <tbody ref="tableBody">
        <tr v-for="user in users.data" :key="user.id">
          <td>{{ user.name }}</td>
        </tr>
      </tbody>
      <tfoot ref="tableFooter">
        <tr>
          <td>Footer</td>
        </tr>
      </tfoot>
    </table>
  </InfiniteScroll>
</template>
```

```jsx
// framework: react
import { useRef } from "react";
export default ({ users }) => {
  const tableHeader = useRef();
  const tableFooter = useRef();
  const tableBody = useRef();
  return (
    <InfiniteScroll
      data="users"
      itemsElement={() => tableBody.current}
      startElement={() => tableHeader.current}
      endElement={() => tableFooter.current}
    >
      <table>
        <thead ref={tableHeader}>
          <tr>
            <th>Name</th>
          </tr>
        </thead>
        <tbody ref={tableBody}>
          {users.data.map((user) => (
            <tr key={user.id}>
              <td>{user.name}</td>
            </tr>
          ))}
        </tbody>
        <tfoot ref={tableFooter}>
          <tr>
            <td>Footer</td>
          </tr>
        </tfoot>
      </table>
    </InfiniteScroll>
  );
};
```

```html
<!-- framework: svelte4 -->
<script>
import { InfiniteScroll } from '@inertiajs/svelte'
export let users
let tableHeader
let tableFooter
let tableBody
</script>
<InfiniteScroll
  data="users"
  items-element={() => tableBody}
  start-element={() => tableHeader}
  end-element={() => tableFooter}
>
  <table>
    <thead bind:this={tableHeader}>
      <tr><th>Name</th></tr>
    </thead>
    <tbody bind:this={tableBody}>
      {#each users.data as user (user.id)}
        <tr>
          <td>{user.name}</td>
        </tr>
      {/each}
    </tbody>
    <tfoot bind:this={tableFooter}>
      <tr><td>Footer</td></tr>
    </tfoot>
  </table>
</InfiniteScroll>
```

## Scroll containers

The infinite scroll component works within any scrollable container, not just the main document. The component automatically adapts to use the custom scroll container for trigger detection and calculations instead of the main document scroll.

```html
<!-- framework: vue -->
<div style="height: 400px; overflow-y: auto;">
  <InfiniteScroll data="users">
    <div v-for="user in users.data" :key="user.id">{{ user.name }}</div>
  </InfiniteScroll>
</div>
```

```jsx
// framework: react
<div style={{ height: "400px", overflowY: "auto" }}>
  <InfiniteScroll data="users">
    {users.data.map((user) => (
      <div key={user.id}>{user.name}</div>
    ))}
  </InfiniteScroll>
</div>
```

```html
<!-- framework: svelte4 -->
<div style="height: 400px; overflow-y: auto;">
  <InfiniteScroll data="users">
    {#each users.data as user (user.id)}
    <div>{user.name}</div>
    {/each}
  </InfiniteScroll>
</div>
```

### Multiple scroll containers

Sometimes you may need to render multiple infinite scroll components on a single page. However, if both components use the default `page` query parameter for [URL synchronization](#url-synchronization), they will conflict with each other. To resolve this, instruct each paginator to use a custom `pageName`.

```ts
// framework: hono
import { Hono } from "hono";
import { scroll } from "@inertianode/hono";

const app = new Hono();

app.get("/dashboard", async (c) => {
  return await c.Inertia("Dashboard", {
    users: scroll(() => userService.getPaginated({ pageName: "users" })),
    orders: scroll(() => orderService.getPaginated({ pageName: "orders" })),
  });
});
```

```ts
// framework: express
import express from "express";
import { scroll } from "@inertianode/express";

const app = express();

app.get("/dashboard", async (req, res) => {
  await res.Inertia("Dashboard", {
    users: scroll(() => userService.getPaginated({ pageName: "users" })),
    orders: scroll(() => orderService.getPaginated({ pageName: "orders" })),
  });
});
```

```ts
// framework: nestjs
import { Controller, Get } from "@nestjs/common";
import { Inert, type Inertia } from "@inertianode/nestjs";
import { scroll } from "@inertianode/express";

@Controller()
export class DashboardController {
  @Get("/dashboard")
  async index(@Inert() inertia: Inertia) {
    await inertia("Dashboard", {
      users: scroll(() => userService.getPaginated({ pageName: "users" })),
      orders: scroll(() => orderService.getPaginated({ pageName: "orders" })),
    });
  }
}
```

```ts
// framework: koa
import Koa from "koa";
import Router from "@koa/router";
import { scroll } from "@inertianode/koa";

const app = new Koa();
const router = new Router();

router.get("/dashboard", async (ctx) => {
  await ctx.Inertia("Dashboard", {
    users: scroll(() => userService.getPaginated({ pageName: "users" })),
    orders: scroll(() => orderService.getPaginated({ pageName: "orders" })),
  });
});

app.use(router.routes());
```

The `scroll()` helper automatically detects the `pageName` from each paginator, allowing both scroll containers to maintain independent pagination state. This results in URLs like `?users=2&orders=3` instead of conflicting `?page=` parameters.

## Programmatic access

When you need to trigger loading actions programmatically, you may use a template ref.

```html
<!-- framework: vue -->
<script setup>
  import { ref } from "vue";
  const infiniteScrollRef = ref(null);
  const fetchNext = () => {
    infiniteScrollRef.value?.fetchNext();
  };
</script>
<template>
  <button @click="fetchNext">Load More</button>
  <InfiniteScroll ref="infiniteScrollRef" data="users" manual>
    <!-- Your content -->
  </InfiniteScroll>
</template>
```

```jsx
// framework: react
import { InfiniteScroll } from '@inertiajs/react'
import { useRef } from 'react'
export default ({ users }) => {
const infiniteScrollRef = useRef(null)
const fetchNext = () => {
infiniteScrollRef.current?.fetchNext()
}
return (
<button onClick={fetchNext}>Load More</button>
<InfiniteScroll ref={infiniteScrollRef} data="users" manual>
{users.data.map(user => (
<div key={user.id}>{user.name}</div>
))}
</InfiniteScroll>
)
}
```

```html
<!-- framework: svelte4 -->
<script>
  import { InfiniteScroll } from "@inertiajs/svelte";
  export let users;
  let infiniteScrollRef;
  const fetchNext = () => {
    infiniteScrollRef?.fetchNext();
  };
</script>
<button on:click="{fetchNext}">Load More</button>
<InfiniteScroll bind:this="{infiniteScrollRef}" data="users" manual>
  {#each users.data as user (user.id)}
  <div>{user.name}</div>
  {/each}
</InfiniteScroll>
```

The component exposes the following methods:

- `fetchNext()` - Manually fetch the next page
- `fetchPrevious()` - Manually fetch the previous page
- `hasNext()` - Whether there is a next page
- `hasPrevious()` - Whether there is a previous page

## scroll() method

The `scroll()` helper provides server-side configuration for infinite scrolling. It automatically configures the proper merge behavior so that new data is appended or prepended to existing content instead of replacing it, and normalizes pagination metadata for the frontend component.

```ts
// framework: hono
import { scroll } from "@inertianode/core";

// Works with various pagination methods...
scroll(() => userService.getPaginated(20));
scroll(() => userService.getSimplePaginated(20));
scroll(() => userService.getCursorPaginated(20));
```

```ts
// framework: express
import { scroll } from "@inertianode/express";

// Works with various pagination methods...
scroll(() => userService.getPaginated(20));
scroll(() => userService.getSimplePaginated(20));
scroll(() => userService.getCursorPaginated(20));
```

```ts
// framework: nestjs
import { scroll } from "@inertianode/express";

// Works with various pagination methods...
scroll(() => userService.getPaginated(20));
scroll(() => userService.getSimplePaginated(20));
scroll(() => userService.getCursorPaginated(20));
```

```ts
// framework: koa
import { scroll } from "@inertianode/koa";

// Works with various pagination methods...
scroll(() => userService.getPaginated(20));
scroll(() => userService.getSimplePaginated(20));
scroll(() => userService.getCursorPaginated(20));
```

If you use a different pagination approach or need custom configuration, you may use the additional parameters that `scroll()` accepts.

```ts
// framework: hono
import { scroll } from "@inertianode/core";

// Customize the data wrapper key (defaults to 'data')...
scroll(customPaginatedData, { wrapper: "items" });
// Provide custom metadata resolution...
scroll(data, { metadata: metadataProvider });
```

```ts
// framework: express
import { scroll } from "@inertianode/express";

// Customize the data wrapper key (defaults to 'data')...
scroll(customPaginatedData, { wrapper: "items" });
// Provide custom metadata resolution...
scroll(data, { metadata: metadataProvider });
```

```ts
// framework: nestjs
import { scroll } from "@inertianode/express";

// Customize the data wrapper key (defaults to 'data')...
scroll(customPaginatedData, { wrapper: "items" });
// Provide custom metadata resolution...
scroll(data, { metadata: metadataProvider });
```

```ts
// framework: koa
import { scroll } from "@inertianode/koa";

// Customize the data wrapper key (defaults to 'data')...
scroll(customPaginatedData, { wrapper: "items" });
// Provide custom metadata resolution...
scroll(data, { metadata: metadataProvider });
```

The metadata parameter accepts an object that implements the scroll metadata interface or a callback that returns such an object. This is useful when integrating with custom pagination libraries.

```ts
// Custom metadata interface
interface ScrollMetadata {
  getPageName(): string;
  getPreviousPage(): number | null;
  getNextPage(): number | null;
  getCurrentPage(): number | null;
}

// Example implementation
class CustomScrollMetadata implements ScrollMetadata {
  constructor(private resource: PaginatedCollection) {}
  getPageName() {
    return "page";
  }
  getPreviousPage() {
    return this.resource.previousPage;
  }
  getNextPage() {
    return this.resource.nextPage;
  }
  getCurrentPage() {
    return this.resource.currentPage;
  }
}
```

You may then use this custom metadata provider in your scroll helper.

```ts
// framework: hono
import { scroll } from "@inertianode/core";

// Using an instance directly
scroll(data, { metadata: new CustomScrollMetadata(data) });

// Using a callback
scroll(() => transformData(data), {
  metadata: (d) => new CustomScrollMetadata(d),
});
```

```ts
// framework: express
import { scroll } from "@inertianode/express";

// Using an instance directly
scroll(data, { metadata: new CustomScrollMetadata(data) });

// Using a callback
scroll(() => transformData(data), {
  metadata: (d) => new CustomScrollMetadata(d),
});
```

```ts
// framework: nestjs
import { scroll } from "@inertianode/express";

// Using an instance directly
scroll(data, { metadata: new CustomScrollMetadata(data) });

// Using a callback
scroll(() => transformData(data), {
  metadata: (d) => new CustomScrollMetadata(d),
});
```

```ts
// framework: koa
import { scroll } from "@inertianode/koa";

// Using an instance directly
scroll(data, { metadata: new CustomScrollMetadata(data) });

// Using a callback
scroll(() => transformData(data), {
  metadata: (d) => new CustomScrollMetadata(d),
});
```

To avoid repeating this setup in multiple routes, you may create a helper function.

```ts
// framework: hono
import { Hono } from "hono";
import { scroll } from "@inertianode/hono";

// Helper function
function customScroll(data: PaginatedCollection) {
  return scroll(data, {
    metadata: (d) => new CustomScrollMetadata(d),
  });
}

// Then use it in your routes
const app = new Hono();
app.get("/users", async (c) => {
  return await c.Inertia("Users/Index", {
    users: customScroll(userService.getCustomPaginated()),
  });
});
```

```ts
// framework: express
import express from "express";
import { scroll } from "@inertianode/express";

// Helper function
function customScroll(data: PaginatedCollection) {
  return scroll(data, {
    metadata: (d) => new CustomScrollMetadata(d),
  });
}

// Then use it in your routes
const app = express();
app.get("/users", async (req, res) => {
  await res.Inertia("Users/Index", {
    users: customScroll(userService.getCustomPaginated()),
  });
});
```

```ts
// framework: nestjs
import { Controller, Get } from "@nestjs/common";
import { Inert, type Inertia } from "@inertianode/nestjs";
import { scroll } from "@inertianode/express";

// Helper function
function customScroll(data: PaginatedCollection) {
  return scroll(data, {
    metadata: (d) => new CustomScrollMetadata(d),
  });
}

// Then use it in your routes
@Controller()
export class UsersController {
  @Get("/users")
  async index(@Inert() inertia: Inertia) {
    await inertia("Users/Index", {
      users: customScroll(userService.getCustomPaginated()),
    });
  }
}
```

```ts
// framework: koa
import Koa from "koa";
import Router from "@koa/router";
import { scroll } from "@inertianode/koa";

// Helper function
function customScroll(data: PaginatedCollection) {
  return scroll(data, {
    metadata: (d) => new CustomScrollMetadata(d),
  });
}

// Then use it in your routes
const app = new Koa();
const router = new Router();
router.get("/users", async (ctx) => {
  await ctx.Inertia("Users/Index", {
    users: customScroll(userService.getCustomPaginated()),
  });
});

app.use(router.routes());
```
