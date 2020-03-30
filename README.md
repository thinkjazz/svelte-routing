[![npm][npm]][npm-url]

# Svelte Routing

Декларативная библиотека маршрутизации Svelte с поддержкой SSR.

## Начало работы

Посмотрите [example folder][example-folder-url] на для примера настройки проекта.
## Установить

```bash
npm install --save svelte-routing
```

## Использование

```html
<!-- App.svelte -->
<script>
  import { Router, Link, Route } from "svelte-routing";
  import Home from "./routes/Home.svelte";
  import About from "./routes/About.svelte";
  import Blog from "./routes/Blog.svelte";

  export let url = "";
</script>

<Router url="{url}">
  <nav>
    <Link to="/">Home</Link>
    <Link to="about">About</Link>
    <Link to="blog">Blog</Link>
  </nav>
  <div>
    <Route path="blog/:id" component="{BlogPost}" />
    <Route path="blog" component="{Blog}" />
    <Route path="about" component="{About}" />
    <Route path="/"><Home /></Route>
  </div>
</Router>
```

```javascript
// main.js
import App from "./App.svelte";

const app = new App({
  target: document.getElementById("app"),
  hydrate: true
});
```

```javascript
// server.js
const { createServer } = require("http");
const app = require("./dist/App.js");

createServer((req, res) => {
  const { html } = app.render({ url: req.url });

  res.write(`
    <!DOCTYPE html>
    <div id="app">${html}</div>
    <script src="/dist/bundle.js"></script>
  `);

  res.end();
}).listen(3000);
```

## API

#### `Router`

Компонент `Router` поставляет `Link` и `Route` потомкам компонентов с маршрутизацией информации через контекст, так что вам нужно по крайней мере один `Router` в верхней части вашего приложения. Он присваивает счет всем своим потомкам `Route` и выбирает лучшее соответствие для рендеринга.

Компоненты `Router` также могут быть вложены, чтобы обеспечить бесшовное слияние многих небольших приложений.

###### Свойства

 Свойство | Обязательное значение | Значение по умолчанию | Описание                                                                                                                                                                                                                                                                                               |
| :--------: | :------: | :-----------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `basepath` |          |     `'/'`     | Свойство `basepath`  будет добавлено ко всем свойствам `to`потомков `Link` и ко всем `path` свойствам потомков `Route`. Это свойство может быть проигнорировано в большинстве случаев, но если вы размещаете ваше приложение на например `https://example.com/my-site`, Свойство `basepath`  должен быть установлен `/my-site`. |
|   `url`    |          |     `''`      | Свойство `url` SSR используется в SSR для проброска текущего URL приложения и будет использоваться всеми `Link` и `Route` потомками. Ложное значение будет проигнорировано `Router`, поэтому достаточно объявить `export let url = '';` для вашего самого верхнего компонента и дать ему значение только в SSR.                      |

#### Ссылка \`Link`

Компонент, используемый для навигации по приложению.

###### Свойства 


 Свойство | Обязательное значение | Значение по умолчанию | Описание                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| :--------: | :------: | :-----------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|    `to`    |   ✔ ️    |     `'#'`     | URL-адрес, на который должен ссылаться компонент.                                                                                                                                                                                                                                                                                                                         |
| `replace`  |          |    `false`    | When `true`, clicking the `Link` will replace the current entry in the history stack instead of adding a new one.                                                                                                                                                                                                                                                                         |
|  `state`   |          |     `{}`      | Объект, который будет перемещен в стек истории при нажатии кнопки `Link`.                                                                                                                                                                                                                                                                                 |
| `getProps` |          | `() => ({})`  | Функция, возвращающая объект, который будет распределен на атрибуты базового якорного элемента. Первый аргумент, переданный функции, - это объект со свойствами `location`, `href`, `isPartiallyCurrent`, `isCurrent`. Посмотрите на [`NavLink` component in the example project setup][example-folder-navlink] чтобы посмотреть, как с помощью этого можно построить свои собственные компоненты связи. |
#### `Route` \ Маршрут

Компонент, который сделает свой `component` собственностью или детей, когда его предок `Router` компонент решит, что это лучшее совпадение.

Все свойства, кроме `path` и `component`, предоставленные `Route`, будут переданы `component`.

Потенциальные параметры пути будут передаваться отображаемому `component` в качестве свойств. Свойству `wildcardName` может быть присвоено имя с `*wildcardName` для передачи символа подстановки в качестве свойства `wildcardName`, а не в качестве свойства `*`.

Потенциальные параметры пути передаются обратно родителю с помощью реквизита, так что они могут быть выставлены в шаблон слота с использованием `let:params`.

```html
<Route path="blog/:id" let:params>
  <BlogPost id="{params.id}" />
</Route>
```

###### Свойства

|  Property   | Required | Default Value | Description                                                                                                                                                              |
| :---------: | :------: | :------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|   `path`    |          | `''`          | Путь к тому, когда этот компонент должен быть визуализирован. Если `path` не указан, то `Route` будет действовать по умолчанию, если ни один другой `Route` в `Router` не будет совпадать. |
| `component` |          | `null`        | Конструктор компонентов, который будет использоваться для рендеринга при совпадении `Route`. Если `component` не задан, вместо него будут отрисовываться дочерние элементы `Route`.         |

#### `navigate`

Функция, позволяющая в обязательном порядке перемещаться по приложению для тех случаев использования, когда компонент `Link` не подходит, например, после отправки формы.

Первый аргумент - это строка, обозначающая, куда нужно перейти, а второй - объект со свойствами `replace` и `state`, эквивалентными свойствам в компоненте `Link`.

```html
<script>
  import { navigate } from "svelte-routing";

  function onSubmit() {
    login().then(() => {
      navigate("/success", { replace: true });
    });
  }
</script>
```

#### `link`

Действие, используемое на якорных метках для навигации по приложению. Вы можете добавить атрибут `replace` для замены текущей записи в стеке истории вместо добавления новой.

```html
<script>
  import { link } from "svelte-routing";
</script>

<Router>
  <a href="/" use:link>Home</a>
  <a href="/replace" use:link replace>Replace this URL</a>
  <!-- ... -->
</Router>
```

#### `links`

Действие, используемое на корневом элементе для того, чтобы заставить все относительные якорные элементы перемещаться вокруг приложения. Вы можете добавить атрибут `replace` на любой якорь, чтобы заменить текущую запись в стеке истории вместо добавления новой. Вы можете добавить атрибут `норут` для этого действия, чтобы пропустить якорь и позволить ему использовать родное действие браузера.

```html
<!-- App.svelte -->
<script>
  import { links } from "svelte-routing";
</script>

<div use:links>
  <Router>
    <a href="/">Home</a>
    <a href="/replace" replace>Replace this URL</a>
    <a href="/native" noroute>Use the native action</a>
    <!-- ... -->
  </Router>
</div>
```

## SSR но есть один нюанс

В браузере мы ждем, пока все дочерние компоненты `Route` не зарегистрируются со своим предком компонентом `Router`, прежде чем мы позволим `Router` выбрать лучшее совпадение. Такой подход невозможен на сервере, потому что когда все компоненты `Route` зарегистрированы и настало время выбрать совпадение, SSR уже завершено, и документ без совпадения маршрута будет возвращен.

Поэтому мы прибегаем к выбору первого совпадения `Route`, которое зарегистрировано на сервере, поэтому крайне важно, чтобы вы `сортировка компонентов  Route от наиболее специфических до наименее специфических, если вы используете SSR `.

## Оригинал на английском
[npm]: https://img.shields.io/npm/v/svelte-routing.svg
[npm-url]: https://npmjs.com/package/svelte-routing
[example-folder-url]: https://github.com/EmilTholin/svelte-routing/tree/master/example
[example-folder-navlink]: https://github.com/EmilTholin/svelte-routing/tree/master/example/src/components/NavLink.svelte
