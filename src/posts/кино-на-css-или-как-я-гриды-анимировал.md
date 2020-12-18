---
layout: layouts/post.njk
title: Кино на CSS, или Как я гриды анимировал
date: 2020-12-18T14:36:23.309Z
---
Если вы имеете дело с CSS, то наверняка задумывались, какие именно CSS-свойства можно анимировать. Если погуглить, то можно найти вот такой список — [«Animatable CSS properties»](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animated_properties). Туда ожидаемо входят всякие цвета, размеры, отступы и прочие свойства, к которым можно применить плавные переходы.

А что будет, если анимацию применить к свойству, значение которого нельзя интерполировать? В [спецификации](https://drafts.csswg.org/css-transitions/#animatable-properties) говорится:

> Since the from and to values cannot be interpolated, the animation is done [in a single step](https://drafts.csswg.org/css-transitions/#step-types).

То есть значение будет переключаться, но без плавного перехода. Получается, что дискретную анимацию можно применить к любому CSS-cвойcтву.

Взяв на вооружение этот факт, я решил собрать небольшую демку с анимированием гридов. Вот такое «кино» получилось:

![Анимация с пришельцем из космоса](/images/1-gf8uq4fl41ixwuatcxycaq.gif "Анимация с пришельцем из космоса")

<iframe height="300" style="width: 100%;" scrolling="no" title="CSS grid movie" src="https://codepen.io/juwain/embed/preview/aqxyxY?height=300&theme-id=9939&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/juwain/pen/aqxyxY'>CSS grid movie</a> by juwain
  (<a href='https://codepen.io/juwain'>@juwain</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

Давайте разберёмся, что тут происходит.

Сначала создадим небольшой грид 10 на 13 фракций:

```css
.field {
  display: grid;
  width: 260px;
  height: 200px;

  grid-template-rows: repeat(10, 1fr);
  grid-template-columns: repeat(13, 1fr);
}
```

В HTML я решил добавить не просто пустые дивы, а абзац с текстовым описанием сюжета «фильма»:

```html
<p class="field">
  <span class="a">В </span>
  <span class="b">этом </span>
  <span class="c">фильме </span>
  <span class="d">рассказывается </span>
  <span class="e">драматическая </span>
  <span class="f">история </span>
  <span class="g">о </span>
  <span class="h">нападении </span>
  <span class="i">космического </span>
  <span class="j">пришельца </span>
  <span class="k">на </span>
  <span class="l">оператора </span>
  <span class="m">снимающего </span>
  <span class="n">этот </span>
  <span class="o">фильм </span>
  <span class="p">на </span> 
  <span class="q">камеру.</span>
</p>
```

Спаны в абзаце — это кирпичики для построения форм в отдельных «кадрах». Текст в них при этом будет скрыт с помощью `font-size: 0;`, а в разметке он останется из соображений доступности.

Для построения отдельных «кадров» я решил использовать свойство `grid-template-areas`. Это свойство описывает раскладку грида в декларативном виде. Вот так, к примеру, будет выглядеть начальный «кадр» будущего фильма:

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

Точками в значении `grid-template-areas` обозначены те области грида, которые не будут закрашены белым, а буквами — те, которые закрашиваются.

Помимо такого визуального объявления внешнего вида грида, нужно задать соответствие блоков в HTML наименованиям в раскладке `grid-template-areas`:

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

/* и так далее… */
```

Вот так это выглядит в браузере:

![Схема разметки грид-областей буквы](/images/1-xpcz-lkdxfta5lzp14ym2q.gif "Схема разметки грид-областей буквы")

По ходу «фильма» значение `grid-template-areas` будет принимать разные значения. Например, такое:

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

Или даже такое:

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

А вот подменять эти значения раскладки грида будет CSS-анимация. Для этого пропишем раскадровку «фильма» в кейфреймах:

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

И затем зададим получившуюся анимацию для грид-контейнера `.field`:

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

Как вы видите, «фильм» изначально поставлен на паузу с помощью свойства `animation-play-state`.

Чтобы запустить анимацию, нужно добавить в HTML триггер, которым удобнее всего будет сделать скрытый чекбокс (чтобы обойтись без скриптов). Добавим его в разметку перед грид-контейнером:

```html
<input class="switch" type="checkbox" id="play-state">
```

Изменение состояния чекбокса будет переключать состояние анимации:

```css
.switch:checked + .field {
  animation-play-state: running;
}
```

А соответствующую этому чекбоксу подпись, которая будет выполнять роль кнопки, запускающей проигрывание анимации, поместим внутрь грид-контейнера `.field`. При этом лейбл нужно спозиционировать абсолютно, чтобы он не влиял на раскладку грида.

К слову, раскладку «кнопки» я тоже решил сделать с помощью грида, который имеет два состояния:

![Схема разметки грид-областей кнопки](/images/1-np6h_i6-1r1i56v-ln9g0g.gif "Схема разметки грид-областей кнопки")

---

Помимо исследования свойств гридов в этой статье я обнаружил для себя новую роль анимации. Её можно использовать не только для плавного изменения внешнего вида элемента, но и как встроенный в CSS таймер, позволяющий управлять состояниями элемента во времени с помощью любых CSS-свойств. Вполне себе такой setTimeout/setInterval.

К сожалению, в Safari и Edge эта фича (по крайней мере применительно к гридам) не заработала. Думаю, что это недоработка самих браузеров. Но, в любом случае, в одной из следующих статей я исследую эту находку поподробнее.