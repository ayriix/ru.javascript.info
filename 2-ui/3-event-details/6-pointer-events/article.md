# События указателя

События указателя - это современный способ обработки ввода с помощью различных указывающих устройств, таких как мышь, перо/стилус, сенсорный экран и так далее.

## Краткая история

Сделаем небольшой обзор, чтобы вы поняли общую картину и место Событий Указателя среди других типов событий.

- Давным-давно, в прошлом, существовали только события мыши

    Затем появились сенсорные устройства. Чтобы старый код продолжал корректно работать, они тоже генерировали события мыши. Например, касание генерирует событие `mousedown`. Но события мыши не были достаточно хороши, поскольку сенсорные устройства во многих аспектах мощнее. Например, они позволяют касаться экрана сразу в нескольких местах, а события мыши не имеют для этого каких-либо свойств.

- Поэтому были внедрены такие события касаний, как `touchstart`, `touchend`, `touchmove`, которые имеют специфичные для касаний свойства (мы не будем здесь рассматривать их подробно, потому что события указателя ещё лучше).

    Но этого всё ещё было недостаточно, так как существует много других устройств, таких как перо, у которых есть свои особенности. Кроме того, код, который отслеживал бы и события касаний и события мыши, просто сложнее писать.

- Для решения этих задач был внедрён стандарт Pointer Events. Он предоставляет единый набор событий для всех типов указывающих устройств.

К настоящему времени спецификация [Pointer Events Level 2](https://www.w3.org/TR/pointerevents2/) поддерживается всеми основными браузерами, а [Pointer Events Level 3](https://w3c.github.io/pointerevents/) находится в разработке. Если вы не разрабатываете под браузеры Internet Explorer 10, Safari 12, или более ранние версии, больше нет необходимости использовать события мыши или касаний – можно переходить на события указателя.

При этом, у них есть некоторые важные особенности, которые нужно знать, чтобы правильно использовать, не сталкиваясь с сюрпризами. Мы отметим их в этой статье.

## Типы событий указателя

События указателя именуются подобно событиям мыши:

| Событие указателя | Событие мыши |
|---------------|-------------|
| `pointerdown` | `mousedown` |
| `pointerup` | `mouseup` |
| `pointermove` | `mousemove` |
| `pointerover` | `mouseover` |
| `pointerout` | `mouseout` |
| `pointerenter` | `mouseenter` |
| `pointerleave` | `mouseleave` |
| `pointercancel` | - |
| `gotpointercapture` | - |
| `lostpointercapture` | - |

Как мы видим, для каждого `mouse<event>` есть соответствующий `pointer<event>`, который играет аналогичную роль. Также есть 3 дополнительных события указателя, у которых нет соответствующего аналога `mouse...`, скоро мы их объясним.

```smart header="Замена `mouse<event>` на `pointer<event>` в коде"
Мы можем заменить события `mouse<event>` на `pointer<event>` в коде и быть увереными, что с мышью по-прежнему всё будет работать нормально.

Поддержка сенсорных устройств будет "волшебным" улучшением, но возможно, нам понадобится добавить `touch-action: none` в CSS. Подробнее смотрите ниже в разделе про `pointercancel`.
```

## Свойства события указателя

События указателя содержат те же свойства, что и события мыши, такие как `clientX/Y`, `target` и т.п., а также несколько других:

- `pointerId` – уникальный идентификатор указателя, вызвавшего событие.
    
    Позволяет обрабатывать несколько указателей, например сенсорный экран со стилусом и мультитач (объясняется ниже).
- `pointerType` – тип указывающего устройства. Должен быть строкой с одним из значений: "mouse", "pen" или "touch".

    Мы можем использовать это свойство, чтобы определять разное поведение для разных типов указателей.
- `isPrimary` – `true` для основного указателя (первое касание, если их несколько).

Для указателей, которые измеряют область контакта и степень надавливания, например палец на сенсорном экране, могут быть полезны дополнительные свойства:

- `width` - ширина области соприкосновения указателя с устройством. Если не поддерживается, например мышью, то всегда равно `1`.
- `height` - высота области соприкосновения указателя с устройством. Если не поддерживается, например мышью, то всегда равно `1`.
- `pressure` - степень давления указателя в диапазоне от 0 до 1. Для устройств, которые не поддерживают давление, равняется либо `0.5` (нажато) либо `0`.
- `tangentialPressure` - нормализованное тангенциальное давление.
- `tiltX`, `tiltY`, `twist` - специфичные для пера свойства, описывающие положение пера относительно сенсорной поверхности.

Эти свойства не очень хорошо поддерживаются на разных устройствах, поэтому редко используются. При необходимости, подробности можно найти в [спецификации](https://w3c.github.io/pointerevents/#pointerevent-interface).

## Мультитач

Одной из функций, которую абсолютно не поддерживают события мыши, является мультитач: возможность  касаться сразу нескольких мест на телефоне или планшете, или выполнять специальные жесты.

События указателя позволяют обрабатывать мультитач с помощью свойств `pointerId` и `isPrimary`.

Вот что происходит, когда пользователь касается экрана в одном месте, а затем другим поальцем в другом:

1. При первом касании:
    - происходит событие `pointerdown` со свойством `isPrimary=true` и некоторым `pointerId`.
2. При втором и последующих касаниях:
    - происходит событие `pointerdown` со свойством `isPrimary=false` и уникальным `pointerId` для каждого касания.

Обратите внимание: `pointerId` присваивается не на всё устройство, а для каждого касающегося пальца. Если коснуться экрана 5 пальцами одновременно, получим 5 событий `pointerdown`, каждое со своими координатами и индивидуальным `pointerId`.

События, связанные с первым пальцем, всегда содержат свойство `isPrimary=true`.

Мы можем отслеживать несколько касающихся экрана пальцев, используя их `pointerId`. Когда пользовтаель перемещает, а затем убирает палец, получаем события `pointermove` и `pointerup` с тем же `pointerId`, что и при событии `pointerdown`.

```online
Здесь представлено демо, фиксирующее события `pointerdown` и `pointerup`:

[iframe src="multitouch" edit height=200]

Обратите внимание: чтобы увидеть разницу, вам нужно использовать устройство с сенсорным экраном, такое как телефон или планшет. Для устройств с одним касанием, таких как мышь, всегда будет один и тот же `pointerId` со свойством `isPrimary=true`, для всех событий указателя.
```

## Событие: pointercancel

Раньше мы уже упоминали о важности `touch-action: none`. Теперь давайте объясним, почему: пропуск этого может привести к сбоям в работе интерфейса.

Событие `pointercancel` происходит, когда текущее действие с указателем по какой-то причине прерывается, и события указателя больше не генерируются.

К таким причинам можно отнести: 
- Указывающее устройство было отключено.
- Изменилась ориентация устройства (перевернули планшет). 
- Браузер решил сам обработать действие, считая его жестом мыши, масштабированием, зумированием, или чем-то еще.

Мы продемонстрируем `pointercancel` на практическом примере, чтобы увидеть, как это влияет на нас.

Допустим, мы реализуем перетаскивание для нашего мяча, как и в начале статьи <info:mouse-drag-and-drop>.

Вот последовательность действий пользователя и соответствующие события:

1) Пользователь нажимает кнопку мыши на изображении, чтобы начать перетаскивание
    - происходит событие `pointerdown`
2) Затем начинается перемещение изображения
    - происходит событие `pointermove` (возможно, несколько раз)
3) Сюрприз! Браузер содержит встроенную поддержку "Drag'n'Drop" для изображений, которая запускает и перехватывает процесс перетаскивания, генерируя при этом событие `pointercancel`.
    - Теперь браузер сам обрабатывает перетаскивание изображения. У пользователя есть возможность перетащить изображение мяча даже за пределы браузера, в свою почтовую программу или файловый менеджер
    - Событий `pointermove` для нас больше нет

Таким образом, браузер "перехватывает" действие: запускается событие `pointercancel` и после этого события `pointermove` больше не генерируются.

```online
Далее представлено демо с событиями указателя (только `up/down`, `move` и `cancel`), которые записываются в элемент textarea

[iframe src="ball" height=240 edit]
```

Мы бы хотели реализовать собственное перетаскивание, поэтому давайте скажем браузеру не перехватывать его.

**Предотвращение действий браузера по умолчанию, чтобы избежать `pointercancel`.**

Нужно выполнить два действия:

1. Предотвратить запуск встроенного drag'n'drop
    - Мы можем сделать это, задав `ball.ondragstart = () => false`, как описано в статье <info:mouse-drag-and-drop>.
    - Это работает для событий мыши.
2. Для устройств с сенсорным экраном тоже существуют действия браузера, связанные с касаниями. С ними у нас тоже возникнут проблемы
    - Мы можем предотвратить их, добавив в CSS свойство `#ball { touch-action: none }`.
    - Затем наш код начнёт корректно работать на устройствах с сенсорным экраном

После того, как мы это сделаем, события будут работать как и ожидается, браузер не будет перехватывать процесс и не будет вызывать событие `pointercancel`.

```online
В данном демо произведены нужные действия:

[iframe src="ball-2" height=240 edit]

Как вы можете видеть, событие `pointercancel` больше не срабатывает.
```

Теперь мы можем добавить код для перемещения мяча и наш drag'n'drop будет работать и для мыши и для устройств с сенсорным экраном.

## Захват указателя

Захват указателя - это специальная функция событий указателя.

Идея состоит в том, что мы можем "привязать" все события с определённым `pointerId` к определённому элементу. Затем все последующие события с таким же `pointerId` будут перенаправлены на этот же элемент. То есть, браузер делает элемент целевым и вызывает связанные с ним обработчики, независимо от того, где на самом деле произошло событие.

Связанные методы:
- `elem.setPointerCapture(pointerId)` - привязывает данный `pointerId` к `elem`.
- `elem.releasePointerCapture(pointerId)` - отвязывает данный `pointerId` от `elem`.

Такая привязка удерживается не долго. Она автоматически удаляется после событий `pointerup` или `pointercancel`, или когда целевой `elem` удаляется из документа.

А теперь давайте разберём, когда это может пригодиться.

**Захват указателя используется для упрощения действий перетаскивания**

Давайте вспомним проблему, с которой столкнулись при создании кастомного слайдера в статье <info:mouse-drag-and-drop>.

1. Сначала пользователь нажимает на ползунок слайдера (`pointerdown`), чтобы начать его перетаскивать.
2. ...Но при перемещении курсор может выйти за пределы слайдера

Но мы продолжаем отслеживать события `pointermove` и перемещать ползунок, пока не произойдёт событие `pointerup`, даже если указатель находится за пределами слайдера.

[Раньше](info:mouse-drag-and-drop) для обработки событий `pointermove`, которые происходят за пределами слайдера, мы отслеживали события `pointermove` на всём `document`.

Захват указателя предоставляет альтернативное решение: мы можем вызвать `thumb.setPointerCapture(event.pointerId)` в обработчике события `pointerdown`, благодаря чему все будущие события указателя будут перенаправляться на `thumb`, пока не произойдёт событие `pointerup`.

То есть, обработчики событий будут вызываться на элементе `thumb`, а `event.target` всегда будет ссылаться на элемент `thumb`, даже если пользовтаель перемещает указатель по всему документу. Таким образом, мы можем отслеживать срабатывание события `pointermove` на элементе `thumb` вне зависимости от того, где оно происходит по факту.

Вот основной код:

```js
thumb.onpointerdown = function(event) {
  // все события указателя перенаправить на меня (пока не произойдёт 'pointerup')
  thumb.setPointerCapture(event.pointerId);
};

thumb.onpointermove = function(event) {
  // перемещайте ползунок: отслежиются события на ползунке, поскольку все события перенаправлены на него
  let newLeft = event.clientX - slider.getBoundingClientRect().left;
  thumb.style.left = newLeft + 'px';
};

// примечание: нет необходимости вызывать thumb.releasePointerCapture, 
// при срабатывании события 'pointerup' это происходит автоматически
```

```online
Полное демо:

[iframe src="slider" height=100 edit]
```

**В конечном счёте код становится чище, поскольку нам больше не нужно добавлять/удалять обработчики для всего документа. Это делает захват указателя.**

Существует два связанных события указателя:

- `gotpointercapture` срабатывает, когда элемент использует `setPointerCapture` для включения захвата.
- `lostpointercapture` срабатывает при освобождении от захвата: явно с помощью `releasePointerCapture` или автоматически, когда происходит событие `pointerup`/`pointercancel`.

## Заключение

События указателя позволяют одновременно обрабатывать действия с помощью мыши, касания и пера.

События указателя расширяют события мыши. Мы можем заменить `mouse` на `pointer` в названиях событий и код продолжит работать для мыши, при этом получив лучшую поддержку и других типов устройств.

Не забывайте добавлять `touch-events: none` в CSS для элементов, с которыми мы взаимодействуем, иначе браузер будет перехватывать множество действий с помощью касаний, а события указателя генерироваться не будут.

Дополнительные возможности событий указателя:

- Поддержка мультитач с помощью `pointerId` и `isPrimary`.
- Специцичные для определённых устройств свойства, такие как `pressure`, `width/height` и другие.
- Захват указателя: мы можем перенаправить все события указателя на определённый элемент до наступления события `pointerup`/`pointercancel`.

На данный момент события указателя поддерживаются в основных браузерах, поэтому мы можем безопасно переходить на них, если нет необходимости в поддержке IE10 и Safari 12. И даже для этих браузеров есть полифилы, которые добавляют эту поддержку.