## Часть 12

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/12/part-12.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/12/part-12.svg)

<em>12.0 Часть 12 (кликабельно)</em>

### Если компоненты в самом деле надо обновлять..

Итак, самое начало обновления, хорошее место чтобы вызвать `componentWillUpdate`, если он был определен (1). Затем, перерисовать компонент и поставить в очередь еще один широко известный метод `componentDidUpdate` (вызов отложенный, потому что его надо вызвать в самом конце обновления).
Что насчет ререндера? По сути, здесь нам надо вызвать у компонента метод `render` и соответствующим образом обновить DOM. Первым шагом мы вызываем `render` (2) у экземпляра нашего компонента (`ExampleApplication`) и сохраняем результат рендера (React-элементы, которые являются возвращаемым значением этого метода). Затем, мы сравниваем результат с предыдущими отрендеренными элементами и смотрим, надо ли обновлять сам DOM.

Собственно это и есть одна из киллер-фич React'а, позволяющая избежать ненужных обновлений DOM'а, что и обеспечивает его производительность.
Согласно комментарию в методе `shouldUpdateReactComponent` (3):
> ‘определяет, нужно ли обновить существующий экземпляр или его надо уничтожить или заменить новым экземпляром’.

Грубо говоря, этот метод проверяем, надо ли заменить элемент целиком, то есть старый нужно сначала `размонтировать`, затем новый элемент (полученный из `render`) надо примонтировать, и полученную из `mount` разметку надо вставить вместо текущего элемента, или частично применить ее. Одна из основных причин целиком заменять элемент это случай, когда новый элемент пустой (был удален в `render') или его тип отличается, то есть был `div`, а стал какой-то другой. Давайте посмотрим код, он простой. 

```javascript
///src/renderers/shared/shared/shouldUpdateReactComponent.js#25

function shouldUpdateReactComponent(prevElement, nextElement) {
    var prevEmpty = prevElement === null || prevElement === false;
    var nextEmpty = nextElement === null || nextElement === false;
    if (prevEmpty || nextEmpty) {
        return prevEmpty === nextEmpty;
    }

    var prevType = typeof prevElement;
    var nextType = typeof nextElement;
    if (prevType === 'string' || prevType === 'number') {
        return (nextType === 'string' || nextType === 'number');
    } else {
        return (
            nextType === 'object' &&
            prevElement.type === nextElement.type &&
            prevElement.key === nextElement.key
        );
    }
}
```
https://github.com/facebook/react/blob/v15.4.2/ssrc/renderers/shared/shared/shouldUpdateReactComponent.js#L25

Хорошо, в случае с нашим `ExampleApplication` мы просто обновили `state` и на `render` это особо не повлияло, так что давайте пойдем посмотрим второй сценарий, где есть само обновление.

### Хорошо, *Часть 12* мы закончили.

Давайте вспомним, как мы сюда попали - глянем еще раз на схему, затем уберем из нее избыточные и менее важные куски, и она станет вот такой:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/12/part-12-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/12/part-12-A.svg)

<em>12.1 Часть 12 упрощенно (кликабельно)</em>

Уберем пустое место и поправим выравнивание:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/12/part-12-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/12/part-12-B.svg)

<em>12.2 Часть 12 упрощенно и с рефакторингом (кликабельно)</em>

Вот теперь отлично. По сути, это все что происходит на этой стадии. Поэтому мы возьмем эту ключевую выдержку из *Части 12* и поместим ее на итоговую схему процесса `обновления`:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/12/part-12-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/12/part-12-C.svg)

<em>12.3 Часть 12 ключевые моменты (кликабельно)</em>

И мы закончили!


[На следующую страницу: Часть 13 >>](./Part-13.md)

[<< На предыдущую страницу: Часть 11](./Part-11.md)


[К оглавлению](./README.md)
