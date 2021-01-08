---
layout: layouts/post.njk
title: Blending modes + Grid = Love
date: 2021-01-07T15:34:07.920Z
---
In this article, I continued to experiment with blend modes (another experiment was [in the previous article](/posts/taming-of-css-blend-modes/)) and decided to make something more "layout"-like with grids.

An advertisement in the subway with tinted areas of the image led me to this idea.

Here's what I've got:

![Stylized picture of a horse](/images/1-uik4w-qsklevjrxwhvb2ew.png)

<iframe height="300" style="width: 100%;" scrolling="no" title="Blend modes + grids = ❤️" src="https://codepen.io/juwain/embed/preview/xWQYXj?height=300&theme-id=9939&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/juwain/pen/xWQYXj'>Blend modes + grids = ❤️</a> by juwain
  (<a href='https://codepen.io/juwain'>@juwain</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

Let's figure it out 🦄.

So let's start with the markup:

```html
<article class="brand">
  <img class="brand__img" src="img/horse.jpg" alt=">
  <section class="brand__message">
    <h1 class="brand__heading">heading</h1>
    <p class="brand__text">text</p>
  </section>
</article>
```

The `.brand` block is displayed as a grid. The grid has 20 "fluid" rows and columns + 1 side fixed width column with text:

```css
.brand {
  display: grid;
  grid-template-columns: repeat(20, 1fr) 300px;
  grid-template-rows: repeat(20, 1fr);
}
```

![Grid container schema](/images/1-rrgvefg99fohm2jmz8owpw.png)

We have created the grid layout, now we will put the existing elements in this grid.

The beauty of a grid layout is that elements can easily overlap within it as if they were absolutely positioned. In our case, the `.brand__img` image will stretch to the full width and height of the grid, and the side column will be positioned on top of it:

```css
/* Picture:
  goes by column - from the first column to the last;
  goes by rows - from the first row to the last */

.brand__img {
  grid-column: 1 / -1;
  grid-row: 1 / -1;
}

/* Column with text:
  goes by column - from the penultimate column to the last;
  goes by rows - from the first row to the last */

.brand__message {
  grid-column: -2 / -1;
  grid-row: 1 / -1;
}
```

Negative index values mean the reverse order of counting: `-1` - the last row or column,` -2` - next to the last, and so on. This is convenient because you don't need to hardcode the total number of lines in the grid when specifying the "coordinates" of a grid item.

![Grid container schema](/images/1-ixusysswrzwew3fcr8kia.png)

![Grid container schema](/images/1-fporvn8iofyobaifwncvva.png)

There is one more detail. you need to select a large enough picture "with a margin" in size to leave the opportunity for our grid to be flexible, not fixed. In addition, you need to make the `img` image behave as if it were a background image in the `background-size: cover` display mode. Fortunately, this is easily handled by the `object-fit` property:

```css
.brand__img {
  object-fit: cover;
  object-position: center left;

  width: 100%;
  height: 100%;
}
```

The result is an excellent responsive layout:

![Responsive grid container demo](/images/1-kl2in55agnjbvozrxwohna.gif)

---

It remains unclear why we needed 20 rows and columns, and how to color a horse with the existing markup?

And here pseudo-elements come to the rescue. Why are `.brand::before` and` .brand::after` not elements of the grid? They will become quite full-fledged participants of the layout, if you set any value of the property `content` to them. Interestingly, although pseudo-elements are not positioned absolutely or relatively, the `z-index` property is applied to them, that is, you can specify the order of the" layers "of grid elements.

```css
.brand::before,
.brand::after {
  content: ";
  z-index: 1;
}

.brand::before {
  grid-column: 1 / -2;
  grid-row: 1 / 11;

  background-color: honeydew;
}

.brand::after {
  grid-column: 1 / -2;
  grid-row: -1 / 11;

  background-color: orchid;
}
```

In general, the top and bottom "color layers" will divide the rows of the grid in half, and they will occupy all the space up to the side column:

![Grid container schema](/images/1-z9x0-dahj17vvkbr97absa.png)

![Grid container schema](/images/1-oj2utlgegk5k0fi-1-qmyw.png)

It remains to add a little magic of the blending modes, and the horse will be colored:

```css
.brand::before,
.brand::after {
  mix-blend-mode: hue;
}
```

![Stylized image of a horse](/images/1-uik4w-qsklevjrxwhvb2ew.png)

The `hue` blend mode takes the hue of the color of the upper layer (in our case, these are the solid colors `honeydew` and `orchid`), and the saturation and luminosity from the lower one, that is, in a simple way: the colored areas will be tinted, but the black and white will remain so the same. [Last time](https://medium.com/@juwain/selective-desaturation-with-blend-modes-54eb1143f105) I got selective desaturation, but now it's something like "selective toning".

By the way, I love named colors in CSS! Just think how great it sounds - `honeydew` 🍀.

In addition, I tried other combinations of blend modes and layouts, it turned out pretty cool too:

![Stylized image of a horse](/images/1-b9uclpu0q2krv_ybgs4maw.png)

![Stylized image of a horse](/images/1-eaki6r56jqijre5jnqcrbg.png)

![Stylized image of a horse](/images/1-ldqafulmngzx_-a-pek0zw.png)

![Stylized image of a horse](/images/1-9t3rltz5ahjouz8ik-7xrw.png)
