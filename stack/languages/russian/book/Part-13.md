## Часть 13

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/13/part-13.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/13/part-13.svg)

<em>13.0 Часть 13 (кликабельно)</em>

### Получить компонент (следующий элемент, если быть точным)

С помощью `ReactReconciler.receiveComponent` React вызывает метод получения компонента `receiveComponent` из класса `ReactDOMComponent` и передает в него следующий элемент. Экземпляр DOM-компонента перезаписывается этим следующим элементом, и вызывается метод обновления. `updateComponent` выполняет две основных задачи: обновляет свойства DOM и дочерние DOM-элементы, опираясь на `prev` и `next` свойства. К счастью, мы уже проанализировали как работает метод `_updateDOMProperties` ([src\renderers\dom\shared\ReactDOMComponent.js#946](https://github.com/facebook/react/blob/v15.4.2/src/renderers/dom/shared/ReactDOMComponent.js#L946)). Как вы помните, он в основном обрабатывает свойства и атрибуты HTML-элементов, вычисляет стили, разбирается с обработчиками событий и т.д. Нам тут остается только посмотреть `_updateDOMChildren` ([src\renderers\dom\shared\ReactDOMComponent.js#1076](https://github.com/facebook/react/blob/v15.4.2/src/renderers/dom/shared/ReactDOMComponent.js#L1076)).

### ### Хорошо, *Часть 13* мы закончили. Коротенькая )

Давайте вспомним, как мы сюда попали - глянем еще раз на схему, затем уберем из нее избыточные и менее важные куски, и она станет вот такой:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/13/part-13-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/13/part-13-A.svg)

<em>13.1 Часть 13 упрощенно (кликабельно)</em>

Уберем пустое место и поправим выравнивание:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/13/part-13-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/13/part-13-B.svg)

<em>13.2 Часть 13 упрощенно и с рефакторингом (кликабельно)</em>

Вот теперь отлично. По сути, это все что происходит на этой стадии. Поэтому мы возьмем эту ключевую выдержку из *Части 13* и поместим ее на итоговую схему процесса `обновления`:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/13/part-13-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/13/part-13-C.svg)

<em>13.3 Часть 13 ключевые моменты (кликабельно)</em>

И мы закончили!


[На следующую страницу: Часть 14 >>](./Part-14.md)

[<< На предыдущую страницу: Часть 12](./Part-12.md)


[К оглавлению](./README.md)
