---
layout: layouts/post.njk
title: Stereotypeless animation
date: 2021-01-04T13:20:59.287Z
---
In this article, I will continue my exploration of the discrete animation topic I've touched in [the previous article](/posts/css-movie-or-how-to-animate-the-grid/). I looked through the list of all the CSS properties and selected a couple that I hadn't thought about animating before.

The first test object was the `content` property. It's cool: it can display text, CSS counters, images and display the value of HTML-attributes.

This is the slideshow created with one pseudo-element and its `content` property:

![Slideshow](/images/2-j6_8bedsw_mjapo43eg77a.gif)

<iframe height="300" style="width: 100%;" scrolling="no" title="Animating content property" src="https://codepen.io/juwain/embed/preview/MVYVrj?height=300&theme-id=9939&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/juwain/pen/MVYVrj'>Animating content property</a> by juwain
  (<a href='https://codepen.io/juwain'>@juwain</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

Let's figure out what's what.

There is the paragraph with the class `.info` in the markup, it has a pseudo-element:

```css
.info::before {
  content: ";
}
```

Animation is applied to this pseudo-element:

```css
.info::before {
  animation-name: presentation;
  animation-duration: 60s;
}
```

The animation changes the value of the `content` property step by step. For example, the first "slide" will be like this:

```css
@keyframes presentation {
  0% {
    content: "Hello! The theme of this slideshow:";
  }
}
```

The value of `content` can be surrounded by the keywords `open-quote` and `close-quote`:

```css
content: open-quote "This text will be enclosed in quotes" close-quote ".";
```

The text will be enclosed in quotation marks corresponding to the system language.

CSS counters can also be used as the value of `content`. I display them in Roman numerals in the "slides". Use the `counter-reset` property to initialize the CSS counter:

```css
.info::before {
  counter-reset: my-counter;
}
```

By default, the counter is set to `0`. Now you can increment the counter with the animation in the `counter-increment` property and compute it in the `content` value with the `counter()` function:

```css
@keyframes presentation {
  10% {
    content: counter(my-counter);
    counter-increment: my-counter;
  }
}

/* By default, the counter is incremented by 1 */
```

The second parameter can be passed to the `counter ()` function, which determines the style of the displayed counter (it can be Arabic or Roman numerals, Greek letters, etc). I decided to use capital Roman numerals and wrote it like this:

```css
content: counter(my-counter, upper-roman);
```

Unfortunately, the counter didn't work as I originally thought. When it was incremented in keyframes, it didn't accumulate *somewhere in the CSS memory*. So I just had to hardcode an incrementation of the original counter value at each animation step:

```css
/* my-counter is initially set to 0 */
25% {
  content: counter(my-counter, upper-roman);
  counter-increment: my-counter 2;

  /* The counter is incremented by 2 */
}

50% {
  content: counter(my-counter, upper-roman);
  counter-increment: my-counter 3;

  /* The counter is incremented by 3 */
}
```

Another kind of *content* of the `content` property is an image. It is displayed using the `url()` function - `content: url("path/to/image.jpg")`. I decided to inject the images directly into the styles, so that at the time of the slide show the image was already loaded. I moved the encoded images to custom properties so that the code remains readable:

```css
:root {
  --bg-image-1: url("data:image/jpeg;base64…");
  --bg-image-2: url("data:image/jpeg;base64…");
}

@keyframes presentation {
  35% {
    content: var(--bg-image-1);
  }

  40% {
    content: var(--bg-image-2);
  }
}
```

There are a few more interesting features in the demo that I won't burden the article with (it's better to [explore them live](https://codepen.io/juwain/pen/MVYVrj)).

In order to justify the inaccessibiliy of all this disgrace somehow, there is a hidden text describing the content of the presentation in the `.info` paragraph in HTML. 🤓

---

The second property for experimenting with discrete animation was `background-image`.

The result is ... Mmmmm, so:

![Picture animation](/images/1-l7erp1dshie6tpefm_l3aa.gif)

<iframe height="300" style="width: 100%;" scrolling="no" title="Animating background-image property" src="https://codepen.io/juwain/embed/preview/GxgdQo?height=300&theme-id=9939&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/juwain/pen/GxgdQo'>Animating background-image property</a> by juwain
  (<a href='https://codepen.io/juwain'>@juwain</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

The action has two parts.

The first part is the animation of the "cat":

```css
.frame {
  animation-name: movie;
  animation-duration: 1s;
  animation-iteration-count: infinite;
}
```

This is literally a quick change of "frames" with pictures, like in a movie:

```css
:root {
  --image-1: url("data:image/png;base64…");
  --image-2: url("data:image/png;base64…");
  --image-3: url("data:image/png;base64…");
}

@keyframes movie {
  0% {
    background-image: var(--image-1);
  }

  10% {
    background-image: var(--image-2);
  }

  20% {
    background-image: var(--image-3);
  }
  …
```

Just 10 "frames" are replaced per second. Sufficient speed to create the illusion of continuous movement.

And the second part is animation of the gradient on the background (I put it in a pseudo-element):

```css
.frame::before {
  animation-name: color-change;
  animation-duration: 0.5s;
  animation-iteration-count: infinite;
}
```

The value of the background gradient will be controled with a `—-colors` custom property:

```css
.frame::before {
  background-image: radial-gradient(var(--colors));
}
```

The value of this custom property will be changed in keyframes.

The first gradient value will be a classic rainbow sequence (*Roy G…*). Then, in the next "frame", all the colors are shifted one forward, with the last color (purple) moving to the beginning:

```css
@keyframes color-change {
  0% {
    --colors: red, orange, yellow, green, skyblue, blue, purple;
  }

  14% {
    --colors: purple, red, orange, yellow, green, skyblue, blue;
  }

  28% {
    --colors: blue, purple, red, orange, yellow, green, skyblue;
  }
  …
}
```

Etc. There are seven colors in total, so the `color-change` animation is divided into seven keyframes.

So it goes 🌈.

Discrete animation has a lot of potential, so it's a pity that it still doesn't fully work in Safari and Firefox, even though it's written in the [spec](https://drafts.csswg.org/css-transitions/#animatable-properties).
