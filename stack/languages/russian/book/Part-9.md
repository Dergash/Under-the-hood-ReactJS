## Часть 9

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/part-9.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/part-9.svg)

<em>9.0 Часть 9 (кликабельно)</em>

### Но давайте сделаем шаг назад...

Как вы заметили по схеме, вызов метода `setState` может быть инициирован несколькими способами, а именно: с внешним (т.е. пользовательским) воздействием или без него. Давайте посмотрим два случая: в первом вызов метода инициирован щелчком мыши, а во втором - из метода обратного вызова в `setTimeout`, помещенного в `componentDidMount`.

В чем же между ними разница? Как вы помните, React обрабатывает обновления по мере накопления пакетами (`batches`), это означает, что в какой-то момент эти обновления должны быть собраны, и затем должен выполниться их сброс (`flushed`). Дело в том, что когда появляется событие мыши, оно обрабатывается на верхнем уровне, и затем, пройдя через несколько слоев оберток начнется пакетное обновление. Кстати, как вы видите, это происходит только если `ReactEventListener` имеет флаг `enabled` (1) и, если помните, во время монтирования компонента, один из врапперов транзакции `ReactReconcileTransaction` как раз и отключает его, обеспечивая безопасное монтирования. Все продумано! Но что насчет сценария с `setTimeout`? Он тоже довольно простой, перед тем как положить компонент в список `dirtyComponents` React сначала убедится, что транзакция началась (открыта), чтобы в момент ее закрытия обновления были сброшены.

Как вы знаете, React реализует синтетические событие `synthetic events` - синтаксический сахар, которым оборачивают нативные события. Потом они все равно стараются вести себя так, как мы ожидаем от обычных событий. Вы можете найти в коде комментарий:
> ‘Чтобы помочь разработке, мы можем получить лучшую интеграцию с инструментарием разработчиков эмулируя настоящие браузерные события’

```javascript
var fakeNode = document.createElement('react');

ReactErrorUtils.invokeGuardedCallback = function (name, func, a) {
      var boundFunc = func.bind(null, a);
      var evtType = 'react-' + name;

      fakeNode.addEventListener(evtType, boundFunc, false);

      var evt = document.createEvent('Event');
      evt.initEvent(evtType, false, false);

      fakeNode.dispatchEvent(evt);
      fakeNode.removeEventListener(evtType, boundFunc, false);
};
```
Хорошо, возвращаясь к нашему процессу обновления, давайте посмотрим еще разок. Общий подход тут следующий:

1. вызвать setState
1. открыть пакетную транзакцию, если она еще не открыта
1. добавить затронутые компоненты в список `dirtyComponents`
1. закрыть транзакцию с вызовом `ReactUpdates.flushBatchedUpdates`, что по сути означает ‘обработать все, что мы собрали в список `dirtyComponents`’.

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/set-state-update-start.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/set-state-update-start.svg)

<em>9.1 Инициация `setState` (кликабельно)</em>

### Хорошо, *Часть 9* мы закончили.

Давайте вспомним, как мы сюда попали - глянем еще раз на схему, затем уберем из нее избыточные и менее важные куски, и она станет вот такой:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/part-9-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/part-9-A.svg)

<em>9.2 Часть 9 упрощенно (кликабельно)</em>

Уберем пустое место и поправим выравнивание:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/part-9-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/part-9-B.svg)

<em>9.3 Часть 9 упрощенно и с рефакторингом (кликабельно)</em>

Вот теперь отлично. По сути, это все что происходит на этой стадии. Поэтому мы возьмем эту ключевую выдержку из *Части 8* и поместим ее на итоговую схему процесса `обновления`:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/part-9-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/9/part-9-C.svg)

<em>9.4 Часть 9 ключевые моменты (кликабельно)</em>

И мы закончили!


[На следующую страницу: Часть 10 >>](../../../../stack/book/Part-10.md)

[<< На предыдущую страницу: Часть 8](./Part-8.md)


[К оглавлению](./README.md)
