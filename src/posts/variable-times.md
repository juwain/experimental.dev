---
layout: layouts/post.njk
title: Variable times
date: 2021-01-02T11:10:36.886Z
tags:
  - css
  - custom properties
---
In this article, I'll show you how I created a demo with CSS custom properties and refined my CSS thinking along the way.

Programming an analog clock is not exactly an original idea, but it seemed a convenient way to show the features of custom properties to me.

So, here is what I've got:

![Clock with moving hands](/images/1-mnazppc5rps6ag07n4b7qa.gif)

<iframe height="300" style="width: 100%;" scrolling="no" title="Custom properties analog clock" src="https://codepen.io/juwain/embed/preview/OQQVKE?height=300&theme-id=9939&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/juwain/pen/OQQVKE'>Custom properties analog clock</a> by juwain
  (<a href='https://codepen.io/juwain'>@juwain</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

Here's how it works:

* JS generates data of the current time and sends it to HTML and CSS;
* HTML uses this data to display the time in text format;
* CSS translates numbers into a visually human-readable graphical form.

- - -

Ok, let's start with the markup.

```html
<time class="clock">
  <span class="clock__hand clock__hand--hour"></span>:
  <span class="clock__hand clock__hand--minute"></span>:
  <span class="clock__hand clock__hand--second"></span>

  <svg class="clock__face" aria-hidden="true" role="presentation">
    <circle class="clock__face-stroke"></circle>
  </svg>
</time>
```

A generic container of the clock `<time>`, three hands, and a decorative SVG for drawing the face. Let's think about accessibility right away:

* in "arrow" spans the current time will be written in numbers using JavaScript;
* the **datetime** attribute of the `<time>` element will be set up with the corresponding value;
* the decorative element `<svg>` will be hidden from screen readers with the **aria-hidden** attribute and remove the semantic meaning from it using the **role** attribute.

- - -

CSS time! I will not describe the entire code of the demo (it is better to [play with it live](https://codepen.io/juwain/pen/OQQVKE)) but will focus only on the key points.

Let's define the watch appearance settings - the diameter and thickness of the watch border:

```css
.clock {
  --clock-diameter: 250px;
  --clock-border-width: 5px;
}
```

Then use these variables right here:

```css
.clock {
  width: var(--clock-diameter);
  height: var(--clock-diameter);
  border: var(--clock-border-width) solid #000000;
}
```

So far, everything is not so complex.

Let's create the watch hands. First, let's define the width and height of the hands and position them in the center of the face. By default, arrows will be **1px** wide and **50%** high. Their **top** and **left** coordinates are calculated depending on the width and height of the absolutely positioned arrows:

```css
.clock__hand {
  --hand-width: 1px;
  --hand-height: 50%;

  position: absolute;

  top: calc(50% - var(--hand-height));
  left: calc(50% - calc(var(--hand-width) / 2));

  width: var(--hand-width);
  height: var(--hand-height);
}

/* With custom properties you can carry out
arithmetic computations with calc() */
```

Now let's determine the color of the hands and set their rotation around the clock axis. The clock hands will initially be black and not turned:

```css
.clock__hand {
  background-color: var(--hand-color, #000000);
  transform: rotate(var(--turn, 0turn));
}

/* You can set fallback value 
when using a custom property */
```

And now let's override the arrow characteristics in the block modifiers:

```css
.clock__hand--hour {
  --hand-width: 6px;
  --hand-height: 30%;
}

.clock__hand--minute {
  --hand-width: 4px;
  --hand-height: 40%;
}

.clock__hand--second {
  --hand-width: 2px;
  --hand-height: 45%;
  --hand-color: red;
}
```

Here's what we've got:

![Clock with arrows pointing up](/images/1-vmcybbwxsvwo2mbijghhag.png)

Let's turn the arrows for clarity:

```css
.clock__hand--hour {
  --turn: 0.25turn;
}

.clock__hand--minute {
  --turn: 0.6turn;
}

.clock__hand--second {
  --turn: 0.8turn;
}

/ * Full circle rotation - 1turn,
yes, there is such a unit of measure in CSS * /
```

![Clock with rotated hands](/images/1-kvfqexw993w5dio5qmtfhw.png)

We will also store the current time in three custom properties:

```css
.clock {
  --hours: 10;
  --minutes: 30;
  --seconds: 15;
}
```

Now the question is how to convert hours, minutes, and seconds into units of rotation of a circle. And here school math comes to the rescue.

Let's start with the second hand. There are **60** seconds in a full circle turn - this corresponds to the **1turn** turn value. And we need to find out how much **turn** is in **n**-second. We've got this proportion:

```text
xturn - nsec
1turn - 60sec
```

That is `x = 1turn * nsec / 60sec`. Let's move these calculations into CSS:

```css
.clock__hand--second {
  --seconds-in-minute: 60;
  --turn: calc(1turn * var(--seconds) / var(--seconds-in-minute));
}
```

Let's check what happened:

![Clock with a rotated second hand](/images/1-zlbvum9zcpld5wlbun2gfq.png)

It's ok, 15 seconds, as specified in `--seconds`.

Next is the minute hand. Here everything is exactly the same as for the second hand - there are 60 minutes in an hour:

```css
.clock__hand--minute {
  --minutes-in-hour: 60;
  --turn: calc(1turn * var(--minutes) / var(--minutes-in-hour));
}
```

The picture confirms that everything is ok:

![Clock with rotated second and minute hands](/images/1-84plpxvnt9j1y_gzug5ahw.png)

And the hour hand remained. There are 12 hours in a full circle:

```css
.clock__hand--hour {
  --hours-in-day-half: 12;
  --turn: calc(1turn * var(--hours) / var(--hours-in-day-half));
}
```

![Clock with all hands turned](/images/1-qwfj5dqn1r_hqnmqznpieg.png)

Everything works, but I want the hour hand to move additionally during an hour, depending on the minutes, and not just jump to the next division once an hour:

![Clock with the hour hand movement scheme](/images/1-zer3dia3wbemiic5rjdcsw.png)

Therefore, for the hour hand, let's calculate the additional rotation within one hour by the same principle as before.

Let's create the proportion again: **1/12 turn** — is 1 hour or 60 minutes. We need to find out what **turn** value will be at the **n**-minute of the hour.

```text
xturn - nmin
1/12turn - 60min
```

That is `x = 1/12turn * nmin / 60min`. Let's move these calculations into CSS and add the main clockwise rotation to the additional one:

```css
.clock__hand--hour {
  --hours-in-day-half: 12;
  --hours-turn: calc(1turn * var(--hours) / var(--hours-in-day-half));

  --min-in-hour: 60;
  --minutes-turn: calc((1 / var(--hours-in-day-half)) * 1turn * var(--minutes) / var(--min-in-hour));

  --turn: calc(var(--hours-turn) + var(--minutes-turn));
}
```

We've got exactly what we need:

![Clock with the hour hand movement scheme](/images/1-ejf1ntal7rpl-1dcuphh2g.png)

And this is how it works in a browser:

![Browser demo](/images/1-m9sd2j_baqt0ia-dthepdw.gif)

- - -

Now let's make the result code alive with real data.

We will set values to our custom properties `--hours`, `--minutes` and `--seconds` every second in JS:

```js
const clock = document.querySelector('.clock&');

const setCustomProperty = (name, value) => {
  clock.style.setProperty(`--${name}`, value);
};

const setTimeVariables = (time) => {
  setCustomProperty('hours', time.getHours());
  setCustomProperty('minutes', time.getMinutes());
  setCustomProperty('seconds', time.getSeconds());
};

const setTime = () => setTimeVariables(new Date());

setTime();

setInterval(setTime, 1000);
```

Let's not forget about screen readers and accessibility. We will update the text content of the `<time>` tag elements and its `datetime` attribute:

```js
const clock = document.querySelector('.clock');

const hoursBox = document.querySelector('.clock__hand--hour');
const minutesBox = document.querySelector('.clock__hand--minute');
const secondsBox = document.querySelector('.clock__hand--second');

const setTimeLayout = (time) => {
  hoursBox.textContent = time.getHours();
  minutesBox.textContent = time.getMinutes();
  secondsBox.textContent = time.getSeconds();

  clock.setAttribute('datetime', time.toISOString());
}

const setTime = () => setTimeLayout(new Date());

setTime();

setInterval(setTime, 1000);
```

Here's what it looks like live in dev-tools:

![Working demo in developer tools](/images/1-otieqbjrl2mr-7_n5k6ckg.gif)

- - -

In general, having tried it once, I want to continue using custom properties everywhere – they turned out to be so organic in use. If IE support doesn't stop you, this is a must-have CSS feature!