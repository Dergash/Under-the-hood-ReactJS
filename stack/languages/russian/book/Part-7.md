## Часть 7

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/part-7.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/part-7.svg)

<em>7.0 Часть 7 (кликабельно)</em>

### Возвращаемся в начало

Выполнение метода закончилось монтированием, и теперь мы имеем готовые для вставки в документ HTML-элементы. Правда, хотя `markup`-разметка (1) и создана, `mountComponent`, несмотря на свое название, возвращает не совсем ее - это объект с полями `children`, `node` (вот здесь уже настоящие DOM-элементы) и некоторыми другими, но все равно - наш элемент мы все-таки получили, и теперь можем положить его в контейнер (тот, который мы явно передали в аргументом в качестве контейнера в нашем вызове `ReactDOM.render`). Добавляя элемент в контейнер, React удаляет все, что там был раньше. `DOMLazyTree` (2) это просто вспомогательный класс для выполнения операций с структурами данных типа "дерево", которые нам нужны при работе с DOM.

Последнее, что тут нужно рассмотреть, это `parentNode.insertBefore(tree.node)` (3), где `parentNode` это `div`-контейнер и `tree.node` это `div` от нашего собственного `ExampleApplication`. Замечательно, те HTML-элементы, которые мы создавали на стадии монтирования наконец-то попали в документ.

Ну, вроде бы все? Не совсем. Как вы помните, вызов `mount` был завернут в транзакцию; это значит, что теперь мы должны ее закрыть. Давайте посмотрим `close`, наш список закрывающих врапперов. В основном нам тут надо восстановить то поведение, которое мы перед этим заблокировали: `ReactInputSelection.restoreSelection()`, `ReactBrowserEventEmitter.setEnabled(previouslyEnabled)`, но помимо этого нам также надо уведомить все функции обратного вызова `this.reactMountReady.notifyAll` (4), которые мы ставили в очередь  `transaction.reactMountReady`. Одна из таких функций - наш любимый метод  `componentDidMount`, который будет вызван именно `close`-враппером.

Поздравляю, вот теперь у нас есть ясная картинка того, что же на самом деле означает ‘component did mount’!

### И еще одна транзакция, которую надо закрыть

Предыдущая транзакция кстати была не единственной, мы забыли еще одну, в которую мы заворачивали вызов `ReactMount.batchedMountComponentIntoNode`. Давайте закроем и ее.

Для этого посмотрим враппер сброса накопленных обновлений `ReactUpdates.flushBatchedUpdates` (5), который обработает `dirtyComponents` ("грязные компоненты". Звучит интересно, но тут есть и хорошие, и плохие новости - мы только закончили с нашими первым монтированием, так что ни один из компонентов пока "грязным" помечен не был, а значит, у нас здесь бесполезный вызов. Получается, эту транзакцию мы можем просто закрыть и сказать, что стратегию обработки накопленных обновлений мы отработали.

### Хорошо, *Часть 7* мы закончили.

Давайте вспомним, как мы сюда попали - глянем еще раз на схему, затем уберем из нее избыточные и менее важные куски, и она станет вот такой:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/part-7-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/part-7-A.svg)

<em>7.1 Часть 7 упрощенно (кликабельно)</em>

Уберем пустое место и поправим выравнивание:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/part-7-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/part-7-B.svg)

<em>7.2 Часть 7 упрощенно и с рефакторингом (кликабельно)</em>

Вот теперь отлично. По сути, это все что происходит на этой стадии. Поэтому мы возьмем эту ключевую выдержку из *Части 5* и поместим ее на итоговую схему процесса `монтирования`:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/part-7-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/part-7-C.svg)

<em>7.3 Часть 7 ключевые моменты (кликабельно)</em>

И мы закончили! И на этот раз мы закончили монтирование целиком. Давайте глянем на него:


[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/mounting-parts-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/7/mounting-parts-C.svg)

<em>7.4 Монтирование (кликабельно)</em>

[На следующую страницу: Часть 8 >>](../../../../stack/book/Part-8.md)

[<< На предыдущую страницу: Часть 6](./Part-6.md)


[К оглавлению](./README.md)
