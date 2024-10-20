![](inertia.png)

# 1. What is Inertia.js

When you want to take advantage of a backend framework like Laravel—using its ORM, routing, and other built-in features—while also building with modern frontend frameworks like Vue or React, the typical solution is to create a REST or GraphQL API to connect them.

However, this setup introduces a lot of complexity. You need to manage authentication, handle CORS, maintain two repositories, and deal with more complicated deployments. It can quickly become overwhelming just to get everything working together smoothly.

This is where Inertia comes in. Inertia works just like a traditional server-side rendered application: you create controllers, fetch data from the database through your ORM, and render views. But instead of Blade templates, these views are JavaScript page components written in Vue, React, or Svelte.

With Inertia, you get the simplicity of server-side development combined with the power of a modern SPA—without the need to build an API. It streamlines your workflow, letting you focus on building your application, not managing unnecessary complexity.

Inertia does this via **adapters**. currently it have three official client-side adapters (`React`, `Vue`, and `Svelte`) and three server-side adapters (`Laravel`, `Rails`, and `Phoenix`), but it's fine-tuned for `Laravel`.

# 2. Installation

## a. Starter kits:

Laravel's starter kits, `Breeze` and `Jetstream`, provide out-of-the-box scaffolding for new Inertia applications. These starter kits are the absolute fastest way to start building a new Inertia project using Laravel and Vue or React. However, if you would like to install Inertia manually into your application, it has two parts: the server side and the client side.

## b. Server-side Setup:

- First, install laravel **adapter**

```shell
composer require inertiajs/inertia-laravel
```

- Setup the root template **(app.blade.php)**

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
    @vite('resources/js/app.js')
    @inertiaHead
  </head>
  <body>
    @inertia
  </body>
</html>
```

-  Setup the Inertia middleware

```shell
php artisan inertia:middleware
```

then append the `HandleInertiaRequests` middleware to the `web` middleware group in your application's `bootstrap/app.php` file.

```php
use App\Http\Middleware\HandleInertiaRequests;

->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        HandleInertiaRequests::class,
    ]);
})
```

## c. Client-side Setup:

- Install your front-end framework **adapter**

```shell
npm install @inertiajs/vue3
```

- Add the vue plugin

```shell
npm i @vitejs/plugin-vue
```

and add it to your **vite.config.js** file:

```js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue'; // --> ADD THIS LINE
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        vue(),  // --> ADD THIS LINE
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
```

- Setup the main JavaScript file to boot your Inertia app **resources/js/app.js**

```js
import { createApp, h } from 'vue'
import { createInertiaApp } from '@inertiajs/vue3'

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.vue', { eager: true })
    return pages[`./Pages/${name}.vue`]
  },
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
})
```

# 3. Pages

When using Inertia, each page in your application has a corresponding controller and route, just like in a traditional server-rendered app. However, instead of Blade templates, these pages are JavaScript components written in Vue, React, or Svelte. This allows you to build modern, dynamic frontends without needing an API.

## a. Rendering Pages from Controllers:

Rendering an Inertia page works similarly to Blade views. In your controller, instead of returning a Blade view, you return an Inertia response using either the `Inertia::render()` method or the `inertia()` helper. The syntax is almost identical to the `view()` helper you’re used to.

```php
class UserController extends Controller
{
    public function show(User $user)
    {
        return inertia('User/Show', ['user' => $user]);
    }
}
```

This makes the transition to Inertia smooth, as you continue to use familiar syntax for rendering views. For example, `User/Show` refers to the component stored in `resources/js/Pages/User/Show.vue`.

## b. Creating Pages:

Inertia pages are regular JavaScript components. They are stored in `resources/js/Pages`, mirroring how Blade views were stored in `resources/views`. If you are familiar with building Vue, React, or Svelte components, you’ll find the process straightforward—pages simply receive data from your controllers as props.

Here’s an example of an Inertia page component in Vue 3:

```html
<script setup>
defineProps({ user: Object })

</script>

<template>
    <p>Hello {{ user.name }}, welcome to your first Inertia app!</p>
</template> 
```

## c. Redirects with Inertia:

One of the biggest advantages of using Inertia is that it eliminates the need to build APIs for every action. With Inertia, you don't have to return JSON responses for non-GET requests; you can simply use regular redirects, just like you would in a Blade-based application.

Here’s an example of a typical store function:

```php
class UsersController extends Controller
{
    public function store(Request $request)
    {
        User::create($request->validate([
            'name' => ['required', 'max:50'],
            'email' => ['required', 'email', 'max:50'],
        ]));

        return to_route('users.index');
    }
}
```

After storing a new user, the `store()` method redirects to the index page. Inertia automatically follows the redirect and updates the page without needing a full page reload. This feels just like a classic server-side app, but with the interactivity of a modern SPA.

For non-GET requests like `PUT`, `PATCH`, or `DELETE`, Inertia uses 303 redirects to ensure the next request is treated as a `GET`. This ensures smooth navigation between pages without unexpected behavior.

## d. Security and Data Handling:

It’s recommended to return only the necessary fields from your models to avoid exposing sensitive data to the frontend. Since all props passed to Inertia components become available on the client-side, it's essential to minimize the data you send. For example, instead of returning all user fields, you can select only what’s needed:

```php
public function index()
{
    return Inertia::render('Users/Index', [
        'users' => User::select('id', 'name', 'email')->get(),
    ]);
}
```

By limiting the data to only the required fields, you reduce the risk of unintentionally leaking sensitive information, such as passwords, API keys, or personal details. This practice not only enhances security but also improves performance by sending lighter responses to the frontend.

# 4. Layouts
