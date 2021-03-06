---
layout: post
title:  "Блокировка прокрутки body для модалок на iOS"
date:   2017-11-11 00:37:29 +0300
category: Frontend
author: "Artamoshkin Maxim"
image: "/assets/prevent-body-scroll.png"
redirect_from: 
    - /frontend/2017/11/10/prevent-scroll-body-modal-ios/
    - /frontend/2017/11/11/prevent-scroll-body-modal-ios/frontend
image_alt: ""
image_height: "auto"
tags: [CSS, iOS, tips]
description: "Одна из известных проблем на iOS с прокручиваемым ``body`` при скролле на фиксированно или абсолютно расположенных элементах."
---

Одна из известных проблем на iOS с прокручиваемым ``body`` при скролле на фиксированно или абсолютно расположенных элементах. 
<!-- more -->
Пример ниже.


 {:.center}
<div><iframe width="560" height="315" src="https://www.youtube.com/embed/IihXWK7nEN8?rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe></div>


На основных браузерах это решается путем добавления ``overflow: hidden`` на ``body``. Но, к сожалению, для iOS этого мало. 
Рабочим решением стало фиксированое ``body`` со 100% шириной и высотой.

```css
body.no-scroll {
   height: 100%;
   width: 100%;
   position: fixed;
   overflow: hidden;
}
```
Этого вполне достаточно, чтобы ``body`` оставался неподвижным и скролл был только на модальном окне.

Буду рад, если эта заметка сократит вам время поиска и решения похожей проблемы.

### UPD ###

У предыдущего метода обнаружен недостаток, в том, что при каждом откритии модального окна страница прокручивается наверх. Да, этот подход сработает на некоторых сайтах, но не на страницах с длинными списками.

Есть другое решение. Но уже с применением js.

Так же устанавливаем для body класс ``no-scroll``, только без свойств ``width`` и ``postion``, они будут динамически задаваться в js.

```css
body.no-scroll {
   height: 100%;
   overflow: hidden;
}
``` 

При открытии модального окна вызываем функцию ``hideScroll``, в которой задаем ``position: fixed``, ``width`` с учетом того, был вертикальный скролл или нет, задается правило ``top`` равное отрицательному значению прокрутки, это и спасает от скролла до верха страницы.

```ts
public hideScroll() {
    this._body.classList.add('no-scroll');

    this._scrollTop = window.pageYOffset; // запоминаем текущую прокрутку сверху
    this._body.style.position = 'fixed';
    if (this._hasScrollbar()) {
    // с учетом горизонтального скролла. Чтобы небыло рывка при открытии модального окна
        this._body.style.width = `calc(100% - ${this._getScrollbarSize()}px)`;
    } else {
        this._body.style.width = '100%';
    }
    this._body.style.top = -this._scrollTop + 'px';
}
```

Необходимые функции для детекции скроллбара и его размера:

```ts
private _getScrollbarSize() { // получение ширины скролла
    let outer = document.createElement('div');
    outer.style.visibility = 'hidden';
    outer.style.width = '100px';
    outer.style.msOverflowStyle = 'scrollbar'; // needed for WinJS apps

    document.body.appendChild(outer);

    let widthNoScroll = outer.offsetWidth;
    // force scrollbars
    outer.style.overflow = 'scroll';

    // add innerdiv
    let inner = document.createElement('div');
    inner.style.width = '100%';
    outer.appendChild(inner);

    let widthWithScroll = inner.offsetWidth;

    // remove divs
    outer.parentNode.removeChild(outer);

    return widthNoScroll - widthWithScroll;
}

private _hasScrollbar() { // проверка на боковой скролл
    return document.body.scrollHeight > document.body.clientHeight;
}
```

При закрытии необходимо убрать класс ``no-scroll`` и занулить инлайновые правила.

```ts
public showScroll() {
    this._body.classList.remove('no-scroll');

    this._body.style.position = '';
    this._body.style.width = '';
    this._body.style.top = '';
    window.scroll(0, this._scrollTop);
}
```
