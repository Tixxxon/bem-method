# Создаем динамический БЭМ-проект

* [Введение](#Введение)
* [Приложение Hello, World](#Приложение-hello-world)
* [Приложение Social Services Search Robot](#Приложение-social-services-search-robot)
  * [Схема работы приложения](#Схема-работы-приложения)
  * [Используемые технологии БЭМ](#Используемые-технологии-БЭМ)
  * [Файловая структура проекта](#Файловая-структура-проекта)
  * [Создание приложения](#Создание-приложения)

## Введение

Многие современные приложения обмениваются данными в режиме реального времени с последующей частичной или полной перезагрузкой страницы.

Цель документа — показать, как разрабатывать БЭМ-проекты, ориентированные на динамические данные.

В документе рассматривается процесс создания двух приложений:
* [Hello, World](#Приложение-hello-world) — быстрый старт по созданию динамических приложений.
* [Social Services Search Robot](#Приложение-social-services-search-robot) —  приложение, которое позволяет найти последние твиты и видео по ключевому слову.

Для работы потребуется установить:

* [Node.js 4+](https://nodejs.org).
* [Git](https://git-scm.com).
* [Git Bash](https://git-for-windows.github.io) (пользователям ОС Windows).

> **Примечание.** В документе не рассматриваются вопросы верстки и клиентского JS.

## Приложение Hello, World

Представляет собой динамическое приложение, которое выводит слова «Hello, World!» в выходной поток, демонстрируя тем самым, что оно запускается и может выполнять операции ввода/вывода.

Давайте создадим это приложение, а затем расширим его до [Social Services Search Robot](#Приложение-social-services-search-robot).

Чтобы создать приложение Hello, World:

1. Склонируйте шаблонный репозиторий [bem-express](https://github.com/bem/bem-express).

    ```bash
    git clone https://github.com/bem/bem-express.git bem-project
    ```

    > **Примечание.** В данном примере используется `bem-express` версии 2.00.

2. Перейдите в директорию проекта.

    ```bash
    cd bem-project
    ```

3. Удалите историю версионирования исходного репозитория.

    ```bash
    rm -rf .git
    ```

4. Инициализируйте собственный Git-репозиторий.

    ```bash
    git init
    ```

5. Установите зависимости.

    ```bash
    npm install
    ```

6. Соберите проект и запустите сервер.

    ```bash
    npm run dev
    ```

    > **Примечание.** За сборку проекта отвечает [ENB](https://ru.bem.info/toolbox/enb/).

    При запуске приложения в терминале выведится сообщение о том, что сервер выполняется на порту 3000:

    `Server is listening on 3000`.

    > **Примечание.** Если порт 3000 занят, его можно переназначить (например, на 8000):
    >
    > ```bash
    > PORT=8000 npm run dev
    > ```

    На компьютере запустился:

    * Cервер — отвечает за обработку динамических данных.
    * [Nodemon](https://github.com/remy/nodemon/) — следит за изменениями на [файловой структуре](https://ru.bem.info/methodology/filestructure/) и перезапускает сервер.
    * [Chokidar](https://github.com/paulmillr/chokidar) — следит за изменениями в файлах директорий `*.blocks/` и перестраивает структуру проекта.
    * [Livereload](https://github.com/napcs/node-livereload) — обновляет страницу в браузере.

7. Откройте браузер и введите адрес [localhost:3000](http://localhost:3000).

    Должна открыться страница со следующим содержимым:

    ```text
    Index page content
    footer content
    ```

8. Откройте файл `server/index.js` и внесите следующие изменения в код, начинающегося строкой `app.get('/', function(req, res)`:

    ```diff
    // ...
    app.get('/', function(req, res) {
    +   var hello = 'Hello';
    +   var world = 'World';
        render(req, res, {
            view: 'page-index',
            title: 'Main page',
            meta: {
                description: 'Page description',
                og: {
                    url: 'https://site.com',
                    siteName: 'Site name'
                }
            },
    +       hello: hello,
    +       world: world
        })
    });
    // ...
    ```

9. Откройте файл `common.blocks/page-index/page-index.bemtree.js` и замените его содержимое на следующее:

    ```js
    block('page-index').content()(function() {
        // Получаем данные из глобального объекта `this`
        var data = this.data;
        // Возвращаем полученные данные: `data.hello: 'Hello'`, `data.world: 'World'`
        return data.hello + ', ' + data.world + '!';
    });
    ```

    После сохранения, сервер автоматически перезапустится, и содержимое страницы изменится на:

    ```text
    Hello, World!
    footer content
    ```

Приложение Hello, World готово.

> **Примечание.** Если при создании приложения возникли сложности, поищите решение на [форуме](https://ru.bem.info/forum/). Если готового ответа не нашлось, задайте вопрос экспертам.

## Приложение Social Services Search Robot

Представляет собой динамическое приложение, которое выводит последние твиты и видео по ключевому слову.

Цель разработки данного приложения — показать взаимосвязь:
* [технологий БЭМ](#Используемые-технологии);
* данных и интерфейса в БЭМ-проекте.

![Demo](start-with-bem-express__demo.png)

### Схема работы приложения

Работу приложения можно представить следующей схемой:

![Chart of Social Services Search Robot](https://rawgit.com/bem-site/bem-method/godfreyd-start-with-bem-express/articles/start-with-bem-express/start-with-bem-express__chart.svg)

#### Шаг 1. Запрос

Пользователь отправляет запрос на сервер.

#### Шаг 2. Получение данных

Приложение обращается за данными к [Twitter Search API](https://dev.twitter.com/rest/public/search) и [YouTube Data API](https://developers.google.com/youtube/v3/docs/search/list) в соответствии с полученным от пользователя запросом.

#### Шаг 3. BEMTREE-шаблонизация

Приложение передает полученные данные [BEMTREE-шаблонизатору](#bemtree), который преобразует данные в BEMJSON.

#### Шаг 4. BEMHTML-шаблонизация

Приложение передает BEMJSON [BEMHTML-шаблонизатору](#bemhtml), который преобразует BEMJSON в HTML.

#### Шаг 5. Отправка результата пользователю

Приложение возвращает HTML-страницу пользователю.

## Используемые технологии БЭМ

В работе будем использовать следующие технологии:

* [BEMDECL](#bemdecl) — технология для описания деклараций в БЭМ.
* [DEPS](#deps) — технология для описания зависимостей в БЭМ.
* [BEMTREE](#bemtree) — шаблонизатор преобразующий данные в BEMJSON.
* [BEMHTML](#bemhtml) — шаблонизатор преобразующий BEMJSON в HTML.
* [i-bem.js](#i-bemjs) — JavaScript-фреймворк для БЭМ.

### BEMDECL

Определяет список [БЭМ-сущностей](https://ru.bem.info/methodology/key-concepts/#БЭМ-сущность), используемых на странице.

Такой список в БЭМ называется [декларацией](https://ru.bem.info/methodology/declarations/). Задача декларации — определить, что и в каком порядке подключать в сборку.

Декларации описываются в файлах с расширением `.bemdecl.js`.

Пример декларации из [приложения Hello, World](#Приложение-hello-world):

```js
// Файл `desktop.bundles/index/index.bemdecl.js`
exports.blocks = [
    { name: 'root' }
];  
```

Как видно из примера, в файле `index.bemdecl.js` определен только блок `root`. При использовании технологии [DEPS](#deps), в декларации определяют блок, с которого начинается [сборка проекта](https://ru.bem.info/methodology/build/). Блок `root` следует рассматривать как центральную «точку входа» при работе над проектом. Все остальные БЭМ-сущности попадают в сборку по зависимостям.

Пример сборки проекта по зависимостям:

```files
root(DECL)
|
└──> root(DEPS)
     |
     └──> page(DEPS)
          |
          ├──> header(DEPS)
          |    |
          |    └──> ...
          |
          ├──> body(DEPS)
          |    |
          |    └──> ...
          |
          └──> footer(DEPS)
               |
               └──> ...
```

> Подробнее о [технологии BEMDECL](https://ru.bem.info/methodology/declarations/).

### DEPS

Определяет зависимости между БЭМ-сущностями, которые разнесены по файловой структуре проекта и не отражены в [декларации](#bemdecl).

Зависимости описываются в виде JavaScript-объекта в файлах с расширением `.deps.js`.

Пример зависимостей для блока `root` из [приложения Hello, World](#Приложение-hello-world):

```js
// Файл `common.blocks/root/root.deps.js`
({
    shouldDeps: 'page'
})
```

> Подробнее о [технологии DEPS](https://ru.bem.info/platform/deps/).

### BEMTREE

Является частью шаблонизатора [bem-xjst](https://ru.bem.info/platform/bem-xjst/) и преобразует данные в BEMJSON.

Шаблоны описываются в BEMJSON-формате в файлах с расширением `.bemtree.js`.

Вход и выход шаблонизатора:

![BEMTREE](https://cdn.rawgit.com/bem-site/bem-method/bem-info-data/articles/start-with-bem-express/start-with-bem-express__bemtree.svg)

> Подробнее о [технологии BEMTREE](https://ru.bem.info/platform/bem-xjst/).

### BEMHTML

Является частью шаблонизатора [bem-xjst](https://ru.bem.info/platform/bem-xjst/) и преобразует BEMJSON-описание страницы в HTML.

Шаблоны описываются в файлах с расширением `.bemhtml.js`.

Вход и выход шаблонизатора:

![BEMHTML](https://cdn.rawgit.com/bem-site/bem-method/bem-info-data/articles/start-with-bem-express/start-with-bem-express__bemhtml.svg)

> Подробнее о [технологии BEMHTML](https://ru.bem.info/platform/bem-xjst/).

### i-bem.js

Клиентский JavaScript-фреймворк для веб-разработки в рамках БЭМ-методологии.

JavaScript-код описывается в файлах с расширением `.js`.

Позволяет:

* разрабатывать веб-интерфейс в терминах блоков, элементов, модификаторов;
* описывать логику работы блока в декларативном стиле — как набор состояний;
* легко интегрировать код JavaScript с BEMHTML-шаблонами и CSS;
* гибко переопределять поведение библиотечных блоков.

> Подробнее о [технологии i-bem.js](https://ru.bem.info/platform/i-bem/).

### Файловая структура проекта

Имеет следующий вид:

```files
bem-project/
    .enb/                 # Конфигурационные файлы для сборщика ENB
    common.blocks/        # Базовые реализации блоков
    desktop.bundles/      # Директории бандлов проекта
    development.blocks/   # Блоки, подключаемые в процессе разработки
    node_modules/         # Установленные модули Node (пакеты)
    server/               # Директория с серверным кодом
    static/               # Корневая директория для раздачи статических файлов
    .bemhint.js           # Конфигурация линтера Bemhint
    .borschik             # Конфигурация сборщика файлов Borschik
    .eslintignore         # Исключение файлов и директорий в ESLint
    .eslintrc             # Конфигурация ESLint
    .gitignore            # Исключение файлов и директорий в Git
    .stylelintrc          # Конфигурация Stylelint
    .travis.yml           # Автоматический запуск линтеров в Continuous Integration
    nodemon.json          # Конфигурация для пакета Nodemon
    package.json          # Описание проекта для npm
    README.md             # Текстовое описание проекта
```

### Создание приложения

1. [Изменение файловой структуры проекта](Изменение-файловой-структуры-проекта)
2. [Установка дополнительных модулей](#Установка-дополнительных-модулей)
3. [Получение OAuth-токенов](#Получение-oauth-токенов)
4. [Конфигурация приложения](#Конфигурация-приложения)
5. [Работа с Twitter Search API](#Работа-с-twitter-search-api)
6. [Работа с YouTube Data API](#Работа-с-youtube-data-api)
7. [Верстка](#Верстка)

#### Изменение файловой структуры проекта

Прежде чем начать писать код, давайте изменим структуру взятого за основу приложения [Hello, World](#Приложение-hello-world).

1. Создайте в директории **server** следующие поддиректории и файлы:

   ```diff
   server/
   +    controllers/         # Контроллеры
   +        index.js         # Контроллер обработки запросов и рендеринга HTML
   +    helpers/             # Хелперы
   +        index.js         # Входная точка для модулей-хелперов (E)
   +        twitter.js       # Модуль-хелпер для работы с Twitter Search API (E)
   +        youtube.js       # Модуль-хелпер для работы с YouTube Data API (E)
   +    middleware/          # Модули промежуточного звена
   +        auth.js          # Модуль проверки прохождения аутентификации на YouTube (E)
   +    app.js               # Модуль монтирования промежуточных модулей
   +    auth.js              # Модуль аутентификации на YouTube (E)
        config.js
        index.js
        rebuild.js
        render.js
   +    routes.js            # Модуль маршрутизации запросов
   ```

   > **Примечание.** Файлы помеченные флагом E (Empty) потребуются позже.

2. Добавьте [следующий код](https://gist.github.com/godfreyd/4bda7da3db029890378e15bcc38f32de) в файл **controllers/index.js**.
3. Добавьте [следующий код](https://gist.github.com/godfreyd/a584cee1191833afae70fc059ba1f200) в файл **app.js**.
4. Добавьте [следующий код](https://gist.github.com/godfreyd/f6de1c33a83dda708a0e3ba9312f0c78) в файл **routes.js**.
5. Замените содержимое файла **index.js** на [следующее](https://gist.github.com/godfreyd/37d903c73f863619e2e1be1cd946d4c3).

   > **Примечание.** В `index.js` остается только функциональность, отвечающая за запуск приложения и прослушивание запросов на порте.

После выполненных действий, по адресу [localhost:3000](http://localhost:3000) по-прежнему должна открываться страница со следующим содержимым:

```text
Hello, World!
footer content
```

> **Примечание.** Если ваш код не работает — ищите опечатки.

### Установка дополнительных модулей

Для работы приложения потребуется установить следующие модули:

* [express](http://expressjs.com) — предоставляет функциональность для построения веб-приложения.
* [passport](http://passportjs.org) — предоставляет стратегии аутентификации в приложениях на Node.js.
* [passport-youtube-v3](https://www.npmjs.com/package/passport-youtube-v3) — предоставляет стратегию аутентификации на Youtube посредством аккаунта Youtube и токенов [OAuth 2.0](https://oauth.net/2/).
* [twitter](https://www.npmjs.com/package/twitter) — клиентская библиотека для работы с [Twitter REST API](https://dev.twitter.com/rest/public).
* [googleapis](http://google.github.io/google-api-nodejs-client/) — клиентская библиотека для работы с [Google REST API](https://developers.google.com/apis-explorer/#p/).
* [moment](http://momentjs.com) — JavaScript библиотека для синтаксического анализа, валидации и форматирования дат.

Установить модули можно командой:

```bash
$ npm install express passport passport-youtube-v3 twitter googleapis moment --save
```





### Получение OAuth-токенов

Сервисы Twitter и Google хранят различные данные пользователей — твиты, видео на Youtube, письма в Почте, фотографии и так далее. Чтобы обеспечить удобный доступ к этим данным из других приложений или сторонних сервисов, они используют открытый протокол авторизации [OAuth 2.0](https://oauth.net).

Согласно протоколу, разработчик регистрирует приложение на OAuth-сервере и запрашивает доступ к определенным данным. Авторизованный пользователь разрешает или запрещает его.

#### Получение OAuth-токена для Twitter

Twitter предлагает приложениям возможность выдавать аутентифицированные запросы от имени самого приложения.

**С чего начать?**

1. Изучите [документацию](https://dev.twitter.com/oauth).
2. Зарегистрируйте [приложение](https://apps.twitter.com) и получите ключи (Consumer Key, Consumer Secret).
3. Установите [Postman](https://www.getpostman.com) любым удобным для вас способом.
4. [Закодируйте строку](#codestring) `Consumer Key:Consumer Secret` методом [Base64](https://en.wikipedia.org/wiki/Base64).
5. [Получите OAuth-токен](#gettoken) в обмен на код.
6. Используйте полученные токен и ключи в запросах к Twitter Search API.

  > **Примечание.** Postman необходим для получения OAuth-токена с помощью POST-запроса в обмен на код, полученный методом Base64.

<a name="codestring"></a>
**Как закодировать строку?**

Чтобы закодировать строку методом Base64:

1. Сформируйте строку вида: `Consumer Key:Consumer Secret`.

    **Пример**

    `xvz1evFS4wEEPTGEFPHBog:L8qq9PZyRg6ieKGEKhZolGC0vJWLw8iEJ88DRdyOg`

    > **Примечание.** Получить ключи Consumer Key и Consumer Secret можно, перейдя на вкладку **Keys and Access Tokens** [вашего приложения](https://apps.twitter.com).

2. Запустите терминал или Git Bash.
4. Выполните команду `echo -n "xvz1evFS4wEEPTGEFPHBog:L8qq9PZyRg6ieKGEKhZolGC0vJWLw8iEJ88DRdyOg" | base64`.
5. Скопируйте полученный код.

    **Пример**

    `eHZ6MWV2RlM0d0VFUFRHRUZQSEdFS2hab2xHQzB2SldMdzhpRUo4OERSZHlPZw==`

> **Примечание.** Если возникли сложности, воспользуйтесь онлайн-ресурсом [base64encode.org](https://www.base64encode.org).

<a name="gettoken"></a>
**Как получить OAuth-токен в обмен на код?**

Чтобы получить токен в обмен на код:

1. Запустите Postman.

    > **Примечание.** По умолчанию открывается вкладка, в которой необходимо сформировать POST-запрос к OAuth-серверу Twitter.

2. Выберите тип запроса **POST**.
3. Введите адрес сервера `https://api.twitter.com/oauth2/token`.
4. Перейдите на вкладку **Headers**.
5. Введите в поле **Key** заголовок `Authorization` со значением (поле **Value**) `Basic <закодированная строка Consumer Key:Consumer Secret>`.

    **Пример**

    `Authorization: Basic eHZ6MWV2RlM0d0VFUFRHRUZQSEdFS2hab2xHQzB2SldMdzhpRUo4OERSZHlPZw==`

    > **Примечание.** Basic указывает на базовый метод авторизации.

6. Введите второй заголовок `Content-Type` со значением `application/x-www-form-urlencoded;charset=UTF-8`.

    **Пример**

    `Content-Type: application/x-www-form-urlencoded;charset=UTF-8`

7. Перейдите на вкладку **Body**.
8. Выберите опцию `x-www-form-urlencoded`.
9. Введите в поле **Key** тело запроса `grant_type` со значением `client_credentials`.
10. Нажмите кнопку **Send**.

    OAuth-сервер вернет токен в JSON-формате:

    ```json
    {
      "token_type": "bearer",
      "access_token": "AAAAAAAAAAAAAAAAAAAAAA%2FAAAAAAAAAA%3DAAAAAAAAAAAAAAAAAA"
    }
    ```

    > **Важно!** Сохраните полученные токен и ключи (Consumer Key и Consumer Secret). Они необходимы для [конфигурационного файла](#Конфигурация-приложения) приложения.

#### Получение OAuth-токена для Google

Google предлагает приложениям возможность выдавать аутентифицированные запросы от имени самого приложения.

> **Примечание.** За получение и обновление OAuth-токена с помощью POST-запроса в обмен на код авторизации отвечает модуль [passport-youtube-v3](#passport-youtube-v3).

**С чего начать?**

1. Изучите [документацию](https://developers.google.com/youtube/v3/docs/search/list).
2. Зарегистрируйте [приложение](https://console.developers.google.com/) и получите Client ID и Client Secret.
3. Укажите callback URL (в нашем случае это `http://localhost:3000`) в учетной записи вашего приложения.
4. Используйте полученные Client ID и Client Secret в запросах к YouTube Data API.

> **Важно!** Сохраните полученные ключи (Client ID и Client Secret). Они необходимы для [конфигурационного файла](#Конфигурация-приложения) приложения.

### Конфигурация приложения

После того как все ключи и токены получены, их необходимо добавить в конфигурационный файл приложения:

* Добавьте в файл `server/config.json` поле `services`.

  ```json
  "services": {
    "twitter": {
      "consumer_key": "",
      "consumer_secret": "",
      "bearer_token": ""
    },
    "youtube": {
      "client_id": "",
      "client_secret": "",
      "redirect_url": "http://localhost:3000"
    }
  }
  ```

  [Полный код config.json](https://gist.github.com/godfreyd/3697d1ba5fe5ac298a9b471fe943340f).

* Заполните одноименные поля [полученными данными](#Получение-oauth-токенов).
* Скройте файл `server/config.json` от системы контроля версий Git, чтобы случайно не добавить личные ключи в репозиторий файлов.

  ```bash
  # файл .gitignore
  server/config.json
  ```

  [Полный код .gitignore](https://gist.github.com/godfreyd/71a35fde50f54205fa395230bc97358f).

### Работа с Twitter Search API

[Twitter Search API](https://dev.twitter.com/rest/public/search) позволяет найти последние или популярные твиты, опубликованные на сайте Twitter.com за последние 7 дней.

Подробнее:

* [Доступ к API](#Доступ-к-api)
* [Вызов API](#Вызов-api)

#### Доступ к API

Для успешного вызова API необходимы:

* URL, составленный согласно требованиям к нужному запросу.
* [OAuth-токен](#Получение-oauth-токена-для-twitter), выданный вашему приложению для доступа к API.
* Модуль [twitter](#twitter).

#### Вызов API

Изменения для работы с Twitter Search API:

![twitter-changes](https://cdn.rawgit.com/bem-site/bem-method/bem-info-data/articles/start-with-bem-express/start-with-bem-express__twitter-changes.svg)

**Директория `controllers`**

* Измените весь текущий контент файла `index.js` на [следующий](https://gist.github.com/godfreyd/3420597de46509b02c69707d596c8dc4).

**Директория `helpers`**

* Добавьте в файл `index.js` следующий контент:

  ```js
  module.exports = {
      twitter: require('./twitter')
  };
  ```

* Добавьте [следующий код](https://gist.github.com/godfreyd/e48b6831d785e51ee6ce0892151e3395) в файл `twitter.js`.

### Работа с YouTube Data API

[YouTube Data API](https://developers.google.com/youtube/v3/) позволяет найти видеоролики, опубликованные на сайте Youtube.com. По умолчанию в набор результата поиска включены следующие ресурсы: видео, каналы, списки воспроизведения.

Подробнее:

* [Доступ к API](#Доступ-к-api-1)
* [Вызов API](#Вызов-api-1)

#### Доступ к API

Для успешного вызова API необходимы:

* URL, составленный согласно требованиям к нужному запросу.
* [OAuth-токен](#Получение-oauth-токена-для-google), выданный вашему приложению для доступа к API.
* Модуль [googleapis](#googleapis).

#### Вызов API

Изменения для работы с YouTube Data API:

![youtube-changes](https://cdn.rawgit.com/bem-site/bem-method/bem-info-data/articles/start-with-bem-express/start-with-bem-express__youtube-changes.svg)

**Директория `server`**

* Добавьте [следующий код](https://gist.github.com/godfreyd/68af82df0bc171da54971990f442dddb) в файл `auth.js`.

* Отредактируйте файл `routes.js`.

  Измените:

  ```js
  var router = require('express').Router(),
      controllers = require('./controllers');

  router
      .get('/ping/', function(req, res) {
          res.send('ok');
      })
      .get('/', controllers.getContent);

  module.exports = router;
  ```

  На:

  ```js
  var router = require('express').Router(),
      controllers = require('./controllers'),
      passportYouTube = require('./auth'),
      middleware = require('./middleware/auth'),
      isAuthenticated = middleware.isAuthenticated;

  router
      .get('/auth/youtube', passportYouTube.authenticate('youtube'))
      .get('/auth/youtube/callback', passportYouTube.authenticate('youtube', { failureRedirect: '/error', failureFlash: true }), (req, res) => {
          res.redirect('/');
      })
      .get('/', isAuthenticated, controllers.getContent);

  module.exports = router;
  ```

**Директория `controllers`**

* Измените весь текущий контент файла `index.js` на [следующий](https://gist.github.com/godfreyd/60d5d123c45c067b3fb675688dc74835).

**Директория `helpers`**

* Добавьте в файл `index.js` следующий контент (см. комментарий):

  ```js
  module.exports = {
      twitter: require('./twitter'),
      youtube: require('./youtube')        // Подключаем модуль `youtube.js`
  };
  ```

* Добавьте [следующий код](https://gist.github.com/godfreyd/e103013e1fe480965cd84b3e7040d04b) в файл `youtube.js`.

**Директория `middleware`**

* Добавьте в файл `auth.js` следующий контент:

  ```js
  module.exports = {
      isAuthenticated: function(req, res, next) {
          if (req.isAuthenticated()) return next();

          return res.redirect('/auth/youtube');
      }
  };
  ```

### Верстка

В данном документе первоочередное внимание уделено вопросу взаимодействия [технологий БЭМ](https://ru.bem.info/platform/). Поэтому он не содержит описания верстки и клиентского JavaScript. Описание верстки привело бы к большему объему, а, значит, и к меньшей практической ценности документа.

Процесс верстки сведен к следующим шагам:

1. Удалите все блоки из директории `common.blocks`.
2. Склонируйте [следующие блоки](https://github.com/godfreyd/bem-in-dynamic/tree/master/common.blocks) в директорию `common.blocks`.
3. Добавьте [logo.svg](https://github.com/godfreyd/bem-in-dynamic/blob/master/static/images/logo.svg) в директорию `static/images`.
4. Перезапустите сервер: `npm run dev`.

Приложение Social Services Search Robot готово.

> **Примечание.** Если при создании приложения возникли сложности, поищите решение на [форуме](https://ru.bem.info/forum/). Если готового ответа не нашлось, задайте вопрос экспертам.
