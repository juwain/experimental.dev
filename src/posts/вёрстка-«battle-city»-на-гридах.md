---
layout: layouts/post.njk
title: Вёрстка «Battle city» на гридах
date: 2020-12-16T13:53:46.970Z
tags: []
---
Пришла в голову идея собрать лейаут игры из детства «Battle City» на гридах. Почему бы и нет? Вот что получилось:

![Игровое поле из игры Battle City](/images/1-tp9iififqxpitv1vyfzebq.png "Игровое поле из игры Battle City")

<iframe height="300" style="width: 100%;" scrolling="no" title="Battle city grid layout" src="https://codepen.io/juwain/embed/preview/xYryva?height=300&theme-id=9939&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/juwain/pen/xYryva'>Battle city grid layout</a> by juwain
  (<a href='https://codepen.io/juwain'>@juwain</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

Тут всё просто:

1. Создадим разметку 26 на 26 фракций с шириной и высотой по 16 пикселей.

![Сетка грида игрового поля без элементов](/images/1-2wftcgddjkxd4nwfhuh19q.png "Сетка грида игрового поля без элементов")

```css
.field {
  display: grid;
  grid-template-rows: repeat(26, 16px);
  grid-template-columns: repeat(26, 16px);
}
```

2. Нашпигуем разметку кучей дивов (да, демка лишена какого-либо смысла и поэтому доступности).

Поставим первый блок:

![Сетка грида игрового поля с одним элементом](/images/1-e_h_qbx0fgdulntherdwnw.png "Сетка грида игрового поля с одним элементом")

```css
.wall-1 {
  grid-row: 3 / span 4;
  grid-column: 3 / span 2;
}
```

Этот блок начинается с третьей линии грида по горизонтали и вертикали; в ширину он занимает 2 полосы, а в высоту 4 полосы.

Поставим второй блок…

3. *Нарисуем остальную сову*:

![Сетка грида игрового поля со всеми элементами](/images/1-mb71mzdkiis1175bdb-nhw.png "Сетка грида игрового поля со всеми элементами")

---

Гриды на деле оказались довольно удобным инструментом для того, чтобы собирать что-то прямоугольное. Конечно, основная мощь гридов в адаптивности, и такую демку вполне можно было собрать хоть на флоатах. Но, в отличие от флоатов, гриды способны на больше.