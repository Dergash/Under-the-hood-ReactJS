## Часть 4

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/4/part-4.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/4/part-4.svg)

<em>4.0 Часть 4 (кликабельно)</em>

### Монтирование дочерних элементов

Захватывающе, правда? Давайте продолжим изучение метода `mount`.

Итак, если `_tag` содержит "сложный" тэг (1) вроде video, form, textarea и им подобных, то он потребует компонента-обертки, который добавит обработчики событий для каждого медиа-события вроде `volumechange` для `audio`-тэгов, или просто обернет нативное поведение таких тэгов как `select`, `textarea` и так далее.
Существуют несколько таких оберток для элементов, например `ReactDOMSelect` и `ReactDOMTextarea` (в папке [src\\renderers\\dom\\client\\wrappers\\](https://github.com/facebook/react/blob/v15.4.2/src/renderers/dom/client/wrappers/)). В нашем примере мы имеет простой `div`, не требующий какой-либо дополнительный обработки.

### Валидация пропсов

Следующий метод валидации вызывается для проверки того, что внутренние `props` выставлены корректно, иначе он бросает ошибки. Например, если мы выставили `props.dangerouslySetInnerHTML` (обычно используется когда мы хотим вставить HTML из строки), а поле `__html` пропустили, то покажется следующая ошибка:

> `props.dangerouslySetInnerHTML` must be in the form `{__html: ...}`.  Please visit https://fb.me/react-invariant-dangerously-set-inner-html for more information.

### Создание HTML элемента

Затем будет создан HTML-элемент с помощью вызова `document.createElement` (3), который создаст нам настоящий `div`. До этого мы все время мы работали только с представлениями элементом в VDOM'е, и сейчас мы в первый раз видим реальный элемент.


### Хорошо, *Часть 4* мы закончили.

Давайте вспомним, как мы сюда попали - глянем еще раз на схему, затем уберем из нее избыточные и менее важные куски, и она станет вот такой:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/4/part-4-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/4/part-4-A.svg)

<em>4.1 Часть 4 упрощенно (кликабельно)</em>

Уберем пустое место и поправим выравнивание:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/4/part-4-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/4/part-4-B.svg)

<em>4.2 Часть 4 упрощенно и с рефакторингом (кликабельно)</em>

Вот теперь отлично. По сути, это все что происходит на этой стадии. Поэтому мы возьмем эту ключевую выдержку из *Части 3* и поместим ее на итоговую схему процесса `монтирования`:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/4/part-4-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/4/part-4-C.svg)

<em>4.3 Часть 4 ключевые моменты (кликабельно)</em>

И мы закончили!


[На следующую страницу: Часть 5 >>](../../../../stack/book/Part-5.md)

[<< На предыдущую страницу: Часть 3](./Part-3.md)


[К оглавлению](../../README.md)
