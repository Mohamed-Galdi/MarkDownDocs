![Vue Header](Vue.js.png)

# Table of Contents

1. [What is Vue ?](#1-what-is-vue)
2. [API Styles](#2-API-Styles)
3. [Creating a Vue Application](#3-Creating-a-Vue-Application)
4. [The application instance](#4-The-application-instance)
5. [Directives](#5-Directives)
6. [Template Syntax](#6-Template-Syntax)
7. [Attribute Bindings](#7-Attribute-Bindings)
8. [Event Handling](#8-Event-Handling)
9. [Reactive State](#9-Reactive-State)
10. [Computed Properties](#10-Computed-Properties)
11. [Component Classes](#11-Component-Classes)
12. [Conditional Rendering](#12-Conditional-Rendering)
13. [List Rendering (v-for)](#13-list-rendering-v-for)
14. [Form Inputs (v-model)](#14-form-inputs-v-model)
15. [Lifecycle Hooks](#15-Lifecycle-Hooks)
16. [Watchers](#16-Watchers)
17. [useTemplateRef](#17-useTemplateRef)
18. [Components](#18-Components)
19. [Provide / Inject](#19-provide--inject)
20. [Composables](#20-Composables)
21. [Built-in Components](#21-Built-in-Components)

# 1. What is Vue?

Vue is a framework for building user interfaces. The two core features of Vue are:

- **Declarative Rendering**: Vue extends standard HTML with a template syntax that allows us to declaratively describe HTML output based on JavaScript state.

- **Reactivity**: Vue automatically tracks JavaScript state changes and efficiently updates the DOM when changes happen.

We author Vue components using an HTML-like file format called **Single-File Component** . A Vue SFC, as the name suggests, encapsulates the component's logic (JavaScript), template (HTML), and styles (CSS) in a single file.

# 2. API Styles:

Vue components can be authored in two different API styles: **Options API** and **Composition API**.

## a. Options API:

With Options API, we define a component's logic using an object of options such as `data`, `methods`, and `mounted`. Properties defined by options are exposed on `this` inside functions, which points to the component instance:

```html
<script>
  export default {
      data() {
          return { count: 0 }
      },
      methods: {
          increment() { this.count++ }
      }
      mounted() { console.log(`The initial count is ${this.count}.`) }
  }
</script>
<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

## b. Composition API:

With Composition API, we define a component's logic using imported API functions. In SFCs, Composition API is typically used with [`<script setup>`](https://vuejs.org/api/sfc-script-setup.html). The `setup` attribute is a hint that makes Vue perform compile-time transforms that allow us to use Composition API with less boilerplate. For example, imports and top-level variables / functions declared in `<script setup>` are directly usable in the template.

Here is the same component, with the exact same template, but using Composition API and `<script setup>` instead:

```html
<script setup>
  import { ref, onMounted } from "vue";

  const count = ref(0);

  function increment() {
    count.value++;
  }

  onMounted(() => {
    console.log(`The initial count is ${count.value}.`);
  });
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

# 3. Creating a Vue Application:

```shell
npm create vue@latest
```

This command will install and execute [create-vue](https://github.com/vuejs/create-vue), the official Vue project scaffolding tool. You will be presented with prompts for several optional features such as TypeScript and testing support:

```shell
✔ Project name: … <your-project-name>
✔ Add TypeScript? … No / Yes
✔ Add JSX Support? … No / Yes
✔ Add Vue Router for Single Page Application development? … No / Yes
✔ Add Pinia for state management? … No / Yes
✔ Add Vitest for Unit testing? … No / Yes
✔ Add an End-to-End Testing Solution? … No / Cypress / Nightwatch / Playwright
✔ Add ESLint for code quality? … No / Yes
✔ Add Prettier for code formatting? … No / Yes
✔ Add Vue DevTools 7 extension for debugging? (experimental) … No / Yes
```

The created project will be using a build setup based on [Vite](https://vitejs.dev/) and allow us to use Vue [Single-File Components](https://vuejs.org/guide/scaling-up/sfc.html) (SFCs).

# 4. The application instance:

Every Vue application starts by creating a new **application instance** with the [`createApp`](https://vuejs.org/api/application.html#createapp) function:

```js
import { createApp } from "vue";
import App from "./App.vue";

createApp(App).mount("#app");
```

An application instance won't render anything until its `.mount()` method is called. It expects a "container" argument, which can either be an actual DOM element or a selector:

```html
<div id="app"></div>
```

You are not limited to a single application instance on the same page. The `createApp` API allows multiple Vue applications to co-exist on the same page:

```js
const app1 = createApp({
  /* ... */
});
app1.mount("#container-1");

const app2 = createApp({
  /* ... */
});
app2.mount("#container-2");
```

# 5. Directives:

Directives are special attributes with the `v-` prefix. A directive's job is to reactively apply updates to the DOM when the value of its expression changes.

Some directives can take an "argument", denoted by a colon after the directive name.

Example of directives: `v-bind:` , `v-if` , `v-on`.

# 6. Template Syntax:

## a. Text Interpolation

The most basic form of data binding is text interpolation using the "Mustache" syntax (double curly braces):

```html
<p>Message: {{ msg }}</p>
```

The mustache tag will be replaced with the value of the `msg` property [from the corresponding component instance](https://vuejs.org/guide/essentials/reactivity-fundamentals.html#declaring-reactive-state). It will also be updated whenever the `msg` property changes.

## b. Raw HTML

The double mustaches interpret the data as plain text, not HTML. In order to output real HTML, you will need to use the [`v-html` directive](https://vuejs.org/api/built-in-directives.html#v-html):

```html
<p v-html="rawHtml"></p>
```

# 7. Attribute Bindings:

Mustaches cannot be used inside HTML attributes. Instead, use a [`v-bind` directive](https://vuejs.org/api/built-in-directives.html#v-bind):

```html
<div v-bind:id="dynamicId"></div>
```

The `v-bind` directive instructs Vue to keep the element's `id` attribute in sync with the component's `dynamicId` property. If the bound value is `null` or `undefined`, then the attribute will be removed from the rendered element.

- **Shorthand** :

Because `v-bind` is so commonly used, it has a dedicated shorthand syntax:

```html
<div :id="dynamicId"></div>
```

- **Same-name Shorthand** : _(only available in Vue 3.4 and above.)_

If the attribute has the same name with the JavaScript value being bound, the syntax can be further shortened to omit the attribute value:

```html
<!-- same as :id="id" -->
<div :id></div>
```

# 8. Event Handling:

We can use the `v-on` directive, which we typically shorten to the `@` symbol, to listen to DOM events and run some JavaScript when they're triggered. The usage would be `v-on:click="handler"` or with the shortcut, `@click="handler"`.

```js
const count = ref(0);
```

```html
<button @click="count++">Add 1</button>
<p>Count is: {{ count }}</p>
```

## a. the $event object

Sometimes we need to access the event in an event handler. You can pass it into a method using the special `$event` variable, or use an inline arrow function:

```js
function warn(message, event) {
  if (event) {
    event.preventDefault();
  }
  alert(message);
}
```

```html
<!-- using $event special variable -->
<button @click="warn('Form cannot be submitted yet.', $event)">Submit</button>

<!-- using inline arrow function -->
<button @click="(event) => warn('Form cannot be submitted yet.', event)">
  Submit
</button>
```

## b. Event Modifiers

It is a very common need to call `event.preventDefault()` or `event.stopPropagation()` inside event handlers.

Vue provides **event modifiers** for `v-on`. Recall that modifiers are directive postfixes denoted by a dot.

```html
<!-- the click event's propagation will be stopped -->
<a @click.stop="doThis"></a>

<!-- the submit event will no longer reload the page -->
<form @submit.prevent="onSubmit"></form>

<!-- modifiers can be chained -->
<a @click.stop.prevent="doThat"></a>

<!-- just the modifier -->
<form @submit.prevent></form>
```

## c. Key Modifiers

When listening for keyboard events, we often need to check for specific keys. Vue allows adding key modifiers for `v-on` or `@` when listening for key events:

```html
<!-- only call `submit` when the `key` is `Enter` -->
<input @keyup.enter="submit" />
```

You can directly use any valid key names exposed via [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) as modifiers by converting them to kebab-case.

```html
<input @keyup.page-down="onPageDown" />
```

In the above example, the handler will only be called if `$event.key` is equal to `'PageDown'`.

# 9. Reactive State:

## a. ref()

In Composition API, the recommended way to declare reactive state is using the `ref()` function.

It takes the argument and returns it wrapped within a ref object with a `.value` property:

```js
import { ref } from "vue";

const count = ref(0);

console.log(count); // { value: 0 }
console.log(count.value); // 0

count.value++;
console.log(count.value); // 1
```

For convenience, refs are automatically unwrapped when used inside templates, we can also mutate a ref directly in event handlers:

```html
<script setup>
  import { ref } from "vue";

  const count = ref(0);

  function increment() {
    count.value++;
  }
</script>

<template>
  <button @click="increment">{{ count }}</button>
</template>
```

Refs can hold any value type, including deeply nested objects, arrays, or JavaScript built-in data structures like `Map`.

A ref will make its value deeply reactive. This means you can expect changes to be detected even when you mutate nested objects or arrays:

```js
import { ref } from "vue";

const obj = ref({
  nested: { count: 0 },
  arr: ["foo", "bar"],
});

function mutateDeeply() {
  // these will work as expected.
  obj.value.nested.count++;
  obj.value.arr.push("baz");
}
```

## b. reactive()

There is another way to declare reactive state, with the `reactive()` API. Unlike a ref which wraps the inner value in a special object, `reactive()` makes an object itself reactive:

```html
<script setup>
  import { reactive } from "vue";

  const state = reactive({ count: 0 });
</script>

<template>
  <button @click="state.count++">{{ state.count }}</button>
</template>
```

Due to `recative()` limitations, we recommend using `ref()` as the primary API for declaring reactive state.

# 10. Computed Properties:

Putting too much logic in your templates can make them complex and hard to maintain. For example, if we have an object with a nested array:

```js
const author = reactive({
  name: "John Doe",
  books: [
    "Vue 2 - Advanced Guide",
    "Vue 3 - Basic Guide",
    "Vue 4 - The Mystery",
  ],
});
```

And we want to display different messages depending on if `author` already has some books or not:

```html
<p>Has published books:</p>
<span>{{ author.books.length > 0 ? 'Yes' : 'No' }}</span>
```

At this point, the template is getting a bit cluttered. also, we probably don't want to repeat ourselves if we need to include this calculation in the template more than once.

That's why for complex logic that includes reactive data, it is recommended to use a **computed property**. Here's the same example, refactored:

```html
<script setup>
  import { reactive, computed } from "vue";

  const author = reactive({
    name: "John Doe",
    books: [
      "Vue 2 - Advanced Guide",
      "Vue 3 - Basic Guide",
      "Vue 4 - The Mystery",
    ],
  });

  // a computed ref
  const publishedBooksMessage = computed(() => {
    return author.books.length > 0 ? "Yes" : "No";
  });
</script>

<template>
  <p>Has published books:</p>
  <span>{{ publishedBooksMessage }}</span>
</template>
```

The `computed()` function expects to be passed a function, and the returned value is a **computed ref**. Similar to normal refs, you can access the computed result as `publishedBooksMessage.value`. Computed refs are also auto-unwrapped in templates so you can reference them without `.value` in template expressions.

A computed property automatically tracks its reactive dependencies. Vue is aware that the computation of `publishedBooksMessage` depends on `author.books`, so it will update any bindings that depend on `publishedBooksMessage` when `author.books` changes.

You may have noticed we can achieve the same result using a method like:

```js
function calculateBooksMessage() {
  return author.books.length > 0 ? "Yes" : "No";
}
```

The difference is that **computed properties are cached based on their reactive dependencies.** A computed property will only re-evaluate when some of its reactive dependencies have changed. This means as long as `author.books` has not changed, multiple access to `publishedBooksMessage` will immediately return the previously computed result without having to run the getter function again.

# 11. Component Classes:

When you use the `class` attribute on a component with a single root element, those classes will be added to the component's root element and merged with any existing class already on it.

```html
<!-- child component template -->
<p class="foo bar">Hi!</p>
```

```html
<!-- when using the component -->
<MyComponent class="baz boo" />
```

```html
<!-- The rendered HTML -->
<p class="foo bar baz boo">Hi!</p>
```

If your component has multiple root elements, you would need to define which element will receive this class. You can do this using the `$attrs` component property:

```html
<!-- MyComponent template using $attrs -->
<p :class="$attrs.class">Hi!</p>
<span>This is a child component</span>
```

```html
<MyComponent class="baz" />
```

Will render:

```html
<p class="baz">Hi!</p>
<span>This is a child component</span>
```

# 12. Conditional Rendering:

## a. v-if

The directive `v-if` is used to conditionally render a block. The block will only be rendered if the directive's expression returns a truthy value.

You can use the `v-else` directive to indicate an "else block" for `v-if`.

The `v-else-if`, as the name suggests, serves as an "else if block" for `v-if`. It can also be chained multiple times.

A `v-else` element must immediately follow a `v-if` or a `v-else-if` element - otherwise it will not be recognized.

Similar to `v-else`, a `v-else-if` element must immediately follow a `v-if` or a `v-else-if` element.

```html
<button @click="awesome = !awesome">Toggle</button>

<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>
```

Because `v-if` is a directive, it has to be attached to a single element. But what if we want to toggle more than one element? In this case we can use `v-if` on a `<template>` element, which serves as an invisible wrapper. The final rendered result will not include the `<template>` element.

```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

## b. v-show

Another option for conditionally displaying an element is the `v-show` directive. The usage is largely the same:

```html
<h1 v-show="ok">Hello!</h1>
```

The difference is that an element with `v-show` will always be rendered and remain in the DOM; `v-show` only toggles the `display` CSS property of the element.

`v-show` doesn't support the `<template>` element, nor does it work with `v-else`.

Generally speaking, `v-if` has higher toggle costs while `v-show` has higher initial render costs. So prefer `v-show` if you need to toggle something very often, and prefer `v-if` if the condition is unlikely to change at runtime.

# 13. List Rendering (v-for):

We can use the `v-for` directive to render a list of items based on an array. The `v-for` directive requires a special syntax in the form of `item in items`, where `items` is the source data array and `item` is an **alias** for the array element being iterated on:

```js
const items = ref([{ message: "Foo" }, { message: "Bar" }]);
```

```html
<li v-for="item in items">{{ item.message }}</li>
```

You can also use `of` as the delimiter instead of `in`, so that it is closer to JavaScript's syntax for iterators:

```html
<li v-for="item of items">{{ item.message }</li>
```

For nested `v-for`, scoping also works similar to nested functions. Each `v-for` scope has access to parent scopes:

```html
<li v-for="item in items">
  <span v-for="childItem in item.children">
    {{ item.message }} {{ childItem }}
  </span>
</li>
```

You can also use `v-for` to iterate through the properties of an object. The iteration order will be based on the result of calling `Object.values()` on the object:

```js
const myObject = reactive({
  title: "How to do lists in Vue",
  author: "Jane Doe",
  publishedAt: "2016-04-10",
});
```

```html
<ul>
  <li v-for="value in myObject">{{ value }}</li>
</ul>
```

`v-for` can also take an integer. In this case it will repeat the template that many times, based on a range of `1...n`.

```html
<span v-for="n in 10">{{ n }}</span>
```

Note here `n` starts with an initial value of `1` instead of `0`.

To give Vue a hint so that it can track each node's identity, and thus reuse and reorder existing elements, you need to provide a unique `key` attribute for each item:

```html
<div v-for="item in items" :key="item.id">
  <!-- content -->
</div>
```

It is recommended to provide a `key` attribute with `v-for` whenever possible, unless the iterated DOM content is simple. The `key` binding expects primitive values - i.e. strings and numbers. Do not use objects as `v-for` keys.

You can directly use `v-for` on a component, like any normal element:

```html
<MyComponent v-for="item in items" :key="item.id" />
```

# 14. Form Inputs (v-model):

The `v-model` directive helps us sync the state of form input elements with corresponding state.

```html
<input v-model="text" />
```

`v-model` can be used on inputs of different types:

- `<input>` with text types and `<textarea>` elements use `value` property and `input` event;
- `<input type="checkbox">` and `<input type="radio">` use `checked` property and `change` event;
- `<select>` uses `value` as a prop and `change` as an event.

## a. Text Input

```html
<input v-model="message" />
<p>Message is: {{ message }}</p>
```

## b. CheckBox

- Single checkbox, boolean value:
  
  ```html
  <input type="checkbox" id="checkbox" v-model="checked" />
  <label for="checkbox">{{ checked }}</label>
  ```

- Multiple checkboxes to the same array:
  
  ```js
  const checkedNames = ref([]);
  ```
  
  ```html
  <div>Checked names: {{ checkedNames }}</div>
  
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames" />
  <label for="jack">Jack</label>
  
  <input type="checkbox" id="john" value="John" v-model="checkedNames" />
  <label for="john">John</label>
  
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames" />
  <label for="mike">Mike</label>
  ```

![multiple checkboxes to same array](<multiple checkboxes to same array.png>)

## c. Radio Button

```html
<div>Picked: {{ picked }}</div>

<input type="radio" id="one" value="One" v-model="picked" />
<label for="one">One</label>

<input type="radio" id="two" value="Two" v-model="picked" />
<label for="two">Two</label>
```

![radio button](<radio button.png>)

## d. Select

```html
<div>Selected: {{ selected }}</div>

<select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```

Select options can be dynamically rendered with `v-for`:

```js
const selected = ref("A");

const options = ref([
  { text: "One", value: "A" },
  { text: "Two", value: "B" },
  { text: "Three", value: "C" },
]);
```

```html
<select v-model="selected">
  <option v-for="option in options" :value="option.value">
    {{ option.text }}
  </option>
</select>

<div>Selected: {{ selected }}</div>
```

# 15. Lifecycle Hooks:

the `onMounted` hook can be used to run code after the component has finished the initial rendering and created the DOM nodes:

```html
<script setup>
  import { onMounted } from "vue";

  onMounted(() => {
    console.log(`the component is now mounted.`);
  });
</script>
```

There are also other hooks which will be called at different stages of the instance's lifecycle, with the most commonly used being [`onMounted`](https://vuejs.org/api/composition-api-lifecycle#onmounted), [`onUpdated`](https://vuejs.org/api/composition-api-lifecycle#onupdated), and [`onUnmounted`](https://vuejs.org/api/composition-api-lifecycle#onunmounted).

# 16. Watchers

Watchers in Vue allow you to observe and react to data changes, (kind of useEffect() in React).

We can use the `watch` function to trigger a callback whenever a piece of reactive state changes:

```js
import { ref, watch } from "vue";

const count = ref(0);

watch(count, (newValue, oldValue) => {
  console.log(`Count changed from ${oldValue} to ${newValue}`);
});

// Somewhere in your code
count.value++;
// Console: "Count changed from 0 to 1"
```

- Watching Multiple Sources:
  
  ```js
  const x = ref(0);
  const y = ref(0);
  
  watch([x, y], ([newX, newY], [oldX, oldY]) => {
    console.log(`x: ${oldX} -> ${newX}, y: ${oldY} -> ${newY}`);
  });
  ```

- Deep Watching:
  
  ```js
  const obj = reactive({ nested: { count: 0 } });
  
  watch(
    obj,
    (newVal, oldVal) => {
      console.log("obj changed");
    },
    { deep: true }
  );
  
  // This will trigger the watcher
  obj.nested.count++;
  ```

- watchEffect:
  
  Automatically tracks reactive dependencies:
  
  ```js
  const count = ref(0);
  const double = computed(() => count.value * 2);
  
  watchEffect(() => {
    console.log(`Count is ${count.value}, double is ${double.value}`);
  });
  
  // Logs immediately and on every change to count
  ```

- Stop watching:
  
  ```js
  const stop = watch(source, callback);
  
  // Later, when you want to stop watching
  stop();
  ```

Watchers are used to handle side effects, so we use them to perform API calls, here is a good example:

```html
<script setup>
  import { ref, watch } from "vue";

  const question = ref("");
  const answer = ref("Questions usually contain a question mark. ;-)");
  const loading = ref(false);

  // watch works directly on a ref
  watch(question, async (newQuestion, oldQuestion) => {
    if (newQuestion.includes("?")) {
      loading.value = true;
      answer.value = "Thinking...";
      try {
        const res = await fetch("https://yesno.wtf/api");
        answer.value = (await res.json()).answer;
      } catch (error) {
        answer.value = "Error! Could not reach the API. " + error;
      } finally {
        loading.value = false;
      }
    }
  });
</script>

<template>
  <p>
    Ask a yes/no question:
    <input v-model="question" :disabled="loading" />
  </p>
  <p>{{ answer }}</p>
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNp9U8Fy0zAQ/ZVFF9tDah96C2mZ0umhHKBAj7oIe52oUSQjyXEyGf87KytyoDC9JPa+p+e3b1cndtd15b5HtmQrV1vZeXDo++6Wa7nrjPVwAovtAgbh6w2M0Fqzg4xOZFxzXRvtPPzq0XlpNNwEbp5lRUKEdgPaVP925jnoXS+UOgKxvJAaxEVjJ+y2hA9XxUVFGdFIvT7LtEI5JIzrqjrbGozdOmikxdqTKqmIQOV6gvOkvQDhjrqGXOOQvCzAqCa9FHBzCyeuAWT7F6uUulZ9gy7PPmZFETmQjJV7oXoke972GJHY+Axkzxupt4FalhRcYHh7TDIQcqA+LTriikFIDy0G59nG+84tq+qITpty8G0lOhmSiedefSaPZ0mnfHFG50VRRkbkj1BPceVorbFzF/+6fQj4O7g3vWpAm6Ao6JzfINw9PZaQwXuYNJJuK/U0z1nxdTLT0M7s8Ec/I3WxquLS0brRi8ddp4RHegNYhR0M/Du3pXFSAJU285osI7aSuus97K92pkF1w1nCOYNlI534qbCh8tkOVasoXkV1+sjplLZ0HGN5Vc1G2IJ5R8Np5XpKlK7J1CJntdl1UqH92k0bzdkyNc8ZRWGGz1MtbMQi1esN1tv/1F/cIdQ4e6LJod0jZzPmhV2jj/DDjy94oOcZpK57Rew3wO/ojOpjJIH2qdcN2f6DN7l9nC47RfTsHg4etUtNpZUeJz5ndPPv32j9Yve6vE6DZuNvu1R2Tg==)

# 17. useTemplateRef:

`ref` is a special attribute <u>_(different than the ref() used for reactive states)_</u> , it allows us to obtain a direct reference to a specific DOM element or child component instance after it's mounted. This may be useful when you want to, for example, programmatically focus an input on component mount, or initialize a 3rd party library on an element.

To obtain the reference with Composition API, we can use the `useTemplateRef()` helper:

```html
<script setup>
  import { useTemplateRef, onMounted } from "vue";

  // the first argument must match the ref value in the template
  const input = useTemplateRef("my-input");

  onMounted(() => {
    input.value.focus();
  });
</script>

<template>
  <input ref="my-input" />
</template>
```

Note that you can only access the ref **after the component is mounted.** If you try to access `input` in a template expression, it will be `null` on the first render.

# 18. Components:

To use a child component, we need to import it in the parent component.

It's also possible to globally register a component, making it available to all components in a given app without having to import it.

```html
<script setup>
  import ButtonCounter from "./ButtonCounter.vue";
</script>

<template>
  <h1>Here is a child component!</h1>
  <ButtonCounter />
</template>
```

## a. Props

Props are custom attributes you can register on a component using the [`defineProps`](https://vuejs.org/api/sfc-script-setup.html#defineprops-defineemits) macro.

`defineProps` is a compile-time macro that is only available inside `<script setup>` and does not need to be explicitly imported.

```html
<!-- BlogPost.vue -->
<script setup>
  defineProps(["title"]);
</script>

<template>
  <h4>{{ title }}</h4>
</template>
```

```js
const posts = ref([
  { id: 1, title: "My journey with Vue" },
  { id: 2, title: "Blogging with Vue" },
  { id: 3, title: "Why Vue is so fun" },
]);
```

```html
<BlogPost v-for="post in posts" :key="post.id" :title="post.title" />
```

Notice how [`v-bind` syntax](https://vuejs.org/api/built-in-directives.html#v-bind) (`:title="post.title"`) is used to pass dynamic prop values.

## b. Events

Just like `props` allow passing data **into** a child component, `$emit` allows the child component to send events **back** to the parent component. This is useful when the child needs to notify the parent that something has happened (like a button click).

![events example](<events example.png>)

![event illustrations](<events illustrations.png>)

Similar to `defineProps`, `defineEmits` is only usable in `<script setup>` and doesn't need to be imported.

```html
<!-- Child component (BlogPost.vue) -->
<script setup>
  defineProps(["title"]);
  defineEmits(["sayMyName"]);
</script>

<template>
  <h4>text here: {{ title }}</h4>
  <button @click="$emit('sayMyName', title)">Click me!</button>
</template>
```

```html
<!-- Parent component (App.vue) -->
<script setup>
  import BlogPost from "./components/BlogPost.vue";

  // array of blog posts
  const posts = [
    { title: "My first post" },
    { title: "My second post" },
    { title: "My third post" },
  ];

  function handlePostClick(title) {
    alert(`Post title clicked: ${title}`);
  }
</script>

<template>
  <h1>My Blog</h1>
  <ul>
    <li v-for="post in posts" :key="post.title">
      <BlogPost :title="post.title" @say-my-name="handlePostClick" />
    </li>
  </ul>
</template>
```

## c. Slots

We use the `<slot>` as a placeholder where we want the content to go.

```html
<!-- SubmitButton.vue -->
<template>
  <button type="submit">
    <slot />
  </button>
</template>
```

```html
<SubmitButton>
  Save
</SubmitButtonox>
```

We can have a fallback content if the parent didn't provide any slot content.

```html
<template>
  <button type="submit">
    <slot>
           Submit
      <!-- fallback content -->
          
    </slot>
  </button> </template
>ton>
```

Now when we use `<SubmitButton>` in a parent component, providing no content for the slot, it will render the fallback content, "Submit". But if we provide content, Then the provided content will be rendered instead.

the `<slot>` element has a special attribute, `name`, which can be used to assign a unique ID to different slots so you can determine where content should be rendered:

```html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

To pass a named slot, we need to use a `<template>` element with the `v-slot` directive, and then pass the name of the slot as an argument to `v-slot`:

```html
<BaseLayout>
  <template v-slot:header>
    <!-- content for the header slot -->
  </template>
</BaseLayout>
```

A `<slot>` outlet without `name` implicitly has the name "default".

`v-slot` has a dedicated shorthand `#`, so `<template v-slot:header>` can be shortened to just `<template #header>`.

```html
<template #header>
  <h1>Here might be a page title</h1>
</template>
```

You can use the [$slots](https://vuejs.org/api/component-instance.html#slots) property in combination with a [v-if](https://vuejs.org/guide/essentials/conditional.html#v-if) to render something based on whether or not a slot is present.

```html
<div v-if="$slots.header" class="card-header">
  <slot name="header" />
</div>
```

## d. \<component >

Vue provide the`component` element with the special `is` attribute which is useful to dynamically switch between components, like in a tabbed interface.

```html
<!-- Component changes when currentTab changes -->
<component :is="tabs[currentTab]"></component>
```

In the example above, the value passed to `:is` can contain either:

- the name string of a registered component, OR
- the actual imported component object

![component](<component.png>)

[Open example in the Playground](https://play.vuejs.org/#eNqNVMGOmzAQ/ZURe2BXCiHbrXpwk1X31mMPvS1V5RiTWAEb2SZNhPLvHdvggLZRE6TIM/P8/N5gpk/e2nZ57HhCkrVhWrQWDLdd+1pI0bRKW/iuGg6VVg2ky9wFDp7G8g9lrIl1H80Bb5rtxfFKMcRzUA+aV3AZQKEEhWRKGgus05pL+5NuYeNwj6mTkT4VckRYujVY63GT17twC6/Fr4YjC3kp5DoPNtEgBpY3bU0txwhgXYojsJoasymSkjeqSHweK9vOWoUbXIC/Y1YpjaDH3wt39hMI6TUUSYSQAz8jArPT5Mj+nmIhC6zpAu1TZlEhmXndbBwpXH5NGL6xWrADMsyaMj1lkAzQ92E7mvYe8nCcM24xZApbL5ECiHCSnP73KyseGnvh6V/XedwS2pVjv3C1ziddxNDYc+2WS9fC8E4qJW1W0UbUZwKGSpMZrkX11dW2SpdcE3huT2BULUp44JxPSpmmpegMgU/tyadbWpZC7jCxwj0v+OfTDdU7ITOrWiTjzTS3Vei8IfB5xHZ4PmqoObMEJHryWXXkuqrVn+xEgHZWYRKbh06uLyv4iQq+oIDnkXSQiwKymlc26n75WNdit78FmLWCMeZL+GKMwlKrhLRcBzhlh51WnSwJPFQr9/zLdIZ007w/O6bR4MQe2bseBJMzer5yzwf8MtzbOzYMkNsOY0+HfoZv1d+lZJGMg8fNqdsfbbio4b77uRVv7I0Li8xxZN1PHWbeHdyTWXc/+zgw/8t/+QsROe9h)

## e. Component v-model

`v-model` can also be used on a component using the [`defineModel()`](https://vuejs.org/api/sfc-script-setup.html#definemodel) macro.

```html
<!-- Parent component (App.vue) -->
<script setup>
  import Child from "./Child.vue";
  import { ref } from "vue";

  const msg = ref("Hello World");
</script>

<template>
  <h1>{{ msg }}</h1>
  <Child v-model="msg" />
</template>
```

```html
<!-- Child component (Child.vue) -->
<script setup>
  const test = defineModel();
</script>

<template>
  <input v-model="test" />
</template>
```

`v-model` on a component can also accept an argument:

```html
<MyComponent v-model:title="bookTitle" />
```

In the child component, we can support the corresponding argument by passing a string to `defineModel()` as its first argument:

```html
<!-- MyComponent.vue -->
<script setup>
  const title = defineModel("title");
</script>

<template>
  <input type="text" v-model="title" />
</template>
```

We can also create multiple `v-model` bindings on a single component instance.

```html
<UserName v-model:first-name="first" v-model:last-name="last" />
```

```html
<script setup>
  const firstName = defineModel("firstName");
  const lastName = defineModel("lastName");
</script>

<template>
  <input type="text" v-model="firstName" />
  <input type="text" v-model="lastName" />
</template>
```

## f. Attribute Inheritance

A "fallthrough attribute" is an attribute or `v-on` event listener that is passed to a component, but is not explicitly declared in the receiving component's [props](https://vuejs.org/guide/components/props.html) or [emits](https://vuejs.org/guide/components/events.html#declaring-emitted-events). Common examples of this include `class`, `style`, and `id` attributes.

When a component renders a single root element, fallthrough attributes will be automatically added to the root element's attributes. For example, given a `<MyButton>` component with the following template:

```html
<!-- template of <MyButton> -->
<button>Click Me</button>
```

And a parent using this component with:

```html
<MyButton class="large" />
```

The final rendered DOM would be:

```html
<button class="large">Click Me</button>
```

If the child component's root element already has existing `class` or `style` attributes, it will be merged with the `class` and `style` values that are inherited from the parent.

Suppose we change the template of `<MyButton>` in the previous example to:

```html
<!-- template of <MyButton> -->
<button class="btn">Click Me</button>
```

Then the final rendered DOM would now become:

```html
<button class="btn large">Click Me</button>
```

The same rule applies to `v-on` event listeners:

```html
<MyButton @click="onClick" />
```

The `click` listener will be added to the root element of `<MyButton>`, i.e. the native `<button>` element. When the native `<button>` is clicked, it will trigger the `onClick` method of the parent component. If the native `<button>` already has a `click` listener bound with `v-on`, then both listeners will trigger.

# 19. Provide / Inject:

Vue provides  `provide` and `inject` to solve the props drilling issue.

## a. Provide

To provide data to a component's descendants, use the [`provide()`](https://vuejs.org/api/composition-api-dependency-injection.html#provide) function.

The `provide()` function accepts two arguments. The first argument is called the **injection key**.

The injection key is used by descendant components to lookup the desired value to inject.

A single component can call `provide()` multiple times with different injection keys to provide different values.

The second argument is the provided value. The value can be of any type, including reactive state such as refs:

```html
<script setup>
  import { ref, provide } from 'vue'

  const count = ref(0)
  provide('key', count)!')
</script>
```

## b. Inject

To inject data provided by an ancestor component, use the [`inject()`](https://vuejs.org/api/composition-api-dependency-injection.html#inject) function.

If the provided value is a ref, it will be injected as-is and will **not** be automatically unwrapped. This allows the injector component to retain the reactivity connection to the provider component.

```html
<script setup>
  import { inject } from "vue";

  const message = inject("message");
</script>
```

By default, `inject` assumes that the injected key is provided somewhere in the parent chain. In the case where the key is not provided, there will be a runtime warning.

If we want to make an injected property work with optional providers, we need to declare a default value, similar to props:

```js
// `value` will be "default value"
// if no data matching "message" was provided
const value = inject("message", "default value");
```

When using reactive provide / inject values, **it is recommended to keep any mutations to reactive state inside of the *provider* whenever possible**.

There may be times when we need to update the data from an injector component. In such cases, we recommend providing a function that is responsible for mutating the state:

```html
<!-- inside provider component -->
<script setup>
  import { provide, ref } from "vue";

  const location = ref("North Pole");

  function updateLocation() {
    location.value = "South Pole";
  }

  provide("location", {
    location,
    updateLocation,
  });
</script>
```

```html
<!-- in injector component -->
<script setup>
  import { inject } from "vue";

  const { location, updateLocation } = inject("location");
</script>

<template>
  <button @click="updateLocation">{{ location }}</button>
</template>
```

# 20. Composables

Composables are functions that encapsulate reactive state and related logic. They usually start with "use" by convention (e.g., useCounter, useFetch). When called in a component's setup function, they return an object with reactive data and methods that can be used in the component.

They allow you to extract and reuse stateful logic across your Vue application, making your code more modular and easier to maintain.

Let's look at a simple example of a composable that manages a counter:

```js
// useCounter.js
import { ref } from "vue";

export function useCounter(initialCount = 0) {
  const count = ref(initialCount);

  function increment() {
    count.value++;
  }

  function decrement() {
    count.value--;
  }

  return {
    count,
    increment,
    decrement,
  };
}
```

Now, let's see how we can use this composable in a Vue component:

```html
<script setup>
  import { useCounter } from "./composables/useCounter";

  const { count, increment, decrement } = useCounter(0);
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <button @click="increment">Increment</button>
    <button @click="decrement">Decrement</button>
  </div> </template
>t
```

# 21. Built-in Components:

## a. \<Transition>

The <Transition> component is a built-in component in Vue 3 that allows you to add smooth transitions and animations when elements are inserted, updated, or removed from the DOM. It's a powerful tool for enhancing the user experience of your application.

Here's a basic explanation of how it works:

1. Wrapping: You wrap the element you want to animate with the <Transition> component.
2. CSS Classes: Vue automatically applies CSS classes at different stages of the transition, which you can use to define your animations.
3. JavaScript Hooks: It also provides JavaScript hooks for more complex animations.

The enter or leave can be triggered by one of the following:

- Conditional rendering via `v-if`
- Conditional display via `v-show`
- Dynamic components toggling via the `<component>` special element
- Changing the special `key` attribute

A transition can be named via the `name` prop.

Here's a simple example:

```html
<script setup>
  import { ref } from "vue";

  const show = ref(false);
</script>

<template>
  <div>
    <button @click="show = !show">Toggle</button>

    <Transition name="fade">
      <p v-if="show">Hello, I'm a transitioning paragraph!</p>
    </Transition>
  </div>
</template>

<style>
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.5s ease;
  }

  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
  }
</style>
```

There are six classes applied for enter / leave transitions:

![transition classes](<transition classes.png>)

## b. \<TransitionGroup>

`<TransitionGroup>` is a built-in component designed for animating the insertion, removal, and order change of elements or components that are rendered in a list.

`<TransitionGroup>` supports the same props, CSS transition classes, and JavaScript hook listeners as `<Transition>`, but all elements inside are **always required** to have a unique `key` attribute.

Here is an example of applying enter / leave transitions to a `v-for` list using `<TransitionGroup>`:

```html
<template>
  <TansitionGroup name="list" tag="ul">
    <li v-for="item in items" :key="item">
      {{ item }}
    </li>
  </TransitionGroup>
</template>

<style>
.list-move, /* apply transition to moving elements */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* ensure leaving items are taken out of layout flow so that moving
   animations can be calculated correctly. */
.list-leave-active {
  position: absolute;
}
</style>
```

## c. \<KeepAlive>

`<KeepAlive>` is a built-in component that allows us to conditionally cache component instances when dynamically switching between multiple components.

When using the `<component>` special component, by default an active component instance will be unmounted when switching away from it. This will cause any changed state it holds to be lost. When this component is displayed again, a new instance will be created with only the initial state.

Creating fresh component instance on switch is normally useful behavior, but in this case, we'd really like the two component instances to be preserved even when they are inactive. To solve this problem, we can wrap our dynamic component with the `<KeepAlive>` built-in component:

```html
<!-- Inactive components will be cached! -->
<KeepAlive>
  <component :is="activeComponent" />
</KeepAlive>
```

Now, the state will be persisted across component switches

By default, `<KeepAlive>` will cache any component instance inside. We can customize this behavior via the `include` and `exclude` props. Both props can be a comma-delimited string:

```html
<!-- Will cache only a and b components -->
<KeepAlive include="a,b">
  <component :is="view" />
</KeepAlive>
```
