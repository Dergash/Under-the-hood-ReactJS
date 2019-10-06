## Часть 10

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/10/part-10.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/10/part-10.svg)

<em>10.0 Часть 10 (кликабельно)</em>

### "Грязные" компоненты

Как видите, React в цикле проходит по `dirtyComponents` (1) и вызывает `ReactUpdates.runBatchedUpdates` (2) через транзакцию! Такой мы еще не видели, давайте посмотрим на нее.

Тип этой транзакции - `ReactUpdatesFlushTransaction`, и как уже упоминалось ранее, нам надо проверить `wrappers` этой транзакции чтобы понять, чем она занимается. Небольшая подсказка из комментариев в коде:
> Врапперы ‘ReactUpdatesFlushTransaction's очищают массив dirtyComponents и выполняют все обновления, поставленные в очередь обработчиками события завершения монтирования (например, componentDidUpdate)’

Тем не менее, нам нужно найти подтверждение в коде. У транзакции есть два враппера - `NESTED_UPDATES` и `UPDATE_QUEUEING`. В фазе `initialize` мы сохраняем размер массива "грязных" компонентов  `dirtyComponentsLength` (3) и, как вы можете видеть в фазе `close`, это значение он сравнивает с текущим количеством таких компонентов, на тот случай когда их количество поменялось на стадии обновления, и в таком случае очевидно нужно еще раз запустить `flushBatchedUpdates`. Как видите, никакой магии, подход довольно прямолинейный.

Хотя один момент с магией тут все же есть - `ReactUpdatesFlushTransaction` перезаписывает метод `Transaction.perform`, потому что на самом деле ему нужно поведение `ReactReconcileTransaction` (транзакции, используемой во время монтирования и обеспечивающей целостность состояния приложения). Поэтому внутри метода `ReactUpdatesFlushTransaction.perform` вызывается еще и `ReactReconcileTransaction`, то есть метод транзакции на самом деле завернут в еще одну транзакцию.

С технической стороны это выглядит так:

```javascript
[NESTED_UPDATES, UPDATE_QUEUEING].initialize()
[SELECTION_RESTORATION, EVENT_SUPPRESSION, ON_DOM_READY_QUEUEING].initialize()

method -> ReactUpdates.runBatchedUpdates

[SELECTION_RESTORATION, EVENT_SUPPRESSION, ON_DOM_READY_QUEUEING].close()
[NESTED_UPDATES, UPDATE_QUEUEING].close()
```

В конце мы вернемся к транзакции, чтобы перепроверить, как она помогла нам закончить работу метода, а пока давайте посмотрим детали `ReactUpdates.runBatchedUpdates` (2) ([\src\renderers\shared\stack\reconciler\ReactUpdates.js#125](https://github.com/facebook/react/blob/v15.4.2/src/renderers/shared/stack/reconciler/ReactUpdates.js#L125))


Первым делом нам надо отсортировать массив `dirtyComponets` (4). Как будем сортировать? По `порядку монтирования` (целое число, назначаемое экземпляру компонента на стадии монтирования), то есть родители обновятся первыми (так как они и монтировались первыми), затем их дочерние компоненты, ну и так далее.
Следующим шагом, мы увеличим номер пакета обновлений `updateBatchNumber`, это что-то вроде идентификатора для текущей реконсиляции. Согласно комментарию в коде:
> ‘Любые обновления, поставленные в очередь во время реконсиляции, должны быть выполнены после обработки пакета обновлений этой реконсиляции. Иначе, если массив dirtyComponents содержит компоненты [A, B], где A имеет дочерними компонентами B и C, компонент B может быть обновлен дважды в пределах одного пакета, если рендер компонента C запросит обновление компонента B (т.к. B уже обновлен, мы должны его пропустить, и единственный способ узнать это - проверить номер пакета)’

Это помогает избежать двойного обновления некоторых компонентов.

Хорошо, теперь мы наконец проходим в цикле по массиву `dirtyComponents` и передаем каждый компонент в `ReactReconciler.performUpdateIfNecessary` (5), где у каждого экземпляра `ReactCompositeComponent` будет вызван метод `performUpdateIfNecessary`, так что мы опять переходим в код `ReactCompositeComponent`, а конкретнее, в его метод `updateComponent`. Там нас ждет кое-что интереснее, так что давайте пойдем глубже...

### Хорошо, *Часть 10* мы закончили.

Давайте вспомним, как мы сюда попали - глянем еще раз на схему, затем уберем из нее избыточные и менее важные куски, и она станет вот такой:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/10/part-10-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/10/part-10-A.svg)

<em>10.1 Часть 10 упрощенно (кликабельно)</em>

Уберем пустое место и поправим выравнивание:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/10/part-10-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/10/part-10-B.svg)

<em>10.2 Часит 10 упрощенно и с рефакторингом (кликабельно)</em>

Вот теперь отлично. По сути, это все что происходит на этой стадии. Поэтому мы возьмем эту ключевую выдержку из *Части 10* и поместим ее на итоговую схему процесса `обновления`:

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/10/part-10-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/10/part-10-C.svg)

<em>10.3 Часть 10 ключевые моменты (кликабельно)</em>

И мы закончили!


[На следующую страницу: Часть 11 >>](./Part-11.md)

[<< На предыдущую страницу: Часть 9](./Part-9.md)


[К оглавлению](./README.md)
