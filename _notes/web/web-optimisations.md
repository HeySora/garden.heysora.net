---
title: Caring about performance in websites
short-title: Performance in websites
subtitle: Crafting innovative and dynamic experiences without sacrificing usability
tags: web
---

Here are my personal tips in order to make about any crazy creative webpage run very well on about any device.  
This list is not an exhaustive list of everything you could possibly do. Those are just some of my own tricks that I feel can apply in many situations. I'm expecting this note to expand over time.

## Throttle your own JS perframes

Finding yourself using `requestAnimationFrame()` quite often? Are you re-drawing a GLSL shader, or doing some other heavy realtime calculation?  
Does it *need* to run at every single frame?

If you're generating a background that changes over time, or making slow animations, there's a high chance that you can get away with re-rendering half the time.

```js
// Keep track of the current frame number
let frame = 0;

function loop() {
	requestAnimationFrame(loop);
	
	// Only draw every two frames
	if (frame++ % 2) return;
	
	// ... Your actual logic
}
requestAnimationFrame(loop);
```

Do you want to instead run every three frames? Simply use `frame++ % 3` !  
This concise syntax gives us a lot of flexibility because I sometimes throttle it even further for mobiles when I make use of WebGL for slow decorations, with something like this:

```js
	if (frame++ % (isMobile ? 4 : 2)) return;
```

Another trick you can have, for elements which react to scroll or other events, is to throttle your loop only if nothing is going on! So you can conservatively throttle your effect to half the framerate *except* when running some animation on it (e.g. when clicking a button).

```js
	const animFactor = isAnimating ? 1 : 2; // Halve if not animating
	const mobileFactor = isMobile ? 2 : 1; // Halve if on mobile
	if (frame++ % (isAnimating * isMobile)) return;
```

Assuming a 60Hz monitor, this will run your loop:
* at 60FPS on a tablet / desktop, when animating; 30FPS otherwise
* at 30FPS on a phone, when animating; 15FPS otherwise

Such a throttle can help making your decorative / heavier effects require less horsepower, leaving performance for the rest of your code + browser layout!

## Avoid changes in `resize` events

Listening to `resize` events can be quite useful, but there are a few things you can do to make the experience much better. *(And before you ask, using a throttle() function should only be done as a last-resort!)*

### If using a loop(): Avoid re-renders!

Are you making a crazy visual effect that needs to adjust to the user's screen? Assuming your effect runs through `requestAnimationFrame()`, just set your variables and move on!

It's a common mistake to attempt re-drawing something from such a listener, when in reality you really don't need to, when you already have a loop taking care of drawing. This also helps keeping the code complexity and maintainability low!

```js
let width = 0,
	height = 0;

const resize = () => {
	// Simply update your variables and do whatever calculation
	// you might actually need.
	width = innerWidth / 2;
	height = innerHeight / 3;
	
	// Nothing else is needed! Those variables are used in your loop()
};

addEventListener('resize', resize);
resize(); // Initialize our variables

function loop() {
	// ... Throttling code from above

	// Example operation using our variables set from resize()
	ctx.clearRect(0, 0, width, height);
}
```

With this approach, you do not need to duplicate your drawing code, and your resize listener is very small.

### If not using a loop(): Do you even *really* need `resize` events?

Such approaches are quite common:

```js
const resize = () => {
	const width = innerWidth;
	
	if (width < 800)
		applyMobileLayout();
	else
		applyDesktopLayout();
};

addEventListener('resize', resize);
resize();
```

Some people even go the extra mile and attempt to be clever by "caching" the current layout:

```js
let hasMobileLayout = false; // Keep track of which layout is active
const resize = () => {
	const width = innerWidth;
	
	// Only switch the layout if the current layout does not match
	if (width < 800 && !hasMobileLayout)
	{
		hasMobileLayout = true;
		applyMobileLayout();
	}
	else if (width >= 800 && hasMobileLayout)
	{
		hasMobileLayout = false;
		applyDesktopLayout();
	}
};
```

The reality is that both of these solutions are not optimized, because you actually do not need to run such a function on each resize event. A much better solution involves media queries!

```js
// Matches if the screen width is >= 800px
const desktopQuery = window.matchMedia('(min-width: 800px)')

const handleDesktop = query => {
	// Yes, the screen is >= 800px
	if (query.matches)
		applyDesktopLayout();
	else
		applyMobileLayout();
};

desktopQuery.addEventListener('change', handleDesktop);
handleDesktop(desktopQuery);
```

This code is as simple as the first "bad" example, but actually only runs once per change! So if a user resizes their window from 900 to 700 pixels, your listener won't be called for every single intermediary value, but will instead be called once!

## Load your stylesheets and scripts as early as possible!

Long gone are the days of *"declaring JS right before `</body>`"*, this is bad for a few reasons:

- The browser will actually start downloading your script(s) *after* everything else, which wastes time.
- Under some circumstances, the entire DOM won't even be ready yet, which will trigger extra calculations and reflows both before *and* after your code executed.

The easiest replacement is to simply declare your external assets inside your `<head>` section, and adding the `defer` attribute to your scripts! Your browser is start enough to download multiple assets at the same time, and this will allow your script to be downloaded in parallel with the rest, while still being executed at the end of the DOM loading operation.  
*(Please note that any `<script type="module">` will always be deferred by default, so you do not need to use the `defer` attribute here!)*

If you rely on third-party servers (e.g. CDNs, for fonts or JS libraries) *within* your stylesheets or scripts, feel free to make use of `dns-prefetch` meta tags! These will allow the browser to simultaneously initiate a DNS query, even before it reaches the point where your code will contact this server!

Here is an example of all of these advices applied at the same time:

```html
<head>
	<!-- Critical info -->
	<meta charset="utf-8">
	
	<!-- Start querying DNS in advance -->
	<link rel="dns-prefetch" href="//fonts.googleapis.com">
	<link rel="dns-prefetch" href="//netdna.bootstrapcdn.com">
	<link rel="dns-prefetch" href="//s3.pub1.infomaniak.cloud">
	
	<!-- Browser UI declarations -->
	<title>My webpage</title>
	<meta name="viewport" content="width=device-width,initial-scale=1">
	<meta name="msapplication-TileColor" content="#123456">
    <meta name="theme-color" content="#123456">
	<link rel="mask-icon" href="safari-tab.svg" color="#123456">
	
	<!-- Actual assets -->
	<link rel="stylesheet" href="app.css">
	<script src="app.js" defer></script>

	<!-- ... OpenGraph, Manifests, Canonical links, etc. -->
</head>
```

## And more...

So far, this is everything that crossed my mind. While some of those tips might be obvious for you, I still encounter a lot of people exercising bad practises and not being aware of the performance left on the table. I'm expecting this note to expand over time, so feel free to watch [the recently updated notes](/){: .internal-link} of this garden!

Let me know if those have been useful to you, I'd really appreciate it. 🧡