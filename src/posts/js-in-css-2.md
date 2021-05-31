---
layout: layouts/post.njk
title: JS in CSS
date: 2021-01-05T14:37:16.899Z
tags:
  - css
  - js
  - custom properties
---
Yes, this is not a mistake, today's article is about JavaScript in CSS.

In general, embedding scripts in CSS is a rather strange idea at first glance. And I've got it after reading a fragment of the custom properties [specification](https://drafts.csswg.org/css-variables/#syntax).

It is written there:

> For example, the following is a valid custom property:
> **\--foo: if(x &gt; 5) this.width = 10;**
> While this value is obviously useless as a variable, as it would be invalid in any normal property, it might be read and acted on by JavaScript.

How and why can it be used?

The first thought was about a sophisticated attack on the user's browser. 😈

This requires the victim to download two files: CSS and JS. CSS declares a custom property called `--script` with JS-code as its value:

```css
:root {
  --script: console.log("hack!");
}
```

And now we need a JS file in which the value of `--script` will be calculated and executed:

```js
// Get all document styles
const style = getComputedStyle(document.documentElement);

// Take the value of the --script property
const script = style.getPropertyValue('--script');

// Execute --script value
new Function(script)();
```

It works:

![Browser console with the code running result](/images/1-qizkchbeg-bk4kcx84zikq.png)

You can reset the value of a custom property over time and execute a new code each time with CSS animation.

To do this, let's create an animation like this:

```css
:root {
  animation-name: hack;
  animation-duration: 3s;
  animation-delay: 1s;
  animation-fill-mode: backwards;
}
```

In this animation, we will change the `--script` property value during three seconds:

```css
@keyframes hack {
  0% {
    --script: console.log("🙈");
  }
  33% {
    --script: console.log("🙉");
  }
  66% {
    --script: console.log("🙊");
  }
}
```

We will get and execute the value of the `--script` property every second in JS:

```js
setInterval(() => {
  const style = getComputedStyle(document.documentElement);
  const script = style.getPropertyValue('--script');
  
  new Function(script)();
}, 1000);
```

It works also:

![Browser console with the code running result](/images/1-md2rtxmnx0xt-tvifkx45g.gif).

Here is the [demo](https://codepen.io/juwain/pen/EEgOdr) with a code. 

- - -

Playing hackers is certainly fun, but is it possible to use a feature with JS and custom properties with real use somehow?

Based on [Harry Roberts' idea](https://csswizardry.com/2018/01/finding-dead-css/), I've got the idea of "logging" the CSS code using.

Let's say we have many CSS files on a large project that are included in multiple bundles. You need to find out which of the files are used in users' browsers and which are not.

The idea is to include a unique custom property in each CSS file that will "trace" in the browser's localStorage.

```css
/* ---------------------- File style-1.css ---------------------- */
:root {
  --style-1: localStorage["--style-1"] = new Date();
}

/* ---------------------- File style-2.css ---------------------- */
:root {
  --style-2: localStorage["--style-2"] = new Date();
}

/* ---------------------- File style-3.css ---------------------- */
:root {
  --style-3: localStorage["--style-3"] = new Date();
}

/* and so on… */
```

Each time the site pages are loaded, you can read the values of these custom properties and execute the JS written in them:

```js
const properties = ['--style-1', '--style-2', '--style-3'];
let style = getComputedStyle(document.documentElement);

properties.forEach(property => {
  new Function(style.getPropertyValue(property))();
});
```

I made two conditional style bundles to test it. I included the first and third CSS files into one bundle and only the first file into the other. The second CSS file was not used:

```css
/* ---------------------- File bundle-1.css --------------------- */
@import "style-1.css";
@import "style-3.css";
  
/* ---------------------- File bundle-2.css --------------------- */
@import "style-1.css";
```

Then I created two HTML pages, included the file `bundle-1.css` into one of them, and `bundle-2.css` - into the other. Then I opened the pages.

There were traces in localStorage:

![Browser console with the code running result](/images/1-qw8twl9pzsea27abgzueqq.gif)

The first CSS file left the most recent tag, the third – the older, and the second left no trace at all.

- - -

Instead of a timestamp, you can write to localStorage, for example, a counter, and increment it each time the styles are loaded. You can collect statistics of the CSS files used on the site this way.

This approach is also interesting in that every use of styles will be logged, even if the styles are loaded not from the server, but from the browser cache.