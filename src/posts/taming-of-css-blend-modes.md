---
layout: layouts/post.njk
title: Taming of CSS blend modes
date: 2021-01-06T12:18:00.975Z
tags:
  - css
  - blend modes
---
Recently I came up with the idea to recreate this selective desaturation effect in CSS:

![A scene from the film "Sin City"](/images/1-yltnfq85-8a4wosrnczuiw.jpg "A scene from the “Sin City” movie")

From an artistic point of view, this effect is hackneyed, but I'm interested in the technical side of the issue. That is, the task is to desaturate the entire picture, while leaving the red color intact.

Here's what I've got as a result of the experiments:

![Partially desaturated photo](/images/1-kgtmsjaus_0ntg5ekgecbq.png)

The original photo is [here](https://unsplash.com/@luiskcortes).

<iframe height="300" style="width: 100%;" scrolling="no" title="Blend modes selective desaturation effect" src="https://codepen.io/juwain/embed/preview/mxLJYj?height=300&theme-id=9939&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/juwain/pen/mxLJYj'>Blend modes selective desaturation effect</a> by juwain
  (<a href='https://codepen.io/juwain'>@juwain</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

Let's explore how it works.

I realized that I need to make an SVG-mask for a specific image, then overlay a black-and-white one with a masked part over the color image. As a result, through the holes in the black-and-white image the color will be visible. But such a solution, with creating a mask by hand, is not universal - for each picture you need to make an SVG-mask separately, which is quite difficult.

Looking for a more versatile way to achieve the desired effect, I decided to try making a "mask on-the-fly" from the original image using [blend modes](https://drafts.fxtf.org/compositing-1/#blending).

By default, CSS layers are stacked on top of each other and do not show through:

![Default Normal Blend Mode](/images/1-fmbrjwkcft2bmbjshsi1ka.png "Default Normal Blend Mode")

And you can make the layers not just show one above the other, but "blend" according to a certain algorithm:

![Multiply Blending Mode](/images/1-6na7ll_rcxehlapfxjhg7w.png "Multiply Blending Mode")

Blending modes came to CSS from Photoshop thanks to the developers at Adobe. But not all Photoshop blend modes have moved to CSS - there are 16 layer blending options in total. I will not go into details, you can play with blending modes [in Sarah Sueidan's demo](http://www.sarasoueidan.com/demos/css-blender/).

But back to *selective desaturation*.

First, overlay a copy of the image on top of it using a pseudo-element:

```css
.photo {
  --source: url(https://juwain.github.io/static/girl.jpg);

  /* this is the bottom layer */
  background-image: var(--source);
}

.photo::after {
  /* this is the top layer */
  background-image: var(--source);
}
```

![Layering two identical images in normal blend mode](/images/1-2pxarp_l_m9wepezh056qg.gif)

Next, change the blending mode of the top layer to `lighten`. In this mode, dark areas begin to "shine through", and light ones, on the contrary, remain opaque:

![Lighten blend mode](/images/1-1hg5ykh47im7v-4iu2r0w.png "Lighten blend mode")

The blending mode between layers is set by the `mix-blend-mode` property.

```css
.photo::after {
  mix-blend-mode: lighten;
}
```

We got what we needed: the dark background of the top layer became transparent, but the light areas — didn't:

![Layering two identical images in lighten blend mode](/images/1-l6g6zwl_ibukmfrxoxxawg.gif)

Next, let's desaturate the top layer using a CSS filter:

```css
.photo::after {
  filter: grayscale(1);
}
```

Now the top layer is overlaid on the bottom color photo with light desaturated areas:

![Layering two identical images in lighten mode with one desaturated layer](/images/1-1wzrhh7ti55nrnzeyduagg.gif)

Almost what we need. Now we need to make the light areas of the top layer more opaque.

To achieve this, we'll use the Blending Modes again. In addition to the `mix-blend-mode` property, which "blends" two different layers, there is another property that changes the blend mode - this is `background-blend-mode`. It sets the blending mode of background elements of one layer: image, background color, gradient.

So, let's set the gray background color and the blending mode of the picture with it to `hard-light` for the top layer. This is a mixed mode, it is somewhat similar to the multiply mode - in short, it makes colors brighter. When applied to a black and white image, this mode will make its highlights more contrasting.

Finally, we get the desired result:

![Layering two identical images in lighten mode with one desaturated contrast layer](/images/1-xxrllxel6sf2hpijnms2fg.gif)

---

It remains to figure out which colors in this combination of blending modes become transparent and which do not.

To do this, I made a demo with layers with striped linear gradients:

![Layering colors with desaturtion](/images/1-u-ewvolqzlopqlm6hzzbcw.png)

You can explore it live in [demo](https://codepen.io/juwain/pen/vRjaQb).

That is, the areas of red, blue and purple shades become transparent with the combination of blending modes discussed in this article, and yellow-orange, green and blue shades remain opaque.

This is an interesting feature that you can use somewhere in your projects.