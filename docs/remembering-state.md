# Remembering state

When navigating browser history, Inertia restores pages using prop data cached in history state. However, Inertia does not restore local page component state since this is beyond its reach. This can lead to outdated pages in your browser history.

For example, if a user partially completes a form, then navigates away, and then returns back, the form will be reset and their work will be lost.

To mitigate this issue, you can tell Inertia which local component state to save in the browser history.

## Saving local state

To save local component state to the history state, use the `useRemember` feature to tell Inertia which data it should remember.

```js
// framework: vue
// (Use the "useRemember" hook to tell Inertia which data it should remember.)
import { useRemember } from "@inertiajs/vue3";
const form = useRemember({
  first_name: null,
  last_name: null,
});
```

```jsx
// framework: react
// (Use the "useRemember" hook to tell Inertia which data it should remember.)
import { useRemember } from "@inertiajs/react";
export default function Profile() {
  const [formState, setFormState] = useRemember({
    first_name: null,
    last_name: null,
    // ...
  });
  // ...
}
```

```js
// framework: svelte
// (Use the "useRemember" store to tell Inertia which data it should remember.)
import { useRemember } from "@inertiajs/svelte";
const form = useRemember({
  first_name: null,
  last_name: null,
});
// ...
```

Now, whenever your local `form` state changes, Inertia will automatically save this data to the history state and will also restore it on history navigation.

## Multiple components

If your page contains multiple components that use the remember functionality provided by Inertia, you need to provide a unique key for each component so that Inertia knows which data to restore to each component.

```js
// framework: vue
// (Set a key as the second argument of useRemember().)
import { useRemember } from "@inertiajs/vue3";
const form = useRemember(
  {
    first_name: null,
    last_name: null,
  },
  "Users/Create"
);
```

```jsx
// framework: react
// (Set a key as the second argument of useRemember().)
import { useRemember } from "@inertiajs/react";
export default function Profile() {
  const [formState, setFormState] = useRemember(
    {
      first_name: null,
      last_name: null,
    },
    "Users/Create"
  );
}
```

```js
// framework: svelte
// (Set a key as the second argument of useRemember().)
import { page, useRemember } from "@inertiajs/svelte";
const form = useRemember(
  {
    first_name: null,
    last_name: null,
  },
  "Users/Create"
);
```

If you have multiple instances of the same component on the page using the remember functionality, be sure to also include a unique key for each component instance, such as a model identifier.

```js
// framework: vue
// (Set a dynamic key as the second argument of useRemember().)
import { useRemember } from '@inertiajs/vue3'
const props = defineProps({ user: Object })
const form = useRemember({
    first_name: null,
    last_name: null,
}, \`Users/Edit:\${props.user.id}\`)
```

```jsx
// framework: react
// (Set a dynamic key as the second argument of useRemember().)
import { useRemember } from '@inertiajs/react'
export default function Profile() {
    const [formState, setFormState] = useRemember({
            first_name: props.user.first_name,
            last_name: props.user.last_name,
    }, \`Users/Edit:\${this.user.id}\`)
}
```

```js
// framework: svelte
// (Set a dynamic key as the second argument of useRemember().)
import { page, useRemember } from '@inertiajs/svelte'
const form = useRemember({
    first_name: $page.props.user.first_name,
    last_name: $page.props.user.last_name,
}, \`Users/Edit:\${$page.props.user.id}\`)
```

## Form helper

If you're using the [Inertia form helper](/forms#form-helper), you can pass a unique form key as the first argument when instantiating your form. This will cause the form data and errors to automatically be remembered.

```js
// framework: vue
import { useForm } from '@inertiajs/vue3'
const form = useForm('CreateUser', data)
const form = useForm(\`EditUser:\${props.user.id}\
```

```js
// framework: react
import { useForm } from '@inertiajs/react'
const form = useForm('CreateUser', data)
const form = useForm(\`EditUser:\${user.id}\
```

```js
// framework: svelte
import { useForm } from '@inertiajs/svelte'
const form = useForm('CreateUser', data)
const form = useForm(\`EditUser:\${user.id}\
```

## Manually saving state

The `useRemember` hook watches for data changes and automatically saves those changes to the history state. Then, Inertia will restore the data on page load.

However, it's also possible to manage this manually using the underlying `remember()` and `restore()` methods exposed by Inertia.

```js
// framework: vue
import { router } from "@inertiajs/vue3";
// Save local component state to history state
router.remember(data, "my-key");
// Restore local component state from history state
let data = router.restore("my-key");
```

```js
// framework: react
import { router } from "@inertiajs/react";
// Save local component state to history state
router.remember(data, "my-key");
// Restore local component state from history state
let data = router.restore("my-key");
```

```js
// framework: svelte
import { router } from "@inertiajs/svelte";
// Save local component state to history state
router.remember(data, "my-key");
// Restore local component state from history state
let data = router.restore("my-key");
```
