## Part 0

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0.svg)

<em>0.0 Часть 0 (кликабельно)</em>

### ReactDOM.render
Итак, давайте начнем с вызова ReactDOM.render.

Точка входа - ReactDOM.render, отсюда наше приложение начинает отрисовку в DOM. Чтобы проще было отлаживать, я написал простой компонент `<ExampleApplication/>`. Первым делом здесь происходит **преобразование JSX в React-элементы**. Элементы - довольно простая штука, по сути это обычные объекты (plain objects) с очень простой структурой. render компонента возвращает именно эти объекты и ничего больше. Некоторые поля вам наверняка уже знакомы, например `props`, `key` и `ref`. Поле `type` указывает на объект разметки, описываемой JSX'ом - в нашем случае оно указывает на  класс `ExampleApplication`, но там могла бы быть и просто строка `button` для `<button>`-тэга и ему подобных. Также во время создания React-элемента, произойдет слияние `defaultProps` (свойств по-умолчанию) с `props` (если они были были указаны) и валидация по `propTypes`.

Детали реализации можно глянуть в исходниках: [src\isomorphic\classic\element\ReactElement.js](https://github.com/facebook/react/blob/v15.4.2/src/isomorphic/classic/element/ReactElement.js)

### ReactMount
Здесь можно увидеть модуль `ReactMount` (01). Он содержит логику монтирования компонентов. На самом деле, никакой логики внутри `ReactDOM` нет, это просто интерфейс для работы с `ReactMount`, поэтому когда вы пишите `ReactDOM.render` технически вы обращаетесь к `ReactMount.render`. Что же такое монтирование?
> Монтирование это процесс инициализации React-компонента путем создания представляющих его DOM-элементов и их вставки в указанный контейнер (`container`).

По крайней мере так написано в комментариях к коду. Что же это на самом деле значит? Давайте представим следующее преобразование:


[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-small.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-small.svg)

<em>0.1 JSX в HTML (кликабельно)</em>

React'у надо **преобразовать описание вашего компонента (или нескольких) в HTML** и вставить его в документ. Как нам это сделать? Ага, React должен обработать все **пропсы, обработчики событий, вложенные компоненты**, а также логику. Ему надо разбить ваше высокоуровневое описание (компонентов) на очень низкоуровые кусочки данных (HTML), которые можно будет вставить на страницу. Именно это и называется монтированием.


[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-big.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-big.svg)

<em>0.1 JSX в HTML, расширенно (кликабельно)</em>

Хорошо, давайте продолжим. Но... время для любопытных фактов! Давайте добавим немного интересных штук в процесс изучения, чтобы он шел веселее.

>  Любопытный факт: Убедимся, что состояние скроллбара отслеживается (02)

> Забавно, но во время первой отрисовки корневого компонента, React инициализирует обработчики событий прокрутки и кэширует текущее положение скролла, чтобы приложение могло обращаться к нему не вызывая перерисовок (reflows). Из-за того, что разные браузеры реализуют рендеринг по-разному, некоторые DOM-значения не статичные, а пересчитываются каждый раз когда вы их дергаете в коде. Конечно, это влияет на производительность. В случае со скроллом это касается только старых браузеров, которые не поддерживают `pageX` и `pageY`. React пытается оптимизировать и этот кейс. Как видите, разработка быстрого инструментария требует использования всяких интересных методик, случай со скроллом это довольно показательный пример таких оптимизаций.

### Создание экземпляра React-компонента

Посмотрите на схему, на ней есть создание экземпляра компонента (instance) под номер (03). Создавать экземпляр компонента `<ExampleApplication />` тут на самом деле пока рановато, на самом деле мы здесь создаем экземпляр `TopLevelWrapper` (внутренний React'овский класс). Давайте сначала глянем следующую схему.

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/jsx-to-vdom.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/jsx-to-vdom.svg)

<em>0.3 JSX в VDOM (кликабельно)</em>

Можно выделить три фазы, через которые могут пройти полученные из JSX React-элементы для их преобразования в типы внутренних React'овских компонентов: `ReactCompositeComponent` (для наших собственных компонентов), `ReactDOMComponent` (для HTML-тэгов) и `ReactDOMTextComponent` (для текстовых элементов). Мы сейчас опустим `ReactDOMTextComponent` и просто сосредоточимся на первых двух.

"Внутренние компоненты"? Любопытно. Вы наверняка ведь уже слышали про **Виртуальный DOM** (Virtual DOM), правда же? Виртуальный DOM это в определенном роде представление DOM-а, используемое React'ом для того чтобы не трогать дерево элементов напрямую во время вычисления разницы состояний. Оно делает React ыстрым! Но, как ни странно, никаких файлов или классов в кодовой базе React'а под названием "Virtual DOM" нет. Забавно, правда? Это потому что V-DOM - всего лишь концепция, один из подходов, как можно работать с настоящим DOM'ом. Некотрые люди говорят, что элементами V-DOM'а являются React-элементы, но с моей точки зрения это не совсем так. Я думаю, что "Virtual DOM" это как раз про вот эти три класса: `ReactCompositeComponent`, `ReactDOMComponent`, `ReactDOMTextComponent`, и чуть позже вы увидите почему.

Хорошо, давайте закончим с созданием экземпляра: мы создадим экземпляр компонента `ReactCompositeComponent`, но это не потому что мы передали `<ExampleApplication/>` в `ReactDOM.render`. React всегда начинает рендеринг дерева компонентов с верхнеуровневой обертки `TopLevelWrapper` - в нем по сути ничего и нет, его render (метод отрисовки компонента) позже просто вернет `<ExampleApplication />` и все.
```javascript
//src\renderers\dom\client\ReactMount.js#277
TopLevelWrapper.prototype.render = function () {
  return this.props.child;
};
```
https://github.com/facebook/react/blob/v15.4.2/src/renderers/dom/client/ReactMount.js#L277

Итак, мы просто создали `TopLevelWrapper` и на этом пока все, пошли дальше. Но сначала... любопытный факт!
>  Любопытный факт: Валидация вложенности DOM-элементов

> Практически каждый раз когда рендерятся вложенные компоненты, они валидируются специальным модулем HTML-валидации который называется `validateDOMNesting`. Валидация вложенности DOM-элементов означает проверку правильности иерархиии тэгов `потомок -> родитель`. Например, если родительский тэг - `<select>`, то допустимым вложенным в него тэгом могут бытть только `option`, `optgroup` или `#text`. Эти правила на самом деле определены в спецификации HTML: https://html.spec.whatwg.org/multipage/syntax.html#parsing-main-inselect. Вы наверняка видели, как этот модуль работает, он бросает ошибки вида:
<em> &lt;div&gt; cannot appear as a descendant of &lt;p&gt; </em>.

т.е. `<div> не может быть потомком <p>`. 


### Хорошо, *Часть 0* мы закончили. 

Давайте вспомним, как мы сюда попали - глянем еще раз на схему, затем уберем из нее менее важные куски, и она станет вот такой:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-A.svg)

<em>0.4 Часть 0 упрощенно (кликабельно)</em>

И наверное мы еще поправим тут пробелы и выравнивание:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-B.svg)

<em>0.5 Часть 0 упрощенно и с рефакторингом (кликабельно)</em>

Вот теперь отлично. По сути, это все что происходит на этой стадии. Поэтому мы возьмем эту ключевую выдержку из *Части 0* и поместим ее на итоговую схему процесса `монтирования`:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-C.svg)

<em>0.6 Часть 0 ключевые моменты (кликабельно)</em>

И мы закончили!


[На следующую страницу: Часть 1 >>](../../../../stack/book/Part-1.md)

[<< На предыдущую страницу: Вступление](./Intro.md)


[К оглавлению](../../README.md)
