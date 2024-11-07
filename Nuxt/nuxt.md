![nuxt](nuxt.png)

# Table of Contents

1. [Get Started With Nuxt](#1-get-started-with-nuxt)
2. [Directory Structure](#2-directory-structure)
3. [Assets](#3-assets)
4. [Components](#4-components)
5. [Composables](#5-composables)
6. [Layouts](#6-layouts)
7. [Middleware](#7-middleware)
8. [Pages](#8-pages)
9. [Data Fetching](#9-data-fetching)
10. [Utils](#10-utils)
11. [Helpful Resources](#11-helpful-resources)
12. [Additional Notes](#12-additional-notes)

# 1. Get Started With Nuxt

Nuxt is a free and open-source framework to create type-safe, performant and production-grade full-stack web applications and websites with Vue.js.

The core features of Nuxt are:

- **File-based routing:** define routes based on the structure of your `pages/directory`. This can make it easier to organize your application and avoid the need for manual route configuration.

- **Code splitting:** Nuxt automatically splits your code into smaller chunks, which can help reduce the initial load time of your application.

- **Server-side rendering out of the box:** Nuxt comes with built-in SSR capabilities, so you don't have to set up a separate server yourself.

- **Auto-imports:** write Vue composables and components in their respective directories and use them without having to import them with the benefits of tree-shaking and optimized JS bundles.

- **Data-fetching utilities:** Nuxt provides composables to handle SSR-compatible data fetching as well as different strategies.

- **Zero-config TypeScript support:** write type-safe code without having to learn TypeScript with our auto-generated types and `tsconfig.json`.

- **Configured build tools:** we use **Vite** by default to support hot module replacement (HMR) in development and bundling your code for production with best-practices baked-in.

## Installation

Use the following command to create a new Nuxt project:

```shell
# create the project
npx nuxi@latest init <project-name>

#start the project
npm run dev
```

# 2. Directory Structure

```
📦 Your Nuxt Project
├── 📁 assets
│   └── Contains uncompiled assets such as SCSS, images, or fonts.
│
├── 📁 components
│   └── Stores reusable Vue components, which can be used across pages.
│
├── 📁 composables
│   └── Contains composable functions using Vue 3’s Composition API.
│
├── 📁 layouts
│   └── Defines the structure of pages and layout templates (header, footer).
│
├── 📁 middleware
│   └── Functions to run before navigating to a page or route, used for authentication, redirects, etc.
│
├── 📁 pages
│   └── Holds all the pages in your application, with automatic routing based on folder/file structure.
│
├── 📁 plugins
│   └── Allows you to register external plugins and libraries globally (e.g., Vue plugins, Axios).
│
├── 📁 public
│   └── Static files that won't change (like images, favicons). These are served directly without processing.
│
├── 📁 node_modules
│   └── Dependencies installed by npm/yarn.
|
├── 📄 app.vue
│   └── The entrypoint of the application, Nuxt will render its content for every route of the application.
│
├── 📄 nuxt.config.js
│   └── The main configuration file for Nuxt, where you set global settings, modules, and build options.
│
├── 📄 package.json
│   └── Stores project metadata and dependencies (npm or yarn).
│
```

The number and type of folders/files may vary.

We'll dive deeper into each one as we move forward.

# 3. Assets

Nuxt uses two directories to handle assets like stylesheets, fonts or images.

- The `public/` directory content is served at the server root as-is.
- The `assets/` directory contains by convention every asset that you want the build tool (Vite or webpack) to process.

## a. public:

The `public/` directory is used as a public server for static assets publicly available at a defined URL of your application.

You can get a file in the `public/` directory from your application's code or from a browser by the root URL `/`.

For example, referencing an image file in the `public/img/` directory, available at the static URL `/img/nuxt.png`:

```html
<template>
  <img src="/img/nuxt.png"/>
</template>
```

## b. assets:

By convention, Nuxt uses the `assets/` directory to store these files but there is no auto-scan functionality for this directory, and you can use any other name for it.

In your application's code, you can reference a file located in the `assets/` directory by using the `~/assets/` path.

For example, referencing an image file that will be processed if a build tool is configured to handle this file extension:

```html
<template>
  <img src="~/assets/img/nuxt.png" />
</template>
```

# 4. Components

Nuxt automatically imports any components in this directory (along with components that are registered by any modules you may be using).

If you have a component in nested directories, the component's name will be based on its own path directory and filename, with duplicate segments being removed.

```js
// Directory Structure
| components/
--| base/
----| foo/
------| Button.vue
```

```html
<!-- the component's name will be: -->
<BaseFooButton />
```

If you want to auto-import components based only on its name, not path, then you need to set `pathPrefix` option to `false` using extended form of the configuration object:

```js
// nuxt.config.ts
export default defineNuxtConfig({
  components: [
    {
      path: '~/components',
      pathPrefix: false, // [!code ++]
    },
  ],
});
```

Now, `~/components/Some/MyComponent.vue` will be usable as `<MyComponent>` and not `<SomeMyComponent>`.

You can also explicitly import components from `#components` if you want or need to bypass Nuxt's auto-importing functionality.

```js
import { MyComp } from '#components'
```

If a component is meant to be rendered only client-side, you can add the `.client` suffix to your component.

```js
| components/
--| Comments.client.vue
```

# 5. Composables

`Composables` directory is used to auto-import your Vue composables into your application.

Using named export:

```js
export const useFoo = () => {
  return useState('foo', () => 'bar')
}
```

Using default export:

```js
export default function () {
  return useState('foo', () => 'bar')
}
```

You can now use the auto imported composable in `.js`, `.ts` and `.vue` files:

```html
<script setup lang="ts">
const foo = useFoo()
</script>

<template>
  <div>
    {{ foo }}
  </div>
</template>
```

# 6. Layouts

Layouts are enabled by adding `<NuxtLayout>`  to your `app.vue`.

```html
<!-- app.vue -->
<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>
```

To use a layout:

- Set a `layout` property in your page with `definePageMeta`.
- Set the `name` prop of `<NuxtLayout>`.

```js
// Directory Structure
-| layouts/
---| default.vue
---| custom.vue
```

```html
<script setup lang="ts">
definePageMeta({
  layout: 'custom'
})
</script>
```

The layout name is normalized to kebab-case, so `someLayout` becomes `some-layout`.

If no layout is specified, `layouts/default.vue` will be used.

If you only have a single layout in your application, we recommend using `app.vue` instead.

Unlike other components, your layouts must have a single root element to allow Nuxt to apply transitions between layout changes and this root element cannot be `<slot/>`

In a layout file, the content of the page will be displayed in the `<slot />` component.

```html
<template>
  <div>
    <p>Some default layout content shared across all pages</p>
    <slot />
  </div>
</template>
```

# 7. Middlewares

Middlewares in Nuxt.js are functions that runs before navigating to a specific route. It's useful for performing checks, redirections, or any other logic you want to execute before a page loads.

Types of Middleware:

1. Anonymous (inline) middleware: Defined directly in the page component.
2. Named middleware: Placed in the `middleware/` directory and imported when needed.
3. Global middleware: Placed in the `middleware/` directory with a `.global` suffix, runs on every route change.

Let's create an example using named middleware to check if a user is over 18:

```js
// middlewares/checkAge.ts
export default defineNuxtRouteMiddleware((to, from) => {

  const userAge = 16; // This is just for demonstration

  if (userAge < 18) {
    // If user is under 18, redirect to a different page
    return navigateTo('/restricted')
  }
  // If user is 18 or older, allow navigation to continue
})
```

Use the middleware in a page:

```html
<script setup>
definePageMeta({
  middleware: ['checkAge']
})
</script>

<template>
  <div>
    <h1>Adult Content</h1>
    <p>This page is only accessible to users 18 and older.</p>
  </div>
</template>
```

Middleware runs in the following order:

1. Global Middleware
2. Page defined middleware order (if there are multiple middleware declared with the array syntax)

For example, assuming you have the following middleware and component:

```js
middleware/
--| analytics.global.ts
--| setup.global.ts
--| auth.ts
```

```html
<script setup lang="ts">
definePageMeta({
      // Custom inline middleware
  middleware: [ function (to, from) {}, 'auth', ],
});
</script>
```

You can expect the middleware to be run in the following order:

1. `analytics.global.ts`
2. `setup.global.ts`
3. Custom inline middleware
4. `auth.ts`

Global middleware is executed alphabetically based on the filename.

However, there may be times you want to define a specific order. For example, in the last scenario, `setup.global.ts` may need to run before `analytics.global.ts`. In that case, we recommend prefixing global middleware with 'alphabetical' numbering.

```js
// Directory structure
middleware/
--| 01.setup.global.ts
--| 02.analytics.global.ts
--| auth.ts
```

# 8.Pages (routing)

One core feature of Nuxt is the file system router. Every Vue file inside the `pages/` directory creates a corresponding URL (or route) that displays the contents of the file.

The `pages/index.vue` file will be mapped to the `/` route of your application.

This file system routing uses naming conventions to create dynamic and nested routes:

```js
// Directory structure
| pages/
---| about.vue
---| index.vue
---| posts/
-----| [id].vue
```

```js
// generated routes
{  
      "path": "/about",
      "component": "pages/about.vue",

      "path": "/",
      "component": "pages/index.vue",

      "path": "/posts/:id",
      "component": "pages/posts/[id].vue",
}
```

If you are using `app.vue`, make sure to use the `<NuxtPage/>` component to display the current page:

```html
<template>
  <div>
    <!--Here add any Markup shared across all pages, ex: NavBar -->
    <NuxtPage />
  </div>
</template>
```

Pages **must have a single root element** to allow route transitions between pages. HTML comments are considered elements as well. 

## a. Dynamic Routes:

If you place anything within square brackets, it will be turned into a dynamic route parameter. You can mix and match multiple parameters and even non-dynamic text within a file name or directory.

If you want a parameter to be *optional*, you must enclose it in double square brackets - for example, `~/pages/[[slug]]/index.vue` or `~/pages/[[slug]].vue` will match both `/` and `/test`.

```js
// Directory Structure
-| pages/
---| index.vue
---| users-[group]/
-----| [id].vue
```

Given the example above, you can access group/id within your component via the `$route` object:

```html
<!-- pages/users-[group]/[id].vue -->
<template>
  <p>{{ $route.params.group }} - {{ $route.params.id }}</p>
</template>
```

Navigating to `/users-admins/123` would render:

```html
<p>admins - 123</p>
```

If you want to access the route using Composition API, there is a global `useRoute` function that will allow you to access the route.

```html
<script setup lang="ts">
const route = useRoute()

if (route.params.group === 'admins' && !route.params.id) {
  console.log('Warning! Make sure user is authenticated!')
}
</script>
```

In some cases, you may want to group a set of routes together in a way which doesn't affect file-based routing. For this purpose, you can put files in a folder which is wrapped in parentheses - `(` and `)`.

```js
// Directory Structure
-| pages/
---| index.vue
---| (marketing)/
-----| about.vue
-----| contact.vue
```

This will produce `/`, `/about` and `/contact` pages in your app. The `marketing` group is ignored for purposes of your URL structure.

## b. Navigation:

The [`<NuxtLink>`](https://nuxt.com/docs/api/components/nuxt-link) component links pages between them. It renders an `<a>` tag with the `href` attribute set to the route of the page. Once the application is hydrated, page transitions are performed in JavaScript by updating the browser URL. This prevents full-page refreshes and allows for animated transitions.

```html
<template>
  <header>
    <nav>
      <ul>
        <li><NuxtLink to="/about">About</NuxtLink></li>
        <li><NuxtLink to="/posts/1">Post 1</NuxtLink></li>
        <li><NuxtLink to="/posts/2">Post 2</NuxtLink></li>
      </ul>
    </nav>
  </header>
</template>
```

Nuxt allows programmatic navigation through the `navigateTo()` utility method. Using this utility method, you will be able to programmatically navigate the user in your app. 

```html
<script setup lang="ts">
const name = ref('');
const type = ref(1);

function navigate(){
  return navigateTo({
    path: '/search',
    query: {
      name: name.value,
      type: type.value
    }
  })
}
</script>
```

## c. app.vue

With Nuxt 3, the `pages/` directory is optional. If not present, Nuxt won't include `vue-router` dependency. This is useful when working on a landing page or an application that does not need routing.

## d. error.vue

During the lifespan of your application, some errors may appear unexpectedly at runtime. In such case, we can use the `error.vue` file to override the default error files and display the error nicely.

```html
<script setup lang="ts">
import type { NuxtError } from '#app'

const props = defineProps({
  error: Object as () => NuxtError
})
</script>

<template>
  <div>
    <h1>{{ error.statusCode }}</h1>
    <NuxtLink to="/">Go back home</NuxtLink>
  </div>
</template>
```

The error page has a single prop - `error` which contains an error for you to handle.

The `error` object provides the following fields:

```js
{
  statusCode: number
  fatal: boolean
  unhandled: boolean
  statusMessage: string
  data: unknown
  cause: unknown
}
```

Although it is called an 'error page' it's not a route and shouldn't be placed in your `~/pages` directory. For the same reason, you shouldn't use `definePageMeta` within this page. That being said, you can still use layouts in the error file, by utilizing the `NuxtLayout` component and specifying the name of the layout.

# 9. Data Fetching

Nuxt comes with two composables and a built-in library to perform data-fetching in browser or server environments: `useFetch`, `useAsyncData` and `$fetch`.

In a nutshell:

- `useFetch` is the most straightforward way to handle data fetching in a component setup function.
- `$fetch` is great to make network requests based on user interaction.
- `useAsyncData` combined with `$fetch`, offers more fine-grained control.

## a. useFetch:

The `useFetch` composable is the most straightforward way to perform data fetching:

```html
<script setup lang="ts">
const { data: count } = await useFetch('/api/count')
</script>

<template>
  <p>Page visits: {{ count }}</p>
</template>
```

## b. $fetch:

Nuxt includes the [ofetch](https://github.com/unjs/ofetch) library, and is auto-imported as the `$fetch` alias globally across your application. It's what `useFetch` uses behind the scenes.

```html
<script setup lang="ts">
async function addTodo() {
  const todo = await $fetch('/api/todos', {
    method: 'POST',
    body: {
      // My todo data
    }
  })
}
</script>
```

## c. useAsyncData

The `useAsyncData` composable is responsible for wrapping async logic and returning the result once it is resolved.

```html
<script setup lang="ts">
const { data, error } = await useAsyncData('users', () => myGetFunction('users'))

// This is also possible:
const { data, error } = await useAsyncData(() => myGetFunction('users'))
</script>
```

# 10. Utils

## a. definePageMeta

`definePageMeta` is a function provided by Nuxt that you can use to set metadata for your `page` components located in the `pages/` directory. This way you can set custom metadata for each static or dynamic route of your Nuxt application.

It is used also to set a specific layout and middleware for a specific page:

```js
definePageMeta({
  title: 'My Page Title',
  layout: 'custom',
  middleware: ['auth']
})
```

Middlewares can be defined using a `function` directly within the `definePageMeta` or set as a `string` that matches the middleware file name located in the `middleware/` directory:

```html
<script setup lang="ts">
definePageMeta({
  // define middleware as a function
  middleware: [
    function (to, from) {
      const auth = useState('auth')

      if (!auth.value.authenticated) {
          return navigateTo('/login')
      }

      if (to.path !== '/checkout') {
        return navigateTo('/checkout')
      }
    }
  ],

  // ... or a string
  middleware: 'auth'

  // ... or multiple strings
  middleware: ['auth', 'another-named-middleware']
})
</script>
```

You can define the layout that matches the layout's file name located (by default) in the `layouts/` directory. You can also disable the layout by setting the `layout` to `false`:

```html
<script setup lang="ts">
definePageMeta({
  // set custom layout
  layout: 'admin'

  // ... or disable a default layout
  layout: false
})
</script>
```

## b. useState

`useState` is an SSR-friendly `ref` replacement. Its value will be preserved after server-side rendering (during client-side hydration) and shared across all components using a unique key.

```js
// Create a reactive state and set default value
const Counter = useState('counter', () => 0);
```

The main differences between `ref()` and `useState()` in Nuxt 3 are:

1. Origin and Scope:
   - `ref()` is from Vue's Composition API
   - `useState()` is a Nuxt 3-specific composable
2. Persistence:
   - `ref()` creates reactive references that don't persist between page navigations
   - `useState()` creates reactive state that persists across page navigations
3. Server-Side Rendering (SSR) Behavior:
   - `ref()` values are not shared between server and client
   - `useState()` hydrates state from server to client
4. Usage Context:
   - `ref()` can be used anywhere in Vue components
   - `useState()` is typically used in Nuxt pages, layouts, or plugins

# 11. Helpful Resources

## a. Nuxt Modules

For extending your Nuxt application, check out the [Nuxt Modules](https://nuxt.com/modules). It offers a wide variety of modules for common needs, including integrations with Firebase, Supabase, Pinia, and many more. These modules can significantly speed up development and add powerful features to your project.

## b. VueUse

Another invaluable resource is [VueUse](https://vueuse.org/), an open-source collection of Vue composition utilities. VueUse provides a wealth of composables for common tasks like handling drag-and-drop, detecting clicks outside elements, managing localStorage, and much more. These composables are crafted by experienced developers and benefit from community contributions, often resulting in more robust and efficient solutions than individual implementations.

# 12. Additional Notes

## a. Passing Assets Between Components

When using Nuxt 3 with Vite, images in the `assets` folder are processed during bundling. To pass these images as props between components, you need to **import** the image in the parent component and pass the resolved image path to the child component.

**Steps**:

1. **Import the image** from the `assets` folder in the parent component.
2. **Pass the image** as a prop to the child component.
3. In the child component, use the **resolved image path** from the prop.

```html
<!-- MyComponent.vue -->
<script setup>
// Define props to accept the image path
const props = defineProps({
  image: String,
});
</script>

<template>
  <!-- Use the image prop in the img src attribute -->
  <div>
    <img :src="image" class="w-full h-full object-cover" />
  </div>
</template>
```

```html
<!-- Parent Component-->
<script setup>
// Import image from the assets folder
import myImage from '~/assets/myimage.png';
</script>

<template>
  <!-- Pass the resolved image path as a prop -->
  <MyComponent :image="myImage" />
</template>
```
