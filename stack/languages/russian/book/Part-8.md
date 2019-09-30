## Часть 8

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/8/part-8.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/8/part-8.svg)

<em>8.0 Часть 8 (кликабельно)</em>

### `this.setState`

Теперь мы знаем, как работает монтирование, поэтому давайте заглянем с другой стороны и сделаем еще один простенький разбор, на это раз - метода `setState`!

Для начала, почему мы вообще можем вызывать какой-то метод с таким именем? Ну, с этим-то более-менее понятно, мы отнаследовали наш компонент от `ReactComponent`. Значит, нам будет несложно найти этот класс в исходном коде React'а и глянуть на этот `setState`-метод.

```javascript
//src\isomorphic\modern\class\ReactComponent.js#68
this.updater.enqueueSetState(this, partialState)
```
https://github.com/facebook/react/blob/v15.4.2/src/isomorphic/modern/class/ReactComponent.js#L68

Как вы видите, тут есть некий интерфейс `updater`. Что же это за `updater`? [Если вы глянете процесс монтирования](./Part-3.md#Назначаем-модуль-обновления-экземпляра-updater), который мы недавно проанализировали, во время `mountComponent` экземпляры компонентов VDOM'а получают свойство `updater` в виде ссылки на ReactUpdateQueue ([src\renderers\shared\stack\reconciler\ReactUpdateQueue.js](https://github.com/facebook/react/blob/v15.4.2/src/renderers/shared/stack/reconciler/ReactUpdateQueue.js)).

Итак, давайте заглянем внутрь метода `enqueueSetState` (1) и обранужим там, что во-первых, он кладет частичное состояние (частичное состояние это объект, который вы передаете в `this.setState`) в `_pendingStateQueue` (2) внутреннего экземпляра (напомним, что публичный экземпляр - это наш собственный компонент `ExampleApplication`, а внутренний экземпляр - это `ReactCompositeComponent` который был создан во время монтирования), и во-вторых ставим обновление в очередь вызовом `enqueueUpdate`, который собственно и проверяет, идет ли уже какое-либо обновление и если да, то кладет наш компонент в список `dirtyComponents`, а если нет - инициализирует транзакцию и затем опять же кладет в список `dirtyComponents`.

Подводя итоги, каждый компонент имеет собственный список висящих состояний, и каждый раз когда вы вызываете `setState` в пределах одной транзакции, вы просто кладете объекты с частичными состояниями в очередь, и затем эти частичные состояния одно за другим будут объединены в одно общее состояние компонента. Также, когда вы вызываете `setState`, вы добавляете ваш компонент в список `dirtyComponents`. Наверное, вы уже гадаете, как же этот `dirtyComponents` обрабатывается? Действительно, это следующий важный кусочек нашего паззла... 

### Хорошо, *Часть 8* мы закончили.

Давайте вспомним, как мы сюда попали - глянем еще раз на схему, затем уберем из нее избыточные и менее важные куски, и она станет вот такой:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/8/part-8-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/8/part-8-A.svg)

<em>8.1 Часть 8 упрощенно (кликабельно)</em>

Уберем пустое место и поправим выравнивание:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/8/part-8-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/8/part-8-B.svg)

<em>8.2 Часть 8 упрощенно и с рефакторингом (кликабельно)</em>

Вот теперь отлично. По сути, это все что происходит на этой стадии. Поэтому мы возьмем эту ключевую выдержку из *Части 8* и поместим ее на итоговую схему процесса `обновления`:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/8/part-8-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/8/part-8-C.svg)

<em>8.3 Часть 8 ключевые моменты (кликабельно)</em>

И мы закончили!


[На следующую страницу: Часть 9 >>](../../../../stack/book/Part-9.md)

[<< На предыдущую страницу: Часть 7](./Part-7.md)


[К оглавлению](./README.md)
