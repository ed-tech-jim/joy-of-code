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

## Reactivity Using Runes

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

The `$state` syntax is called a **rune** and is part of the Svelte language. Under the hood Svelte turns the `$state` rune into a signal. The three main important runes we're going to learn about are `$state`, `$derived`, and `$effect`.

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
	let count = $state(0)
	let double = $derived(count * 2)
	let history = $derived.by(() => {
		let doubles = []

		return {
			get doubles() {
				doubles.push(double)
				return doubles.join(', ')
			}
		}
	})
</script>

<button onclick={() => count++}>
	{history.doubles}
</button>
```

Derived values are lazy evaluted. The derived value only updates when it changes and not when their dependencies change.

The last rune you should know about is the `$effect` rune. Effects are functions that run when the component mounts and when reactive values inside of them update. You can also return a function from an effect that reruns when the reactive values update and when the component unmounts:

```svelte:app.svelte
<script>
	let count = $state(0)

	$effect(() => {
		console.log(`The count is ${count}`)
		return () => console.log('🧹 cleanup')
	})
</script>

<button onclick={() => count++}>
	{count}
</button>
```

**You should never use effects to synchronize state** because Svelte queues effects and runs them last after the code ran and everything is updated.

Effects should only be used for side-effects like fetching data from an API, working with the DOM directly, or to synchronize with an external system that doesn't understand Svelte's reactivity:

```svelte:app.svelte
<script>
	let pokemon = $state()

	$effect(() => {
		const cache = JSON.deserialize(localStorage.getItem('pokemon'))
		if (!cache) {
			fetch('https://pokeapi.co/api/v2/')
				.then(response => pokemon = response.json())
		} else {
			pokemon = cache
		}
	})

	$effect(() => {
		localStorage.setItem('pokemon', JSON.stringify(pokemon))
	})
</script>

<pre>{JSON.stringify(pokemon, null, 2)}</pre>
```

## Template Logic

Since HTML can't express logic such as conditionals and loops you would have to write something like this using JavaScript.

```svelte:app.html
<script>
  let appElement = document.querySelector('#app')

  let user = {
	  loggedIn: false
	}

	function toggle() {
	  user.loggedIn = !user.loggedIn
		updateUI()
	}

	function updateUI() {
    let html

	  if (user.loggedIn) {
		  html = `<button>Log out</button>`
		}

		if (!user.loggedIn) {
		  html = `<button>Log in</button>`
		}

	  appElement.innerHTML = html
    appElement.querySelector('button').onclick = toggle
	}

  updateUI()
</script>

<div id="app"></div>
```

It doesn't look bad but I think we can do better. This is the same example using an `#if` block in Svelte.

```svelte:App.svelte {11-13, 15-17} showLineNumbers
<script>
	let user = {
		loggedIn: false
	}

	function toggle() {
		user.loggedIn = !user.loggedIn
	}
</script>

{#if user.loggedIn}
  <button on:click={toggle}>Log out</button>
{/if}

{#if !user.loggedIn}
  <button on:click={toggle}>Log in</button>
{/if}
```

Awesome, right? I want to emphasize how close Svelte is to HTML and I hope you're excited about it.

There's more logic blocks like `#if`, `#each`, `#await`, and `#key` for you to play around with.

This is using JavaScript to loop over a list of items and render them.

```svelte:Example.html showLineNumbers
<div id="app"></div>

<script>
  let appElement = document.querySelector('#app')

	let todos = [
		{ id: 1, text: 'Todo 1', completed: true },
		{ id: 2, text: 'Todo 2', completed: false },
		{ id: 3, text: 'Todo 3', completed: false },
		{ id: 4, text: 'Todo 4', completed: false },
	]

	let todosHtml = ''

	for (let todo of todos) {
    let checked = todo.completed ? 'checked' : null

	  todosHtml += `
      <li data-id=${todo.id}>
		    <input ${checked} type="checkbox" />
		    <span>${todo.text}</span>
	    </li>
		`
	}

	appElement.innerHTML = `<ul>${todosHtml}</ul>`
</script>
```

The same example using `#each` in Svelte.

```svelte:App.svelte {11-16} showLineNumbers
<script>
	let todos = [
		{ id: 1, text: 'Todo 1', completed: true },
		{ id: 2, text: 'Todo 2', completed: false },
		{ id: 3, text: 'Todo 3', completed: false },
		{ id: 4, text: 'Todo 4', completed: false },
	]
</script>

<ul>
{#each todos as todo}
  <li>
		<input checked={todo.completed} type="checkbox" />
		<span>{todo.text}</span>
	</li>
{/each}
</ul>
```

You can [destructure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) values from the item you're iterating over, get the index, and provide a key so Svelte can keep track of changes. **Avoid using the index as the key** because it's not guaranteed to be unique so use a unique value instead.

```svelte:App.svelte {2} showLineNumbers
<ul>
{#each todos as {id, text, completed}, index (id)}
  <li>
		<input checked={completed} type="checkbox" />
		<span>{text}</span>
	</li>
{/each}
</ul>
```

If you're fetching data on the client this is how it would look using JavaScript.

```svelte:Example.html showLineNumbers
<div id="app"></div>

<script>
  let appElement = document.querySelector('#app')

  async function fetchPokemon(pokemonName) {
    let url = `https://pokeapi.co/api/v2/pokemon/`
    let response = await fetch(`${url}${pokemonName}`)
    let { name, sprites } = await response.json()

    return {
      name,
      image: sprites['front_default']
    }
  }

  async function renderUI() {
    let { name, image } = await fetchPokemon('pikachu')

    appElement.innerHTML = `
      <h1>${name}</h1>
      <img src=${image} alt=${name} />
    `
  }

  renderUI()
</script>
```

In Svelte you can easily resolve a promise using the `#await` block but you can also resolve the promise in the `<script>` tag if you want .

```svelte:App.svelte {14-21} showLineNumbers
<script>
  async function fetchPokemon(pokemonName) {
    let url = `https://pokeapi.co/api/v2/pokemon/`
    let response = await fetch(`${url}${pokemonName}`)
    let { name, sprites } = await response.json()

		 return {
      name,
      image: sprites['front_default']
    }
  }
</script>

{#await fetchPokemon('pikachu')}
	<p>Fetching Pokemon...</p>
{:then pokemon}
	<h1>{pokemon.name}</h1>
	<img src={pokemon.image} alt={pokemon.name} />
{:catch error}
	<p>Something went wrong: {error.message}</p>
{/await}
```

In the JavaScript example we didn't even add checks for scenarios where the promise could be pending, fulfilled, or rejected and just hope it works. 😬 Using Svelte you don't have to think about it.

## Events

If you're new to JavaScript frameworks you might be confused by the use of **inline event handlers** because so far everyone told you to avoid doing so in JavaScript. That's for a good reason because of **separation of concerns** to have our markup, styles, and logic separate which makes it easy to change and maintain.

Using a modern JavaScript framework all our **concerns are in one place** using components and writing declarative code and the framework doing the heavy DOM lifting and under the hood performance optimization.

If you want to know about all the Svelte events you won't find them in the Svelte documentation because it's just JavaScript so you can look at the MDN documentation for [event listing](https://developer.mozilla.org/en-US/docs/Web/Events#event_listing). Under the generic `Element` you can find the event listener `dblclick` when someone performs a double click or `mousemove` when the user moves the mouse.

This is an example of an event listener in JavaScript.

```svelte:Example.html showLineNumbers
<style>
  html,
  body {
    width: 100%;
    height: 100%;
    margin: 0;
  }

  div {
    height: 100%;
  }
</style>

<div id="app"></div>

<script>
  let appElement = document.querySelector('#app')

  let mouse = { x: 0, y: 0 }

  function handleMouseMove(event) {
    mouse.x = event.clientX
    mouse.y = event.clientY
    updateUI()
  }

  function updateUI() {
    appElement.innerHTML = `
      The mouse position is ${mouse.x} x ${mouse.y}
    `
  }

  appElement.addEventListener('mousemove', handleMouseMove)
</script>
```

In Svelte you use the `on:` directive to listen to DOM events.

```svelte:App.svelte {10} showLineNumbers
<script>
	let mouse = { x: 0, y: 0 }

	function handleMouseMove(event) {
		mouse.x = event.clientX
		mouse.y = event.clientY
	}
</script>

<div on:mousemove={handleMouseMove}>
	The mouse position is {mouse.x} x {mouse.y}
</div>

<style>
	div {
		height: 100vh;
	}
</style>
```

Svelte sends the `event` alongside your function if you do `on:mousemove={handleMouseMove}` but if you do it inline you have to pass it yourself `on:mousemove={(event) => handleMouseMove(event)}`.

Svelte has special modifiers for DOM events such as `preventDefault`. You can find a complete list under [element directives](https://svelte.dev/docs#Element_directives) and you can chain special modifiers together.

```svelte:App.svelte {7} showLineNumbers
<script>
	function handleSubmit() {
		console.log('Submit')
	}
</script>

<form on:submit|preventDefault={handleSubmit}>
	<input type="text" />
	<button type="submit">Submit</button>
</form>
```

Using `preventDefault` which is short for `event.preventDefault()` prevents the default behavior such as the form submitting causing a page reload because we want to control it using JavaScript on the client.

## Bindings

**Data binding is keeping your application state and user interface synchronized.**

Svelte supports **data binding** using the `bind:` directive.

Often you have a value that other parts depend on for example if you had a text search input and want to filter a list of items whenever the user types a search query.

You can implement data binding in JavaScript but it's not part of the language so you often get the value from the `event`. This is true for other JavaScript frameworks like React that use a synthetic event system and doesn't have data binding.

Filtering a list of items using JavaScript.

```svelte:Example.html showLineNumbers
<input type="text" />
<ul></ul>

<script>
  let list = ['React', 'Vue', 'Svelte']
  let filteredList = []

  let inputElement = document.querySelector('input')
  let listElement = document.querySelector('ul')

  function filterList(event) {
    let searchQuery = event.target.value
    filteredList = list.filter(item => {
      return item
              .toLowerCase()
              .includes(searchQuery.toLowerCase())
    })
    updateUI()
  }

  function updateUI() {
    listElement.innerHTML = filteredList.map(item =>
			`<li>${item}</li>`).join('')
  }

  inputElement.addEventListener('input', filterList)
</script>
```

Instead of using `event.target.value` which we could also do in Svelte we can bind the value of the text input field to `searchQuery` instead.

```svelte:App.svelte {4, 17} showLineNumbers
<script>
 	let list = ['React', 'Vue', 'Svelte']
  let filteredList = []
	let searchQuery = ''

	function filterList() {
    filteredList = list.filter(item => {
      return item
              .toLowerCase()
              .includes(searchQuery.toLowerCase())
    })
  }
</script>

<input
	on:input={filterList}
	bind:value={searchQuery}
	type="text"
/>

<ul>
{#each filteredList as item}
  <li>{item}</li>
{/each}
</ul>
```

You can have text, numeric, checkbox, group and textarea among other bindings. Instead of overwhelming you with examples you lack context for to find useful right now you can learn more about [bindings in the Svelte documentation](https://svelte.dev/docs#template-syntax-element-directives-bind-property) or by following the [Svelte tutorial](https://learn.svelte.dev/tutorial/text-inputs) when you encounter it in your project.

## Components

**Components are the primary reason of using any modern JavaScript framework** because it lets you **organize** code and have your **concerns in one place**.

If you ever used classes you can think of components as new instances of a class that can be used as a blueprint to have its own independent state.

It's easy to get carried away with components so in general **don't look for what to turn into a component** but write everything inside a single file until it becomes hard to manage and you start noticing repeating parts.

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
