---
title: Svelte 5 For Beginners
description: The ultimate guide for the most beloved JavaScript framework.
slug: svelte-5-for-beginners
published: '2025-7-14'
category: svelte
---

## Table of Contents

## What is Svelte?

If we look at the definition from the [Svelte](https://svelte.dev/) website, it says:

> Svelte is a UI framework that uses a compiler to let you write breathtakingly concise components that do minimal work in the browser, using languages you already know — HTML, CSS and JavaScript.

Svelte is not just a UI framework, it's a compiled language. This means having the best developer experience as it's not constrained by the limitations of JavaScript. You also ship less code and features like animations are built-in because only what you use gets bundled.

If you're looking for an application framework with more opinions, routing, and server-side rendering among other things, Svelte has a meta-framework called [SvelteKit](https://svelte.dev/docs/kit/introduction) that's comparable to [Next.js](https://nextjs.org/) for React.

## Try Svelte

You can try Svelte in the browser using the [Svelte Playground](https://svelte.dev/playground) and follow along without having to set up anything.

If you're a creature of comfort and prefer your development environment, you can scaffold a Vite project and pick Svelte as the option from the CLI if you run `npm create vite@latest` in a terminal — you're going to need [Node.js](https://nodejs.org/) for that.

I also recommend using the [Svelte for VS Code extension](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode) for syntax highlighting and code completion, or a similar extension for your editor.

## Single File Components

Let's start with a simple counter example using regular HTML and JavaScript:

```svelte:app.html
<script>
  let count = 0
  let text = document.querySelector('p')

  function increment() {
    count++
    text.innerText = `Clicked ${count} ${count === 1 ? 'time' : 'times'}`
  }
</script>

<p>Clicked 0 times</p>
<button onclick="increment()">Click</button>
```

Having to keep track of state and [Document Object Model (DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) updates is tedious even in the glorious age of AI.

Let's look at the same example in Svelte:

```svelte:App.svelte
<script>
  let count = $state(0)

  function increment() {
		count++
  }
</script>

<p>Clicked {count} {count === 1 ? 'time' : 'times'}</p>
<button onclick={increment}>Click</button>
```

Don't worry if you don't understand the code yet, we'll go over it in the next section.

**You can think of Svelte as HTML with superpowers.**

You write code in a **declarative** way like HTML, but you don't have to think about querying elements and keeping the state of your application in sync with the user interface.

In Svelte, files ending with `.svelte` are called **single file components** because the JavaScript, HTML, and CSS are contained inside a single file.

You can use JavaScript expressions like `{count === 1 ? 'time' : 'times'}` in the template, but we're also going to look into using logic blocks like `if` and `each` to conditionally render content.

Let's create a `<style>` tag to add some styles:

```svelte:App.svelte
<style>
  p {
    color: red;
  }
</style>
```

Styles are scoped to the component by default. This means that styles used in one component aren't going to affect styles in other components. If you look at the CSS output in the Svelte Playground, you can see Svelte generated a unique class name for the styles `p.svelte-omwhvp {color: red }`.

To make your styles global inside a component, you can use the `global` modifier `:global(p)`. Having to use `:global(selector)` for everything is tedious, so you can nest everything inside the `:global { ... }` block. You can also have "scoped global styles" by saying `.prose :global(p)`:

```svelte:App.svelte
<style>
	/* global styles */
	:global(p) {
		color: red;
	}

	/* global block */
	.prose :global {
		p {
			color: red;
		}
		/* ... */
	}

	/* scoped global styles */
	.prose :global(p) {
		color: red;
	}
</style>
```

You can preprocess the styles with [SCSS](https://sass-lang.com/) by simply adding `lang="scss"` to the `<style>` tag, or use TypeScript by adding `lang="ts"` to the `<script>` tag:

```svelte:App.svelte
<script lang="ts">
	let count: number = 0
</script>

<style lang="scss">
	.prose {
		p {
			color: red;
		}
	}
</style>
```

## State Management And Reactivity Using Runes

In the last example, we defined a reactive variable `count` using the `$state` syntax:

```svelte:App.svelte
<script>
  let count = $state(0)

  function increment() {
    count++
  }
</script>

<p>Clicked {count} {count === 1 ? 'time' : 'times'}</p>
<button onclick={increment}>Click</button>
```

The `$state` syntax is called a **rune** and is part of the Svelte language. Under the hood Svelte turns the `$state` rune into a signal. The three main important runes we're going to learn about are the `$state`, `$derived`, and `$effect` rune.

The `$state` rune marks a variable as reactive. Svelte's reactivity is based on **assignments**. To update the UI, you just assign a new value to a reactive variable:

```svelte:App.svelte
<script>
	// reactive value
  let count = $state(0)

  function increment() {
		// reactive assignment
    count++
  }
</script>

<!-- update every time count changes -->
<p>Clicked {count} {count === 1 ? 'time' : 'times'}</p>
<button onclick={increment}>Click</button>
```

In Svelte, components don't rerun when a value changes like in React. Instead, Svelte surgically updates the DOM in place when a value updates.

If you want a value to automatically update when other values it depends on update, you should use the `$derived` rune to create a computed property:

```svelte:App.svelte
<script>
  let count = $state(0)
	let double = $derived(count * 2)

  function increment() {
    count++
  }
</script>

<button onclick={increment}>
	{doubled}
</button>
```

The `$derived` rune only accepts an expression by default, but you can use the `$derived.by` rune if you want to pass a function for a more complex derivation:

```svelte:App.svelte
<script>
	let cart = $state([
		{ item: 'apple', total: 10 },
		{ item: 'banana', total: 10 }
	])
	let total = $derived.by(() => {
		let sum = 0
		for (let item of cart) {
			sum += item.total
		}
		return sum
	})
</script>

<p>Total: {total}€</p>
```

Derived values are lazy evaluted. The derived value only updates when it changes and not when their dependencies change.

The last rune you should know about is the `$effect` rune. Effects are functions that run when the component is added (mounted) and when their dependencies change. You can also return a function from an effect which reruns when the effect dependencies change, or when the component removed (unmounted).

**Effects don't need a dependency array** because of how signals work — if a reactive value is read inside of an effect, it will be tracked and the effect will rerun when the tracked value changes:

```svelte:App.svelte
<script>
	let count = $state(0)
	let double = $derived(count * 2)

	$effect(() => {
		// reruns if `count` or `double` changes
		console.log({ count, double })
		// also runs when the component is removed
		return () => console.log('🧹 cleanup')
	})
</script>

<button onclick={() => count++}>
	{double}
</button>
```

{% warning text="You can use the [$inspect](https://svelte.dev/docs/svelte/$inspect) rune instead of effects to log when a reactive value updates." %}

**You should never use effects for updating state** because Svelte queues effects and runs them after everything is updated.

Here's an example how using effects to synchronize state can cause unexpected behavior:

```svelte:App.svelte
<script>
	let count = $state(0)
	let double = $state(0)

	$effect(() => {
		// `double` is updated after we log `double`
		double = count * 2
	})
</script>

<button onclick={() => {
	count++
	console.log(double) // ⚠️ out of sync
}}>
	{double}
</button>
```

**Always derive your state** using the `$derived` rune when you can and reach for the `$effect` rune sparingly:

```svelte:App.svelte
<script>
	let count = $state(0)
	let double = $derived(count * 2)
</script>

<button onclick={() => {
	count++
	console.log(double) // 👍️ latest value
}}>
	{double}
</button>
```

{% warning text="Derived values are effects under the hood, but they rerun immediately when their dependencies change." %}

Effects should only be used for side-effects like fetching data from an API, working with the DOM directly, or to synchronize with an external system that doesn't understand Svelte's reactivity:

```svelte:App.svelte
<script>
	let pokemon = $state()

	$effect(() => {
		const savedPokemon = localStorage.getItem('pokemon')

		if (!savedPokemon) {
			// fetching data from an API
			fetch('https://pokeapi.co/api/v2/pokemon')
				.then((response) => response.json())
				.then((data) => {
					// sync with an external system
					pokemon = data
					localStorage.setItem('pokemon', JSON.stringify(data))
				})
		} else {
			pokemon = JSON.parse(savedPokemon)
		}
	})
</script>

<pre>{JSON.stringify(pokemon, null, 2)}</pre>
```

## Template Logic

There are no conditionals and loops in HTML unless you're using a templating language. In Svelte, you can use the `#if` block to conditionally render content:

```svelte:App.svelte
<script>
	let user = $state({ loggedIn: false })

	function toggle() {
		user.loggedIn = !user.loggedIn
	}
</script>

{#if user.loggedIn}
  <button onclick={toggle}>Log out</button>
{:else}
	<button onclick={toggle}>Log in</button>
{/if}
```

To loop over a list of items, you use the `#each` block:

```svelte:App.svelte
<script>
	let todos = [
		{ id: 1, text: 'Todo 1', done: true },
		{ id: 2, text: 'Todo 2', done: false },
		{ id: 3, text: 'Todo 3', done: false },
		{ id: 4, text: 'Todo 4', done: false },
	]
</script>

<ul>
	{#each todos as todo}
		<li>
			<input checked={todo.done} type="checkbox" />
			<span>{todo.text}</span>
		</li>
	{:else}
		<p>No items</p>
	{/each}
</ul>
```

{% info text="The else clause is optional." %}

You can [destructure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) the items values you're iterating over, get the current item index and provide a key, so Svelte can keep track of changes:

```svelte:App.svelte
<ul>
	{#each todos as { id, text, done }, index (id)}
		<li>
			<input checked={done} type="checkbox" />
			<span>{text}</span>
		</li>
	{/each}
</ul>
```

Sometimes you just want to create an arbitrary amount of items like a grid, so you can ignore the `as` part. Here's an example of a 10x10 grid:

```svelte:App.svelte
<script>
	let size = 10
</script>

<div class="grid">
  {#each { length: size }, row}
    {#each { length: size }, col}
      <div class="cell">{row},{col}</div>
    {/each}
  {/each}
</div>

<style>
  .grid {
		max-width: 400px;
    display: grid;
    grid-template-columns: repeat(10, 1fr);
    gap: 0.5rem;
  }

  .cell {
    padding: 1rem;
    border: 1px solid #ccc;
  }
</style>
```

In a previous example, we fetched some Pokemon data inside of an effect. That approach works, but we haven't handled any of the the error and success states which quickly becomes a mess.

Thankfully, Svelte has a built-in solution for async data loading using the `#await` block:

```svelte:App.svelte
<script>
  async function getPokemon(name) {
    let response = await fetch(`https://pokeapi.co/api/v2/pokemon/${name}`)
    let { name, sprites } = await response.json()
		return { name, image: sprites['front_default'] }
  }
</script>

{#await getPokemon('charizard')}
	<p>loading...</p>
{:then pokemon}
	<p>{pokemon.name}</p>
	<img src={pokemon.image} alt={pokemon.name} />
{:catch error}
	<p>{error.message}</p>
{/await}
```

In the near future, you're going to be able to `await` a promise directly in a Svelte component. You can try it today by enabling the [experimental async flag](https://github.com/sveltejs/svelte/discussions/15845) in your Svelte config:

```ts:svelte.config.js
export default {
  compilerOptions: {
    experimental: {
      async: true
    }
  }
}
```

At the moment you have to create a [boundary](https://svelte.dev/docs/svelte/svelte-boundary) which you can put at the root of your app, or where you want to use the `await` keyword:

```svelte:App.svelte
<script>
	// pretend this is an import
	import { getPokemon } from 'api/pokemon'

	// you could `await` the data here if the boundary was declared higher up
	let pokemon = getPokemon('charizard')
</script>

<svelte:boundary>
	{#snippet pending()}
		<!-- only shows when the component is added -->
		<p>loading...</p>
	{/snippet}

	<!-- for loading new data -->
	{#if $effect.pending()}
		<p>loading...</p>
	{:else}
		<p>{(await pokemon).name}</p>
		<img src={(await pokemon).image} alt={(await pokemon).name} />
	{/if}
</svelte:boundary>
```

## Listening To Events

Events in Svelte use the same naming convention as standard [JavaScript events](https://developer.mozilla.org/en-US/docs/Web/Events#event_listing).

You can listen to DOM events by adding attributes that start with `on` to elements. In the case of a mouse click, you would add the `onclick` attribute to a `<button>`:

```svelte:App.svelte
<script>
	function onclick() {
		console.log('clicked')
	}
</script>

<!-- using an inline function -->
<button onclick={() => console.log('clicked')}>Click</button>

<!-- passing a function -->
<button onclick={onclick}>Click</button>

<!-- using the shorthand -->
<button {onclick}>Click</button>
```

You can spread events, since they're just attributes:

```svelte:App.svelte
<script>
	const events = {
		onclick: () => console.log('clicked'),
		ondblclick: () => console.log('double clicked')
	}
</script>

<button {...events}>Click</button>
```

Here's an example of using the `onmousemove` event to update the mouse position:

```svelte:App.svelte
<script>
	let mouse = $state({ x: 0, y: 0 })

	function onmousemove(e) {
		mouse.x = e.clientX
		mouse.y = e.clientY
	}
</script>

<div {onmousemove}>
	The mouse position is {mouse.x} x {mouse.y}
</div>
```

The `event` is automatically passed to the function, so you don't have to do `onmousemove={(e) => onmousemove(e)}`.

You can also prevent default behavior by using `e.preventDefault()`. This is useful when you want to control a form with JavaScript and avoid a page reload:

```svelte:App.svelte
<script>
	function onsubmit(e) {
		e.preventDefault()
		// sign up to newsletter
	}
</script>

<form {onsubmit}>
	<input type="email" />
	<button type="submit">Sign up</button>
</form>
```

## Data Binding

In this example we take the user input by listening to the `input` event and filter the list of items based on it:

```svelte:App.svelte
<script>
 	let list = $state(['angular', 'react', 'svelte', 'vue'])
  let filteredList = $derived(list.filter(item => item.includes(search)))
	let search = $state('')
</script>

<input
	oninput={(e) => search = e.target.value}
	value={search}
	type="search"
/>

<ul>
	{#each filteredList as item}
		<li>{item}</li>
	{/each}
</ul>
```

This is a lot of boilerplate code for something that's so common in web development. Thankfully, Svelte supports two-way data binding using the `bind:` directive:

```svelte:App.svelte
<input bind:value={search} type="search" />
```

Svelte provides many two-way bindings, and some readonly bindings. There are input, group, files, media and more bindings you can find in the [Svelte documentation](https://svelte.dev/docs/svelte/bind):

```svelte:App.svelte
<script>
	let text = $state('Hello 👋')
	let number = $state(0)
	let checkbox = $state(false)
	let range = $state(0)
</script>

<input type="text" bind:value={text} />
<p>{text}</p>

<input type="number" bind:value={number} />
<p>{number}</p>

<input type="checkbox" bind:checked={checkbox} />

<input type="range" bind:value={range} min="0" max="100" />
```

One of the more useful bindings is `bind:this` to get a reference to a DOM node such as the `<canvas>` element for example:

```svelte:App.svelte
<script>
	// `undefined` until the component is added
	let canvas

	$effect(() => {
		// ⛔️ don't do this
		// const canvas = document.querySelector('canvas')

		// 👍️ bind the value instead
		const ctx = canvas.getContext('2d')
	})
</script>

<canvas bind:this={canvas}></canvas>
```

Another useful thing to know about are **function bindings** when you need to do something with a value when it changes. This works by passing `bind:property={get, set}`, where `get` and `set` are functions:

```svelte:App.svelte
<script>
 	let celsius = $state(0)
 	let fahrenheit = $state(32)
</script>

<input bind:value={
  () => celsius,
  (v) => {
    celsius = v
    fahrenheit = (celsius * 9/5 + 32).toFixed()
  }
} />

<input bind:value={
  () => fahrenheit,
  (v) => {
    fahrenheit = v
    celsius = ((fahrenheit - 32) * 5/9).toFixed()
  }
} />
```

## Components

> Frameworks are not tools for organizing your code, they are tools for organizing your mind. — [Rich Harris](https://www.youtube.com/watch?v=AdNJ3fydeao)

You can think of components as reusable lego blocks that can include the markup, styles, and logic that can be reused across your app. You can use them with other blocks to compose bigger parts of your application.

A Svelte component is a file that ends with the `.svelte` extension.

In my opinion, **you should avoid creating components**. If you're not sure what to turn into a component — don't. Instead, write everything inside a single component until it gets complicated, or the reusable parts become obvious.

Let's use a basic todo list app as an example:

```svelte:Todos.svelte
<script>
	import { slide } from 'svelte/transition'

	let todo = $state('')
	let todos = $state([])
	let filter = $state('all')
	let filteredTodos = $derived(filterTodos())
	let remaining = $derived(remainingTodos())

	function addTodo(e) {
		e.preventDefault()
		todos.push({
			id: crypto.randomUUID(),
			text: todo,
			completed: false
		})
		todo = ''
	}

	function removeTodo(todo) {
		todos = todos.filter((t) => t.id !== todo.id)
	}

	function filterTodos() {
		return todos.filter((todo) => {
			if (filter === 'all') return true
			if (filter === 'active') return !todo.completed
			if (filter === 'completed') return todo.completed
		})
	}

	function setFilter(newFilter) {
		filter = newFilter
	}

	function remainingTodos() {
		return todos.filter((todo) => !todo.completed).length
	}

	function clearCompleted() {
		todos = todos.filter((todo) => !todo.completed)
	}
</script>

<form onsubmit={addTodo}>
	<input type="text" bind:value={todo} />
</form>

<ul>
	{#each filteredTodos as todo (todo.id)}
		<li transition:slide>
			<input type="checkbox" bind:checked={todo.completed} />
			<input type="text" bind:value={todo.text} />
			<button onclick={() => removeTodo(todo)}>🗙</button>
		</li>
	{/each}
</ul>

<div>
	<p>{remaining} {remaining === 1 ? 'item' : 'items'} left</p>

	{#each ['all', 'active', 'completed'] as filter}
		<button onclick={() => setFilter(filter)}>{filter}</button>
	{/each}

	<button onclick={clearCompleted}>Clear completed</button>
</div>
```

Let's take the contents of the `Todos.svelte` file and break it into multiple components. You can keep everything organized and place the files inside a `todos` folder:

```console:files
todos/
├── Todos.svelte
├── AddTodo.svelte
├── TodoList.svelte
├── TodoItem.svelte
└── TodoFilter.svelte
```

The way Svelte knows something is a component is by a capitalized tag such as `<Component>`, or dot notation like `<my.component>`. How you name the file is irrelevant. Most often you're going to see the PascalCase naming convention, so that's what I'm going to use. Personally, I prefer kebab-case.

First we'll create the component that handles adding a new todo:

```svelte:AddTodo.svelte
<script>
	let { todo = $bindable(), addTodo } = $props()
</script>

<form onsubmit={addTodo}>
	<input type="text" bind:value={todo} />
</form>
```

To receive and destructure props we use the `$props` rune. Here we bind the input value to the `todo` variable in the parent component, so we have to let Svelte know it's okay for the child to mutate the parent state by using the `$bindable` rune. You can now bind the `todo` prop:

```diff:todos.svelte
<script>
+	import AddTodo from './AddTodo.svelte'
</script>

+ <AddTodo bind:todo {addTodo} />
```

In reality, you don't have to do this. I just wanted to demonstrate how to use the `$bindable` rune if you have to. It makes more sense to move the `todo` state inside the component for adding todos:

```diff:todos.svelte
<script>
-	let todo = $state('')

-	function addTodo(e) {
+	function addTodo(todo) {
-		e.preventDefault()
		todos.push({
			id: crypto.randomUUID(),
			text: todo,
			completed: false
		})
-		todo = ''
}
</script>

- <AddTodo {todo} {addTodo} />
+ <AddTodo {addTodo} />
```

```svelte:AddTodo.svelte
<script>
	let { addTodo } = $props()
	let todo = $state('')

	function onsubmit(e) {
		e.preventDefault()
		addTodo(todo)
		todo = ''
	}
</script>

<form {onsubmit}>
	<input type="text" bind:value={todo} />
</form>
```

I'm mostly using a form because you can just press enter to submit. Instead of binding the value, you can get the value from the form `onsubmit` event. Later in this section, I'm going to show you what to do instead.

Let's create a component that renders the list of todos and spice it up with a built-in Svelte transition:

```svelte:TodoList.svelte
<script lang="ts">
	import { slide } from 'svelte/transition'

	let { todos, removeTodo } = $props()
</script>

<ul>
	{#each todos as todo, i (todo.id)}
		<li transition:slide>
			<input type="checkbox" bind:checked={todo.completed} />
			<input type="text" bind:value={todo.text} />
			<button onclick={() => removeTodo(todo.id)}>🗙</button>
		</li>
	{/each}
</ul>
```

```diff:todos.svelte
<script>
	import AddTodo from './AddTodo.svelte'
+	import TodoList from './TodoList.svelte'
</script>

<AddTodo {todo} {addTodo} />
+ <TodoList todos={filteredTodos} {removeTodo} />
```

Now we can create the component that filters the todos:

```svelte:TodoFilter.svelte
<script>
	let { remaining, setFilter, clearCompleted } = $props()
</script>

<div>
	<p>{remaining} {remaining === 1 ? 'item' : 'items'} left</p>

	{#each ['all', 'active', 'completed'] as filter}
		<button onclick={() => setFilter(filter)}>{filter}</button>
	{/each}

	<button onclick={clearCompleted}>Clear completed</button>
</div>
```

```diff:todos.svelte
<script>
	import AddTodo from './AddTodo.svelte'
	import TodoList from './TodoList.svelte'
+	import TodoFilter from './TodoFilter.svelte'
</script>

<AddTodo {todo} {addTodo} />
<TodoList todos={filteredTodos} {removeTodo} />
+ <TodoFilter {remaining} {setFilter} {clearCompleted} />
```

I left the todo item component for last to show you the downside of abusing bind:

```svelte:TodoItem.svelte
<script>
	import { slide } from 'svelte/transition'

	let { todo = $bindable(), removeTodo } = $props()
</script>

<li transition:slide>
	<input type="checkbox" bind:checked={todo.completed} />
	<input type="text" bind:value={todo.text} />
	<button onclick={() => removeTodo(todo.id)}>🗙</button>
</li>
```

This works, but Svelte is going to throw a bunch of warnings because you're mutating `todos` in the parent state, so now we have to make `todos` bindable:

```diff:todos.svelte
<script>
	import AddTodo from './AddTodo.svelte'
	import TodoList from './TodoList.svelte'
	import TodoFilter from './TodoFilter.svelte'
</script>

<AddTodo {todo} {addTodo} />
- <TodoList todos={filteredTodos} {removeTodo} />
+ <TodoList bind:todos={filteredTodos} {removeTodo} />
<TodoFilter {remaining} {setFilter} {clearCompleted} />
```

```diff:todoItem.svelte
<script>
	import TodoItem from './TodoItem.svelte'

-	let { todos, removeTodo } = $props()
+	let { todos = $bindable(), removeTodo } = $props()
</script>

<ul>
	{#each todos as todo, i (todo.id)}
		<li transition:slide>
+			<TodoItem bind:todo={todos[i]} {removeTodo} />
		</li>
	{/each}
</ul>
```

In general, avoid mutating props to avoid unexpected state changes. If you want to update a value from a child component, callback props are a better option. Let's change the todos component to show you what I mean:

```diff:todos.svelte
<script>
function addTodo(e) {
+	e.preventDefault()
+	const form = e.currentTarget
+	const formData = new FormData(form)
	todos.push({
		id: crypto.randomUUID(),
-		text: todo,
+		text: formData.get('todo'),
		completed: false
	})
+	form.reset()
}

+	function toggleTodo(todo: Todo) {
+		const index = todos.findIndex((t) => t.id === todo.id)
+		todos[index].completed = !todos[index].completed
+	}

+	function updateTodo(todo: Todo) {
+		const index = todos.findIndex((t) => t.id === todo.id)
+		todos[index].text = todo.text
+	}
</script>

+ <AddTodo {addTodo} />
+ <TodoList todos={filteredTodos} {toggleTodo} {updateTodo} {removeTodo} />
+ <TodoFilter {remaining} {setFilter} {clearCompleted} />
```

Let's update the offending components to use callback props to update the todos instead of binding props everywhere, which could lead to unpredictable behavior:

```svelte:AddTodo.svelte
<script>
	let { addTodo } = $props()
</script>

<form onsubmit={addTodo}>
	<input type="text" name="todo" />
</form>
```

```svelte:TodoList.svelte
<script>
	import TodoItem from './TodoItem.svelte'

	let { todos, toggleTodo, updateTodo, removeTodo } = $props()
</script>

<ul>
	{#each todos as todo (todo.id)}
		<TodoItem {todo} {toggleTodo} {updateTodo} {removeTodo} />
	{/each}
</ul>
```

```svelte:TodoItem.svelte
<script lang="ts">
	import { slide } from 'svelte/transition'

	let { todo, toggleTodo, updateTodo, removeTodo } = $props()
</script>

<li transition:slide>
	<input
		type="checkbox"
		onchange={() => toggleTodo(todo)}
		checked={todo.completed}
	/>
	<input
		type="text"
		oninput={() => updateTodo(todo)}
		bind:value={todo.text}
	/>
	<button onclick={() => removeTodo(todo)}>🗙</button>
</li>
```

As a cherry on top, let's save the todos in local storage:

```svelte:Todos.svelte
<script>
	// ...
	$effect(() => {
		todos = JSON.parse(localStorage.getItem('todos') || '[]')
	})

	$effect(() => {
		localStorage.setItem('todos', JSON.stringify(todos))
	})
</script>
```

There are more ways to do this, but I'm going to leave it here for now. Later we're going to learn how to talk between components without props, using the context API.

## Component Composition In Svelte

You can compose components by nesting them, using snippets which hold content that can be passed as props to components similar to slots, and communicate between components with the context API without props or events.

To demonstrate how wonderful component composition is in Svelte, let's create an accordion component that can have many accordion items. You can create these files inside an `accordion` folder:

```console:files
accordion/
├── Accordion.svelte
├── AccordionItem.svelte
└── index.ts
```

Let's export the accordion components from the `index.ts` file:

```ts:index.ts
export { default as Accordion } from './Accordion.svelte'
export { default as AccordionItem } from './AccordionItem.svelte'
```

In HTML, you can nest elements inside other elements:

```html:accordion.html
<div class="accordion">
	<div class="accordion-item">
		<button>
			<div>Item A</div>
			<div class="accordion-icon">👈️</div>
		</button>
		<div class="accordion-content">Content</div>
	</div>
</div>
```

The fun part of using a framework like Svelte is that you get to decide the API of your components and how to compose them. Here's one way how we can take the accordion HTML and turn it into a component ready to be used across your app:

```svelte:App.svelte
<script>
	import { Accordion, AccordionItem } from './accordion'
</script>

<Accordion>
	<AccordionItem title="Item A">
		Content
	</AccordionItem>
</Accordion>
```

The `<Accordion>` component accepts children like HTML — which can be anything. In this case, it's a `<AccordionItem>` component which accepts a `title` prop. Every component has an implicit `children` prop which is a snippet you can render using the `@render` tag. Any content inside the component tags becomes part of the `children` snippet:

```svelte:Accordion.svelte
<script>
	let { children } = $props()
</script>

<div class="accordion">
	<!-- a) conditional with a fallback -->
	{#if children}
		{@render children()}
	{:else}
		<p>Fallback content</p>
	{/if}

	<!-- b) optional chaining -->
	{@render children?.()}
</div>
```

The `<AccordionItem>` accepts a `label` prop and we can show the accordion item content using the `children` prop which acts like a catch-all for any content inside the component:

```svelte:AccordionItem.svelte
<script>
	let { label, children } = $props()

	let open = $state(false)

	function toggle() {
		open = !open
	}
</script>

<div class="accordion-item">
	<button onclick={toggle} class="accordion-heading">
		<div>{label}</div>
		<div class="accordion-icon">👈️</div>
	</button>

	{#if open}
		<div transition:slide class="accordion-content">
			{@render children?.()}
		</div>
	{/if}
</div>
```

That's it! You can now use the `<Accordion>` component in your app. That being said, this has limited composability. Let's say you don't like the icon, or position of the individual accordion elements. This could lead to a silly amount of props and conditionals:

```svelte:App.svelte
<script>
	import { Accordion, AccordionItem } from './accordion'
</script>

<Accordion>
	<AccordionItem
		title="Item A"
		icon="👈️"
		iconPosition="left"
		...
	>
		Content
	</AccordionItem>
</Accordion>
```

That's not a way to live your life! Instead, you can use [inversion of control](https://en.wikipedia.org/wiki/Inversion_of_control) so the user can render the accordion item however they want.

Let's modify the `<AccordionItem>` component to accept an `accordionItem` snippet as a prop instead, and pass it the `open` state and `toggle` function so we have access to them inside the snippet:

```svelte:AccordionItem.svelte
<script lang="ts">
	let { accordionItem } = $props()

	function toggle() {
		open = !open
	}
</script>

<div class="accordion-item">
	{@render accordionItem?.({ open, toggle })}
</div>
```

Snippets are just functions! You can define and render a snippet in your component for markup reuse, or delegate the rendering to another component like `<AccordionItem>` by passing it as a prop:

```svelte:App.svelte
<script>
	import { slide } from 'svelte/transition'
	import { Accordion, AccordionItem } from './accordion'
</script>

{#snippet accordionItem({ open, toggle })}
	<button onclick={toggle} class="accordion-heading">
		<div>Item A</div>
		<div class"accordion-icon">👈️</div>
	</button>

	{#if open}
		<div transition:slide class="accordion-content">
			Content
		</div>
	{/if}
{/snippet}

<Accordion>
	</AccordionItem {accordionItem}>
</Accordion>
```

If you use a snippet inside the component, it implicitly becomes a prop on the component for convenience:

```svelte:App.svelte
<script>
	import { slide } from 'svelte/transition'
	import { Accordion, AccordionItem } from './accordion'
</script>

<Accordion>
	<AccordionItem>
		<!-- the snippet becomes a prop -->
		{#snippet accordionItem({ open, toggle })}
			<button onclick={toggle} class="accordion-heading">
				<div>Item A</div>
				<div class"accordion-icon">👈️</div>
			</button>

			{#if open}
				<div transition:slide class="accordion-content">
					Content
				</div>
			{/if}
		{/snippet}
	</AccordionItem>
</Accordion>
```

This gives you complete control how the accordion item is rendered. I don't know about you, but that's really cool.

Alright, but what if you're asked to add a feature to let the user control the open and closed state of the accordion items? You might bind the `open` prop from the `<Accordion>` component and pass the prop which works but then you have to add another prop:

```svelte:App.svelte
<script>
	import { slide } from 'svelte/transition'
	import { Accordion, AccordionItem } from './accordion'

	let open = $state(false)
</script>

<button onclick={() => (open = !open)}>
	{open ? 'Close' : 'Open'}
</button>

<Accordion bind:open>
	<!-- tedious -->
	<AccordionItem {open} />
</Accordion>
```

How do we communicate this change from the `<Accordion>` component to its child components? You could "lift state up" and use props everywhere, or you could use the context API which was made to solve this problem. Under the hood, the context API is just a JavaScript [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) object that holds key-value pairs.

First you have to set the context in the parent component using `setContext` which accepts a key and a value:

```svelte:Accordion.svelte
<script lang="ts">
	import { setContext } from 'svelte'

	let { open = $bindable(), children } = $props()

	setContext('accordion', {
		get open() { return open }
	})
</script>

<div class="accordion">
	{@render children?.()}
</div>
```

Now you can use `getContext` in a child component to get the context value. Since `accordion.open` is a reactive value, we can change the `open` state to be a derived value which updates when `accordion.open` changes:

```svelte:AccordionItem.svelte
<script lang="ts">
	import { getContext } from 'svelte'

	let { accordionItem } = $props()

	const accordion = getContext('accordion')
	let open = $derived(accordion.open)

	function toggle() {
		open = !open
	}
</script>

<div>
	{@render accordionItem?.({ open, toggle })}
</div>
```

That's it! As you can see from the example, you can also store reactive state in context. Let's take a step back and explain this code because it's very important to understand:

```ts:example
// why this?
setContext('accordion', {
	get open() { return open }
})

// ...and not this?
setContext('accordion', { open })
```

When you're referencing state in Svelte, you're accessing the current value. The reason why passing the `open` state loses reactivity is because how JavaScript works. If you just pass the current value of `open` state, it's never going to update.

Let's say this is the context API:

```ts:example
const context = new Map()

function setContext(key, value) {
	context.set(key, value)
}

function getContext(key) {
	return context.get(key)
}
```

The value passed to context is not a reference to the `value` variable, but the `🍌` value itself. If `value` changes after the context is set, it won't update:

```ts:example
let emoji = '🍌'

setContext('ctx', { emoji })

const ctx = getContext('ctx')
console.log(ctx.emoji) // 🍌

emoji = '🍎'
console.log(ctx.emoji) // 🍌
```

Svelte doesn't change how JavaScript works — you need a mechanism which gets and returns the latest value:

```ts:example
let emoji = '🍌'

setContext('ctx', {
	getLatestValue() { return emoji }
})

const ctx = getContext('ctx')
console.log(ctx.getLatestValue()) // 🍌

emoji = '🍎'
console.log(ctx.getLatestValue()) // 🍎
```

I used a [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) because the syntax is nicer than invoking a function. To get the latest value, you just access `accordion.open`. That being said, you can use a **function**, **class**, **accessor**, or **proxied state** to get and set the value:

```ts:example
import { setContext } from 'svelte'

let emoji = $state('🍌')

// 👍 function
setContext('ctx', {
	getEmoji() { return emoji },
	updateEmoji(v) { emoji = v },
})

const ctx = getContext('ctx')
ctx.getEmoji()
ctx.updateEmoji('🍎')

// 👍 class
class Emoji {
	current = $state('🍌')
}
setContext('ctx', { emoji: new Emoji() })

const ctx = getContext('ctx')
ctx.emoji.current
ctx.emoji.current = '🍎'

// 👍 property accesors
setContext('ctx', {
	get emoji() { return emoji },
	set emoji(v) { emoji = v },
})

const ctx = getContext('ctx')
ctx.emoji
ctx.emoji = '🍎'

// 👍 proxied state
let emoji = $state({ current: '🍌'})
setContext('ctx', { emoji })

const ctx = getContext('ctx')
ctx.emoji.current
ctx.emoji.current = '🍎'
```

## Using Animations For Delightful User Interactions

In this section, I'm going to show you how you can use Svelte's built-in transitions and animations to create delightful user interactions.

### Transitions

To use a transition, you use the `transition:` directive on an element. Transitions play when the element is added to the DOM, and in reverse when the element is removed from the DOM.

This example uses the `fade` transition from Svelte to fade in and out two elements. The first element has a `duration` option of `600` milliseconds, and the second element has a `delay` option of `600` milliseconds:

```svelte:App.svelte
<script>
	import { fade } from 'svelte/transition'

	let play = $state(false)
</script>

<button onclick={() => (play = !play)}>Play</button>

{#if play}
	<div>
		<span transition:fade={{ duration: 600 }}>Hello</span>
		<span transition:fade={{ delay: 600 }}>World</span>
	</div>
{/if}
```

You can have separate intro and outro transitions using the `in:` and `out:` directives:

```svelte:App.svelte
<script>
	import { fade, fly } from 'svelte/transition'
	import { cubicInOut } from 'svelte/easing'

	let play = $state(false)
</script>

<button onclick={() => (play = !play)}>Play</button>

{#if play}
	<div class="flex gap-1">
		<span
			in:fly={{ x: -10, duration: 600, easing: cubicInOut }}
			out:fade
		>
			Hello
		</span>
		<span
			in:fly={{ x: 10, delay: 600, easing: cubicInOut }}
			out:fade
		>
			World
		</span>
	</div>
{/if}
```

Svelte also has a lot of [built-in easing functions](https://svelte.dev/docs/svelte/svelte-easing) you can use to make a transition feel more natural, or give it more character.

### Local And Global Transitions

Let's say you have an `each` block that renders a list of items using a staggered animation inside of an `if` block:

```svelte:problem
<script>
	import { fade } from 'svelte/transition'

	let play = $state(false)
</script>

{#if play}
	<div class="grid">
		{#each { length: 50 }, i}
			<div transition:fade={{ delay: i * 100 }}>{i + 1}</div>
		{/each}
	</div>
{/if}
```

It doesn't work! Why?

**Transitions are local by default** which means they only play when the block they belong to is added or removed from the DOM, and not the parent block unless you use the `global` modifier:

```svelte:solution
<div transition:fade|global={{ delay: i * 100 }}>{i + 1}</div>
```

In older version of Svelte, transitions were global by default for historical reasons. Keep that in mind if you come across some old Svelte code.

### Playing Transitions Immediately

You might have noticed that transitions don't play immediately when you open a page.

If you want that behavior, you can create a component with an effect to trigger the transition when it's added to the DOM:

```svelte:Fade.svelte
<script>
	let { children, options } = $props()
	let play = $state(false)

	$effect(() => {
		play = true
	})
</script>

{#if play}
	<div transition:fade={options}>
		{@render children?.()}
	</div>
{/if}
```

Now you can use the `<Fade>` component in your app:

```svelte:Example.svelte
<script>
	import { Fade } from './transitions'
</script>

<Fade options={{ duration: 2000 }}>
	Yo, time to fade!
</Fade>
```

You could also create a more general `<Transition>` component that accepts a prop for the transition you want to use like `<Transition type="fade">` and conditionally that type of transition.

### Custom Transitions

You can find more built-in transitions in the [Svelte documentation](https://svelte.dev/docs/svelte/svelte-transition). If that isn't enough, you can also create custom transitions.

Custom transitions are regular function which have to return an object with the transition options and a `css`, or `tick` function:

```svelte:App.svelte
<script>
	import { elasticOut } from 'svelte/easing'

	function customTransition(node, options) {
		return {
			delay: options.delay || 0,
			duration: options.duration || 2000,
			easing: options.easing || elasticOut,
			css: (t) => `
				color: hsl(${360 * t} , 100%, 80%);
				transform: scale(${t});
			`
		}
	}

	let play = $state(false)
</script>

<button onclick={() => (play = !play)}>Play</button>

{#if play}
	<div in:customTransition>Whoooo!</div>
{/if}
```

You should always return a `css` function, because Svelte is going to create keyframes using the [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) which is always more performant.

The `t` argument is the transition progress from `0` to `1` after the easing has been applied. If you have a transition that lasts `2` seconds, where you move an item from `0` pixels to `100` pixels, it's going to start from `0` pixels and end at `100` pixels.

You can reverse the transition by using the `u` argument which is the transition progress from `1` to `0`. If you have a transition that lasts `2` seconds, where you move an item from `100` pixels to `0` pixels, it's going to start from `100` pixels and end at `0` pixels.

Alternatively, you can return a `tick` function when you need to use JavaScript for transition and Svelte is going to use the [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) API:

```svelte:App.svelte
<script>
	const chars = '!@#$%&*1234567890-=_+[]{}|;:,.<>/?'

	function getRandomCharacter() {
		return chars[Math.floor(Math.random() * chars.length)]
	}

	function scrambleText(node, options) {
		const finalText = node.textContent
		const length = finalText.length

		return {
			duration: options.duration || 2000,
			tick: (t) => {
				let output = ''
				for (let i = 0; i < length; i++) {
					if (t > i / length) {
						output += finalText[i]
					} else {
						output += getRandomCharacter()
					}
				}
				node.textContent = output
			}
		}
	}

	let play = $state(false)
</script>

<button onclick={() => (play = !play)}>Scramble text</button>

{#if play}
	<p transition:scrambleText>Scrambling Text Effect</p>
{/if}
```

You would of course define these custom transitions using whichever method you prefer in a separate file and import them in your app.

### Deferred Transitions

### Flip Animations

### Tween And Spring Animations

## Todo

- Universal reactivity
- Using third party libraries in Svelte
- Module context
- Special elements
- Deployment
