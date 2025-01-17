---
title: Native Page Transitions With SvelteKit Using The View Transitions API
description: Learn how to animate state and page transitions with ease using the View Transitions API.
slug: sveltekit-view-transitions
published: '2023-09-08'
category: sveltekit
draft: true
---

## Table of Contents

https://github.com/paoloricciuti/sveltekit-view-transition

## What Is The View Transitions API?

The [View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API) is a new browser API that lets you easily create animated transitions between different states in your app.

In the past you had to reach for the [FLIP animation technique](https://aerotwist.com/blog/flip-your-animations/) if you wanted to do [impossible layout animations](https://joyofcode.xyz/impossible-layout-animations-with-svelte) but it wasn't easy until now.

The View Transitions API makes it easy to create state and page transitions for single-page (SPA) and traditional multi-page applications (MPA).

It works by creating a snapshot of the DOM before and after the change and does a cross-fade by default, but it can be customized with CSS transitions.

You can read [Smooth and simple transitions with the View Transitions API](https://developer.chrome.com/docs/web-platform/view-transitions/) which includes a lot of examples to learn everything about the View Transitions API.

At the time of writing the View Transitions API is only [supported](https://caniuse.com/view-transitions) in Chromium based browsers, but Safari and Firefox are working on implementing it.

That being said Chrome has the majority share of the browser market at over 60%. If a browser doesn't support the View Transitions API your site is going to work as normal.

## Animating Page Transitions

Using the View Transitions API is simple.

```ts:example.ts
// snapshot of old DOM 📸
document.startViewTransition(() => {
  // DOM update
  // snapshot of new DOM 📸
})
```

This is fine for state transitions but to animate page transitions we have to know when the page changed.

To know when the page changed we can use the [onNavigate](https://kit.svelte.dev/docs/modules#$app-navigation-onnavigate) lifecycle function in SvelteKit.

```html:src/routes/+layout.svelte showLineNumbers
<script lang="ts">
	import { onNavigate } from '$app/navigation'

	onNavigate((navigation) => {
		if (!document.startViewTransition) return

		return new Promise((resolve) => {
			document.startViewTransition(async () => {
				resolve()
				await navigation.complete
			})
		})
	})
</script>
```

If the browser doesn't support the View Transitions API we don't do anything.

If the browser supports the View Transitions API we return a promise to wait for the page to navigate and load the data. After that is done we resolve the promise and do the transition.

> 🐿️ If you're using TypeScript you're going to get an error because the View Transitions API types don't exist yet in which case you can add a `@ts-expect-error` or `@ts-ignore` line comment to ignore the error.

To control the animation there's a new `::view-transition` pseudo-element that has the selectors for the old and new transition.

```text:example showLineNumbers
::view-transition
└─ ::view-transition-group(root)
   └─ ::view-transition-image-pair(root)
      ├─ ::view-transition-old(root)
      └─ ::view-transition-new(root)
```

You can target the pseudo-elements with CSS to customize the transition.

```css:src/app.css showLineNumbers
::view-transition-old(root),
::view-transition-new(root) {
	animation-duration: 4s;
}
```

You can use the [animations tab](https://developer.chrome.com/docs/devtools/css/animations/) in your developer tools to inspect view transitions.

You can make the page fly in and out of view but also do other fun transitions like the [Star Wars wipe effect](https://simple-set-demos.glitch.me/star-wars-wipe/) using CSS [blend modes](https://developer.mozilla.org/en-US/docs/Web/CSS/mix-blend-mode), or the [Batman transition](https://simple-set-demos.glitch.me/batman-transition/).

If you don't want to use page transitions you can set `view-transition-name` to `none` on `:root`.

```css:src/app.css showLineNumbers
:root {
	view-transition-name: none;
}
```

The `view-transition-name` value can be anything including `none`.

If you don't want the header to be included in the page transition you can separate it from the rest of the page.

```css:src/routes/header.svelte showLineNumbers
header {
  view-transition-name: header;
}
```

This is how simple it is to animate the active page marker.

```css:src/routes/header.svelte showLineNumbers
&[aria-current='page']::before {
  view-transition-name: active-page;
}
```

You can also animate different DOM elements between page transitions like the planet title and image.

```html:src/routes/+page.svelte {4,5,17,30} showLineNumbers
<div class="planets">
  {#each planets as { name, image }}
    <a href="planets/{name.toLowerCase()}" class="planet">
      <img src={image} alt={name} style:--planet="image-{name}" />
      <h2 style:--title="title-{name}">{name}</h2>
    </a>
  {/each}
</div>

<style>
.planets {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 4rem;

  & h2 {
    view-transition-name: var(--title);
  }

  & .planet {
    display: grid;
    place-content: center;
    text-align: center;

    & img {
      width: 280px;
      opacity: 0.8;
      transition: opacity 0.3s ease;
      user-select: none;
      view-transition-name: var(--planet);

      &:hover {
        opacity: 1;
      }
    }
  }
}
</style>
```

You need to give the `view-transition-name` a unique name to know what changed which is why we set the name using CSS variables.

The same goes for the new elements your old elements are going to transition to.

```html:src/routes/planets/[planet]/+page.svelte {3,21,37,48} showLineNumbers
<div class="container">
  <div class="description">
    <h1 style:--title="title-{planet.name}">{planet.name}</h1>

    <p>{planet.description}</p>

    <MakeReservation />

    <div class="details">
      {#if planet.details}
        {#each planet.details as { title, value }}
          <div class="item">
            <div>{title}</div>
            <div>{value}</div>
          </div>
        {/each}
      {/if}
    </div>
  </div>

  <img src={planet.image} alt={planet.name} style:--planet="image-{planet.name}" />
</div>

<style>
	.container {
		display: grid;
		grid-template-columns: 50ch 1fr;
		max-width: 1024px;
		margin-top: 12rem;
		margin-inline: auto;

		& img {
			width: 100%;
			height: 100%;
			scale: 1.4;
			z-index: -1;
			view-transition-name: var(--planet);
		}
	}

	.description {
		align-self: center;

		& h1 {
			width: fit-content;
			font-size: 3rem;
			text-transform: capitalize;
			view-transition-name: var(--title);
		}
    /* ... */
  }
</style>
```

## Animating State Transitions

The View Transitions API is not only useful for animating page transitions but also animating state transitions.

Let's use the View Transitions API to animate the state change when a user does a reservation for a flight to some planet.

```html:src/routes/[planet]/button.svelte {5-8,13,16,17,19,36-41,55} showLineNumbers
<script lang="ts">
	type State = 'idle' | 'loading' | 'success' | 'error'
	let state: State = 'idle'

	function transition(action: () => void) {
		if (!document.startViewTransition) return
		document.startViewTransition(action)
	}

	function makeReservation() {
		if (state !== 'idle') return

		transition(() => (state = 'loading'))

		Math.random() > 0.5
			? setTimeout(() => transition(() => (state = 'success')), 2000)
			: setTimeout(() => transition(() => (state = 'error')), 2000)

		setTimeout(() => transition(() => (state = 'idle')), 3000)
	}
</script>

<button on:click={makeReservation} data-state={state}>
	{#if state === 'idle'}
		Make reservation
	{:else if state === 'loading'}
		Making reservation...
	{:else if state === 'success'}
		Your ticket has been reserved
	{:else if state === 'error'}
		No tickets available
	{/if}
</button>

<style>
	/* ignore aspect-ratio */
	:global(html)::view-transition-old(reservation),
	:global(html)::view-transition-new(reservation) {
		width: 100%;
		height: 100%;
	}

	button {
		--background: hsl(220 40% 28%);

		all: unset;
		margin-top: 2rem;
		padding: 1rem;
		font-weight: 600;
		background-color: var(--background);
		text-shadow: 1px 1px 1px hsl(0 0% 0% / 60%);
		border-radius: 4px;
		cursor: pointer;
		transition: background-color 0.3s ease;
		view-transition-name: reservation;

		&[data-state='loading'],
		&[data-state='success'] {
			display: flex;
			gap: 0.5rem;
		}

		&[data-state='loading'] {
			--background: hsl(60 40% 28%);
		}

		&[data-state='success'] {
			--background: hsl(120 40% 28%);
		}

		&[data-state='error'] {
			--background: hsl(9 40% 28%);
		}
	}

	@keyframes loading {
		to {
			rotate: 1turn;
		}
	}

	.loading {
		animation: loading 1s infinite;
	}
</style>
```

That's how simple that was and it only required a couple of lines of code.

## Prefers Reduced Motion

```css:src/app.css showLineNumbers
@media (prefers-reduced-motion) {
	::view-transition-group(*),
	::view-transition-old(*),
	::view-transition-new(*) {
		animation: none !important;
	}
}
```
