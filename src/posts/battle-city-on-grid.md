---
layout: layouts/post.njk
title: '"Battle City" on Grid Layout'
date: 2021-01-01T13:11:29.275Z
tags:
  - css
  - grid layout
---
I've got the idea – recreate ["Battle City"](https://en.wikipedia.org/wiki/Battle_City) layout with CSS Grid. Why not? Here is a result:

![Playfield from Battle City video game](/images/1-tp9iififqxpitv1vyfzebq.png "Playfield from Battle City video game")

<iframe height="300" style="width: 100%;" scrolling="no" title="Battle city grid layout" src="https://codepen.io/juwain/embed/preview/xYryva?height=300&theme-id=9939&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/juwain/pen/xYryva'>Battle city grid layout</a> by juwain
  (<a href='https://codepen.io/juwain'>@juwain</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

The code creation process isn't much complex:

1. Let's create a layout 26 by 26 fractions with a width and height of 16 pixels each:

![Playfield layout without any elements](/images/1-2wftcgddjkxd4nwfhuh19q.png "Playfield layout without any elements")

```css
.field {
  display: grid;
  grid-template-rows: repeat(26, 16px);
  grid-template-columns: repeat(26, 16px);
}
```

2. Let's fill the markup with a bunch of divs (yes, the demo doesn't have any sense and therefore accessibility).

Ok, let's create the first block:

![Playfield layout with one element](/images/1-e_h_qbx0fgdulntherdwnw.png "Playfield layout with one element")

```css
.wall-1 {
  grid-row: 3 / span 4;
  grid-column: 3 / span 2;
}
```

This block starts at the third horizontal and vertical grid line; it occupies 2 grid lines wide, and 4 lines high.

Ok, let's create the second block…

3. * Draw the rest of the owl*:

![Playfield layout with all elements](/images/1-mb71mzdkiis1175bdb-nhw.png "Playfield layout with all elements")

---

CSS Grid actually turned out to be a pretty handy tool for creating some fixed width rectangular layouts. Of course, the main power of Grid may come out in responsive and adaptive layouts, and such a demo could have been created even with floats. But unlike floats, Grid can do more.