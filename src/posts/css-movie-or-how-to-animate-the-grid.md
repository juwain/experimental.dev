---
layout: layouts/post.njk
title: CSS movie, or How to animate the Grid
date: 2021-01-03T12:25:38.814Z
tags:
  - css
  - grid layout
  - animation
---
If you are dealing with CSS, then you probably wondered which CSS properties you can animate. If you google, you can find this list - ["Animatable CSS properties"](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animated_properties). As expected, this includes all sorts of colors, sizes, margins, and other properties to which you can apply smooth transitions.

What happens if the animation will be applied to a property which value cannot be interpolated? The [spec](https://drafts.csswg.org/css-transitions/#animatable-properties) says:

> Since the from and to values cannot be interpolated, the animation is done [in a single step](https://drafts.csswg.org/css-transitions/#step-types).

That is, the value will switch, but without a smooth transition. It turns out that discrete animation can be applied to any CSS property.

Taking this fact into account, I decided to create a small demo with grid animation. I've got such a "movie":

![Alien from space animation](/images/1-gf8uq4fl41ixwuatcxycaq.gif)

<iframe height="300" style="width: 100%;" scrolling="no" title="CSS grid movie" src="https://codepen.io/juwain/embed/preview/aqxyxY?height=300&theme-id=9939&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/juwain/pen/aqxyxY'>CSS grid movie</a> by juwain
  (<a href='https://codepen.io/juwain'>@juwain</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

Let's take a look at what's going on here.

First, let's create a small 10 by 13 fractions grid:

```css
.field {
  display: grid;
  width: 260px;
  height: 200px;

  grid-template-rows: repeat(10, 1fr);
  grid-template-columns: repeat(13, 1fr);
}
```

I decided to add not just empty divs into HTML, but a paragraph with a "movie script":

```html
<p class="field">
  <span class="a">This </span>
  <span class="b">movie </span>
  <span class="c">tells </span>
  <span class="d">the dramatic </span>
  <span class="e">story </span>
  <span class="f">about </span>
  <span class="g">the attack </span>
  <span class="h">of </span>
  <span class="i">the </span>
  <span class="j">space </span>
  <span class="k">invader </span>
  <span class="l">on </span>
  <span class="m">the </span>
  <span class="n">cameraman </span>
  <span class="o">filming </span>
  <span class="p">the </span> 
  <span class="q">movie.</span>
</p>
```

Spans in a paragraph are the "bricks" in separate "frames". The text will be hidden with `font-size: 0;`, and it will remain in the HTML because of accessibility reasons.

I decided to use the `grid-template-areas` property to draw individual "frames". This property describes the layout of the grid in a declarative way. This is how, for example, the opening "frame" of a future movie will look like:

```css
grid-template-areas:
  ". . . . a a a a a . . . ."
  ". . . . . . . . b . . . ."
  ". . . . . . . . b . . . ."
  ". . . . . . . . b . . . ."
  ". . . . c c c c c . . . ."
  ". . . . . . . . d . . . ."
  ". . . . . . . . d . . . ."
  ". . . . . . . . d . . . ."
  ". . . . . . . . d . . . ."
  ". . . . e e e e e . . . .";
```

The dots in the `grid-template-areas` value represents those areas of the grid that will not be filled with white, and the letters represent those that will be painted over.

In addition to such a visual declaration of the grid appearance, we need to connect blocks in HTML with names in the `grid-template-areas` layout:

```css
.a {
  grid-area: a;
}

.b {
  grid-area: b;
}

.c {
  grid-area: c;
}

/* and so on… */
```

This is how it looks in the browser:

![Layout scheme for grid areas of the letter](/images/1-xpcz-lkdxfta5lzp14ym2q.gif)

As the "movie" progresses, the value of `grid-template-areas` will take on different values. For example, this:

```css
grid-template-areas:
  ". . . . . . . . . . . . ."
  ". . . . . . . . . . . . ."
  ". a a a . c . e . f g g ."
  ". . b . . c . e . f . . ."
  ". . b . . c d e . f h h ."
  ". . b . . c . e . f . . ."
  ". . b . . c . e . f i i ."
  ". . . . . . . . . . . . ."
  ". . . . . . . . . . . . ."
  ". . . . . . . . . . . . .";
```

Or even this:

```css
grid-template-areas:
  ". . . . . . . . . . . . ."
  ". . . a . . . . . b . . ."
  ". c . . d . . . e . . g ."
  ". c . f f f f f f f . g ."
  ". c h h . i i i . j j g ."
  ". c k k k k k k k k k g ."
  ". . k k k k k k k k k . ."
  ". . . l . . . . . m . . ."
  ". . n . . . . . . . o . ."
  ". . . . . . . . . . . . .";
```

CSS animation will replace the grid layout values. To do this, let's write the "storyboard" of the "movie" in keyframes:

```css
@keyframes movie {
  0% {
    grid-template-areas: …;
  }
  
  10% {
    grid-template-areas: …;
  }
  
  …

  100% {
    grid-template-areas: …;
  }
}
```

And then we assign the animation to the `.field` grid container:

```css
.field {
    animation-name: movie;
    animation-duration: 10s;
    animation-play-state: paused;
    animation-timing-function: step-end;
    animation-iteration-count: 1;
    animation-fill-mode: both;
}
```

As you can see, the "movie" is initially paused with the `animation-play-state` property.

To start the animation, we need to add a trigger to the HTML (it will be convenient to use a hidden checkbox to implement functionality without scripts). Let's add it to the markup before the grid container:

```html
<input class="switch" type="checkbox" id="play-state">
```

Changing the state of the checkbox will toggle the state of the animation:

```css
.switch:checked + .field {
  animation-play-state: running;
}
```

And the label connected to this checkbox, which will act as a button that starts playing the animation, will be placed inside the `.field` grid container. In this case, the label must be absolutely positioned so that it does not affect the layout of the grid.

By the way, I also decided to make the layout of the "button" with a grid, which has two states:

![Layout scheme for grid areas of the letter](/images/1-np6h_i6-1r1i56v-ln9g0g.gif)

- - -

In addition to exploring features of CSS Grid, I've discovered a new role for animation in this article. It can be used not only to smoothly change the appearance of an element but also as a built-in CSS-timer that allows you to control the states of an element over time using any CSS properties. It's like a setTimeout / setInterval in JS.

Unfortunately, this feature (at least with grid) didn't work in Safari. I think this is a bug in the browser. But, in any case, in one of the next articles, I will explore this feature in more detail.