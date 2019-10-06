## Часть 14

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/part-14.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/part-14.svg)

<em>14.0 Часть 14 (кликабельно)</em>

### Последняя!

Этот метод выполняет реконсиляцию дочерних компонентов с различными свойствами, которые могут повлиять на содержимое дочерних элементов. Тут есть несколько возможных сценариев, но технически они все сводятся к двум основным случаям - либо дочерние компоненты все еще "сложные", в том смысле что они все еще являются React-компонентами и надо рекурсивно через них пройти несколько раз, пока наконец не доберемся до уровня с реальным содержимым, либо дочерние компоненты относятся к простым типам, строкам или числам (то есть являются содержимым).

Проверяется это по типу `nextProps.children` (1), и в нашем случае наш компонент `ExampleApplication` имеет три дочерних компонента: `button`, `ChildCmp` и `text string`. 

Давайте посмотрим, как это работает.

Итак, первая итерация по `ExampleApplication children`. Очевидно, что тип дочернего компонента тут не "содержимое", так что пойдем по "сложному" пути. Мы берем все дочерние компоненты, и один за другим проводим их почти через тот же самый сценарий, через который мы чуть раньше проводили их родителя. Кстати, блок с верификацией `shouldUpdateReactComponent` (2) может навести на неправильную мысль, что в нем проверяется, надо ли делать обновление или нет; на самом деле, он проверяет, будет ли происходить обновление, или же его надо удалить и создать заново (мы пропустили false-ветку на схеме чтобы упростить ее). Кроме того, после этого мы сравниваем старые и новые дочерние компоненты, и если какой-то из них был удален, мы размонтируем компонент и удаляем его.

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/children-update.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/children-update.svg)

<em>14.1 Обновление дочерних компонентов (кликабельно)</em>

На второй итерации мы обрабатываем `button`, это относится к простым случаям, потому что тип дочерних компонентов кнопки - простой "текст", потому что кнопка содержит только надпись "Кнопка обновления состояния". Затем мы проверяем, что предыдущий текст был таким же как сейчас, и действительно - текст не изменился, так что обновлять `button` нам не надо. Вот так VirtualDOM и работает, теперь уже не так абстрактно звучат слова о том, что React поддерживает внутреннее представление DOM-дерева и меняет реальные DOM-элементы только по необходимости. Отсюда и замечательная производительность.
Я думаю, основную идею вы уже уловили, теперь мы ставим на обновление `ChildCmp` и его дочерние компоненты, пока не дойдем до нижнего уровня (с содержимым) и сможем обновить его. Их содержимое как раз поменялось, если вы помните, мы обновили `this.props.message` новым значением "сообщение в состоянии по щелчку" с помощью щелчка и вызова `setState`.

```javascript
//...
onClickHandler() {
	this.setState({ message: 'сообщение в состоянии по щелчку' });
}

render() {
    return <div>
		<button onClick={this.onClickHandler.bind(this)}>Кнопка обновления состояния</button>
		<ChildCmp childMessage={this.state.message} />
//...

```

Итак, мы собираемся обновить содержимое элемента, вернее даже заменить его. Чем на самом деле является обновление? Это описывающий некоторое действие объект, который будет разобран с применением действия, которое в нем было описано. В нашем случае с обновлением текста он выглядит вот так:

```javascript
{
  afterNode: null,
  content: "сообщение в состоянии по щелчку",
  fromIndex: null,
  fromNode: null,
  toIndex: null,
  type: "TEXT_CONTENT"
}
```
Как вы видите, он почти пустой, обновление текста - операция довольно прямолинейная. Остальньные свойства которые вы тут видите нужны для ситуаций, когда вы, например, перещаете элементы - обновление тогда уже будет сложнее, чем простая замена текста.

Гляньте код метода, чтобы получить более ясную картинку:

```javascript
//src\renderers\dom\client\utils\DOMChildrenOperations.js#172
processUpdates: function(parentNode, updates) {
    for (var k = 0; k < updates.length; k++) {
      var update = updates[k];

      switch (update.type) {
        case 'INSERT_MARKUP':
          insertLazyTreeChildAt(
            parentNode,
            update.content,
            getNodeAfter(parentNode, update.afterNode)
          );
          break;
        case 'MOVE_EXISTING':
          moveChild(
            parentNode,
            update.fromNode,
            getNodeAfter(parentNode, update.afterNode)
          );
          break;
        case 'SET_MARKUP':
          setInnerHTML(
            parentNode,
            update.content
          );
          break;
        case 'TEXT_CONTENT':
          setTextContent(
            parentNode,
            update.content
          );
          break;
        case 'REMOVE_NODE':
          removeChild(parentNode, update.fromNode);
          break;
      }
    }
  }
```
https://github.com/facebook/react/blob/v15.4.2/src/renderers/dom/client/utils/DOMChildrenOperations.js#L172

Мы попадаем в ветку ‘TEXT_CONTENT’ и, в принципе, это уже последний шаг - мы вызываем `setTextContent` (3) и изменяем содержимое HTML-узла (настоящего, в DOM).

Хорошая работа! Содержимое обновилось, и на странице оно перерисовалось для пользователя. Что еще нам осталось? Давайте закончим с нашим обновлением! Все готово, так что можно вызвать у компонента `componentDidUpdate`. Как обычно вызываются отложенные методы? Правильно, через заворачивание в  транзакцию. Как вы помните, обновление "грязных" компонентов было завернутно в `ReactUpdatesFlushTransaction`, и один из врапперов этой транзакции содержит логику `this.callbackQueue.notifyAll()`, которая и вызовет `componentDidUpdate`. Замечательно!

Похоже, что мы закончили, и на этот раз - целиком.

### Хорошо, *Часть 14* мы закончили.

Давайте вспомним, как мы сюда попали - глянем еще раз на схему, затем уберем из нее избыточные и менее важные куски, и она станет вот такой:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/part-14-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/part-14-A.svg)

<em>14.2 Часть 14 упрощенно (кликабельно)</em>

Уберем пустое место и поправим выравнивание:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/part-14-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/part-14-B.svg)

<em>14.3 Часть 14 упрощенно и с рефакторингом (кликабельно)</em>

Вот теперь отлично. По сути, это все что происходит на этой стадии. Поэтому мы возьмем эту ключевую выдержку из *Части 14* и поместим ее на итоговую схему процесса `обновления`:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/part-14-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/part-14-C.svg)

<em>14.4 Часть 14 ключевые моменты (кликабельно)</em>

И мы закончили! И на этот раз мы закончили обновление целиком. Давайте глянем на него:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/updating-parts-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/14/updating-parts-C.svg)

<em>14.5 Обновление (кликабельно)</em>

[<< На предыдущую страницу: Часть 13](./Part-13.md)


[К оглавлению](./README.md)
