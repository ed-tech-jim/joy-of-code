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

```svelte:app.svelte
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

```svelte:app.svelte
<style>
  p {
    color: red;
  }
</style>
```

Styles are scoped to the component by default. This means that styles used in one component aren't going to affect styles in other components. If you look at the CSS output in the Svelte Playground, you can see Svelte generated a unique class name for the styles `p.svelte-omwhvp {color: red }`.

To make your styles global inside a component, you can use the `global` modifier `:global(p)`. Having to use `:global(selector)` for everything is tedious, so you can nest everything inside the `:global { ... }` block. You can also have "scoped global styles" by saying `.prose :global(p)`:

```svelte:app.svelte
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

```svelte:app.svelte
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

## Reactivity And Svelte Runes

In the last example, we defined a reactive variable `count` using the `$state` syntax:

```svelte:app.svelte
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

```svelte:app.svelte
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

```svelte:app.svelte
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

```svelte:app.svelte
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

The last rune you should know about is the `$effect` rune. Effects are functions that run when the component mounts and when their dependencies change. You can also return a function from an effect which reruns when the effect dependencies change, or when the component unmounts.

**Effects don't need a dependency array** because of how signals work — if a reactive value is read inside of an effect, it will be tracked and the effect will rerun when the tracked value changes:

```svelte:app.svelte
<script>
	let count = $state(0)
	let double = $derived(count * 2)

	$effect(() => {
		// reruns if `count` or `double` changes
		console.log({ count, double })
		// also runs when the component unmounts
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

```svelte:app.svelte
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

```svelte:app.svelte
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

```svelte:app.svelte
<script>
	let pokemon = $state()

	$effect(() => {
		const cache = JSON.parse(localStorage.getItem('pokemon'))

		if (!cache) {
			// fetching data from an API
			fetch('https://pokeapi.co/api/v2/pokemon')
				.then((response) => response.json())
				.then((data) => {
					// sync with an external system
					pokemon = data
					localStorage.setItem('pokemon', JSON.stringify(data))
				})
		} else {
			pokemon = cache
		}
	})
</script>

<pre>{JSON.stringify(pokemon, null, 2)}</pre>
```

## Template Logic

There are no conditionals and loops in HTML unless you're using a templating language. In Svelte, you can use the `#if` block to conditionally render content:

```svelte:app.svelte
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

```svelte:app.svelte
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

```svelte:app.svelte
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

```svelte:app.svelte
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

In a previous example, we fetched some Pokemon data inside of an effect. That approach works, but we haven't handled any of the the error and success states which quickly becomes a mess.

Thankfully, Svelte has a built-in solution for async data loading using the `#await` block:

```svelte:app.svelte
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

In the near future you're going to be able to use the `await` keyword directly in the `<script>` tag, inside a `$derived` expression, and in your markup. You can try it today by enabling the [experimental async flag](https://github.com/sveltejs/svelte/discussions/15845) in your Svelte config:

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

```svelte:app.svelte
<script>
	// pretend this is an import
	import { getPokemon } from 'api/pokemon'

	// you could use `await` here if the boundary was declared higher up
	let pokemon = getPokemon('charizard')
</script>

<svelte:boundary>
	{#snippet pending()}
		<!-- this only shows when the component mounts -->
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

```svelte:app.svelte
<script>
	function onclick() {
		console.log('clicked')
	}
</script>

<!-- using an inline function -->
<button onclick={() => console.log('clicked')}>
	Click
</button>

<!-- passing a function -->
<button onclick={onclick}>
	Click
</button>

<!-- using the shorthand -->
<button {onclick}>Click</button>
```

You can spread events, since they're just attributes:

```svelte:app.svelte
<script>
	const events = {
		onclick: () => console.log('clicked'),
		ondblclick: () => console.log('double clicked')
	}
</script>

<button {...events}>Click</button>
```

Here's an example of using the `onmousemove` event to update the mouse position:

```svelte:app.svelte
<script>
	let mouse = $state({ x: 0, y: 0 })

	function onmousemove(event) {
		mouse.x = event.clientX
		mouse.y = event.clientY
	}
</script>

<div {onmousemove}>
	The mouse position is {mouse.x} x {mouse.y}
</div>
```

The `event` is automatically passed to the function, so you don't have to do `onmousemove={(event) => onmousemove(event)}`.

You can also prevent default behavior by using `event.preventDefault()`. This is useful when you want to control a form with JavaScript and avoid a page reload:

```svelte:app.svelte
<script>
	function onsubmit(event) {
		event.preventDefault()
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

```svelte:app.svelte
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

```svelte:app.svelte
<input bind:value={search} type="search" />
```

Svelte provides many two-way bindings, and some readonly bindings. There are input, group, files, media and more bindings you can find in the [Svelte documentation](https://svelte.dev/docs/svelte/bind):

```svelte:app.svelte
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

```svelte:app.svelte
<script>
	// `undefined` until the component mounts
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

```svelte:app.svelte
<script>
 	let celsius = $state(0)
 	let fahrenheit = $state(0)
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

It might be tempting, but **avoid creating components**. If you're not sure what to turn into a component — don't. Instead, write everything inside a single component until it becomes obvious what which parts warrant their own component, either for reusability, or to make it easier to reason about.

You might have an `<Artist />` component:

- `<Artist />` components gets passed the `artistName` property
- `<Album />` component has `albumTitle` and `albumTracks` property passed to `<AlbumTrack />`
- `<AlbumTrack />` component has `track` and `length` properties but also a `playing` state

The filename can be whatever but a capitalised tag such as `<Artist />` indicates to Svelte that something is a component. You import another Svelte component using the `import Component as './Component'` syntax.

Pretend that `artists` is some data we fetched as a JSON response from the Spotify API.

```svelte:App.svelte {2, 3} showLineNumbers
<script>
	import Artist from './Artist.svelte'
	import Album from './Album.svelte'

	let artists = [
		{
			name: 'Fleetwood Mac',
			albums: [
				{
					name: 'Tango in the Night',
					year: 1987,
					tracks: [
						{ title: 'Big Love', length: '3:37' },
						{ title: 'Seven Wonders', length: '3:38' },
						{ title: 'Everywhere', length: '3:48' },
						{ title: 'Caroline', length: '3:50' },
						{ title: 'Tango in the Night', length: '3:56' },
						{ title: 'Mystified', length: '3:08' },
					],
				},
			],
		},
	]
</script>

{#each artists as artist}
  <Artist artistName={artist.name} />
  {#each artist.albums as album}
		<Album
			albumTitle={album.name}
			albumTracks={album.tracks}
		/>
  {/each}
{/each}
```

The `<Artist />` component takes an `artistName` prop. To define something as a prop that's passed in to your component you use the `export let prop` syntax. You can define multiple props on the same line such as `export let prop1, prop2`.

```svelte:Artist.svelte {2, 5} showLineNumbers
<script>
	export let artistName
</script>

<h1>{artistName}</h1>
```

The `<Album />` component imports `<AlbumTrack />` and loops over the tracks. The `{...track}` syntax is just spreading the `track` props which is equivalent to `title={title} length={length}`. If your props share the same name as the value you can do `{title} {length}`.

```svelte:Album.svelte {2, 18} showLineNumbers
<script>
	import AlbumTrack from './AlbumTrack.svelte'

	export let albumTitle
	export let albumTracks

	let playing

	function setPlaying(track) {
		playing = track
	}
</script>

<h2>{albumTitle}</h2>

<ul>
  {#each albumTracks as track}
    <AlbumTrack {setPlaying} {playing} {...track} />
  {/each}
</ul>
```

We're passing `setPlaying` to the child component so we can set the currently playing song and check if `currentlyPlaying` is equal to the current track.

The `<AlbumTrack />` component applies a `.playing` style using the `class:` directive based on what song is playing which is shorter than using a ternary inside an expression `class={playing === title ? 'playing' : ''}`.

```svelte:AlbumTrack.svelte {3, 4, 8, 15-17} showLineNumbers
<script>
	export let setPlaying
	export let playing
  export let title
	export let length
</script>

<li class:playing={playing === title}>
	<button on:click={() => setPlaying(title)}>▶️</button>
  <span>{title}</span>
	<span>🕒️ {length}</span>
</li>

<style>
	.playing {
		color: teal;
	}
</style>
```

We can also use a reactive statement `$: playing = playing === title` for `playing` and since it matches the class name we want to apply we can simplify the code and write `class:playing`.

```svelte:AlbumTrack.svelte {7, 10} showLineNumbers
<script>
	export let setPlaying
	export let playing
  export let title
	export let length

	$: playing = playing === title
</script>

<li class:playing>
	<button on:click={() => setPlaying(title)}>▶️</button>
  <span>{title}</span>
	<span>🕒️ {length}</span>
</li>

<style>
	.playing {
		color: teal;
	}
</style>
```

## Slots

**In Svelte we can use slots to compose components** meaning our components can contain other components and elements to be more reusable like regular HTML.

```svelte:Example.html showLineNumbers
<button>
	<span>Child</span>
</button>
```

The `<slot>` element lets us do that with components. If you're familiar with React this is similar to the `children` prop and Vue also has slots. We can provide a **fallback** if no content is provided.

```svelte:Button.svelte {2} showLineNumbers
<button>
  <slot>Placeholder</slot>
</button>

<style>
	button {
		color: teal;
	}
</style>
```

```svelte:App.svelte {2, 6} showLineNumbers
<script>
	import Button from './Button.svelte'
</script>

<Button>
  <span>Child</span>
</Button>

<Button />
```

You can use **named slots** for more control over the placement of elements. If you want multiple elements going into the same slot use the `<svelte:fragment>` element as the wrapper.

```svelte:Button.svelte {2-3} showLineNumbers
<button>
	<slot name="icon"></slot>
	<slot name="text"></slot>
</button>
```

```svelte:App.svelte {6-7, 11-12} showLineNumbers
<script>
	import Button from './Button.svelte'
</script>

<Button>
  <span slot="icon">➕</span>
	<span slot="text">Add</span>
</Button>

<Button>
  <span slot="icon">💩</span>
	<span slot="text">Delete</span>
</Button>
```

You might be asking when you'd use slots over regular components and the answer might be not often and that's fine.

Here's an example of slots and composition used in a real-world scenario in [Svelte Cubed](https://svelte-cubed.vercel.app/) that's a wrapper around [Three.js](https://threejs.org/) so you write less code because it's more declarative:

```svelte:Example.svelte
<script>
	import * as SC from 'svelte-cubed';
	import * as THREE from 'three';
</script>

<SC.Canvas>
	<SC.Mesh geometry={new THREE.BoxGeometry()} />
	<SC.PerspectiveCamera position={[1, 1, 3]} />
</SC.Canvas>
```

This is only a couple of lines of code compared to the equivalent Three.js code which has more than 20 lines of code and it's harder to read.

There's a lot more you can do with slot props but I encourage you to [read the slots documentation](https://svelte.dev/docs#template-syntax-slot) because slots deserve their separate post.

## Transitions

**Animations in Svelte are first-class** so you don't have to reach for an animation library unless you want to. To use transitions you can import `blur`, `fly`, `slide`, `scale`, `draw` and `crossfade` from `svelte/transition`.

To use a transition use `transition:fade`. You can specify parameters such as `delay`, `duration`, `easing` for `fade`. To learn what they are for each transition [consult the documentation](https://svelte.dev/docs#svelte_transition).

```svelte:App.svelte {2, 14} showLineNumbers
<script>
	import { fade } from 'svelte/transition'

	let showFade = false

	function toggleFade() {
		showFade = !showFade
	}
</script>

<button on:click={toggleFade}>Wax poetic</button>

{#if showFade}
	<blockquote transition:fade={{delay: 250, duration: 300}}>
		Memories fade, but friends are forever
	</blockquote>
{/if}
```

You can specify a enter animation with `in:fade` and exit animation with `out:fade` but you're not limited to one transition.

In Svelte you can define custom animations such as this [typewriter effect](https://learn.svelte.dev/tutorial/custom-js-transitions), use [spring and tweened motion](https://learn.svelte.dev/tutorial/tweens) and make smooth transitions between elements using [flip animations](https://learn.svelte.dev/tutorial/animate).

## Svelte Store

Passing data from parent to child component is described as **data flowing top to bottom** but Svelte lets you **reverse** the flow using **bindings**, **event forwarding** and the **context API** which you don't have to know right now because passing props is fine in most cases where you don't have deeply nested components.

However, one feature you're going to use all the time is the [Svelte store](https://svelte.dev/docs#run-time-svelte-store) which is Svelte's answer to **global state management**. You would reach for a store if you have **information that is required by multiple unrelated components** such as the logged in user or theme.

The Svelte store is just an object you can `subscribe` to for updates when the store value changes and `set` and `update` values. You can have `writable` stores to read and write to, `readable` stores if you don't want values to be set from the outside and `derived` stores if you want to use values from multiple stores.

(If you're trying this out in the Svelte REPL it's not obvious how to change the file extension but if you just type the file name such as `stores.js` it's going to change it.)

```js:stores.js showLineNumbers
import { writable } from 'svelte/store'

export let message = writable('Hello 👋')
```

You can use the reactive `$message` syntax to access the value. This also **subscribes and unsubscribes** to the store for you.

```svelte:Alert.svelte {2, 5, 9} showLineNumbers
<script>
	import { message } from './stores.js'

	function updateStore() {
		$message = 'Bye 👋'
	}
</script>

<p>{$message}</p>
<button on:click={updateStore}>Click</button>
```

You can create your own stores by implementing the [store contract](https://svelte.dev/docs#component-format-script-4-prefix-stores-with-$-to-access-their-values-store-contract). It must contain a `subscribe` method that's going to be a subscription function and return a `unsubscribe` function and it may include a `set` method to update the value.

Here's an example of a writable local storage store you can use to set and update a value in local storage.

```js:localStorageStore.js showLineNumbers
import { writable } from 'svelte/store'

export function localStorageStore(key, initial) {
  if (!localStorage.getItem(key)) {
    localStorage.setItem(key, JSON.stringify(initial))
  }

  let saved = JSON.parse(localStorage.getItem(key))
  let { subscribe, set, update } = writable(saved)

  return {
    subscribe,
    set: (value) => {
      localStorage.setItem(key, JSON.stringify(value))
      return set(value)
    },
    update
  }
}
```

```svelte:App.svelte showLineNumbers
<script>
import { localStorageStore } from './localStorageStore.js'

let message = localStorageStore('message', 'Hello 👋')

$message = 'Bye 👋'
</script>

{$message}
```

The Svelte store is incredibly powerful and deserves an entire post so I encourage you to go through the [Svelte tutorial](https://learn.svelte.dev/tutorial/writable-stores) and [consult the documentation](https://svelte.dev/docs#run-time-svelte-store) to learn more.

## Further Reading

Join me in the next post where we take what we learned to build a [Svelte todo app](https://joyofcode.xyz/svelte-todo-app) with animations and persistent storage 😍.

You're ready to build some Svelte apps! 👏 When you get more comfortable or encounter problems that require these solutions then you should learn and reach for them:

- [Lifecycle functions](https://learn.svelte.dev/tutorial/onmount)
- [Tick](https://svelte.dev/docs#run-time-svelte-tick)
- [Actions](https://learn.svelte.dev/tutorial/actions)
- [Event forwarding](https://learn.svelte.dev/tutorial/event-forwarding)
- [Context API](https://learn.svelte.dev/tutorial/context-api)
- [Module context](https://learn.svelte.dev/tutorial/sharing-code)
- [Special Elements](https://learn.svelte.dev/tutorial/svelte-self)
- [The @debug tag](https://learn.svelte.dev/tutorial/debug)

## Conclusion

Svelte is amazing for building all kinds of things and delivers a great developer and user experience.

I hope what stuck most is learning the concepts behind JavaScript frameworks because **if you understand JavaScript you can learn any JavaScript framework** and be productive quicker.

Thanks for reading! 🏄️
