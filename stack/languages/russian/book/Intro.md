## Вступление

### Схема, первое знакомство


[![](../../../images/intro/all-page-stack-reconciler-25-scale.jpg)](../images/intro/all-page-stack-reconciler.svg)

<em>Вступление.0 Вся схема (кликабельна)</em>

Итак... взгляните. Не торопитесь. В целом она выглядит сложной, но на самом деле она описывает всего два процеса: mount и update. Я пропустил unmount, потому что он по сути просто "mount наоборот", и убрав его можно упростить схему. Кроме того, ***это не 100%*** переложение кода, а только основных его частей, которые описывают архитектуру. В целом, это около 60% кода, но остальные 40% мало помогают выразительности визуализации, поэтому я, опять же, их опустил. 

С первого взгляда вы наверняка заметите на схеме множество цветов. Каждая логическая единица (фигура на схема) раскрашена в цвет родительского модуля. Например, `methodA` будет красным если он вызывается из `moduleB`, который красный. Ниже представлена легенда для модулей на схеме с путями к каждому файлу.

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-src-path.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-src-path.svg)

<em>Вступление.1 Цвета модулей (кликабельно)</em>

Давай поместим их на схему чтобы увидеть **зависимости между модулями**.

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/files-scheme.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/files-scheme.svg)

<em>Вступление.2 Зависимости модулей (кликабельно)</em>

Как вы наверное знаете, React разрабатывается с **поддержкой множества окружение**. 
- Мобильные устройства (**ReactNative**)
- Браузеры (**ReactDOM**)
- Серверный рендеринг
- **ReactART** (для рисования векторной графики с помощью React'а)
- и т.д.

В результате, на самом деле число файлов гораздо больше, чем кажется по схеме. Ниже представлена та же самая схема с включением в нее поддержки разных окружений.

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-per-platform-scheme.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-per-platform-scheme.svg)

<em>Вступление.3 Зависимости платформ (кликабельно)</em>

Как вы видите, некоторые элементы задвоились. Таким образом показано, что они имеют раздельные реализации под каждую платфору. Давайте возьмем что-нибудь простое, например ReactEventListener. Очевидно, что его реализация для разных платформ будет разной! Технически, как вы понимаете, зависимые модули для этих платформ должны быть как-то подключены или внедрены в текущий логический поток выполнения, и в действительности таких точек подключения много. Поскольку их использование это часть стандартного паттерна "Компоновщик", я решил опустить их. Опять же, для упрощения.

Давайте познакомимся с логическим потоком выполнения кода для **React DOM** в **обычном браузере**. Это наиболее используемая платформа, и она полностью покрывает все архитектурные идеи React'а, так что почему бы и не ее.


### Пример кода

Какой наилучший способ изучить код фреймворка или библиотеки? Правильно, прочитать и отладить код. Хорошо, мы будем отлаживать **два процесса**: **ReactDOM.render** и **component.setState**, которые транслируются в mount и update. Давайте глянем на код, с которого мы смогли бы бы начать. Что нам понадобится? Наверное, несколько маленьких компонентов с простыми render'ами, с которыми их будет проще отлаживать.

```javascript
class ChildCmp extends React.Component {
    render() {
        return <div> {this.props.childMessage} </div>
    }
}

class ExampleApplication extends React.Component {
    constructor(props) {
        super(props);
        this.state = {message: 'нет сообщения'};
    }

    componentWillMount() {
        //...
    }

    componentDidMount() {
        /* setTimeout(()=> {
            this.setState({ message: 'сообщение в состоянии по таймауту' });
        }, 1000); */
    }

    shouldComponentUpdate(nextProps, nextState, nextContext) {
        return true;
    }

    componentDidUpdate(prevProps, prevState, prevContext) {
        //...
    }

    componentWillReceiveProps(nextProps) {
        //...
    }

    componentWillUnmount() {
        //...
    }

    onClickHandler() {
        /* this.setState({ message: 'сообщение в состоянии по щелчку' }); */
    }

    render() {
        return <div>
            <button onClick={this.onClickHandler.bind(this)}> Кнопка обновления состояния </button>
            <ChildCmp childMessage={this.state.message} />
            А также некоторый текст!
        </div>
    }
}

ReactDOM.render(
    <ExampleApplication hello={'world'} />,
    document.getElementById('container'),
    function() {}
);
```

Итак, можно начинать. Давайте перейдем к первой части схемы. Шаг за шагом, мы пройдем ее целиком.

[На следующую страницу: Часть 0 >>](./Part-0.md)


[К оглавлению](./README.md)
