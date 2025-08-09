---
title: The Complete Svelte 5 Guide
description: The ultimate guide for the most beloved JavaScript framework.
slug: learn-svelte
published: '2025-7-14'
category: svelte
---

<script lang="ts">
	import Card from '$lib/components/card.svelte'
	import YouTube from '$lib/components/youtube.svelte'
</script>

## Table of Contents

## What is Svelte?

If we look at the definition from the [Svelte](https://svelte.dev/) website, it says:

> Svelte is a UI framework that uses a compiler to let you write breathtakingly concise components that do minimal work in the browser, using languages you already know — HTML, CSS and JavaScript.

Because Svelte is a compiled language, it can wield the same syntax of a language that's not great at making user interfaces like JavaScript and change the semantics for a better developer experience:

```svelte:App.svelte
<script lang="ts">
	// reactive state
	let count = $state(0)

	// reassignment updates the UI
	setInterval(() => count += 1, 1000)
</script>

<p>{count}</p>
```

You might think how Svelte does some crazy compiler stuff under the hood to make this work, but the output is human readable JavaScript:

```ts:output
function App($$anchor) {
	// create signal
	let count = state(0)

	// update signal
	setInterval(() => set(count, get(count) + 1), 1000)

	// create element
	var p = from_html(`<p> </p>`)
	var text = child(p, true)

	// update DOM when `count` changes
	template_effect(() => set_text(text, get(count)))

	// add to DOM
	append($$anchor, p)
}
```

In fact, Svelte's reactivity is just based on [signals](https://www.youtube.com/watch?v=1TSLEzNzGQM)! There's nothing magical about it. You could write a basic version of Svelte by hand without using a compiler.

<!-- <YouTube id="1TSLEzNzGQM" title="Signals" /> -->

Just by reading the output code, you can start to understand how Svelte works. There's no virtual DOM, or rerendering the component when state updates like in React — Svelte only updates the part of the DOM that changed.

This is what "does minimal work in the browser" means!

Svelte also has a more opinionated application framework called [SvelteKit](https://svelte.dev/docs/kit/introduction) (equivalent to [Next.js](https://nextjs.org/) for React) if you need routing, server-side rendering, adapters to deploy to different platforms and so on.

## Try Svelte

You can try Svelte in the browser using the [Svelte Playground](https://svelte.dev/playground) and follow along without having to set up anything.

<Card type="warning">
	Some of the examples use browser APIs like <code>localStorage</code> that aren't supported in the Svelte Playground.
</Card>

If you're a creature of comfort and prefer your development environment, you can scaffold a Vite project and pick Svelte as the option from the CLI if you run `npm create vite@latest` in a terminal — you're going to need [Node.js](https://nodejs.org/) for that.

I also recommend using the [Svelte for VS Code extension](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode) for syntax highlighting and code completion, or a similar extension for your editor.

## TypeScript Aside

[TypeScript](https://www.typescriptlang.org/) has become table stakes when it comes to frontend development. For that reason, the examples are going to use TypeScript, but you can use JavaScript if you prefer.

If you're unfamiliar with TypeScript, code after `:` usually represents a type. You can omit the types and your code will work:

```ts:example
// TypeScript 👍️
let items: string[] = [...]

// JavaScript 👍️
let items = [...]
```

Some developers prefer writing JavaScript with [JSDoc](https://jsdoc.app/) comments because it gives you the same benefits of TypeScript at the cost of a more verbose syntax:

```ts:example
/**
 * This is a list of items.
 * @type {string[]}
 */
let items = [...]
```

That is completely up to you!

## Single File Components

In Svelte, files ending with `.svelte` are called **single file components** because they contain the JavaScript, HTML, and CSS in a single file.

Here's an example of a Svelte component:

```svelte:App.svelte
<!-- logic -->
<script lang="ts">
	let title = 'Svelte'
</script>

<!-- markup -->
<h1>{title}</h1>

<!-- styles -->
<style>
	h1 {
		color: orangered;
	}
</style>
```

A Svelte component can only have one top-level `<script>` and `<style>` block and is unique for every component instance. A code formatter like Prettier might arrange the blocks for you, but **the order of the blocks doesn't matter**.

There's also a special `<script module>` block used for sharing code across component instances we'll learn about later.

## Component Logic

Your component logic goes inside the `<script>` tag. Since Svelte 5, TypeScript is [natively supported](https://svelte.dev/docs/kit/integrations):

```svelte:App.svelte
<script lang="ts">
	let title = 'Svelte'
</script>

<h1>{title as string}</h1>
```

Later we're going to learn how you can even define values inside your markup which can be helpful in some cases.

## Markup Poetry

In Svelte, anything that's outside the `<script>` and `<style>` blocks is considered markup:

```svelte:App.svelte
<!-- markup -->
<h1>Svelte</h1>
```

You can use JavaScript expressions in the template using curly braces and Svelte is going to evalute it:

```svelte:App.svelte
<script>
	let banana = 1
</script>

<p>There's {banana} {banana === 1 ? 'banana' : 'bananas'} left</p>
```

Later we're going to learn about logic blocks like `if` and `each` to conditionally render content.

Tags with lowercase names are treated like regular HTML elements by Svelte and accept normal attributes:

```svelte:App.svelte
<img src="dance.gif" alt="Person dancing" />
```

You can pass values to attributes using curly braces:

```svelte:App.svelte
<script lang="ts">
	let src = 'dance.gif'
	let alt = 'Person dancing'
</script>

<img src={src} alt={alt} />
```

If the attribute name and value are the same, you can use a shorthand attribute:

```svelte:App.svelte
<!-- 👍️ longhand -->
<img src={src} alt={alt} />

<!-- 👍️ shorthand -->
<img {src} {alt} />
```

Attributes can have expressions inside the curly braces:

```svelte:App.svelte
<script lang="ts">
	let src = 'dance.gif'
	let alt = 'Person dancing'
	let lazy = true
</script>

<img src={src} alt={alt} loading={lazy ? 'lazy' : 'eager'} />
```

If you want to conditionally render attributes, don't use `&&` for short-circuit evaluation or empty strings. Instead, use `null` or `undefined` as the value:

```svelte:App.svelte
<script lang="ts">
	let src = 'dance.gif'
	let alt = 'Person dancing'
	let lazy = false
</script>

<!-- ⛔️ -->
<img src={src} alt={alt} loading={lazy && 'lazy'} />
<img src={src} alt={alt} loading={lazy ? 'lazy' : ''} />

<!-- 👍 -->
<img src={src} alt={alt} loading={lazy ? 'lazy' : null} />
<img src={src} alt={alt} loading={lazy ? 'lazy' : undefined} />
```

You can spread attributes on elements:

```svelte:App.svelte
<script lang="ts">
	let obj = {
		src: 'dance.gif',
		alt: 'Person dancing'
	}
</script>

<img {...obj} />
```

## Component Styles

There are many ways you can style a Svelte component. 💅 I've heard people love inline styles with [Tailwind CSS](https://tailwindcss.com/), so you could just use the `style` tag...I'm joking! 😄

That being said, the `style` tag can be useful. You can use the `style` attribute like in regular HTML, but Svelte also has a shorthand `style:` directive you can use. The only thing you can't pass is an object:

```svelte:App.svelte
<script lang="ts">
	let color = 'orangered'
</script>

<!-- 👍️ attribute -->
<h1 style="color: {color}">Banana</h1>

<!-- 👍️ directive -->
<h1 style:color>Banana</h1>

<!-- ⛔️ object -->
<h1 style={{ color }}>Banana</h1>
```

You can even add `important` like `style:color|important` to override styles. The `style:` directive is also great for CSS custom properties:

```svelte:App.svelte
<script lang="ts">
	let color = 'orangered'
</script>

<!-- 👍️ custom CSS property -->
<h1 style="--color: {color}">Svelte</h1>

<!-- 👍️ shorthand -->
<h1 style:--color={color}>Svelte</h1>

<style>
	h1 {
		/* custom CSS property with a default value */
		color: var(--color, #fff);
	}
</style>
```

### Scoped Styles

Fortunately, you're not stuck using the `style` attribute. Most of the time, you're going to use the `style` block to define styles in your component. Those styles are scoped to the component by default:

```svelte:App.svelte
<h1>Svelte</h1>

<!-- these styles only apply to this component -->
<style>
	h1 {
		color: orangered;
	}
</style>
```

Scoped styles are unique to that component and don't affect styles in other components. If you're using the Svelte playground, you can open the CSS output tab to view the generated CSS:

```css:output
/* uniquely generated class name */
h1.svelte-ep2x9j {
	color: orangered;
}
```

If you want to define global styles for your app, you can import a CSS stylesheet at the root of your app:

```ts:main.ts {4}
// inside a Vite project
import { mount } from 'svelte'
import App from './App.svelte'
import './app.css'

const app = mount(App, {
  target: document.getElementById('app')!
})

export default app
```

You can also define global styles in components. This is useful if you have content from a content management system (CMS) that you have no control over.

Svelte has to "see" the styles in the component, so it doesn't know they exist and warns you about removing unusued styles:

```svelte:App.svelte {15-17,20-22}
<script lang="ts">
	let content = `
		<h1>Big Banana Exposed</h1>
		<p>The gorillas inside the banana cartel speak out</p>
	`
</script>

<div class="content">
	{@html content}
</div>

<style>
	.content {
		/* ⚠️ Unused CSS selector "h1" */
		h1 {
			font-size: 48px;
		}

		/* ⚠️ Unused CSS selector "p" */
		p {
			font-size: 20px;
		}
	}
</style>
```

In that case, you can make the styles global by using the `:global(selector)` modifier:

```svelte:App.svelte {4-6,8-10}
<!-- ... -->
<style>
	.content {
		:global(h1) {
			font-size: 48px;
		}

		:global(p) {
			font-size: 20px;
		}
	}
</style>
```

Having to use `:global` on every selector is tedious! Thankfully, you can nest global styles inside a `:global { ... }` block:

```svelte:App.svelte {3}
<!-- ... -->
<style>
	:global {
		.prose {
			h1 {
				font-size: 48px;
			}

			p {
				font-size: 20px;
			}
		}
	}
</style>
```

You can also have "global scoped styles" where the styles inside the `:global` block are scoped to the class:

```svelte:App.svelte {3}
<!-- ... -->
<style>
	.prose :global {
		h1 {
			font-size: 48px;
		}

		p {
			font-size: 20px;
		}
	}
</style>
```

Here's the compiled CSS output:

```css:output
.prose.svelte-ju1yn8 {
	h1 {
		font-size: 48px;
	}

	p {
		font-size: 20px;
	}
}
```

You can use different [preprocessors](https://svelte.dev/docs/kit/integrations#vitePreprocess) like [PostCSS](https://postcss.org/) or [SCSS](https://sass-lang.com/) by simply adding the `lang` attribute to the `<style>` tag with the preprocessor you want to use:

```svelte:example
<style lang="postcss">
	<!-- ... -->
</style>

<style lang="scss">
	<!-- ... -->
</style>
```

These days you probably don't need SCSS anymore, since a lot of features such as nesting and CSS variables are supported by CSS.

### Dynamic Classes

You can use an expression to apply a dynamic class, but it's tedious and easy to make mistakes:

```svelte:App.svelte {2,5,11-13}
<script lang="ts">
	let open = false
</script>

<div class="trigger {open ? 'open' : ''}">👈️</div>

<style>
	.trigger {
		transition: all 0.2s ease;

		&.open {
			rotate: -90deg;
		}
	}
</style>
```

Thankfully, Svelte can helps us out here. You can use the `class:` directive to conditionally apply a class:

```svelte:App.svelte
<div class="arrow" class:open>👈️</div>
```

You can also pass an object, array, or both to the `class` attribute and Svelte is going to use [clsx](https://github.com/lukeed/clsx) under the hood to merge the classes:

```svelte:App.svelte
<!-- 👍️ passing an object -->
<div class={{ trigger: true, open }}>👈️</div>

<!-- 👍️ passing an array -->
<div class={['trigger', open && 'open']}>👈️</div>

<!-- 👍️ passing an array and object -->
<div class={['trigger', { open }]}>👈️</div>
```

If you're using Tailwind, this is very useful when you need to apply a bunch of classes:

```svelte:App.svelte
<div class={['transition-all', { '-rotate-90': open }]}>👈️</div>
```

You should also consider using [data attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/How_to/Use_data_attributes) to make the state more explicit instead of using a bunch of classes:

```svelte:App.svelte {2,5,11-13,15-17}
<script lang="ts">
	let status = 'closed'
</script>

<div class="trigger" data-status={status}>👈️</div>

<style>
	.trigger {
		transition: all 2s ease;

		&[data-status="open"] {
			rotate: -90deg;
		}

		&[data-status="closed"] {
			rotate: 0deg;
		}
	}
</style>
```

## Svelte Reactivity

What is state?

In the context of JavaScript frameworks, **application state** refers to values that are essential to your application working and cause the framework to update the UI when changed.

Let's look at a counter example:

```svelte:App.svelte
<!-- only required for this example because of legacy mode -->
<svelte:options runes={true} />

<script lang="ts">
	let count = 0
</script>

<button onclick={() => count += 1}>
	{count}
</button>
```

The Svelte compiler knows that you're trying to update the `count` value and warns you because it's not reactive:

> `count` is updated, but is not declared with `$state(...)`. Changing its value will not correctly trigger updates.

This brings us to our first Svelte rune — the `$state` rune.

## Reactive State

The `$state` rune marks a variable as reactive. Svelte's reactivity is based on **assignments**. To update the UI, you just assign a new value to a reactive variable:

```svelte:App.svelte {3,7}
<script lang="ts">
	// reactive value
	let count = $state(0)
</script>

<!-- reactive assignment -->
<button onclick={() => count += 1}>
	{count}
</button>
```

You can open the developer tools and see that Svelte only updated the part of the DOM that changed.

<Card type="info">
	I used <code>count += 1</code> to emphasize assignment, but you can use <code>count++</code> to increment the value.
</Card>

The `$state(...)` syntax is called a **rune** and is part of the language. It looks like a function, but it's only a hint to Svelte what to do with it. This means as far as TypeScript is concerned, it's just a function:

```ts:example
let value = $state<Type>(...)
```

The three main runes we're going to learn about are the `$state`, `$derived`, and `$effect` rune.

## Deeply Reactive State

If you pass an array, or object to `$state` it becomes a deeply reactive [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy). This lets Svelte perform granular updates when you read or write properties and avoids mutating the state directly.

For example, changing `editor.content` is going to update the UI in every place where `editor.content` is used:

```svelte:App.svelte {2-5,10,13}
<script lang="ts">
	let editor = $state({
		theme: 'dark',
		content: '<h1>Svelte</h1>'
	})
</script>

<textarea
	value={editor.content}
	oninput={e => editor.content = e.target.value}
></textarea>

{@html editor.content}

<style>
	textarea {
		width: 100%;
		height: 200px;
	}
</style>
```

You might not want deeply reactive state where pushing to an array or updating the object would cause an update. In that case, you can use `$state.raw` so state only updates when you reassign it:

```svelte:App.svelte {3-6,13,16-19}
<script lang="ts">
	// this could be a complex object
	let editor = $state.raw({
		theme: 'dark',
		content: '<h1>Svelte</h1>'
	})
</script>

<textarea
	value={editor.content}
	oninput={e => {
		// ⛔️ can't be mutated
		editor.content = e.target.value

		// 👍️ reassignment
		editor = {
			...editor,
			content: e.target.value
		}
	}}
></textarea>

{@html editor.content}

<style>
	textarea {
		width: 100%;
		height: 200px;
	}
</style>
```

Because proxied state is deeply reactive, you could change it on accident when you pass it around, or run into a problem with some API that doesn't expect it. In that case, you can use `$state.snapshot` to get the normal value from the Proxy:

```ts:editor.ts
function saveEditorState(editor) {
	// 💣️ oops! it doesn't like a Proxy object...
	const editorState = structuredClone(editor)
	// 👍️ normal object
	const editorState = structuredClone($state.snapshot(editor))
}
```

<Card type="info">
	Svelte uses <code>$state.snapshot</code> when you <code>console.log</code> deeply reactive values for convenience.
</Card>

You should also be aware that destructuring state loses reactivity because it's just JavaScript, so the values are evaluated when you destructure them:

```svelte:App.svelte {8}
<script lang="ts">
	let editor = $state({
		theme: 'dark',
		content: '<h1>Svelte</h1>'
	})

	// ⛔️ not reactive
	let { theme, content } = editor
</script>

{@html content}
```

If you want to do this, you can use derived state!

## Derived State

You can derive state from other state using the `$derived` rune and it's going to reactively update:

```svelte:App.svelte {4}
<script lang="ts">
	let count = $state(0)
	let factor = $state(2)
	let result = $derived(count * factor)
</script>

<button onclick={() => count++}>Count: {count}</button>
<button onclick={() => factor++}>Factor: {factor}</button>

<p>{count} * {factor} = {result}</p>
```

Derived values **only run when they're read** and are **lazy evaluted** which means they only update when they change and not when their dependencies change to avoid unnecessary work.

Even if `max` depends on `count`, it only updates when `max` updates instead of `count`:

```svelte:App.svelte {3,6,9}
<script lang="ts">
	let count = $state(0)
	let max = $derived(count >= 4)

	// only logs when `max` changes
	$inspect(max)
</script>

<button onclick={() => count++} disabled={max}>
	{count}
</button>
```

<Card type="info">
	The <code>$inspect</code> rune only runs in development and is great for seeing state updates and debugging.
</Card>

You can pass a function to a derived without losing reactivity, because a signal only has to be read to become a dependency:

```svelte:App.svelte {3,5-7}
<script lang="ts">
	let count = $state(0)
	let max = $derived(limit())

	function limit() {
		return count > 4 // 📖
	}
</script>

<button onclick={() => count++} disabled={max}>
	{count}
</button>
```

This might sound like magic, but the only magic here is the system of signals and runtime reactivity! 🪄

The reason you don't have to pass state to the function — unless you want to be explicit — is because signals only care where they're read, as highlighted in the compiled output:

```ts:output {5,9}
// not passing state
let disabled = derived(limit)

function limit() {
	return get(count) > 4 // 📖
}

// passing state
let disabled = derived(() => limit(get(count))) // 📖

function limit(count) {
	return count > 4
}
```

The `$derived` rune only accepts an expression by default, but you can use the `$derived.by` rune if you want to pass a function for a more complex derivation:

```svelte:App.svelte {6-12}
<script lang="ts">
	let cart = $state([
		{ item: '🍎', total: 10 },
		{ item: '🍌', total: 10 }
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

Svelte recommends you keep deriveds free of side-effects. You can't update state inside of deriveds to protect you from unintended side-effects:

```svelte:App.svelte {5}
<script lang="ts">
	let count = $state(0)
	let double = $derived.by(() => {
		// ⛔️ error
		count++
	})
</script>
```

Going back to a previous example, you can also use derived state to keep reactivity when using destructuring:

```svelte:App.svelte {8,11}
<script lang="ts">
	let editor = $state({
		theme: 'dark',
		content: '<h1>Svelte</h1>'
	})

	// ⛔️ not reactive
	let { theme, content } = editor

	// 👍️ reactive
	let { theme, content } = $derived(editor)
</script>

{@html content}
```

## Effects

The last main rune you should know about is the `$effect` rune.

Effects are functions that run when the component is added to the DOM and when their dependencies change. State that is **read** inside of an effect will be tracked:

```svelte:App.svelte {2,6}
<script lang="ts">
	let count = $state(0)

	$effect(() => {
		// 🕵️ tracked
		console.log(count)
	})
</script>

<button onclick={() => count++}>Click</button>
```

**Values are only tracked inside of the effect if they're read.** If `condition` is `true` in the example, then both `condition` and `count` are going to be tracked. If `condition` is false, then the effect is only going to rerun when `condition` changes:

```svelte:App.svelte {3,6-8}
<script lang="ts">
	let count = $state(0)
	let condition = $state(false)

	$effect(() => {
		if (condition) {
			console.log(count) // 📖
		}
	})
</script>

<button onclick={() => condition = !condition}>Toggle</button>
<button onclick={() => count++}>Click</button>
```

<Card type="info">
	Use the <a href="https://svelte.dev/docs/svelte/$inspect" target="_blank">$inspect</a> rune instead of effects to log when a reactive value updates.
</Card>

Svelte provides an `untrack` function if you don't want to track the state:

```svelte:App.svelte {2,9}
<script lang="ts">
	import { untrack } from 'svelte'

	let a = $state(0)
	let b = $state(0)

	$effect(() => {
		// ⛔️ only runs when `b` changes
		console.log(untrack(() => a) + b)
	})
</script>

<button onclick={() => a++}>A</button>
<button onclick={() => b++}>B</button>
```

You can return a function from the effect callback, which reruns when the effect **dependencies change**, or when the component is **removed** from the DOM:

```svelte:App.svelte {9}
<script lang="ts">
	let count = $state(0)
	let delay = $state(1000)

	$effect(() => {
		// 🕵️ only `delay` is tracked
		const interval = setInterval(() => count++, delay)
		// 🧹 clear interval every update
		return () => clearInterval(interval)
	})
</script>

<button onclick={() => delay *= 2}>+</button>
<span>{count}</span>
<button onclick={() => delay /= 2}>-</button>
```

<Card type="warning">
	Values that are read <b>asynchronously</b> inside promises and timers are <b>not tracked</b> inside effects.
</Card>

When it comes to deeply reactive state, effects only rerun when the object it reads changes and not its properties:

```svelte:App.svelte {6,11}
<script lang="ts">
	let obj = $state({ current: 0 })

	$effect(() => {
		// doesn't run if property changes
		console.log(obj)
	})

	$effect(() => {
		// you have to track the property
		console.log(obj.current)
	})
</script>
```

There are ways around it though! 🤫

You can use `JSON.stringify`, `$state.snapshot`, or the `$inspect` rune to react when the object properties change. The `save` function could be some external API used to save the data:

```svelte:App.svelte {5,10,15}
<script lang="ts">
	let obj = $state({ current: 0 })

	$effect(() => {
		JSON.stringify(obj) // 👍️ tracked
		save(obj)
	})

	$effect(() => {
		$state.snapshot(obj) // 👍️ tracked
		save(obj)
	})

	$effect(() => {
		$inspect(obj) // 👍️ tracked
		save(obj)
	})
</script>
```

**Don't use effects to synchronize state**. Svelte queues your effects and runs them last. Using effects to synchronize state can cause unexpected behaviors like state being out of sync:

```svelte:App.svelte {7,12-13}
<script lang="ts">
	let count = $state(0)
	let double = $state(0)

	$effect(() => {
		// effects are queued and run last
		double = count * 2
	})
</script>

<button onclick={() => {
	count++ // 1
	console.log(double) // ⚠️ 0
}}>
	{double}
</button>
```

**Always derive state** when you can instead:

```svelte:App.svelte {3,7-8}
<script lang="ts">
	let count = $state(0)
	let double = $derived(count * 2)
</script>

<button onclick={() => {
	count++ // 1
	console.log(double) // 👍️ 2
}}>
	{double}
</button>
```

<Card type="info">
	Derived values are effects under the hood, but they rerun immediately when their dependencies change.
</Card>

**Effects should be a last resort** when you have to synchronize with an external system that doesn't understand Svelte's reactivity. You should only use them for side-effects like fetching data from an API, or working with the DOM directly:

```svelte:App.svelte {17-21}
<script lang="ts">
	import { getAbortSignal } from 'svelte'

	let pokemon = $state('charizard')
	let image = $state('')

	async function getPokemon(pokemon: string) {
		const baseUrl = 'https://pokeapi.co/api/v2/pokemon'
		const response = await fetch(`${baseUrl}/${pokemon}`, {
			// aborts when derived and effect reruns
			signal: getAbortSignal()
		})
		if (!response.ok) throw new Error('💣️ oops!')
		return response.json()
	}

	$effect(() => {
		getPokemon(pokemon).then(data => {
			image = data.sprites.front_default
		})
	})
</script>

<input
	oninput={e => pokemon = (e.target as HTMLInputElement).value}
	type="search"
/>
<img src={image} alt={pokemon} />
```

If you want to do something **once** when the component is added, you can use the `onMount` lifecycle function instead of an effect to do something when the component is addded (with an optional cleanup function):

```svelte:App.svelte
<script lang="ts">
	import { onMount } from 'svelte'

	onMount(() => {
		console.log('hi 👋')
		return () => console.log('🧹 cleanup')
	})
</script>
```

<Card type="warning">
	Avoid passing async callbacks to <code>onMount</code> and <code>$effect</code> as any cleanup function they have won't run. You can use async functions, or an <a href="https://developer.mozilla.org/en-US/docs/Glossary/IIFE" target="_blank">IIFE</a> inside them instead.
</Card>

Your effects run after the DOM updates in a [microtask](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide), but sometimes you might need to do work before the DOM updates like measuring an element, or scroll position.

A great example is the [GSAP Flip plugin](https://gsap.com/docs/v3/Plugins/Flip/) for animating view changes when you update the DOM. It needs to measure the position, size, and rotation of the element before and after the DOM update.

In that case, you can use the `$effect.pre` rune which runs before the DOM updates:

```svelte:App.svelte {10-20}
<script lang="ts">
	import { gsap } from 'gsap'
	import { Flip } from 'gsap/Flip'
	import { tick } from 'svelte'

	gsap.registerPlugin(Flip)

	let items = $state([...Array(20).keys()])

	$effect.pre(() => {
		// track `items` as a dependency
		items
		// record the element state before the DOM updates
		const state = Flip.getState('.item')
		// wait after the DOM updates
		tick().then(() => {
			// do the FLIP animation
			Flip.from(state, { duration: 1, stagger: 0.01, ease: 'power1.inOut' })
		})
	})

	function shuffle() {
		items = items.toSorted(() => Math.random() - 0.5)
	}
</script>

<div class="container">
	{#each items as item (item)}
		<div class="item">{item}</div>
	{/each}
</div>

<button onclick={shuffle}>Shuffle</button>

<style>
	.container {
		width: 600px;
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 0.5rem;
		color: orangered;
		font-size: 3rem;
		font-weight: 700;
		text-shadow: 2px 2px 0px #000;

		.item {
			display: grid;
			place-content: center;
			aspect-ratio: 1;
			background-color: #222;
			border: 1px solid #333;
			border-radius: 1rem;
		}
	}

	button {
		margin-top: 1rem;
		font-size: 2rem;
	}
</style>
```

`tick` is a useful lifecyle function that schedules a task to run in the next microtask when all the work is done, and before the DOM updates.

## State In Functions And Classes

So far, we only used state at the top-level of our components, but you can use state, deriveds, and effects inside functions and classes which can be used in your components.

If those functions and classes are declared inside of a file, you have to use the `.svelte.js` or `.svelte.ts` extension to tell Svelte that it's a special file and doesn't have to check every file for runes.

Here's an example of a `createCounter` function:

```ts:counter.svelte.ts
export function createCounter(initial: number) {
	let count = $state(initial)

	$effect(() => {
		console.log(count)
	})

	const increment = () => count++
	const decrement = () => count--

	return {
		get count() { return count },
		set count(v) { count = v },
		increment,
		decrement
	}
}
```

Here's how it's used inside of a Svelte component:

```svelte:App.svelte
<script lang="ts">
	import { createCounter } from './counter.svelte'

	const counter = createCounter(0)
</script>

<button onclick={counter.decrement}>-</button>
<span>{counter.count}</span>
<button onclick={counter.increment}>+</button>
```

You're probably wondering what's the deal with the `get` and `set` functions?

Those are called **getters and setters**, and they create **accessor properties** which let you define custom behavior when you read and write to a property using a cleaner syntax.

They're just part of JavaScript, and you could use functions instead:

```ts:counter.svelte.ts {8-9}
export function createCounter(initial: number) {
	let count = $state(initial)

	const increment = () => count++
	const decrement = () => count--

	return {
		count() { return count },
		setCount(v: number) { count = v },
		increment,
		decrement
	}
}
```

You could return a tuple `[count, setCount] = createCounter(0)` instead to make the API nicer using destructuring.

As you can see, the syntax is not as nice compared to using accessors, since you have to use functions everywhere:

```svelte:App.svelte {8-10,13-15}
<script lang="ts">
	import { createCounter } from './counter.svelte'

	const counter = createCounter(0)
</script>

<!-- using functions -->
<button onclick={() => counter.setCurrent(counter.count() + 1)}>
	{counter.count()}
</button>

<!-- using accessors -->
<button onclick={() => counter.count++}>
	{counter.count}
</button>
```

The accessor syntax looks a lot nicer! 😄 You might be wondering, can't you just return state from the function?

```ts:counter.svelte.ts
export function createCounter(initial: number) {
	let count = $state(initial)
	// ⛔️ this doesn't work
	return count
}
```

The reason this doesn't work is because **state is just a regular value**. It's **not** some magic reactive container. If you want something like that, you could return deeply reactive proxied state:

```ts:counter.svelte.ts
export function createCounter(initial: number) {
	let count = $state({ current: initial })
	// 👍️ proxied state
	return count
}
```

You could create a "magic" reactive container yourself like some signal-based frameworks do for you:

```ts:counter.svelte.ts {2-5,9}
// this could be a personal utility
export function reactive<T>(initial: T) {
	let value = $state<{ current: T }>({ current: initial })
	return value
}

export function createCounter(initial: number) {
	// reactive container
	let count = reactive(initial)

	const increment = () => count.current++
	const decrement = () => count.current--

	return { count, increment, decrement }
}
```

Even destructuring works, since `count` is not just a regular value:

```svelte:App.svelte {4}
<script lang="ts">
	import { createCounter } from './counter.svelte'

	const { count } = createCounter(0)
</script>

<button onclick={() => count.current++}>
	{count.current}
</button>
```

That seems super useful...so why doesn't Svelte provide this utility?

It's mostly because you can write one yourself in a couple of lines of code, but another reason is classes. If you use state inside classes, you get extra benefits which you can't get using functions.

Svelte turns any class fields declared with state into private fields with matching `get`/`set` methods, unless you declare them yourself:

```ts:counter.svelte.ts {4}
export class Counter {
	constructor(initial: number) {
		// turned into `get` and `set` methods
		this.count = $state(initial)
	}

	increment() {
		this.count++
	}

	decrement() {
		this.count--
	}
}
```

If you look at the output, you would see something like this:

```ts:output
class Counter {
	#count
	get count() { ... }
	set count(v) { ... }
}
```

There's only one gotcha with classes. Using methods like `counter.increment` inside `onclick` doesn't work, because `this` refers to the context where it ran, and here that's the `<button>` element:

```svelte:App.svelte
<script lang="ts">
	import { Counter } from './counter.svelte'

	const counter = new Counter(0)
</script>

<button onclick={counter.decrement}>-</button>
<span>{counter.current}</span>
<button onclick={counter.increment}>+</button>
```

You either have to use a function like `() => counter.increment()` or define the methods using arrow functions that don't bind to `this`:

```ts:counter.svelte.ts {6-8,10-12}
export class Counter {
	constructor(initial = 0) {
		this.current = $state(initial)
	}

	increment = () =>
		this.current++
	}

	decrement = () => {
		this.current--
	}
}
```

Now that you understand how state is a regular value, it also makes sense why you can't pass it to a function, or a class and expect it to be reactive.

In this example, we pass `count` to a `Doubler` class in hopes that it will double the value when `count` updates. However, it **doesn't** work because `count` is a regular value when it's evaluated:

```svelte:App.svelte {2-6,9}
<script lang="ts">
	class Doubler {
		constructor(count: number) {
			this.current = $derived(count * 2)
		}
	}

	let count = $state(0)
	const double = new Doubler(count) // 0
</script>

<button onclick={() => count++}>
	{double.current}
</button>
```

Svelte even gives you a warning with a hint:

> This reference only captures the initial value of `count`. Did you mean to reference it inside a closure instead?

The hint is that your value is never going to update because it's a regular value, so we can pass a function to get the latest value:

```svelte:App.svelte {3-5,9}
<script lang="ts">
	class Doubler {
		constructor(count: () => number) {
			this.value = $derived(count() * 2)
		}
	}

	let count = $state(0)
	const doubler = new Doubler(() => count)
</script>

<button onclick={() => count++}>
	{doubler.value}
</button>
```

Remember the `reactive` utility we talked about earlier? You could also use that! Let's use a class version this time:

```svelte:App.svelte {2-6,9-11,14-15}
<script lang="ts">
	class Reactive<T> {
		constructor(initial: T) {
			this.current = $state<T>(initial)
		}
	}

	class Doubler {
		constructor(count: Reactive<number>) {
			this.current = $derived(count.current * 2)
		}
	}

	const count = new Reactive(0)
	const double = new Doubler(count)
</script>

<button onclick={() => count.current++}>
	{double.current}
</button>
```

## Reactive Global State

Creating global reactive state in Svelte is simple as exporting deep state from a module, like a config which can be used across your app:

```ts:config.svelte.ts
interface Config {
	theme: 'light' | 'dark'
}

export const config = $state<Config>({ theme: 'dark' })

export function toggleTheme() {
	config.theme = config.theme === 'light' ? 'dark' : 'light'
}
```

```svelte:App.svelte
<script>
	import { config, toggleTheme } from './config.svelte'
</script>

<button onclick={toggleTheme}>
	{config.theme}
</button>
```

You could use a function, or a class for the config:

```ts:config.svelte.ts
type Themes = 'light' | 'dark'

class Config {
	theme = $state<Themes>('dark')

	toggleTheme() {
		this.theme = this.theme === 'light' ? 'dark' : 'light'
	}
}

export const config = new Config()
```

It doesn't matter if you prefer functions or classes. As long as you understand how state works, you can bend it to your will.

To understand it even more, let's learn how reactivity works in Svelte.

## How Svelte Reactivity Works

I believe that understanding how things work gives you greater enjoyment by being more competent at what you do.

I mentioned how Svelte uses signals for reactivity, but signals aren't unique to Svelte! You can find them in other frameworks like Angular, Solid, Vue, and Qwik. There's even a [proposal to add signals to JavaScript](https://github.com/tc39/proposal-signals) itself.

So far we learned that assignments cause updates in Svelte. There's nothing special about `=` though! It just creates a function call to update the value:

```svelte:example {3}
<script lang="ts">
	let value = $state('🍎')
	value = '🍌' // set(value, '🍌')
</script>

<!-- how does this get updated? -->
{value}
```

A signal is just a container that holds a value and subscribers that are notified when that value updates, so it doesn't do anything on its own:

```ts:example
function createSignal(value) {
	const signal = {
		value: null,
		subscribers: new Set(),
		// ...
	}
	return signal
}
```

**You need effects to react to signals** and effects are just functions that run when a signal changes.

In JavaScript frameworks that implement signals for reactivity like Svelte, everything is an effect! That's how Svelte is able to update the DOM when state changes:

```svelte:example
<!-- template_effect(() => set_text(text, get(value))) -->
{value}
```

Everything starts with a root effect — your component is just a nested effect inside of it, so Svelte can keep track of effects for cleanup. When the effect runs, it invokes the callback function and sets it as the active effect in some variable:

```ts:example
let effect = null

function template_effect(fn) {
	// set active effect
	effect = fn
}
```

The magic happens when you read a signal inside of an effect. When `value` is read, it adds the active effect as a subscriber:

```ts:example
let effect = fn

function get(signal) {
	// add effect to subscribers
	signal.subscribers.add(effect)
	// return value
	return signal.value
}
```

Later, when you write to `count` it notifies the subscribers and recreates the dependency graph when it reads the signal inside the effect:

```ts:example
function set(signal, value) {
	// update signal
	signal.value = value
	// notify subscribers
	signal.subscribers.forEach(effect => effect())
}
```

This is oversimplified, but it happens every update and that's why it's called **runtime reactivity**, because it happens as your code runs!

**Svelte doesn't compile reactivity**, it only compiles the implementation details. As far as you're concerned, state is a regular value. In other frameworks that implement signals, you have to read and write them using `value()` and `setValue()` — which is fine if you prefer no "magic".

Deriveds are also effects! That's how they're able to track dependencies. You can pass a function with state to a derived and it's tracked when it's read inside of an effect:

```svelte:example {7,14}
<script lang="ts">
	let value = $state('🍎')
	let code = $derived(getCode())

	function getCode() {
		// `value` is read inside derived effect
		return value.codePointAt(0).toString(16)
	}

	value = '🍌'
</script>

<!-- `code` is read inside template effect -->
{code}
```

I want to emphasize how `$state` is not some magic reactive container, but a regular value; which is why you need a function or a getter to get the latest value when the effect reruns — unless you're using deep state.

If `emoji.code` was a regular value and not a getter, then `() => set_text(text, emoji.code)` would always return the same value, even though it reacts to the change:

```svelte:example {5-6,15}
<script lang="ts">
	class Emoji {
		constructor(emoji: string) {
			// turned into `get` and `set` methods
			this.current = $state(emoji)
			this.code = $derived(this.current.codePointAt(0).toString(16))
		}
	}

	const emoji = new Emoji('🍎')
	emoji.current = '🍌'
</script>

<!-- template_effect(() => set_text(text, emoji.code)) -->
{emoji.code}
```

As the React people love to say, "it's just JavaScript!" 😄

## Why You Should Avoid Effects

Honestly, it's not the end of the world if you **sometimes** use effects when you shouldn't.

The problem is that you can easily overcomplicate your code with effects, when you could just do a side-effect inside an event handler.

In this example, I have a `counter` value that I want to read and write to `localStorage`. That's a side-effect, so using an effect makes sense:

```ts:counter.svelte.ts
class Counter {
	constructor(initial: number) {
		this.count = $state(initial)

		$effect(() => {
			const savedCount = localStorage.getItem('count')
			if (savedCount) this.count = parseInt(savedCount)
		})

		$effect(() => {
			localStorage.setItem('count', this.count.toString())
		})
	}
}
```

There's nothing wrong with this approach. The problem arises if you want to create your favorite counter inside `counter.svelte.ts` to share it with the world:

```ts:counter.svelte.ts
// ...
export const counter = new Counter(10)
```

Oops! Immediately, there's an error:

> effect_orphan `$effect` can only be used inside an effect (e.g. during component initialisation)

In the previous section we learned that everything starts with a root effect, so Svelte can run the teardown logic for nested effects when the component is removed.

In this case, you're trying to create an effect outside that root effect, which is not allowed.

Svelte provides an advanced `$effect.root` to create your own root effect, but now you have to run the cleanup manually:

```ts:counter.svelte.ts
class Counter {
	#cleanup

	constructor(initial: number) {
		this.count = $state(initial)

		// manual cleanup 😮‍💨
		this.cleanup = $effect.root(() => {
			$effect(() => {
				const savedCount = localStorage.getItem('count')
				if (savedCount) this.count = parseInt(savedCount)
			})

			$effect(() => {
				localStorage.setItem('count', this.count.toString())
			})

			return () => console.log('🧹 cleanup')
		})
	}

	cleanup() {
		this.#cleanup()
	}
}
```

Then you learn about the `$effect.tracking` rune to know if you're inside a **tracking context** like the effect in your template, so maybe that's it:

```ts:counter.svelte.ts
class Counter {
	constructor(initial: number) {
		this.count = $state(initial)

		if ($effect.tracking()) {
			$effect(() => {
				const savedCount = localStorage.getItem('count')
				if (savedCount) this.count = parseInt(savedCount)
			})

			$effect(() => {
				localStorage.setItem('count', this.count.toString())
			})
		}
	}
}
```

But there's **another** problem! The effect is never going to run when the counter is created because you're not inside a tracking context. 😩

Alright...how about we move the effects to where you read and write the value, inside of a tracking context like the template effect:

```ts:counter.svelte.ts {7-12,17}
export class Counter {
	constructor(initial: number) {
		this.#count = $state(initial)
	}

	get count() {
		if ($effect.tracking()) {
			$effect(() => {
				const savedCount = localStorage.getItem('count')
				if (savedCount) this.#count = parseInt(savedCount)
			})
		}
		return this.#count
	}

	set count(v: number) {
		localStorage.setItem('count', v.toString())
		this.#count = v
	}
}
```

There's **one more** problem though. Each time we read the value, we're creating an effect! 😱 Alright, that's a simple fix. We can use a variable to track if we already ran the effect:

```ts:counter.svelte.ts {2,11,14}
export class Counter {
	#first = true

	constructor(initial: number) {
		this.#count = $state(initial)
	}

	get count() {
		if ($effect.tracking()) {
			$effect(() => {
				if (!this.#first) return
				const savedCount = localStorage.getItem('count')
				if (savedCount) this.#count = parseInt(savedCount)
				this.#first = false
			})
		}
		return this.#count
	}

	set count(v: number) {
		localStorage.setItem('count', v.toString())
		this.#count = v
	}
}
```

Perfect! 😄 I know what you're thinking. **That's the point**. None of this is necessary. You can make everything simpler by **avoiding effects** and doing side-effects inside event handlers:

```ts:counter.svelte.ts
export class Counter {
	#first = true

	constructor(initial: number) {
		this.#count = $state(initial)
	}

	get count() {
		if (this.#first) {
			const savedCount = localStorage.getItem('count')
			if (savedCount) this.#count = parseInt(savedCount)
			this.#first = false
		}
		return this.#count
	}

	set count(v: number) {
		localStorage.setItem('count', v.toString())
		this.#count = v
	}
}
```

Now you can share your favorite counter with the world and you won't have any problems, unless it's a skill issue:

```svelte:App.svelte
<script lang="ts">
	import { counter } from './counter.svelte'
</script>

<button onclick={() => counter.count++}>
	{counter.count}
</button>
```

If you catch yourself using `$effect.root` or `$effect.tracking`, you're doing something wrong, unless you know what you're doing.

## Control Flow Blocks

There are no conditionals and loops in HTML, unless you're using a templating language.

### Using Conditionals

In Svelte, you can use the `#if` block to conditionally render content:

```svelte:App.svelte
<script>
	let user = $state({ authed: false })

	function toggle() {
		user.authed = !user.authed
	}
</script>

{#if user.authed}
  <button onclick={toggle}>Log out</button>
{:else}
	<button onclick={toggle}>Log in</button>
{/if}
```

### Looping Over Data

To loop over a list of items, you use the `#each` block:

```svelte:App.svelte
<script>
	let todos = $state([
		{ id: 1, text: 'Todo 1', done: true },
		{ id: 2, text: 'Todo 2', done: false },
		{ id: 3, text: 'Todo 3', done: false },
		{ id: 4, text: 'Todo 4', done: false },
	])
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

<Card type="info">
	The <code>else</code> clause is optional.
</Card>

You can [destructure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) the items values you're iterating over, get the current item index and provide a key, so Svelte can keep track of changes:

```svelte:App.svelte {2}
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
<div class="grid">
  {#each { length: 10 }, row}
    {#each { length: 10 }, col}
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

### Asynchronous Data Loading

In a previous example, we fetched some Pokemon data inside of an effect. That approach works, but we haven't handled any of the the error and success states which quickly becomes a mess.

Thankfully, Svelte has a built-in solution for async data loading using the `#await` block:

```svelte:App.svelte
<script lang="ts">
	async function getPokemon(pokemon: string) {
		let response = await fetch(`https://pokeapi.co/api/v2/pokemon/${pokemon}`)
		if (!response.ok) throw new Error('💣️ oops!')
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

### Asynchronous Svelte Aside

TODO: update this with recent changes in mind

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
<script lang="ts">
	import { getPokemon } from './api.ts'

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

### Recreating Elements

You can use the `key` block to recreate elements when state updates. This is useful for replaying transitions, which we're going to learn about later:

```svelte:App.svelte {4,7-9}
<script lang="ts">
	import { fade } from 'svelte/transition'

	let value = $state(0)
</script>

{#key value}
	<div in:fade>👻</div>
{/key}

<button onclick={() => value++}>Spook</button>
```

## Listening To Events

You can listen to DOM events by adding attributes that start with `on` to elements. In the case of a mouse click, you would add the `onclick` attribute to a `<button>`:

```svelte:App.svelte
<script lang="ts">
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
<script lang="ts">
	const events = {
		onclick: () => console.log('clicked'),
		ondblclick: () => console.log('double clicked')
	}
</script>

<button {...events}>Click</button>
```

Here's an example of using the `onmousemove` event to update the mouse position:

```svelte:App.svelte
<script lang="ts">
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

You can prevent the default behavior by using `e.preventDefault()`. This is useful for things like when you want to control a form with JavaScript and avoid a page reload:

```svelte:App.svelte
<script lang="ts">
	function onsubmit(e) {
		e.preventDefault()
		// sign up to newsletter...
	}
</script>

<form {onsubmit}>
	<input type="email" />
	<button type="submit">Sign up</button>
</form>
```

## Using Data Bindings

In JavaScript, it's common to listen for the user input on the `<input>` element through the `input` event and update a value using one-way data binding — which only updates the value from the UI — but what if you could keep the value and UI in sync?

### Two-Way Data Binding

Having to set `value={search}` and do `oninput={(e) => search = e.target.value}` on the `<input>` element to update `search` is mundane for something you do often:

```svelte:App.svelte {3,4,8,14}
<script>
	let list = $state(['angular', 'react', 'svelte', 'vue'])
	let filteredList = $derived(list.filter(item => item.includes(search)))
	let search = $state('')
</script>

<input
	oninput={(e) => search = (e.target as HTMLInputElement).value}
	value={search}
	type="search"
/>

<ul>
	{#each filteredList as item}
		<li>{item}</li>
	{/each}
</ul>
```

Thankfully, Svelte supports two-way data binding using the `bind:` directive. If you update the value, it updates the input and if you update the input, it updates the value:

```svelte:App.svelte
<input bind:value={search} type="search" />
<!-- ... -->
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

```svelte:App.svelte {3,10,14}
<script>
	// `undefined` until the component is added
	let canvas

	$effect(() => {
		// ⛔️ don't do this
		const canvas = document.querySelector('canvas')

		// 👍️ bind the value instead
		const ctx = canvas.getContext('2d')
	})
</script>

<canvas bind:this={canvas}></canvas>
```

### Function Bindings

Another useful thing to know about are **function bindings** if you need to validate some input, or link one value to another.

Let's say you want to make a [Mocking SpongeBob](https://knowyourmeme.com/memes/mocking-spongebob) case converter to transform the text as the user types:

```svelte:App.svelte
<script lang="ts">
	let text = $state('I love Svelte')

	function toSpongeBobCase(text: string) {
		return text
			.split('')
			.map((c) => (Math.random() > 0.5 ? c.toUpperCase() : c.toLowerCase()))
			.join('')
	}
</script>

<textarea
	value={toSpongeBobCase(text)}
	oninput={(e) => {
		text = toSpongeBobCase((e.target as HTMLInputElement).value)
	}}
></textarea>
```

This is a perfectly fine approach, but it could be simpler. Instead of passing an expression like `bind:value={expression}`, you can pass a function binding like `bind:property={get, set}` to have more control what happens when you read and write a value:

```svelte:App.svelte
<!-- ... -->
<textarea
	bind:value={
		() => toSpongeBobCase(text),
		(v: string) => text = toSpongeBobCase(v)
	}
></textarea>
```

### Readonly Bindings

Svelte provides a bunch of two-way bindings, and readonly bindings for different elements. I'm only going to demonstrate a couple of them, but you can find many more bindings in the [Svelte documentation for bind](https://svelte.dev/docs/svelte/bind).

This includes media bindings for `<audio>`, `<video>`, and `<img>` elements:

```svelte:App.svelte {3-5,9}
<script lang="ts">
	let clip = 'video.mp4'
	let currentTime = $state(0)
	let duration = $state(0)
	let paused = $state(true)
</script>

<div class="container">
	<video src={clip} bind:currentTime bind:duration bind:paused></video>

	<div class="controls">
		<button onclick={() => paused = !paused}>{paused ? 'Play' : 'Pause'}</button>
		<span>{currentTime.toFixed()}/{duration.toFixed()}</span>
		<input type="range" bind:value={currentTime} max={duration} />
	</div>
</div>

<style>
	.container {
	  max-width: 600px;

		video {
			width: 100%;
			border-radius: 0.5rem;
		}

		.controls {
			display: flex;
			gap: 0.5rem;

			input[type="range"] {
				flex-grow: 1;
			}
		}
	}
</style>
```

There are also readonly bindings for visible elements that use [ResizeObserver](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver) to measure dimension changes:

```svelte:App.svelte {2-3,6}
<script lang="ts">
	let width = $state()
	let height = $state()
</script>

<div class="container" bind:clientWidth={width} bind:clientHeight={height}>
	<div class="text" contenteditable>
		Edit this text
	</div>
	<div class="size">{width} x {height}</div>
</div>

<style>
	.container {
		position: relative;
		display: inline-block;
		padding: 0.5rem;
		border: 1px solid orangered;

		.text {
			font-size: 2rem;
		}

		.size {
			position: absolute;
			left: 50%;
			bottom: 0px;
			padding: 0.5rem;
			translate: -50% 100%;
			color: black;
			background-color: orangered;
			font-weight: 700;
			white-space: pre;
		}
	}
</style>
```

### Window And Document Bindings

### Component Bindings

In the next section we're going to learn about components and how we can also bind the properties we pass to them, making the data flow from child to parent.

## Svelte Components

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

First we'll create the component that handles adding a new todo. To receive and destructure props we use the `$props` rune. Here we bind the input value to the `todo` variable in the parent component, so we have to let Svelte know it's okay for the child to mutate the parent state by using the `$bindable` rune:

```svelte:AddTodo.svelte {2,6}
<script>
	let { todo = $bindable(), addTodo } = $props()
</script>

<form onsubmit={addTodo}>
	<input type="text" bind:value={todo} />
</form>
```

You can now bind the `todo` prop:

```svelte:Todos.svelte {6}
<script>
	import AddTodo from './AddTodo.svelte'
	// ...
</script>

<AddTodo bind:todo {addTodo} />
```

In reality, you don't have to do this. I just wanted to demonstrate how to use the `$bindable` rune if you have to. It makes more sense to move the `todo` state inside the component for adding todos:

```svelte:Todos.svelte
<script>
	function addTodo(todo) {
		todos.push({
			id: crypto.randomUUID(),
			text: todo,
			completed: false
		})
	}
</script>

<AddTodo {addTodo} />
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
<script>
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

```svelte:Todos.svelte {3,7}
<script>
	import AddTodo from './AddTodo.svelte'
	import TodoList from './TodoList.svelte'
</script>

<AddTodo {todo} {addTodo} />
<TodoList todos={filteredTodos} {removeTodo} />
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

```svelte:Todos.svelte {4,9}
<script>
	import AddTodo from './AddTodo.svelte'
	import TodoList from './TodoList.svelte'
	import TodoFilter from './TodoFilter.svelte'
</script>

<AddTodo {todo} {addTodo} />
<TodoList todos={filteredTodos} {removeTodo} />
<TodoFilter {remaining} {setFilter} {clearCompleted} />
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

```svelte:Todos.svelte {8}
<script>
	import AddTodo from './AddTodo.svelte'
	import TodoList from './TodoList.svelte'
	import TodoFilter from './TodoFilter.svelte'
</script>

<AddTodo {todo} {addTodo} />
<TodoList bind:todos={filteredTodos} {removeTodo} />
<TodoFilter {remaining} {setFilter} {clearCompleted} />
```

```svelte:TodoItem.svelte {4,10}
<script>
	import TodoItem from './TodoItem.svelte'

	let { todos = $bindable(), removeTodo } = $props()
</script>

<ul>
	{#each todos as todo, i (todo.id)}
		<li transition:slide>
			<TodoItem bind:todo={todos[i]} {removeTodo} />
		</li>
	{/each}
</ul>
```

In general, avoid mutating props to avoid unexpected state changes. If you want to update a value from a child component, callback props are a better option. Let's change the todos component to show you what I mean:

```svelte:Todos.svelte {3-11,14-17,19-22,25-27}
<script>
	function addTodo(e) {
		e.preventDefault()
		const form = e.currentTarget
		const formData = new FormData(form)
		todos.push({
			id: crypto.randomUUID(),
			text: formData.get('todo'),
			completed: false
		})
		form.reset()
	}

	function toggleTodo(todo: Todo) {
		const index = todos.findIndex((t) => t.id === todo.id)
		todos[index].completed = !todos[index].completed
	}

	function updateTodo(todo: Todo) {
		const index = todos.findIndex((t) => t.id === todo.id)
		todos[index].text = todo.text
	}
</script>

<AddTodo {addTodo} />
<TodoList todos={filteredTodos} {toggleTodo} {updateTodo} {removeTodo} />
<TodoFilter {remaining} {setFilter} {clearCompleted} />
```

Let's update the offending components to use callback props to update the todos instead of binding props everywhere, which could lead to unpredictable behavior:

```svelte:AddTodo.svelte {2,5}
<script>
	let { addTodo } = $props()
</script>

<form onsubmit={addTodo}>
	<input type="text" name="todo" />
</form>
```

```svelte:TodoList.svelte {4-9}
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

```svelte:TodoItem.svelte {4,10,15,18}
<script>
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

## Component Composition

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

```svelte:Accordion.svelte {6-11,13-14}
<script>
	let { children } = $props()
</script>

<div class="accordion">
	<!-- conditional with a fallback -->
	{#if children}
		{@render children()}
	{:else}
		<p>Fallback content</p>
	{/if}

	<!-- optional chaining -->
	{@render children?.()}
</div>
```

The `<AccordionItem>` accepts a `label` prop and we can show the accordion item content using the `children` prop which acts like a catch-all for any content inside the component:

```svelte:AccordionItem.svelte {2,13,19}
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

```svelte:App.svelte {7-10}
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

```svelte:AccordionItem.svelte {2,10}
<script>
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

```svelte:App.svelte {6-17,20}
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

```svelte:App.svelte {8-20}
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

Alright, but what if you're asked to add a feature to let the user control the open and closed state of the accordion items?

You might bind the `open` prop from the `<Accordion>` component and pass the prop which works but then you have to add another prop:

```svelte:App.svelte {5,12}
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

```svelte:Accordion.svelte {2,6-8}
<script>
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

```svelte:AccordionItem.svelte {2,6,7}
<script>
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

## Transitions And Animations

In this section, I'm going to show you how you can use Svelte's built-in transitions and animations to create delightful user interactions.

### Transitions

To use a transition, you use the `transition:` directive on an element. Transitions play when the element is added to the DOM, and in reverse when the element is removed from the DOM.

This example uses the `fade` transition from Svelte to fade in and out two elements. The first element has a `duration` option of `600` milliseconds, and the second element has a `delay` option of `600` milliseconds:

```svelte:App.svelte {2,11-12}
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

```svelte:App.svelte {2,13-14,19-20}
<script>
	import { fade, fly } from 'svelte/transition'
	import { cubicInOut } from 'svelte/easing'

	let play = $state(false)
</script>

<button onclick={() => (play = !play)}>Play</button>

{#if play}
	<div>
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

There's also a bunch of transition events you can listen to, including `introstart`, `introend`, `outrostart`, and `outroend`.

### Local And Global Transitions

Let's say you have an `each` block that renders a list of items using a staggered transition inside of an `if` block:

```svelte:problem {9-13}
<script>
	import { fade } from 'svelte/transition'

	let play = $state(false)
</script>

{#if play}
	<div class="grid">
		{#each { length: 50 }, i}
			<div transition:fade={{ delay: i * 100 }}>
				{i + 1}
			</div>
		{/each}
	</div>
{/if}
```

It doesn't work! Why?

**Transitions are local by default** which means they only play when the block they belong to is added or removed from the DOM, and not the parent block unless you use the `global` modifier:

```svelte:solution
<div transition:fade|global={{ delay: i * 100 }}>
	{i + 1}
</div>
```

In older version of Svelte, transitions were global by default for historical reasons. Keep that in mind if you come across some old Svelte code.

### Playing Transitions Immediately

You might have noticed that transitions don't play immediately when you open a page.

If you want that behavior, you can create a component with an effect to trigger the transition when it's added to the DOM:

```svelte:Fade.svelte {3,5-7}
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
	Isn't this cool?
</Fade>
```

You could also create a more general `<Transition>` component that accepts a prop for the transition you want to use like `<Transition type="fade">` and conditionally that type of transition.

### Custom Transitions

You can find more built-in transitions in the [Svelte documentation](https://svelte.dev/docs/svelte/svelte-transition). If that isn't enough, you can also create custom transitions.

Custom transitions are regular function which have to return an object with the transition options and a `css`, or `tick` function:

```svelte:App.svelte {4-14,22}
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

The `t` argument is the transition progress from `0` to `1` after the easing has been applied — if you have a transition that lasts `2` seconds, where you move an item from `0` pixels to `100` pixels, it's going to start from `0` pixels and end at `100` pixels.

You can reverse the transition by using the `u` argument which is a transition progress from `1` to `0` — if you have a transition that lasts `2` seconds, where you move an item from `100` pixels to `0` pixels, it's going to start from `100` pixels and end at `0` pixels.

Alternatively, you can return a `tick` function when you need to use JavaScript for a transition and Svelte is going to use the [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) API:

```svelte:App.svelte {8-26,34}
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

### Coordinating Transitions Between Different Elements

In this example, we have a section for published posts and archived posts where you can archive and unarchive post:

```svelte:App.svelte
<script>
	let posts = $state([
		{
			id: 1,
			title: 'Title',
			description: 'Content',
			published: true,
		},
		// ...
	])

	function togglePublished(post) {
		const index = posts.findIndex((p) => p.id === post.id)
		posts[index].published = !posts[index].published
	}

	function removePost(post) {
		const index = posts.findIndex((p) => p.id === post.id)
		posts.splice(index, 1)
	}
</script>

<div>
	<h2>Posts</h2>
	<section>
		{#each posts.filter((posts) => posts.published) as post (post)}
			<article>
				<h3>{post.title}</h3>
				<p >{post.description}</p>
				<div>
					<button onclick={() => togglePublished(post)}>💾</button>
					<button onclick={() => removePost(post)}>❌</button>
				</div>
			</article>
		{:else}
			<p>There are no posts.</p>
		{/each}
	</section>
</div>

<div>
	<h2>Archive</h2>
	<section>
		{#each posts.filter((posts) => !posts.published) as post (post)}
			<article>
				<h3>{post.title}</h3>
				<div>
					<button onclick={() => togglePublished(post)}>♻️</button>
				</div>
			</article>
		{:else}
			<p>Archived items go here.</p>
		{/each}
	</section>
</div>
```

This works, but the user experience is not great! In the real world, items don't simply teleport around like that. The user should have more context for what happened when performing an action.

In Svelte, you can coordinate transitions between different elements using the `crossfade` transition. The `crossfade` transition creates two transitions named `send` and `receive` which accept a unique key to know what to transition:

```svelte:App.svelte {2,4,10-11,17-18}
<script>
	import { crossfade } from 'svelte/transition'

	const [send, receive] = crossfade({})
	// ...
</script>

<!-- published posts -->
<article
	in:receive={{ key: post }}
	out:send={{ key: post }}
>
<!-- ... -->

<!-- archived posts -->
<article
	in:receive={{ key: post }}
	out:send={{ key: post }}
>
<!-- ... -->
```

You can also pass `duration` and a custom `fallback` transition options when there are no matching transitions:

```ts:App.svelte
const [send, receive] = crossfade({
	// the duration is based on the distance
	duration: (d) => Math.sqrt(d * 200),
	// custom transition
	fallback(node, params) {
		return {
			css: (t) => `
				transform: scale(${t});
				opacity: ${t};
			`
		}
	}
})
```

That's it! 😄

These days there are web APIs to transition view changes like the [View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API), but they're not supported in all browsers yet.

### FLIP Animations

In our previous example, we used the `crossfade` transition to coordinate transitions between different elements, but it's not perfect. When you move a post between being archived and published, all the items "wait" for the transition to end before they "snap" into their new position.

We can fix this by using Svelte's `animate:` directive and the `flip` function which calculates the start and end position of an element and animates between them:

```svelte:App.svelte {2,11,19}
<script>
	import { flip } from 'svelte/animate'
	import { crossfade } from 'svelte/transition'

	const [send, receive] = crossfade({})
	// ...
</script>

<!-- published posts -->
<article
	animate:flip={{ duration: 200 }}
	in:receive={{ key: post }}
	out:send={{ key: post }}
>
<!-- ... -->

<!-- archived posts -->
<article
	animate:flip={{ duration: 200 }}
	in:receive={{ key: post }}
	out:send={{ key: post }}
>
<!-- ... -->
```

Isn't it magical? 🪄

[FLIP](https://aerotwist.com/blog/flip-your-animations/) is an animation technique for buttery smooth layout animations. In Svelte, you can only FLIP items inside of an `each` block. **It's not** reliant on `crossfade`, but they work great together.

You can make your own custom animation functions! Animations are triggered only when the contents of an `each` block change. You get a reference to the `node`, a `from` and `to` [DOMRect](https://developer.mozilla.org/en-US/docs/Web/API/DOMRect#Properties) which has the size and position of the element before and after the change and `parameters`.

Here's a simplified version of a custom FLIP animation I _yoinked_ from the Svelte source code:

```ts:animations.ts
function flip(node, { from, to }, params) {
	const dx = from.left - to.left
	const dy = from.top - to.top
	const dsx = from.width / to.width
	const dsy = from.height / to.height

	return {
		duration: params.duration || 2000,
		css: (t, u) => {
			const x = dx * u
			const y = dy * u
			const sx = dsx + (1 - dsx) * t
			const sy = dsy + (1 - dsy) * t
			return `transform: translate(${x}px, ${y}px) scale(${sx}, ${sy})`
		}
	}
}
```

This works the same as custom transitions, so you can remind yourself how that works by revisiting it — like with custom transitions, you can also return a `tick` function with the same arguments.

### Tweened Values And Springs

Imagine if you could take the animation engine from CSS and interpolate any value, including objects and arrays.

This is where the `Tween` and `Spring` classes come in handy.

The `Tween` class accepts a target value and options. You can use the `current` property to get the current value, and `target` to update the value:

```svelte:App.svelte {2,5,8,12,22}
<script>
	import { Tween } from 'svelte/motion'
	import { cubicInOut } from 'svelte/easing'

	const size = new Tween(50, { duration: 300, easing: cubicInOut })

	function onmousedown() {
		size.target = 150
	}

	function onmouseup() {
		size.target = 50
	}
</script>

<svg width="400" height="400" viewBox="0 0 400 400">
	<circle
		{onmousedown}
		{onmouseup}
		cx="200"
		cy="200"
		r={size.current}
		fill="aqua"
	/>
</svg>
```

The `Tween` class has the same methods as `Tween`, but uses spring physics and doesn't have a duration. Instead, it has `stiffness`, `damping`, and `precision` options:

```svelte:App.svelte {2,4,7,11,21}
<script>
	import { Spring } from 'svelte/motion'

	const size = new Spring(50, { stiffness: 0.1, damping: 0.25, precision: 0.1 })

	function onmousedown() {
		size.target = 150
	}

	function onmouseup() {
		size.target = 50
	}
</script>

<svg width="400" height="400" viewBox="0 0 400 400">
	<circle
		{onmousedown}
		{onmouseup}
		cx="200"
		cy="200"
		r={size.current}
		fill="aqua"
	/>
</svg>
```

They both have a `set` function which returns a promise and lets you override the options:

```ts:App.svelte
async function onmousedown() {
	// using `target` to update the value
	size.target = 150
	// using `set` to update the value
	await size.set(150, { duration: 200 })
}
```

If you want to update the `Tween` or `Spring` value when a reactive value changes, you can use the `of` method:

```svelte:App.svelte
<script>
	import { Spring, Tween } from 'svelte/motion'

	let { value, options } = $props()

	Tween.of(() => value, options)
	Spring.of(() => value, options)
</script>
```

## Built-In Reactives

Svelte provides reactive versions of built-in JavaScript objects like `Map`, `Set`, `Date`, and `URL`, including other reactive utilities.

## Using Third Party Libraries

If a specific Svelte package isn't available, you have the entire JavaScript ecosystem at your fingertips. In this section, we're going to learn methods at your disposal you can use to integrate third party JavaScript libraries with Svelte.

### Component Lifecycle Functions

So far, we got used to Svelte's declarative syntax and reactivity. Unfortunately, third-party JavaScript libraries usually require direct access to the DOM, and they don't understand Svelte's reactivity.

Let's look at how we can use the popular [GSAP](https://gsap.com/) JavaScript animation library in Svelte. You can install GSAP with `npm i gsap` (if you're using the Svelte Playground, you can skip this and use imports directly).

Here's a basic GSAP example for creating a tween animation:

```html:index.html
<script type="module">
	import gsap from 'gsap'

	gsap.to('.box', { rotation: 180, x: 100, duration: 1 })
</script>

<div class="box"></div>

<style>
	.box {
		width: 100px;
		height: 100px;
		background-color: orangered;
		border-radius: 1rem;
	}
</style>
```

If you tried this example in Svelte, you would get a `GSAP target .box not found.` warning. This is because the `<script>` part runs first in Svelte, before the component is added to the DOM.

For this reason, Svelte provides an `onMount` lifecycle function. The "lifecyle" part refers to the life of the component, since it accepts a callback that runs when it's added and removed:

```svelte:App.svelte {2,5-7}
<script>
	import { onMount } from 'svelte'
	import gsap from 'gsap'

	onMount(() => {
		gsap.to('.box', { rotation: 180, x: 100, duration: 1 })
	})
</script>

<div class="box"></div>

<style>
	.box {
		width: 100px;
		height: 100px;
		background-color: orangered;
		border-radius: 1rem;
	}
</style>
```

This works! That being said, it's not ideal that we query any element with a `.box` class on the page.

Using Svelte, we should get a reference to the element instead. I also want to show you that you can return a function from `onMount` or use the `onDestroy` lifecycle function for any cleanup when the component is removed:

```svelte:App.svelte {2,5,10,13-16,19}
<script>
	import { onDestroy, onMount } from 'svelte'
	import gsap from 'gsap'

	let tween
	let target

	onMount(() => {
		tween = gsap.to(target, { rotation: 180, x: 100, duration: 1 })
		return () => tween.kill()
	})

	// alternative cleanup
	onDestroy(() => {
		tween.kill()
	})
</script>

<div bind:this={target}></div>

<style>
	.box {
		width: 100px;
		height: 100px;
		background-color: orangered;
		border-radius: 1rem;
	}
</style>
```

### Effects Versus Lifecycle Functions

You can also use effects to achieve the same thing:

```svelte:App.svelte {5,7-10,13}
<script>
	import gsap from 'gsap'

	let tween
	let target

	$effect(() => {
		tween = gsap.to(target, { rotation: 180, x: 100, duration: 1 })
		return () => tween.kill()
	})
</script>

<div bind:this={target}></div>

<style>
	.box {
		width: 100px;
		height: 100px;
		background-color: orangered;
		border-radius: 1rem;
	}
</style>
```

So why do both of them exist?

Effects aren't lifecycle functions because their "lifecycle" depends on the value inside of them updating. You could end up tracking some state inside of the effect and then have to [untrack](https://svelte.dev/docs/svelte/svelte#untrack) the value:

```ts:example
import { untrack } from 'svelte'

let value_you_dont_want_to_track = $state('')
let value_you_want_to_track = $state('')

$effect(() => {
	untrack(() => value_you_dont_want_to_track)
	console.log(value_you_want_to_track)
})
```

It's your choice, of course! If you understand how `$effect` works, you won't get unexpected surprises.

Alright, our code works! Let's go a step further and create a `<Tween>` component which accepts `tween`, `vars` and `children` as props:

```svelte:Tween.svelte
<script lang="ts">
	import gsap from 'gsap'
	import type { Snippet } from 'svelte'

	type Props = {
		tween: gsap.core.Tween
		vars: gsap.TweenVars
		children?: Snippet
	}

	let { tween = $bindable(), vars, children }: Props = $props()
	let target: HTMLElement

	$effect(() => {
		tween = gsap.to(target, vars)
		return () => tween.kill()
	})
</script>

<div bind:this={target}>
	{@render children?.()}
</div>
```

This gives us a generic animation component we can pass any element to, and bind the `tween` prop to get the animation controls:

```svelte:App.svelte
<script lang="ts">
	import Tween from './Tween.svelte'

	let animation: gsap.core.Tween
</script>

<Tween bind:tween={animation} vars={{ rotation: 180, x: 100, duration: 1 }}>
	<div class="box"></div>
</Tween>

<button onclick={() => animation.restart()}>Play</button>

<style>
	.box {
		width: 100px;
		height: 100px;
		background-color: #ff4500;
		border-radius: 1rem;
	}
</style>
```

### Element Lifecycle Functions Using Attachments

So far we learned how we can use `onMount` to get a reference to an element when the component is added. What if you had `onMount` for elements instead of components? You would have attachments.

Attachments are functions you can "attach" to regular elements that run when the element is added to the DOM, or when state inside of them updates:

```svelte:example {2,8-15}
<script>
	let color = $state('orangered')
</script>

<canvas
	width={400}
	height={400}
	{@attach (canvas) => {
		const context = canvas.getContext('2d')

		$effect(() => {
			context.fillStyle = color
			context.fillRect(0, 0, canvas.width, canvas.height)
		})
	}}
></canvas>
```

Instead of the animation component, we can create an attachment function which can be used on any element. The `tween` function accept the animations options and an optional callback to get a reference to the tween:

```svelte:App.svelte {4-12,18-21}
<script lang="ts">
	import { gsap } from 'gsap'

	function tween(vars, ref) {
		let tween: gsap.core.Tween

		return (target: HTMLElement) => {
			tween = gsap.to(target, vars)
			ref?.(tween)
			return () => tween.kill()
		}
	}

	let animation: gsap.core.Tween
</script>

<div
	{@attach tween(
		{ rotation: 180, x: 100, duration: 1 },
		(tween) => animation = tween
	)}
	class="box"
></div>

<button onclick={() => animation.restart()}>Play</button>
```

A cool idea would be to have different attachments like `{@attach tween.from(...)}` or `{@attach tween.to(...)}`. The fun comes from picking the API shape you want that works in harmony with Svelte.

## Using Reactivity With Events

This is a more advanced topic, but I think it's useful to know whenever you're trying to make an external event-based system reactive in Svelte.

An external event is any event you can subscribe to and listen for changes. For example, let's say I want to create a GSAP animation timeline that I can control with state.

Let's start by creating the GSAP timeline:

```svelte:App.svelte
<script lang="ts">
	import { onMount } from 'svelte'
	import gsap from 'gsap'

	type Tween = [string | HTMLElement, gsap.TweenVars]

	class Timeline {
		#timeline = gsap.timeline()

		constructor(tweens: Tween[]) {
			this.populateTimeline(tweens)
		}

		populateTimeline(tweens: Tween[]) {
			onMount(() => {
				tweens.forEach(([element, vars]) => {
					this.#timeline.to(element, vars)
				})
			})
		}
	}

	const tl = new Timeline([
		['.box1', { x: 200, duration: 1 }],
		['.box2', { x: 200, duration: 1 }]
	])
</script>

<div class="box box1"></div>
<div class="box box2"></div>

<style>
	.box {
		aspect-ratio: 1;
		width: 100px;
		background-color: orangered;
		border-radius: 1rem;
	}
</style>
```

The next step is to subscribe for updates using `eventCallback` from GSAP. Because we need to synchronize with an external system, we need an effect so when we update the time, it updates the playhead and causes `onUpdate` to fire:

```svelte:App.svelte {9,14-16,18-20,31-33,35-37,48}
<script lang="ts">
	import { onMount } from 'svelte'
	import gsap from 'gsap'

	type Tween = [string | HTMLElement, gsap.TweenVars]

	class Timeline {
		#timeline = gsap.timeline()
		#time = $state(0)

		constructor(tweens: Tween[]) {
			this.populateTimeline(tweens)

			$effect(() => {
				this.#timeline.seek(this.#time)
			})

			this.#timeline.eventCallback('onUpdate', () => {
				this.#time = this.#timeline.time()
			})
		}

		populateTimeline(tweens: Tween[]) {
			onMount(() => {
				tweens.forEach(([element, vars]) => {
					this.#timeline.to(element, vars)
				})
			})
		}

		get time() {
			return this.#time
		}

		set time(v) {
			this.#time = v
		}
	}

	const tl = new Timeline([
		['.box1', { x: 200, duration: 1 }],
		['.box2', { x: 200, duration: 1 }]
	])
</script>

<label>
	<p>Time:</p>
	<input bind:value={tl.time} type="range" min={0} max={2} step={0.01} />
</label>

<div class="box box1"></div>
<div class="box box2"></div>

<style>
	.box {
		aspect-ratio: 1;
		width: 100px;
		background-color: orangered;
		border-radius: 1rem;
	}
</style>
```

So what is the downside of this approach? If you create an effect outside of a Svelte component, you might run into an effect orphan error in the constructor:

```ts:timeline.ts
export const tl = new Timeline(...) // ⚠️ effect orphan
```

The reason this happens is because effects need to be inside a parent root effect. This is how Svelte keeps track of every effect and knows what to cleanup when the component is removed from the DOM.

There's an advanced [$effect.root](https://svelte.dev/docs/svelte/$effect#$effect.root) rune you can use and provide a manual cleanup, but **I'm only telling you this if you encounter it because you should never have to use it.**

Not only that, but we're not doing any cleanup for the event either!

Alright, so how can we improve this? It makes more sense to create the effect when we read the value. This creates another problem, because we're creating an effect each time we read the value:

```ts:example
// ...
get time() {
	// oops 😅
	$effect(() => {
		this.#timeline.seek(this.#time)
	})
	return this.#time
}
```

Thankfully, Svelte has a [createSubscriber](https://svelte.dev/docs/svelte/svelte-reactivity#createSubscriber) function you can use to create a subscriber to subscribe to! The `createSubscriber` function provides a callback which gives you an `update` function. When `update` is invoked, it reruns the subscriber. In our example, the subscriber is the `time` method:

```svelte:App.svelte {10,14-17,28-31,33-35}
<script lang="ts">
	import { onMount } from 'svelte'
	import { createSubscriber } from 'svelte/reactivity'
	import gsap from 'gsap'

	type Tween = [string | HTMLElement, gsap.TweenVars]

	class Timeline {
		#timeline = gsap.timeline()
		#subscribe

		constructor(tweens: Tween[]) {
			this.populateTimeline(tweens)
			this.#subscribe = createSubscriber((update) => {
				this.#timeline.eventCallback('onUpdate', update)
				return () => this.#timeline.eventCallback('onUpdate', null)
			})
		}

		populateTimeline(tweens: Tween[]) {
			onMount(() => {
				tweens.forEach(([element, vars]) => {
					this.#timeline.to(element, vars)
				})
			})
		}

		get time() {
			this.#subscribe()
			return this.#timeline.time()
		}

		set time(v) {
			this.#timeline.seek(v)
		}
	}

	const tl = new Timeline([
		['.box1', { x: 200, duration: 1 }],
		['.box2', { x: 200, duration: 1 }]
	])
</script>

<label>
	<p>Time:</p>
	<input bind:value={tl.time} type="range" min={0} max={2} step={0.01} />
</label>

<div class="box box1"></div>
<div class="box box2"></div>

<style>
	.box {
		aspect-ratio: 1;
		width: 100px;
		background-color: orangered;
		border-radius: 1rem;
	}
</style>
```

This makes our code much simpler. We don't need extra state to keep track of the time. Instead, we can just return and set the current time for the timeline using the methods it provides. Also, we can easily do a cleanup! 🧹

How it works is that `createSubscriber` uses an effect that watches a value that increments when `update` runs, and reruns subscribers while keeping track of the active effects.

Do you remember the counter example from before, when we talked about how you don't need effects and you can do side-effects inside event handlers?

```ts:counter.svelte.ts
export class Counter {
	#first = true

	constructor(initial: number) {
		this.#count = $state(initial)
	}

	get count() {
		if (this.#first) {
			const savedCount = localStorage.getItem('count')
			if (savedCount) this.#count = parseInt(savedCount)
			this.#first = false
		}
		return this.#count
	}

	set count(v: number) {
		localStorage.setItem('count', v.toString())
		this.#count = v
	}
}
```

This can also be made simpler by using `createSubscriber`. You only have to listen for the `storage` event on the `window` and run `update` when it changes to notify subscribers, so you don't even need to use state:

```ts.counter.svelte.ts {5,8-14,17-20,22-24}
import { createSubscriber } from 'svelte/reactivity'
import { on } from 'svelte/events'

class Counter {
	#subscribe

	constructor(initial: number) {
		this.#subscribe = createSubscriber((update) => {
			if (!localStorage.getItem('count')) {
				localStorage.setItem('count', initial.toString())
			}
			const off = on(window, 'storage', update)
			return () => off()
		})
	}

	get count() {
		this.#subscribe()
		return parseInt(localStorage.getItem('count') ?? '0')
	}

	set count(v: number) {
		localStorage.setItem('count', v.toString())
	}
}
```

In this example, we also use the `on` event from Svelte rather than `addEventListener`, because it returns a cleanup function that removes the handler for convenience.

## The Svelte Ecosystem

## Using Svelte With AI
