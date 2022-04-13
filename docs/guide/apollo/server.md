# Apollo Server

## Начало работы

### Создание проекта

1. Создаем директорию для проекта и заходим в нее:

```bash
mkdir graphql-server-example
cd !$
```

2. Инициализируем Node.js-проект:

```bash
yarn init -y
# or
npm init -y
```

### Установка зависимостей

Для Сервера требуется 2 зависимости:

- `apollo-server` - библиотека, позволяющая определять форму данных (data shape) и способы их получения
- `graphql` - библиотека для создания GraphQL-схемы и выполнения запросов

```bash
yarn add apollo-server graphql
# or
npm i ...
```

Создаем файл `index.js`:

```bash
touch index.js
```

### Определение схемы

Каждый сервер `GraphQL` (включая `Apollo Server`) нуждается в схеме, определяющей структуру данных, которые могут запрашиваться клиентом. В следующем примере мы получаем коллекцию книг по названию и автору:

```js
const { ApolloServer, gql } = require('apollo-server')

// схема - это коллекция типов (`typeDefs`),
// которые определяют "форму" выполняемых запросов
const typeDefs = gql`
  # Пример комментария

  # Тип "Book" определяет поля, которые можно получить для книги
  type Book {
    title: String
    author: String
  }

  # Тип "Query" является особым: в нем указываются все запросы, которые
  # могут выполняться клиентом, а также тип, возвращаемый запросом. В данном случае
  # запрос "books" возвращает массив из 0 или более книг
  type Query {
    books: [Book]
  }
`
```

### Определение набора данных

После определения структуры мы можем определить данные. Сервер может получать данные из любого источника (база данных, `REST API`, статический объект - хранилище или даже другой GraphQL-сервер).

Определяем данные в `index.js`:

```js
const books = [
  {
    title: 'The Awakening',
    author: 'Kate Chopin'
  },
  {
    title: 'City of Glass',
    author: 'Paul Auster'
  }
]
```

_Обратите внимание_, что оба объекта в массиве соответствуют типу `Book`, определенному в схеме.

### Определение резолвера

Резолверы сообщают Серверу, как получать данные того или иного типа. Создаем резолвер в `index.js`:

```js
// Резолверы определяют способ получения типов, определенных в схеме.
// Данный резолвер извлекает книги из соответствующего массива
const resolvers = {
  Query: {
    books: () => books,
  }
}
```

### Создание экземпляра Сервера

Схема, данные и резолвер должны быть переданы Серверу.

Создаем экземпляр Сервера в `index.js`:

```js
// В конструктор Сервера передается 2 аргумента:
// схема и набор резолверов
const server = new ApolloServer({ typeDefs, resolvers })

// Запускаем сервер
server.listen().then(({ url }) => {
  console.log(`🚀  Сервер запущен по адресу: ${url}`)
})
```

### Запуск сервера

Все готово к запуску сервера. Выполняем команду:

```bash
node index.js
```

Получаем сообщение:

```bash
🚀  Сервер запущен по адресу: http://localhost:4000/
```

### Выполнение запроса

Для выполнения запросов можно использовать `ApolloSandbox`.

Переходим по адресу `http://localhost:4000` и нажимаем на кнопку `Query your server`.

Интерфейс песочницы включает в себя следующее:

- панель операций (Operations) для создания и выполнения запросов (посередине)
- панель ответа (Response) для отображения результатов запросов (справа)
- вкладку для изучения, поиска и настроек схемы (слева)
- `URL` для подключения к другим серверам (наверху слева)

Запрос на получение книг может выглядеть так:

```gql
query GetBooks {
  books {
    title
    author
  }
}
```

Вставляем данную строку в панель операций и нажимаем синюю кнопку наверху справа. Результат запроса отображается в панели ответа.

Одной из ключевых особенностей `GraphQL` является то, что клиент может запрашивать только те данные, которые ему нужны. Удалите `author` из запроса и выполните его повторно. Вы увидите, что ответ теперь содержит только названия книг.

## Основы работы со схемой

Сервер использует схему для описания формы доступных данных. Схема определяет иерархию типов с полями, которые наполняются (populate) данными из серверного хранилища. Она также определяет, какие запросы и мутации могут выполняться клиентом.

### Язык для определения схемы

Спецификация `GraphQL` определяет высокоуровневый язык определения схемы (schema definition language, SDL), который используется для определения схемы и ее хранения в виде строки.

Пример определения 2 объектных типов:

```gql
type Book {
  title: String
  author: Author
}

type Author {
  name: String
  books: [Book]
}
```

Схема определяет коллекцию типов и отношения между ними. В приведенном примере `Book` имеет связанного с ней `author`, а `Author` - список `book`.

_Обратите внимание_: структура схемы не зависит от реализации.

### Определение полей

Как правило, типы имеют одно или более поле:

```gql
# Тип Book имеет 2 поля: `title` и `author`
type Book {
  title: String  # возвращает `String`
  author: Author # возвращает `Author`
}
```

Каждое поле возвращает данные определенного типа. Возвращаемым типом может быть `scalar`, `object`, `enum`, `union` или `interface` (⬇).

#### Списки

Поле может возвращать список, содержащий элементы определенного типа. Индикатором списка являются квадратные скобки (`[]`):

```gql
type Author {
  name: String

  books: [Book] # Список `Books`
}
```

#### Поля с нулевым значением

По умолчанию поле может возвращать `null` вместо определенного типа. Указание `!` после типа делает поле ненулевым - это означает, что такое поле не может возвращать `null`:

```gql
type Author {
  name: String! # Не может возвращать `null`
  books: [Book]
}
```

При попытке сервера вернуть `null` для ненулевого поля будет выброшено исключение.

__Нулевые значения и списки__

В случае со списками символ `!` может указываться в двух позициях:

```gql
type Author {
  books: [Book!]!
  # Данный список сам не может быть `null` и его элементы также не могут быть `null`
}
```

- использование `!` внутри квадратных скобок означает, что возвращаемый список не может содержать элементы, которые имеют значение `null`
- использование `!` после скобок означает, что сам список не может иметь значение `null`
- в любом случае возврат пустого массива будет валидным

### Поддерживаемые типы

- Scalar
- Object
  - это включает в себя 3 специальных типа корневых (root) операций: `Query`, `Mutation` и `Subscription`
- Input
- Enum
- Union
- Interface

__Скалярные типы__

Скалярные типы похожи на примитивы, они всегда разрешаются конкретными данными:

- Int
- Float
- String
- Boolean
- ID (сериализуется как `String`): уникальный идентификатор, который часто используется для повторного получения объекта или в качестве ключа для кеша. Несмотря на то, что он сериализуется в строку, он не обязательно имеет человекочитаемый формат

__Объектные типы__

Большая часть типов, определяемых в схеме, являются объектными. Такие типы содержат коллекцию полей, каждое из которых имеет собственный тип.

Два объектных типа могут включать друг друга в качестве полей:

```gql
type Book {
  title: String
  author: Author
}

type Author {
  name: String
  books: [Book]
}
```

__Тип `Query`__

Тип `Query` - это специальный объектный тип, который определяет все конечные точки верхнего уровня, которые могут выполняться клиентом на сервере.

Каждое поле типа `Query` определяет название и возвращаемый тип определенной конечной точки:

```js
type Query {
  books: [Book]
  authors: [Author]
}
```

В `REST API` книги и авторы, скорее всего, будут возвращаться разными конечными точками (например, `/api/books` и `/api/authors`). Гибкость `GraphQL` позволяет получить эти ресурсы с помощью одного запроса.

_Формирование запроса_

Запрос должен соответствовать форме запрашиваемых объектных типов:

```gql
query GetBooksAndAuthors {
  books {
    title
  }

  authors {
    name
  }
}
```

В данном случае ответ сервера будет полностью соответствовать структуре запроса:

```js
{
  "data": {
    "books": [
      {
        "title": "City of Glass"
      },
      ...
    ],
    "authors": [
      {
        "name": "Paul Auster"
      },
      ...
    ]
  }
}
```

Поскольку тип `Book` имеет поле `author` с типом `Author`, запрос может выглядеть так:

```gql
query GetBooks {
  books {
    title
    author {
      name
    }
  }
}
```

Ответ сервера будет соответствовать структуре запроса:

```js
{
  "data": {
    "books": [
      {
        "title": "City of Glass",
        "author": {
          "name": "Paul Auster"
        }
      },
      ...
    ]
  }
}
```

__Тип `Mutation`__

Тип `Mutation` определяет конечные точки для операций записи.

```gql
type Mutation {
  addBook(title: String, author: String): Book
}
```

Данный тип `Mutation` определяет единственную доступную мутацию `addBook`. Эта мутация принимает 2 аргумента (`title` и `author`) и возвращает созданный объект `Book`, соответствующий структуре, определенной в схеме.

_Формирование мутации_

Следующая мутация создает новую книгу и запрашивает определенные поля созданного объекта:

```gql
mutation CreateBook {
  addBook(title: "Fox in Socks", author: "Dr. Seuss") {
    title
    author {
      name
    }
  }
}
```

Как и в случае с запросами, ответ сервера будет полностью соответствовать структуре мутации:

```gql
{
  "data": {
    "addBook": {
      "title": "Fox in Socks",
      "author": {
        "name": "Dr. Seuss"
      }
    }
  }
}
```

Одна операция мутации может содержать несколько верхнеуровневых полей типа `Mutation`. Это означает, что одна мутация может приводить к выполнению нескольких операций записи. Во избежание гонки условий (race conditions) верхнеуровневые поля типа `Mutation` разрешаются последовательно в том порядке, в котором они определены (другие поля разрешаются параллельно).

__Типы для ввода данных__

Типы для ввода данных (input types) - это специальные объектные типы, позволяющие передавать данные как аргументы полей:

```gql
input BlogPostContent {
  title: String
  body: String
}
```

Каждое поле такого типа может быть только скалярным типом, перечислением или другим типом для ввода:

```gql
input BlogPostContent {
  title: String
  body: String
  media: [MediaDetails!]
}

input MediaDetails {
  format: MediaFormat!
  url: String!
}

enum MediaFormat {
  IMAGE
  VIDEO
}
```

После определения типа для ввода, любые поля объектных типов могут принимать этот тип в качестве аргумента:

```gql
type Mutation {
  createBlogPost(content: BlogPostContent!): Post
  updateBlogPost(id: ID!, content: BlogPostContent!): Post
}
```

__Перечисления__

Типы-перечисления похожи на скалярные типы, но все допустимые значения таких типов определяются в схеме в явном виде:

```gql
enum AllowedColor {
  RED
  GREEN
  BLUE
}
```

Перечисления используются в ситуациях, когда пользователь должен выбрать одно значение из указанных.

Перечисления могут использоваться вместо скалярных типов, поскольку они сериализуются в строки:

```gql
type Query {
  favoriteColor: AllowedColor # Значение, возвращаемое перечислением
  avatar(borderColor: AllowedColor): String # Аргумент, передаваемый перечислению
}
```

Запрос может выглядеть следующим образом:

```gql
query GetAvatar {
  avatar(borderColor: RED)
}
```

__Описания (`docstrings`)__

`SDL` поддерживает так называемые описания - markdown-подобные строки:

```gql
"Описание типа"
type MyObjectType {
  """
  Описание поля
  Поддерживается **многострочное** описание [API](http://example.com)!
  """
  myField: String!

  otherField(
    "Описание аргумента"
    arg: Int
  )
}
```

__Соглашение об именовании__

Разработчики `Apollo` рекомендуют придерживаться следующих правил именования сущностей:

- поля должны именоваться в стиле `camelCase`
- типы - в стиле `PascalCase`
- названия перечислений - в стиле `PascalCase`
- значения перечислений - в стиле `ALL_CAPS` (подобно константам)

### Проектирование схемы на основе запросов

Схема должна проектироваться на основе того, как данные используются, а не на основе того, как они хранятся.

Предположим, что мы разрабатываем приложение, которое отображает список приближающихся событий в нашей локации. Мы хотим, чтобы приложение показывало название, дату, локацию каждого события, а также погоду.

Мы хотим, чтобы наше приложение имело возможность выполнять такие запросы:

```gql
query EventList {
  upcomingEvents {
    name
    date
    location {
      name
      weather {
        temperature
        description
      }
    }
  }
}
```

Поскольку нам известна структура данных, которые используются в приложении, мы можем спроектировать схему:

```gql
type Query {
  upcomingEvents: [Event!]!
}

type Event {
  name: String!
  date: String!
  location: Location
}

type Location {
  name: String!
  weather: WeatherInfo
}

type WeatherInfo {
  temperature: Float
  description: String
}
```

Каждый тип может заполняться данными из разных источников. Например, поля `name` и `date` типа `Event` могут заполняться данными из нашей БД, а тип `WeatherInfo` - из стороннего интерфейса.

### Проектирование мутаций

В ответ мутации рекомендуется включать обновленные данные.

Пример мутации для обновления поля `email` типа `User`:

```gql
type Mutation {
  # Данная мутация принимает параметры `id` и `email`
  # и возвращает `User`
  updateUserEmail(id: ID!, email: String!): User
}

type User {
  id: ID!
  name: String!
  email: String!
}
```

Структура мутации, выполняемой клиентом:

```gql
mutation updateMyUser {
  updateUserEmail(id: 1, email: "jane@mail.com"){
    id
    name
    email
  }
}
```

После того, как сервер выполнит мутацию и сохранит новый адрес электронной почты пользователя, он ответит клиенту следующим:

```gql
{
  "data": {
    "updateUserEmail": {
      "id": "1",
      "name": "Jane Doe",
      "email": "jane@mail.com"
    }
  }
}
```

__Формирование ответа мутации__

Одна мутация может модифицировать несколько типов или несколько экземпляров одного типа.

Также мутации сильнее подвержены ошибкам, чем запросы.

Для решения этих проблем рекомендуется определять в схеме интерфейс `MutationResponse` вместе с коллекцией объектных типов, реализующих этот интерфейс (по одному для каждой мутации).

Пример такого интерфейса:

```gql
interface MutationResponse {
  code: String!
  success: Boolean!
  message: String!
}
```

Пример объекта, реализующего этот интерфейс:

```gql
type UpdateUserEmailMutationResponse implements MutationResponse {
  code: String!
  success: Boolean!
  message: String!
  user: User
}
```

Если возвращаемым типом мутации `updateUserEmail` вместо `User` будет `UpdateUserEmailMutationResponse`, то ответ сервера будет следующим:

```gql
{
  "data": {
    "updateUser": {
      "code": "200",
      "success": true,
      "message": "Email пользователя был успешно обновлен",
      "user": {
        "id": "1",
        "name": "Jane Doe",
        "email": "jane@mail.com"
      }
    }
  }
}
```

- `code` - строка, представляющая статус передачи данных. Вы можете думать об этом, как о HTTP-статус-коде
- `success` - индикатор успешного выполнения мутации
- `message` - строка в человекочитаемом формате, описывающая результат мутации
- `user` - новый пользователь

Если мутация модифицирует несколько типов, ее реализация может включать отдельные поля для каждого типа:

```gql
type LikePostMutationResponse implements MutationResponse {
  code: String!
  success: Boolean!
  message: String!
  post: Post
  user: User
}
```

Поскольку наша гипотетическая мутация `likePost` модифицирует поля `Post` и `User`, объект ответа будет включать поля из обоих типов:

```gql
{
  "data": {
    "likePost": {
      "code": "200",
      "success": true,
      "message": "Спасибо!",
      "post": {
        "id": "123",
        "likes": 5040
      },
      "user": {
        "likedPosts": ["123"]
      }
    }
  }
}
```

## Объединения и интерфейсы

Объединения и интерфейсы - это абстрактные типы, которые позволяют полю возвращать один из нескольких объектных типов.

### Объединения

В случае с объединением мы определяем, какие объектные типы оно в себя включает:

```gql
union Media = Book | Movie
```

Поле может возвращать объединение или список объединений. В данном случае оно может возвращать любой объектный тип из объединения:

```gql
type Query {
  allMedia: [Media] # Данный список может включать как объект `Book`, так и объект `Movie`
}
```

_Обратите внимание_: все типы в объединении должны быть объектами.

__Пример__

В следующем примере мы определяем объединение `SearchResult`, которое может возвращать `Book` или `Author`:

```gql
union SearchResult = Book | Author

type Book {
  title: String!
}

type Author {
  name: String!
}

type Query {
  search(contains: String): [SearchResult!]
}
```

`SearchResult` включает `Query.search`, что позволяет возвращать список, содержащий как `Book`, так и `Author`.

__Получение объединения__

Клиент не знает, какие объекты будут возвращены полем, содержащим объединение. Поэтому запрос может включать вложенные поля с несколькими возможными типами:

```gql
query GetSearchResults {
  search(contains: "Shakespeare") {
    # Запрос поля `__typename` рекомендуется включать всегда,
    # это особенно важно при запросе поля, которое
    # может возвращать один из нескольких типов
    __typename
    ... on Book {
      title
    }
    ... on Author {
      name
    }
  }
}
```

Данный запрос использует встроенные фрагменты (inline fragments) для получения заголовка (если типом результата является книга) или имени (если типом результата является автор).

Результат может выглядеть так:

```gql
{
  "data": {
    "search": [
      {
        "__typename": "Book",
        "title": "The Complete Works of William Shakespeare"
      },
      {
        "__typename": "Author",
        "name": "William Shakespeare"
      }
    ]
  }
}
```

__Разрешение объединения__

Для полноценного разрешения объединения Серверу необходимо определить, какие типы оно возвращает. Для этого в карте резолверов (resolver map) определяется функция `__resolveType`.

`__resolveType` определяет тип объекта и возвращает название типа в виде строки. Она может использоваться для реализации такой логики, как:

- проверка наличия или отсутствия полей, которые являются уникальными для определенного типа, входящего в состав объединения
- использование `instanceof`, если JavaScript-объект связан с объектным типом `GraphQL`

```js
const resolvers = {
  SearchResult: {

    __resolveType(obj, context, info){
      // Только `Author` имеет поле `name`
      if(obj.name){
        return 'Author'
      }
      // Только `Book` имеет поле `title`
      if(obj.title){
        return 'Book'
      }
      return null // Выбрасывается `GraphQLError`
    },
  },
  Query: {
    search: () => { ... }
  }
}

const server = new ApolloServer({
  typeDefs,
  resolvers,
})

server.listen().then(({ url }) => {
  console.log(`🚀 Сервер запущен по адресу: ${url}`)
})
```

_Обратите внимание_: если функция `__resolveType` возвращает значение, которое не является названием валидного типа, соответствующая операция заканчивается ошибкой.

__Интерфейс__

Интерфейс определяет набор полей, которые могут включать несколько объектных типов:

```gql
interface Book {
  title: String!
  author: Author!
}
```

Если объектный тип реализует (implements) интерфейс, он должен включать все его поля:

```gql
type Textbook implements Book {
  title: String! # Должно быть включено
  author: Author! # Должно быть включено
  courses: [Course!]!
}
```

Поле может возвращать интерфейс или список интерфейсов. В этом случае поле может возвращать любой объектный тип, реализующий интерфейс:

```gql
type Query {
  books: [Book!] # Может включать объекты `Textbook`
}
```

__Пример__

В следующем примере мы определяем интерфейс `Book` и 2 объекта, реализующих этот интерфейс:

```gql
interface Book {
  title: String!
  author: Author!
}

type Textbook implements Book {
  title: String!
  author: Author!
  courses: [Course!]!
}

type ColoringBook implements Book {
  title: String!
  author: Author!
  colors: [String!]!
}

type Query {
  books: [Book!]!
}
```

`Query.books` возвращает список, который может включать как `Textbook`, так и `ColoringBook`.

__Получение интерфейса__

Если поле возвращает интерфейс, клиент может запрашивать любые вложенные поля, включенные в этот интерфейс:

```gql
query GetBooks {
  books {
    title
    author
  }
}
```

Клиент также может запрашивать поля, которые в интерфейсе отсутствуют:

```gql
query GetBooks {
  books {
    __typename
    title
    ... on Textbook {
      courses { # Имеется только в `Textbook`
        name
      }
    }
    ... on ColoringBook {
      colors # Имеется только в `ColoringBook`
    }
  }
}
```

Результат этого запроса может выглядеть так:

```gql
{
  "data": {
    "books": [
      {
        "__typename": "Textbook",
        "title": "Wheelock's Latin",
        "courses": [
          {
            "name": "Latin I"
          }
        ]
      },
      {
        "__typename": "ColoringBook",
        "title": "Oops All Water",
        "colors": [
          "Blue"
        ]
      }
    ]
  }
}
```

__Разрешение интерфейса__

Пример использования функции `__resolveType` для определенного выше интерфейса `Book`:

```js
const resolvers = {
  Book: {
    __resolveType(book, context, info){
      // Только `Textbook` имеет поле `courses`
      if(book.courses){
        return 'Textbook'
      }
      // Только `ColoringBook` имеет поле `colors`
      if(book.colors){
        return 'ColoringBook'
      }
      return null // Выбрасывается `GraphQLError`
    },
  },
  Query: {
    schoolBooks: () => { ... }
  }
}
```

### Кастомные скалярные значения

Спецификация `GraphQL` включает дефолтные скалярные типы `Int`, `Float`, `String`, `Boolean` и `ID`. Несмотря на то, что этих типов в большинстве случаев достаточно, нам могут потребоваться другие типы данных (такие как `Date`) или нам может потребоваться валидация существующего типа. Для этого мы можем определить кастомный скалярный тип.

__Определение кастомного скалярного типа__

Для определения кастомного скалярного типа следует добавить его в схему следующим образом:

```gql
scalar MyCustomScalar
```

После этого `MyCustomScalar` может использоваться где угодно (например, как тип поля объекта, поля для ввода данных или в качестве аргумента).

Однако, Серверу нужно знать, как взаимодействовать с этим новым типом.

__Определение логики кастомного скалярного типа__

Определение логики кастомного скалярного типа включает в себя определение следующего:

- как скалярное значение представлено на сервере
- как это представление сериализуется в совместимый с `JSON` тип
- как JSON-представление десериализуется

Данная логика определяется в экземпляре класса `GraphQLScalarType`.

__Пример: скалярный тип `Date`__

В следующем примере мы исходим из предположения, что дата представлена на сервере в виде JavaScript-объекта `Date`:

```js
const { GraphQLScalarType, Kind } = require('graphql');

const dateScalar = new GraphQLScalarType({
  name: 'Date',
  description: 'Кастомный скалярный тип Date',
  serialize(value) {
    return value.getTime() // Конвертируем исходящий объект `Date` в целое число для `JSON`
  },
  parseValue(value) {
    return new Date(value) // Конвертируем входящее целое число в `Date`
  },
  parseLiteral(ast) {
    if (ast.kind === Kind.INT) {
      return new Date(parseInt(ast.value, 10)) // Конвертируем строку `AST` сначала в целое число, затем в `Date`
    }
    return null // Невалидное значение (не целое число)
  }
})
```

При инициализация используются следующие методы:

- `serialize`
- `parseValue`
- `parseLiteral`

_`serialize`_

Метод `serialize` преобразует серверное представление кастомного скалярного типа в совместимый с `JSON` формат, чтобы Сервер мог включить его в ответ.

_`parseValue`_

Метод `parseValue` преобразует значение скалярного типа в формате `JSON` в его серверное представление перед добавлением в аргументы (`args`), передаваемые резолверу.

Сервер вызывает этот метод при предоставлении скалярного типа в качестве переменной для аргумента (когда скалярный тип предоставляется в качестве аргумента в строку операции, вызывается метод `parseLiteral`).

_`parseLiteral`_

Когда входящая строка запроса включает скалярный тип как значение аргумента, это значение является частью абстрактного синтаксического дерева (AST) документа запроса. Сервер вызывает метод `parseLiteral` для преобразование AST в серверное представление скалярного типа.

__Передача кастомного скаляра Серверу__

После определения экземпляра `GraphQLScalarType` он включается в карту, содержащую резолверы для других типов и полей, определенных в схеме:

```js
const { ApolloServer, gql } = require('apollo-server')
const { GraphQLScalarType, Kind } = require('graphql')

const typeDefs = gql`
  scalar Date

  type Event {
    id: ID!
    date: Date!
  }

  type Query {
    events: [Event!]
  }
`

const dateScalar = new GraphQLScalarType({
  // ...
})

const resolvers = {
  Date: dateScalar
  // другие резолверы
}

const server = new ApolloServer({
  typeDefs,
  resolvers
})
```

__Пример: фильтрация чисел__

В следующем примере мы определяем скаляр `Odd`, который может содержать только нечетные целые числа:

```js
const { ApolloServer, gql, UserInputError } = require('apollo-server')
const { GraphQLScalarType, Kind } = require('graphql')

// Основная схема
const typeDefs = gql`
  scalar Odd

  type Query {
    # Выводит переданное нечетное целое число
    echoOdd(odd: Odd!): Odd!
  }
`

// Функция для проверки "нечетности" числа
function oddValue(value) {
  if (typeof value === "number" && Number.isInteger(value) && value % 2 !== 0) {
    return value;
  }
  throw new UserInputError("Переданное значение не является целым нечетным числом");
}

const resolvers = {
  Odd: new GraphQLScalarType({
    name: 'Odd',
    description: 'Кастомный скалярный тип для целых нечетных чисел',
    parseValue: oddValue,
    serialize: oddValue,
    parseLiteral(ast) {
      if (ast.kind === Kind.INT) {
        return oddValue(parseInt(ast.value, 10))
      }
      throw new UserInputError("Переданное значение не является целым нечетным числом")
    },
  }),
  Query: {
    echoOdd(_, {odd}) {
      return odd
    }
  }
}

const server = new ApolloServer({ typeDefs, resolvers })

server.listen().then(({ url }) => {
  console.log(`🚀 Сервер запущен по адресу: ${url}`)
})
```

__Импорт сторонних скаляров__

Кастомные скаляры из сторонней библиотеки могут импортироваться и использоваться как любые другие типы.

Например, пакет <a href="https://github.com/taion/graphql-type-json">`graphql-type-json`</a> определяет объект `GraphQLJSON`, который является экземпляром `GraphQLScalarType`. Он может использоваться для определения скаляра `JSON`, принимающего любое значение, которое является валидным `JSON`.

Устанавливаем эту библиотеку:

```bash
yarn add graphql-type-json
```

Получаем `GraphQLJSON` и добавляем его в карту резолверов:

```js
const { ApolloServer, gql } = require('apollo-server')
const GraphQLJSON = require('graphql-type-json')

const typeDefs = gql`
  scalar JSON

  type MyObject {
    myField: JSON
  }

  type Query {
    objects: [MyObject]
  }
`

const resolvers = {
  JSON: GraphQLJSON
  // другие резолверы
}

const server = new ApolloServer({ typeDefs, resolvers })

server.listen().then(({ url }) => {
  console.log(`🚀 Сервер запущен по адресу: ${url}`)
})
```

### Директивы

Директива декорирует часть схемы или операции, добавляя в них дополнительную логику. Инструменты, такие как `Apollo Server`, могут читать директивы и выполнять соответствующую логику.

Директивы объявляются следующим образом:

```gql
type ExampleType {
  oldField: String @deprecated(reason: "Используется `newField`.")
  newField: String
}
```

В приведенном примере используется одна из дефолтных директив - `@deprecated`. Она показывает, что для директив характерно следующее:

- директивы могут принимать аргументы (в данном случае `reason`)
- директивы указываются после декорируемых ими полей (`oldField` в данном случае)

Существует еще 2 дефолтные директивы:

- `@skip(if: Boolean)` - если имеет значение `true`, декорируемое поле или фрагмент в операции не разрешаются сервером
- `@include(if: Boolean)` - если имеет значение `false`, декорируемое поле или фрагмент в операции не разрешаются сервером

Имеется возможность определения собственных директив.

## Получение данных

### Резолверы

Резолвер - это функция для заполнения данными определенного поля схемы.

Если мы не определяем резолвер для определенного поля, Сервер автоматически создает его для такого поля.

__Определение резолвера__

Предположим, что у нас имеется такая схема:

```js
const resolvers = {
  Query: {
    numberSix() {
      return 6
    },
    numberSeven() {
      return 7
    }
  }
};
```

Как видно из примера:

- резолверы определяются в виде простого объекта (`resolvers`) - данный объект называется картой резолверов (resolver map)
- карта резолверов включает поля верхнего уровня, соответствующие типам, определенным в схеме (например, `Query`)
- каждый резолвер принадлежит типу соответствующего поля

__Обработка аргументов__

Предположим, что наша схема выглядит так:

```gql
type User {
  id: ID!
  name: String
}

type Query {
  user(id: ID!): User
}
```

Мы хотим иметь возможность получать `user` по `id`.

Допустим, что у нас имеется такой массив:

```js
const users = [
  {
    id: '1',
    name: 'Elizabeth Bennet'
  },
  {
    id: '2',
    name: 'Fitzwilliam Darcy'
  }
];
```

Резолвер для поля `user` может выглядеть так:

```js
const resolvers = {
  Query: {
    user(parent, args, context, info) {
      return users.find(user => user.id === args.id);
    }
  }
}
```

Как видно из примера:

- резолвер принимает 4 опциональных аргумента: `parent, args, context, info`
- аргумент `args` - это объект, содержащий все переданные аргументы

_Обратите внимание_: в примере мы не определяем резолверы для полей `User` (`id` и `name`). Это возможно благодаря тому, что Сервер автоматически определяет для них дефолтные резолверы.

___Передача резолверов Серверу__

После определения резолверы передаются в конструктор `ApolloServer` в качестве свойства `resolvers`, наряду с определением схемы (свойство `typeDefs`):

```js
const { ApolloServer, gql } = require('apollo-server');

// Статичные данные
const books = [
  {
    title: 'The Awakening',
    author: 'Kate Chopin',
  },
  {
    title: 'City of Glass',
    author: 'Paul Auster',
  }
]

// Определение схемы
const typeDefs = gql`
  type Book {
    title: String
    author: String
  }

  type Query {
    books: [Book]
  }
`

// Карта резолверов
const resolvers = {
  Query: {
    books() {
      return books
    }
  }
}

// Передаем схему и резолверы в конструктор `ApolloServer`
const server = new ApolloServer({ typeDefs, resolvers })

// Запускаем сервер
server.listen().then(({ url }) => {
  console.log(`🚀  Сервер запущен по адресу: ${url}`)
})
```

__Цепочка резолверов__

Когда Сервер разрешает поле, содержащее объектный тип, он затем разрешает одно или более поле этого объекта. Эти вложенные поля, в свою очередь, также могут содержать объектные типы. Так будет продолжаться до тех пор, пока Сервер не доберется до скалярного значения или перечисления. Количество уровней вложенности в данном случае зависит от схемы. Такая цепочка объектов называется цепочкой резолверов.

Предположим, что у нас имеется такая схема:

```gql
# У библиотеки есть филиал и книги
type Library {
  branch: String!
  books: [Book!]
}

# У книги есть название и автор
type Book {
  title: String!
  author: Author!
}

# У автора есть имя
type Author {
  name: String!
}

# В ответ на запрос может возвращаться список библиотек
type Query {
  libraries: [Library]
}
```

Вот один из примеров валидного запроса:

```gql
query GetBooksByLibrary {
  libraries {
    books {
      author {
        name
      }
    }
  }
}
```

Эти резолверы разрешаются в указанном порядке. Возвращаемое резолвером значение передается следующему резолверу через аргумент `parent`.

Полный пример:

```js
const { ApolloServer, gql } = require('apollo-server')

const libraries = [
  {
    branch: 'downtown'
  },
  {
    branch: 'riverside'
  }
]

// Поле `branch` определяет, в какой библиотеке хранится книга
const books = [
  {
    title: 'The Awakening',
    author: 'Kate Chopin',
    branch: 'riverside'
  },
  {
    title: 'City of Glass',
    author: 'Paul Auster',
    branch: 'downtown'
  }
]

// Определение схемы
const typeDefs = gql`
  type Library {
    branch: String!
    books: [Book!]
  }

  type Book {
    title: String!
    author: Author!
  }

  type Author {
    name: String!
  }

  type Query {
    libraries: [Library]
  }
`

// Карта резолверов
const resolvers = {
  Query: {
    libraries() {
      return libraries
    }
  },
  Library: {
    books(parent) {
      // Фильтруем массив книг из указанного филиала
      return books.filter(book => book.branch === parent.branch);
    }
  },
  Book: {
    // Родительский резолвер (`Library.books`) возвращает объект с именем автора
    // в поле `author`. Возвращаем объект в формате `JSON`, содержащий имя,
    // поскольку значением данного поля должен быть объект
    author(parent) {
      return {
        name: parent.author
      }
    }
  }

  // Поскольку `Book.author` возвращает объект с полем `name`,
  // для `Author.name` будет создан дефолтный резолвер.
  // Нам не нужно определять его самостоятельно
}

// Передаем схему и резовлеры в конструктор `ApolloServer`
const server = new ApolloServer({ typeDefs, resolvers })

// Запускаем сервер
server.listen().then(({ url }) => {
  console.log(`🚀  Сервер запущен по адресу: ${url}`)
})
```

Если после этого мы обновим запрос для получения названия книги:

```gql
query GetBooksByLibrary {
  libraries {
    books {
      title
      author {
        name
      }
    }
  }
}
```

Ветки, находящиеся на одном уровне, выполняются параллельно.

__Аргументы резолверов__

Как было отмечено выше, резолверы принимают 4 аргумента: `parent, args, context, info`.

- `parent` - значение, возвращаемое резолвером и передаваемое дочернему резолверу
- `args` - объект, содержащий аргументы, переданные для поля
- `context` - объект, который является общим для всех резолверов определенной операции. Может использоваться для хранения состояния, данных для аутентификации, индикатора загрузки и т.д.
- `info` - объект, содержащий информацию о состоянии выполняемой операции, включая название поля, путь поля, начиная от корневого и т.д.

__Аргумент `context`__

Данный аргумент может использоваться для передачи данных, которые могут потребоваться определенному резолверу, например, информации об авторизации пользователя, соединении с базой данных, кастомной функции для получения данных и т.п.

Для передачи контекста в конструктор `ApolloServer` передается функция инициализации `context`. Эта функция вызывается при каждом запросе, поэтому контекст может определяться на основе запроса (например, на основе HTTP-заголовков):

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => ({
    authScope: getScope(req.headers.authorization)
  })
})

// Пример резолвера
(parent, args, context, info) => {
  if(context.authScope !== ADMIN) throw new AuthenticationError('Пользователь не имеет статуса администратора');
  // ...
}
```

Инициализация контекста может быть асинхронной:

```js
context: async () => ({
  db: await client.connect(),
})

// Резолвер
(parent, args, context, info) => {
  return context.db.query('SELECT * FROM table_name');
}
```

__Дефолтные резолверы__

Как было отмечено выше, при отсутствии кастомного резолвера, для поля автоматически создается дефолтный резолвер.

Рассмотрим такую схему:

```gql
type Book {
  title: String
}

type Author {
  books: [Book]
}
```

Если резолвер поля `books` возвращает массив объектов, каждый из которых содержит поле `title`, тогда мы можем использовать дефолтный резолвер для поля `title`. Дефолтный резолвер вернет `parent.title`.

### Источники данных

Источники данных (data sources) - это классы, которые Сервер может использовать для инкапсуляции получения данных из определенного источника, например, базы данных или `REST API`.

Использовать источники данных необязательно, но настоятельно рекомендуется.

__Реализации с открытым исходным кодом__

Все реализации источников данных расширяют общий абстрактный класс `DataSource`, входящий в состав пакета `apollo-datasource`.

На сегодняшний день существуют следующие реализации источников данных:

- `RESTDataSource` - REST API
- `HTTPDataSource` - HTTP/REST API (свежая альтернатива `RESTDataSource`, разработанная сообществом)
- `SQLDataSource` - SQL БД
- `MongoDataSource` - MongoDB
- `CosmosDataSource` - Azure Cosmos БД
- `FirestoreDataSource` - Cloud Firestore

При отсутствии нужного источника данных, его можно создать самостоятельно.

__Добавление источника данных__

Подклассы `DataSource` передаются в конструктор `ApolloServer`:

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,

  dataSources: () => {
    return {
      moviesAPI: new MoviesAPI(),
      personalizationAPI: new PersonalizationAPI()
    }
  }
})
```

- как мы видим, настройка `dataSources` - это функция, возвращающая объект с экземплярами подклассов `DataSource` (в данном случае `MoviesAPI` и `PersonalizationAPI`)
- Сервер вызывает эту операцию для каждого входящей операции. Он автоматически присваивает возвращаемый объект полю `dataSources` объекта контекста, передаваемого резолверам
- функция должна возвращать новый экземпляр каждого источника данных для каждой операции. Если несколько операций использует один источник, результаты нескольких операций можно комбинировать

После этого резолверы могут получать доступ к источникам данных через распределенный объект `context` для получения данных:

```js
const resolvers = {
  Query: {
    movie: async (_, { id }, { dataSources }) => {
      return dataSources.moviesAPI.getMovie(id);
    },
    mostViewedMovies: async (_, __, { dataSources }) => {
      return dataSources.moviesAPI.getMostViewedMovies();
    },
    favorites: async (_, __, { dataSources }) => {
      return dataSources.personalizationAPI.getFavorites();
    }
  }
}
```

__Кеширование__

По умолчанию источники данных используют `InMemoryLRUCache` для хранения результатов предыдущих запросов.

При инициализации Сервера в его конструктор можно передать другой объект кеша, реализующий интерфейс `KeyValueCache`. Это позволяет хранить кеш в распределенных хранилищах типа `Memcached` или `Redis`.

__Использование `Memcached/Redis` в качестве хранилища для кеша__

При запуске нескольких экземпляров сервера следует использовать распределенный кеш. Это позволяет одному экземпляру сервера использовать кешированные результаты другого экземпляра.

Сервер поддерживает использование `Memcached` и `Redis` в качестве хранилищ для кеша через пакеты `apollo-server-cache-memcached` и `apollo-server-cache-redis`, соответственно.

_`Memcached`_

```js
const { MemcachedCache } = require('apollo-server-cache-memcached')

const server = new ApolloServer({
  typeDefs,
  resolvers,
  cache: new MemcachedCache(
    ['memcached-server-1', 'memcached-server-2', 'memcached-server-3'],
    { retries: 10, retry: 10000 } // Настройки
  )
  dataSources: () => ({
    moviesAPI: new MoviesAPI(),
  })
})
```

_`Redis`_

```js
const { BaseRedisCache } = require('apollo-server-cache-redis')
const Redis = require('ioredis')

const server = new ApolloServer({
  typeDefs,
  resolvers,
  cache: new BaseRedisCache({
    client: new Redis({
      host: 'redis-server'
    })
  }),
  dataSources: () => ({
    moviesAPI: new MoviesAPI()
  })
})
```

__`RESTDataSource`__

`RESTDataSource` помогает получать данные из `REST API`.

Устанавливаем пакет `apollo-datasource-rest`:

```bash
yarn add apollo-datasource-rest
```

_Пример_

Пример подкласса `RESTDataSource`, определяющего 2 метода для получения данных - `getMovie` и `getMostViewedMovies`:

```js
const { RESTDataSource } = require('apollo-datasource-rest')

class MoviesAPI extends RESTDataSource {
  constructor() {
    // Всегда вызываем `super()`
    super()
    // Устанавливаем базовый `URL` для `REST API`
    this.baseURL = 'https://movies-api.example.com/'
  }

  async getMovie(id) {
    // Отправляем GET-запрос request по указанному адресу
    return this.get(`movies/${id}`)
  }

  async getMostViewedMovies(limit = 10) {
    const data = await this.get('movies', {
      // Параметры строки запроса
      per_page: limit,
      order_by: 'most_viewed'
    })
    return data.results
  }
}
```

_Методы `HTTP`_

`RESTDataSource` включает методы, соответствующие наиболее распространенным HTTP-методам: `get, post, put, patch, delete`.

Пример каждого из них:

```js
class MoviesAPI extends RESTDataSource {
  constructor() {
    super()
    this.baseURL = 'https://movies-api.example.com/'
  }

  // GET
  async getMovie(id) {
    return this.get(
      `movies/${id}` // путь
    )
  }

  // POST
  async postMovie(movie) {
    return this.post(
      `movies`, // путь
      movie // тело запроса
    )
  }

  // PUT
  async newMovie(movie) {
    return this.put(
      `movies`, // путь
      movie // тело запроса
    )
  }

  // PATCH
  async updateMovie(movie) {
    return this.patch(
      `movies`, // путь
      { id: movie.id, movie } // тело запроса
    )
  }

  // DELETE
  async deleteMovie(movie) {
    return this.delete(
      `movies/${movie.id}` // путь
    )
  }
}
```

_Параметры методов_

Все методы в качестве первого параметра принимают относительный путь для отправки запроса.

Второй параметр зависит от метода:

- для методов, включающих тело запроса (`post, put, patch`) - вторым параметром является тело запроса
- для методов без тела запроса - второй параметр является объектом с параметрами строки запроса в виде пар ключ/значение

В качестве третьего параметра все методы принимают объект `init`, позволяющий устанавливать дополнительные настройки (такие как заголовки и рефереры).

_Перехват запросов_

Метод `willSendRequest` позволяет модифицировать исходящий запрос перед его отправкой. Этот метод можно использовать, например, для добавления заголовков или параметров строки запроса. В основном, он используется для авторизации и т.п.

Источники данных также имеют доступ к контексту операций, что может использоваться для хранения токенов или другой чувствительной информации.

Установка заголовка:

```js
class PersonalizationAPI extends RESTDataSource {
  willSendRequest(request) {
    request.headers.set('Authorization', this.context.token)
  }
}
```

Добавление параметров строки запроса:

```js
class PersonalizationAPI extends RESTDataSource {
  willSendRequest(request) {
    request.params.set('api_key', this.context.token)
  }
}
```

_Использование `TypeScript`_

```ts
import { RESTDataSource, RequestOptions } from 'apollo-datasource-rest'

class PersonalizationAPI extends RESTDataSource {
  baseURL = 'https://personalization-api.example.com/'

  willSendRequest(request: RequestOptions) {
    request.headers.set('Authorization', this.context.token)
  }
}
```

__Динамическое разрешение `URL`__

В некоторых случаях может потребоваться устанавливать `URL` на основе окружения или значений контекста. Для этого можно перезаписать `resolveURL`:

```js
async resolveURL(request: RequestOptions) {
  if (!this.baseURL) {
    const addresses = await resolveSrv(request.path.split("/")[1] + ".service.consul")
    this.baseURL = addresses[0]
  }
  return super.resolveURL(request)
}
```

__Использование `DataLoader`__

[`DataLoader`](https://github.com/graphql/dataloader) - это утилита для удаления дубликатов (дедупликации, deduplication) и пакетирования (батчинга, batching) объекта, загружаемого их хранилища данных. Она предоставляет мемоизированный кеш, позволяющий избежать повторных загрузок одного объекта в процессе выполнения запроса. Она также комбинирует загрузки в процессе одного тика (tick) цикла событий (event loop) в пакетный запрос, что позволяет получать несколько объектов за раз.

Основной задачей `DataLoader` является батчинг, а не кеширование, поэтому данная утилита не очень полезна при получении данных из `REST API`.

## Обработка ошибок

При возникновении ошибки Сервер возвращает ответ, содержащий массив `errors`. Каждая ошибка в этом массиве содержит свойство `extensions` с дополнительной информацией, такой как `code` и `exception.stacktrace` (в режиме для разработки).

Пример ошибки, возникшей из-за опечатки в `__typename`:

```json
{
  "errors":[
    {
      "message":"Cannot query field \"__typenam\" on type \"Query\".",
      "locations":[
        {
          "line":1,
          "column":2
        }
      ],
      "extensions":{
        "code":"GRAPHQL_VALIDATION_FAILED",
        "exception":{
          "stacktrace":[
            "GraphQLError: Cannot query field \"__typenam\" on type \"Query\".",
            "    at Object.Field (/my_project/node_modules/graphql/validation/rules/FieldsOnCorrectTypeRule.js:48:31)",
            "    ...и т.д.",
          ]
        }
      }
    }
  ]
}
```

Сервер предоставляет подклассы класса `ApolloError` из пакета [`apollo-server-errors`](https://github.com/apollographql/apollo-server/blob/main/packages/apollo-server-errors/src/index.ts) (такие как `SyntaxError` и `ValidationError`), которые могут использоваться для отладки.

__Коды ошибок__

- `GRAPHQL_PARSE_FAILED` (`SyntaxError`) - строка операции содержит синтаксическую ошибку
- `GRAPHQL_VALIDATION_FAILED` (`ValidationError`) - операция не соответствует схеме
- `BAD_USER_INPUT` (`UserInputError`) - операция включает невалидное значение для аргумента поля
- `UNAUTHENTICATED` (`AuthenticationError`) - сервер не смог выполнить аутентификацию в запрашиваемом источнике данных
- `FORBIDDEN` (`ForbiddenError`) - сервер не имел полномочий на доступ к запрашиваемому источнику данных
- `PERSISTED_QUERY_NOT_FOUND` (`PersistedQueryNotFoundError`) - клиент отправил хеш строки запроса для выполнения автоматически сохраняемого запроса, но запрос в таком кеше отсутствует
- `PERSISTED_QUERY_NOT_SUPPORTED` (`PersistedQueryNotSupportedError`) - на сервере отключены автоматически сохраняемые запросы
- `INTERNAL_SERVER_ERROR` (`None`) - неизвестная ошибка

__Вызов ошибки__

В следующем примере резолвер выбрасывает `UserInputError`, если `id` пользователя меньше `1`:

```js
const {
  ApolloServer,
  gql,
  UserInputError
} = require('apollo-server')

const typeDefs = gql`
  type Query {
    userWithID(id: ID!): User
  }

  type User {
    id: ID!
    name: String!
  }
`

const resolvers = {
  Query: {

    userWithID: (parent, args, context) => {
      if (args.id < 1) {
        throw new UserInputError('Неверное значение аргумента');
      }
      // ...
    }
  }
}
```

__Кастомизация ошибки__

В следующем примере в объект ошибки добавляется поле `argumentName` с названием невалидного аргумента:

```js
const {
  ApolloServer,
  gql,
  UserInputError
} = require('apollo-server')

const typeDefs = gql`
  type Query {
    userWithID(id: ID!): User
  }

  type User {
    id: ID!
    name: String!
  }
`

const resolvers = {
  Query: {
    userWithID: (parent, args, context) => {
      if (args.id < 1) {

        throw new UserInputError('Неверное значение аргумента', {
          argumentName: 'id'
        })
      }
      // ...
    }
  }
}
```

Ответ сервера будет выглядеть так:

```json
{
  "errors": [
    {
      "message": "Неверное значение аргумента",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "userWithID"
      ],
      "extensions": {

        "argumentName": "id",
        "code": "BAD_USER_INPUT",
        "exception": {
          "stacktrace": [
            "UserInputError: Неверное значение аргумента",
            "    at userWithID (/my-project/index.js:25:13)",
            "    ...и т.д."
          ]
        }
      }
    }
  ]
}
```

__Кастомные ошибки__

Мы можем создавать собственные ошибки либо путем определения подкласса `ApolloError`, либо путем прямой инициализации объекта `ApolloError`.

Пример определения подкласса:

```js
import { ApolloError } from 'apollo-server-errors'

export class MyError extends ApolloError {
  constructor(message: string) {
    super(message, 'MY_ERROR_CODE')

    Object.defineProperty(this, 'name', { value: 'MyError' })
  }
}

throw new MyError('Сообщение об ошибке')
```

Прямая инициализация:

```js
import { ApolloError } from 'apollo-server-errors'

throw new ApolloError('Сообщение об ошибке', 'MY_ERROR_CODE', myCustomExtensions)
```

## Загрузка файлов

Для загрузки файлов используется библиотека [`graphql-upload`](https://www.npmjs.com/package/graphql-upload). Данный пакет предоставляет поддержку для `multipart/form-data`.

Эту библиотеку нельзя использовать совместно с `apollo-server`. Для этого нужна интеграция с каким-либо фреймворком, такая как `apollo-server-express`.

```js
const express = require('express')
const { ApolloServer, gql } = require('apollo-server-express')
const {
  GraphQLUpload,
  graphqlUploadExpress
} = require('graphql-upload')
const { finished } = require('stream/promises')

const typeDefs = gql`
  # Реализация этого скалярного типа экспортируется из
  # 'GraphQLUpload' из пакета 'graphql-upload'
  # в карте резолверов ниже
  scalar Upload

  type File {
    filename: String!
    mimetype: String!
    encoding: String!
  }

  type Query {
    # В типе 'Query' должно присутствовать хотя бы одно поле. Данный пример
    # показывает как получать загруженные файлы
    otherFields: Boolean!
  }

  type Mutation {
    # Поддерживаются разные способы загрузки
    singleUpload(file: Upload!): File!
  }
`

const resolvers = {
  // Это связывает скаляр `Upload` с реализацией,
  // предоставляемой `graphql-upload`
  Upload: GraphQLUpload,

  Mutation: {
    singleUpload: async (parent, { file }) => {
      const { createReadStream, filename, mimetype, encoding } = await file

      // Вызов `createReadStream` возвращает поток для чтения.
      // См. https://nodejs.org/api/stream.html#stream_readable_streams
      const stream = createReadStream()

      // Перезаписываем файл local-file-output.txt в текущей директории
      // при каждой загрузке
      const out = require('fs').createWriteStream('local-file-output.txt')
      stream.pipe(out)
      await finished(out)

      return { filename, mimetype, encoding }
    }
  }
}

async function startServer() {
  const server = new ApolloServer({
    typeDefs,
    resolvers
  })
  await server.start()

  const app = express()

  // Этот посредник должен быть добавлен перед вызовом `applyMiddleware`
  app.use(graphqlUploadExpress())

  server.applyMiddleware({ app })

  await new Promise(r => app.listen({ port: 4000 }, r))

  console.log(`🚀 Сервер запущен по адресу: http://localhost:4000${server.graphqlPath}`)
}

startServer()
```

## Подписки

Подписки - это длящиеся операции чтения, которые могут обновлять свои результаты при возникновении определенного события на сервере.

Они, как правило, используют веб-сокеты вместо `HTTP`.

__Определение схемы__

Тип `Subscription` определяет поле верхнего уровня, на которое может подписаться клиент:

```gql
type Subscription {
  postCreated: Post
}
```

Поле `postCreated` будет обновляться при каждом создании нового `Post` на сервере и отправлять `Post` подписанным клиентам.

Клиенты могут подписаться на поле `postCreated` следующим образом:

```gql
subscription PostFeed {
  postCreated {
    author
    comment
  }
}
```

_Обратите внимание_: каждая операция может быть подписана только на одно поле типа `Subscription`.

__Включение подписок__

В следующих примерах вместо `apollo-server` используется `apollo-server-express`, поскольку первый не поддерживает подписки.

Для одновременного запуска Express-сервера и сервера для подписок мы создаем экземпляр `http.Server`, который оборачивает оба сервера и становится новым "слушателем" (listener). Мы также создаем `SubscriptionServer`, что требует определенных настроек.

1. Устанавливаем `subscriptions-transport-ws` и `@graphql-tools/schema`:

```bash
yarn add subscriptions-transport-ws @graphql-tools/schema
```

2. Импортируем необходимые утилиты в файле, где инициализируется экземпляр `ApolloServer`:

```js
import { createServer } from 'http'
import { execute, subscribe } from 'graphql'
import { SubscriptionServer } from 'subscriptions-transport-ws'
import { makeExecutableSchema } from '@graphql-tools/schema'
```

3. Далее необходимо создать `http.Server`. Для этого мы передаем приложение `Express` (app) в функцию `createServer`, импортированную из модуля `http`:

```js
// `app` - значение, которое вернул вызов `express()`
const httpServer = createServer(app);
```

4. Создаем экземпляр `GraphQLSchema`.

Вместо `typeDefs` и `resolvers` `SubscriptionServer` принимает выполняемую `GraphQLSchema`. Мы можем передать этот объект `schema` в `SubscriptionServer` и `ApolloServer`. Таким образом, мы обеспечим использование одной и той же схемы в обоих местах.

```js
const schema = makeExecutableSchema({ typeDefs, resolvers })
// ...
const server = new ApolloServer({
  schema
})
```

5. Создаем `SubscriptionServer`:

```js
const subscriptionServer = SubscriptionServer.create({
   // Созданная нами схема
   schema,
   // Это мы импортировали из `graphql`
   execute,
   subscribe
}, {
   // `httpServer`, созданный на предыдущем шаге
   server: httpServer,
   // `server` - это экземпляр `ApolloServer`
   path: server.graphqlPath
})
```

6. Добавляем в конструктор `ApolloServer` плагин для закрытия `SubscriptionServer`:

```js
const server = new ApolloServer({
  schema,
  plugins: [{
    async serverWillStart() {
      return {
        async drainServer() {
          subscriptionServer.close()
        }
      }
    }
  }]
})
```

7. Вызываем `listen`.

Вместо `app.listen` мы вызываем `httpServer.listen` с аналогичными аргументами. Это позволяет одновременно запустить `HTTP` и веб-сокет-серверы.

Полный пример:

```js
import { createServer } from "http"
import { execute, subscribe } from "graphql"
import { SubscriptionServer } from "subscriptions-transport-ws"
import { makeExecutableSchema } from "@graphql-tools/schema"
import express from "express"
import { ApolloServer } from "apollo-server-express"
import resolvers from "./resolvers"
import typeDefs from "./typeDefs"

;(async function () {
  const app = express()

  const httpServer = createServer(app)

  const schema = makeExecutableSchema({
    typeDefs,
    resolvers
  })

  const subscriptionServer = SubscriptionServer.create(
    { schema, execute, subscribe },
    { server: httpServer, path: server.graphqlPath }
  )

  const server = new ApolloServer({
    schema,
    plugins: [{
      async serverWillStart() {
        return {
          async drainServer() {
            subscriptionServer.close()
          }
        }
      }
    }]
  })
  await server.start()
  server.applyMiddleware({ app })

  const PORT = 4000
  httpServer.listen(PORT, () =>
    console.log(`Сервер запущен по адресу: http://localhost:${PORT}/graphql`)
  )
})()
```

__Разрешение подписки__

Резолверы для полей `Subscription` отличаются от резолверов для полей других типов. Резолверы подписок - это объекты с функцией `subscribe`:

```js
const resolvers = {
  Subscription: {
    postCreated: {
      // Подробнее об этом ниже
      subscribe: () => pubsub.asyncIterator(['POST_CREATED'])
    }
  },
  // другие резолверы
}
```

Функция `subscribe` должна возвращать `AsyncIterator` - стандартный интерфейс для перебора результатов асинхронных операций. В приведенном примере `AsyncIterator` генерируется `pubsub.asyncIterator`.

__Класс `PubSub`__

_Обратите внимание_: данный класс не рекомендуется использовать в продакшне, поскольку он представляет собой систему событий, хранящуюся в памяти, которая поддерживает только один экземпляр сервера. В продакшне должен использоваться другой подкласс абстрактного класса `PubSubEngine` (см. в конце раздела).

Сервер использует модель "публикация-подписка" (pub-sub) для отслеживания событий, которые обновляют активные подписки. Библиотека `graphql-subscription` предоставляет класс `PubSub`, с которого можно начать разработку.

Устанавливаем данную библиотеку:

```bash
yarn add graphql-subscription
```

Экземпляр `PubSub` позволяет как публиковать события определенного типа, так и регистрировать такие события. Создаем экземпляр `PubSub`:

```js
import { PubSub } from 'graphql-subscriptions'

const pubsub = new PubSub()
```

__Публикация события__

Для публикации события используется метод `publish`:

```js
pubsub.publish('POST_CREATED', {
  postCreated: {
    author: 'Али Баба',
    comment: 'Сезам, откройся!'
  }
})
```

- первый параметр - это название публикуемого события в виде строки
- второй параметр - полезная нагрузка, связанная с событием

Предположим, что у нас имеется такая мутация:

```gql
type Mutation {
  createPost(author: String, comment: String): Post
}
```

Базовый резолвер для этой мутации может выглядеть так:

```js
const resolvers = {
  Mutation: {

    createPost(parent, args, context) {
      // Логика работы с БД живет в `postController`
      return postController.createPost(args)
    },
  }
  // другие резолверы
}
```

Перед сохранением поста в БД мы можем опубликовать соответствующее событие:

```js
const resolvers = {
  Mutation: {
    createPost(parent, args, context) {

      pubsub.publish('POST_CREATED', { postCreated: args })
      return postController.createPost(args)
    }
  },
  // другие резолверы
}
```

__Регистрация событий__

Объект `AsyncIterator` "слушает" события определенного типа и добавляет их в очередь на обработку. `AsyncIterator` создается путем вызова метода `asyncIterator` экземпляра `PubSub`:

```js
pubsub.asyncIterator(['POST_CREATED'])
```

Данный метод принимает массив регистрируемых событий.

Каждая функция `subscribe` резолвера для поля типа `Subscription` должна возвращать объект `AsyncIterator`. Это возвращает нас к коду, с которого мы начинали:

```js
const resolvers = {
  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator(['POST_CREATED'])
    }
  },
  // другие резолверы
}
```

После определения функции `subscribe` Сервер будет использовать полезную нагрузку события `POST_CREATED` для отправки обновленных значений в поле `postCreated`.

__Фильтрация событий__

Функция `withFilter` позволяет фильтровать обновления перед их отправкой клиенту.

Предположим,что сервер предоставляет подписку `commendAdded`, которая уведомляет клиентов о добавлении нового комментария в определенный репозиторий. Клиент может выполнить подписку следующим образом:

```gql
subscription($repoName: String!){
  commentAdded(repoFullName: $repoName) {
    id
    content
  }
}
```

Что здесь не так? Сервер публикует событие `COMMENT_ADDED` при добавлении комментария в любой репозиторий. Это означает, что резолвер `commenAdded` выполняется для каждого комментария независимо от репозитория. Как результат, подписанные клиенты могут получать данные, которые им не нужны (или даже такие данные, к которым у них нет доступа).

Пример использования функции `withFilter`:

```js
import { withFilter } from 'graphql-subscriptions'

const resolvers = {
  Subscription: {

    commentAdded: {
      subscribe: withFilter(
        () => pubsub.asyncIterator('COMMENT_ADDED'),
        (payload, variables) => {
          // Отправлять обновление только в случае, когда комментарий добавляется
          // в правильный для этой операции репозиторий
          return (payload.commentAdded.repository_name === variables.repoFullName)
        }
      )
    }
  },
    // другие резолверы
}
```

`withFilter` принимает 2 параметра:

- первый параметр - функция, используемая для `subscribe` при отсутствии фильтра
- второй параметр - функция фильтрации, возвращающая `true`, если обновление должно быть отправлено определенному клиенту, или `false` в противном случае (`Promise<boolean>` также является валидным). Данная функция, в свою очередь, также принимает 2 параметра:
  - `payload` - полезная нагрузка опубликованного события
  - `variables` - объект, содержащий переменные, переданные клиентом при инициализации подписки

__Контекст операции__

При инициализации контекста для запроса или мутации мы, как правило, извлекаем HTTP-заголовки и другие детали запроса из объекта `req`, переданного в функцию `context`.

Для подписок мы передаем функцию `onConnect` в конструктор `SubscriptionServer`. Данная функция принимает `connectionParams` в объекте в качестве первого аргумента, а также `WebSocket` и `ConnectionContext` в качестве второго и третьего аргументов, соответственно. Если `onConnect` возвращает объект, он передается резолверам в качестве аргумента `context`:

```js
SubscriptionServer.create({
  // другие настройки
  async onConnect(connectionParams) {
    if (connectionParams.authorization) {
      const currentUser = await findUser(connectionParams.authorization)
      return { currentUser }
    }
    throw new Error('Отсутствует токен аутентификации!')
  }
})
```

Это важно, поскольку такие метаданные, как токены аутентификации, отправляются по-разному в зависимости от используемого транспортного протокола.

__`onConnect` и `onDisconnect`__

Мы можем определить функции, которые выполняются сервером подписок при каждом подключении - подписке (`onConnect`) или отключении (`onDisconnect`).

Определение `onConnect` предоставляет следующие преимущества:

- мы можем отклонять определенное подключение, выбрасывая исключение или возвращая `false`. Это может быть полезным при аутентификации
- если `onConnect` возвращает объект, он передается резолверам как контекст операции

Мы передаем эти функции в конструктор `SubscriptionServer`:

```js
SubscriptionServer.create({
  schema,
  execute,
  subscribe,

  onConnect(connectionParams, webSocket, context) {
    console.log('Подключение установлено!')
  },
  onDisconnect(webSocket, context) {
    console.log('Подключение прервано!')

  }
})
```

Эти функции принимают следующие параметры:

- `connectionParams` - объект, содержащий параметры, включенные в запрос, такие как токен
- `webSocket` - подключаемый или отключаемый `WebSocket`
- `context` - объект контекста для подключения `WebSocket`. Это не объект `context`, связанный с определенной операцией

__Пример: аутентификация с помощью `onConnect`__

На клиенте `SubscriptionClient` поддерживает добавление информации в `connectionParams`, которая отправляется с первым сообщением. На сервере все подписки ожидают полной аутентификации соединения и возвращения колбеком `onConnect` истинного значения.

Предположим, что наш `SubscriptionClient` настроен следующим образом:

```js
new SubscriptionClient(subscriptionUrl, {
  // другие настройки
  connectionParams: {
    authorization: clientToken
  }
})
```

Аргумент `connectionParams` в колбеке `onConnect` содержит информацию, переданную клиентом, и может использоваться для проверки полномочий пользователя.

Например, мы можем использовать токен `authorization`, переданный клиентом, для поиска соответствующего пользователя и его передачи резолверам:

```js
async function findUser(authToken) {
  // ищем юзера по токену
}

SubscriptionServer.create({
  schema,
  execute,
  subscribe,
  async onConnect(connectionParams, webSocket) {
    if (connectionParams.authorization) {
      const currentUser = await findUser(connectionParams.authorization)
      return { currentUser }
    }
    throw new Error('Отсутствует токен аутентификации!')
  },
  { server, path }
})
```

Объект пользователя будет доступен в резолверах через `context.currentUser`. При возникновении ошибки аутентификации промис будет отклонен, что предотвратит подключение клиента.

__Библиотеки `PubSub` для продакшна__

В настоящее время сообществом разработано несколько библиотек `PubSub` для таких популярных систем публикации событий, как:

- Redis
- Google PubSub
- MQTT enabled broker
- RabbitMQ
- Kafka
- Postgres
- Google Cloud Firestore
- Ably Realtime

## Формат запросов

Сервер принимает запросы и мутации, отправленные методом `POST`. Он также принимает запросы, отправленные методом `GET`.

__POST-запросы__

Сервер принимает POST-запросы с телом в формате `JSON`. Валидный запрос содержит поле `field`, а также опциональные поля `variables` и `operationName` (если запрос содержит несколько возможных операций).

Предположим, что мы хотим выполнить следующий запрос:

```gql
query GetBestSellers($category:ProductCategory) {
  bestSellers(category: $category) {
    title
  }
}
```

Пример валидного тела POST-запроса:

```gql
{
  "query":"query GetBestSellers($category:ProductCategory){bestSellers(category: $category){title}}",
  "operationName": "GetBestSellers",
  "variables": { "category": "BOOKS" }
}
```

Выполнить этот запрос можно с помощью такой команды `curl`:

```bash
curl --request POST \
  -H 'Content-Type: application/json' \
  --data '{"query":"query GetBestSellers($category:ProductCategory){bestSellers(category: $category){title}}", "operationName":"GetBestSellers", "variables":{"category":"BOOKS"}}' \
  https://rover.apollo.dev/quickstart/products/graphql
```

__Отправка групповых запросов__

В одном POST-запросе может быть отправлено сразу несколько запросов посредством передачи закодированного в формат `JSON` массива объектов:

```json
[
  {
    "query": "query { testString }"
  },
  {
    "query": "query AnotherQuery{ test(who: \"you\" ) }"
  }
]
```

На такой запрос сервер также отвечает массивом объектов.

__GET-запросы__

Сервер также принимает GET-запросы для запросов (но не для мутаций). В этом случае детали запроса (`query`, `operationName` и `variables`) передаются в виде параметров строки запроса. При этом настройка `variables` должна быть "экранированным" объектом.

Вот пример аналогичного GET-запроса:

```bash
curl --request GET \
  https://rover.apollo.dev/quickstart/products/graphql?query=query%20GetBestSellers%28%24category%3AProductCategory%29%7BbestSellers%28category%3A%20%24category%29%7Btitle%7D%7D&operationName=GetBestSellers&variables=%7B%22category%22%3A%22BOOKS%22%7D
```

## Кеширование на стороне сервера

Сервер позволяет определять настройки кеширования (`maxAge` и `scope`), применяемые к каждому полю в схеме:

```gql
type Post {
  id: ID!
  title: String
  author: Author

  votes: Int @cacheControl(maxAge: 30)
  comments: [Comment]

  readByCurrentUser: Boolean! @cacheControl(maxAge: 10, scope: PRIVATE)
}
```

Когда Сервер разрешает операцию, он вычисляет правильное поведение кеша на основе наиболее строгих настроек. Это вычисление может использоваться для поддержки любой формы реализации кеша, например, для передачи его `CDN` в качестве значения заголовка `Cache-Control`.

Настройки кеша могут определяться в схеме (статические настройки) или резолверах (динамические настройки).

При настройке кеша следует принимать во внимание следующее:

- какие поля схемы могут быть безопасно кешированы
- на протяжении какого времени кешированное значение должно оставаться валидным
- каким является кешированное значение, глобальным или рассчитанным на определенного пользователя

__Статические настройки (в схеме)__

Для настройки кеширования определенного поля или всех полей, возвращающих определенный тип, в схеме может использоваться директива `@cacheControl`.

Для этого в серверную схему следует добавить следующее:

```gql
enum CacheControlScope {
  PUBLIC
  PRIVATE
}

directive @cacheControl(
  maxAge: Int
  scope: CacheControlScope
  inheritMaxAge: Boolean
) on FIELD_DEFINITION | OBJECT | INTERFACE | UNION
```

При отсутствии данных директив будет выброшено исключение `Unknown directive "@cacheControl"`.

Рассматриваемая директива принимает следующие аргументы:

- `maxAge` - количество секунд, в течение которых кешированное значение считается валидным. Значением по умолчанию является `0`, но это можно изменить
- `scope` - если `PRIVATE`, значение поля будет привязано к конкретному пользователю. Значением по умолчанию является `PUBLIC`
- `inheritMaxAge` - если `true`, поле наследует `maxAge` от родительского поля вместо дефолтного `maxAge`. Не может использоваться совместно с `maxAge`

Директиву `@cacheControl` следует использовать в отношении полей, которые изменяются редко или не изменяются совсем.

_На уровне поля_

В следующем примере мы определяем настройки кеширования для 2 полей типа `Post` - `votes` и `readByCurrentUser`:

```gql
type Post {
  id: ID!
  title: String
  author: Author

  votes: Int @cacheControl(maxAge: 30)
  comments: [Comment]

  readByCurrentUser: Boolean! @cacheControl(maxAge: 10, scope: PRIVATE)
}
```

В данном случае:

- значение поля `votes` кешируется на 30 секунд
- значение поля `readByCurrentUser` кешируется на 10 секунд и его видимость ограничена одним пользователем

_На уровне типа_

В следующем примере мы определяем настройки кеширования для всех полей схемы, которые возвращают объект `Post`:

```gql
type Post @cacheControl(maxAge: 240) {
  id: Int!
  title: String
  author: Author
  votes: Int
  comments: [Comment]
  readByCurrentUser: Boolean!
}
```

Если другой объектный тип включает поле с типом `Post` (или список `Post`), значение такого поля будет кешировано на 240 секунд:

```gql
type Comment {
  post: Post! # Кешируется на 240 сек
  body: String!
}
```

_Обратите внимание_: настройки на уровне поля перезаписывают настройки на уровне типа. В следующем примере `Comment.post` будет кешироваться на 120 сек вместо 240:

```gql
type Comment {
  post: Post! @cacheControl(maxAge: 120)
  body: String!
}
```

__Динамические настройки (в резолверах)__

Кеширование поля может быть настроено во время его разрешения. Для этого используется объект `cacheControl` из параметра `info`, передаваемый каждому резолверу.

_`cacheControl.setCacheHint`_

Объект `cacheControl` включает метод `setCacheHint`, который вызывается следующим образом:

```js
const resolvers = {
  Query: {
    post: (_, { id }, _, info) => {

      info.cacheControl.setCacheHint({ maxAge: 60, scope: 'PRIVATE' });
      return find(posts, { id });
    }
  }
}
```

Данный метод принимает объект с полями `maxAge` и `scope`.

_`cacheControl.cacheHint`_

Данный объект представляет текущие настройки кеширования поля. Он включает в себя следующее:

- текущие `maxAge` и `scope` (которые могут быть установлены статически)
- метод `restrict`, похожий на `setCacheHint`, за исключением того, что он не может делать настройки менее строгими:

```js
// Такой вызов...
info.cacheControl.setCacheHint({ maxAge: 60, scope: 'PRIVATE' });

// ...изменит maxAge (сделает его более строгим), но не изменит scope (не сделает его менее строгим)
info.cacheControl.cacheHint.restrict({ maxAge: 30, scope: 'PUBLIC' });
```

_`cacheControl.cacheHintFromType`_

Данный метод позволяет получить дефолтные настройки кеширования для определенного объектного типа. Это может быть полезным при разрешении объединения или интерфейса, которые могут возвращать один из нескольких объектных типов.

__Вычисление поведения кеша__

По причинам, связанным с безопасностью, поведение кеша для ответа каждой операции вычисляется на основе наиболее строгих настроек:

- `maxAge` ответа равняется наименьшему `maxAge` среди всех полей. Если таковым является `0`, результат не кешируется
- `scope` ответа является `PRIVATE`, если `scope` любого поля имеет такое значение

__`maxAge` по умолчанию__

По умолчанию при отсутствии кастомизации `maxAge` имеет значение `0` для следующих полей:

- корневых (таких как типы `Query`, `Mutation` и `Subscription`)
- полей, которые возвращают нескалярные типы (объект, интерфейс или объединение), а также списки таких типов

Все остальные поля наследуют `maxAge` от своих предков.

_Кастомизация дефолтного `maxAge`_

Для кастомизации дефолтного `maxAge` следует передать плагин управления кешем в конструктор `ApolloServer`:

```js
import { ApolloServerPluginCacheControl } from 'apollo-server-core'

const server = new ApolloServer({
  // другие настройки
  plugins: [ApolloServerPluginCacheControl({ defaultMaxAge: 5 })] // 5 сек
})
```

__Рекомендации__

- для полей, которые не должны кешироваться, следует явно устанавливать `maxAge` в значение `0`
- `maxAge` следует определять для каждого поля с резолвером, получающим данные из источника данных (такого как БД или `REST API`). В этом случае значение `maxAge` зависит от частоты обновления соответствующих данных
- для каждого некорневого поля, возвращающего нескалярный тип, следует устанавливать `inheritMaxAge: true` (это можно сделать только статически)

__Пример вычисления `maxAge`__

Предположим, что у нас имеется такая схема:

```gql
type Query {
  book: Book
  cachedBook: Book @cacheControl(maxAge: 60)
  reader: Reader @cacheControl(maxAge: 40)
}

type Book {
  title: String
  cachedTitle: String @cacheControl(maxAge: 30)
}

type Reader {
  book: Book @cacheControl(inheritMaxAge: true)
}
```

Рассмотрим несколько запросов и их результирующих значений `maxAge`:

```gql
# maxAge: 0
# Query.book не устанавливает maxAge и данное поле является корневым (по умолчанию 0).
query GetBookTitle {
  book {        # 0
    cachedTitle # 30
  }
}

# maxAge: 60
# Query.cachedBook имеет maxAge равный 60, а Book.title - это scalar, поэтому оно
# по умолчанию наследует maxAge от родителя
query GetCachedBookTitle {
  cachedBook { # 60
    title      # наследование
  }
}

# maxAge: 30
# Query.cachedBook имеет maxAge равный 60, но Book.cachedTitle имеет
# maxAge равный 30
query GetCachedBookCachedTitle {
  cachedBook {  # 60
    cachedTitle # 30
  }
}

# maxAge: 40
# Query.reader имеет maxAge равный 40. Для Reader.Book установлено
# inheritMaxAge, а Book.title - это scalar,
# который по умолчанию наследует maxAge от родителя
query GetReaderBookTitle {
  reader {  # 40
    book {  # наследование
      title # наследование
    }
  }
}
```

__Кеширование с помощью `CDN`__

При отправке ответа, содержащего ненулевой `maxAge`, Сервер включает в него HTTP-заголовок `Cache-Control`, который описывает политику кеширования ответа.

Данный заголовок имеет такой формат:

```
Cache-Control: max-age=60, private
```

Если Сервер запущен за `CDN` или другим кеширующим прокси, такие заголовки могут использоваться для правильного кеширования ответов.

__Использование GET-запросов__

`CDN` и кеширующие прокси кешируют только GET-запросы (`Apollo Client` по умолчанию отправляет все операции методом `POST`). Поэтому рекомендуется включать автоматическое сохранение запросов и настройку `useGETForHashedQueries` в `Apollo Client`.

В качестве альтернативы можно установить настройку `useGETForQueries` для `HttpLink` в экземпляре `ApolloClient`. Однако большинство браузеров ограничивает размер GET-запросов, поэтому следует убедиться, что запросы не превышают установленного лимита.

__Кеширование с помощью `responseCachePlugin`__

Ответы на запросы могут сохраняться в хранилищах типа `Redis`, `Memcached` или в кеше Сервера, хранящегося в памяти.

_Сохранение кеша в памяти_

Импортируем `responseCachePlugin` и передаем его в конструктор `ApolloServer`:

```js
import responseCachePlugin from 'apollo-server-plugin-response-cache'

const server = new ApolloServer({
  // другие настройки
  plugins: [responseCachePlugin()]
})
```

Данный плагин использует тот же кеш, что и другие инструменты, предоставляемые Сервером. Для среды выполнения кода с несколькими экземплярами сервера лучше использовать технологии хранения распределенного кеша, такие как `Memcached` или `Redis`.

__Идентификация пользователей для `PRIVATE` ответов__

Если кешированный ответ имеет область видимости `PRIVATE`, значение этого ответа доступно только одному пользователю. Разумеется, кеш должен знать, как определить такого пользователя.

Для этого в `responseCachePlugin` передается функция `sessionId`:

```js
import responseCachePlugin from 'apollo-server-plugin-response-cache'
const server = new ApolloServer({
  // другие настройки
  plugins: [responseCachePlugin({
    sessionId: (requestContext) => (requestContext.request.http.headers.get('sessionid') || null)
  })]
})
```

_Обратите внимание_: при отсутствии функции `sessionId`, `PRIVATE` ответы кешироваться не будут.

Кеш использует возвращаемое этой функцией значение для идентификации пользователя, который может получать доступ к `PRIVATE` ответу. В приведенном примере функция использует заголовок `sessionid` из оригинального запроса.

Если клиент выполнит тот же запрос с таким же идентификатором, Сервер вернет защищенный кешированный ответ при наличии такового.

__Разделение ответов для авторизованных и неавторизованных юзеров__

По умолчанию `PUBLIC` (открытые) ответы доступны всем юзерам. Однако при определении функции `sessionId` Сервер кеширует 2 версии каждого открытого ответа:

- одну версию для юзеров, `sessionId` которых имеет значение `null`
- другую - для юзеров, `sessionId` которых имеет ненулевое значение

Это позволяет кешировать разные ответы для авторизованных и неавторизованных юзеров. Это можно использовать, например, для отображения разных элементов в списке меню.

__Настройка чтения и записи__

Кроме `sessionId`, `responseCachePlugin` принимает следующие функции чтения/записи:

- `extraCacheKeyData` - возвращаемое данной функцией значение (любой "стрингифицируемый" объект) добавляется в качестве ключа кешированного ответа. Например, если наше `API` включает переводимый текст, эта функция может возвращать строку, производную от `requestContext.request.http.headers.get('Accept-Language')`
- `shouldReadFromCache` - если данная функция возвращает `true`, Сервер не читает кеш для входящей операции, даже если в кеше имеется валидный ответ
- `shouldWriteToCache` - если данная функция возвращает `false`, Сервер не кеширует ответ для входящей операции, даже если `maxAge` ответа имеет значение больше `0`

## Автоматически сохраняемые постоянные запросы

Постоянные запросы (persisted queries) предназначены для улучшения производительности за счет кеширования больших запросов и их идентификаторов (в виде `SHA-256` хеша) на сервере и отправки этих идентификаторов вместо оригинального ответа на запрос.

__Настройка клиента__

Сервер поддерживает постоянные запросы из коробки. Но на стороне клиента требуется предварительная настройка.

Сначала импортируем функцию `createPersistedQueryLink`:

```js
import { createPersistedQueryLink } from "@apollo/client/link/persisted-queries"
```

Данная функция создает ссылку, которая может быть добавлена в цепочку ссылок. Эта ссылка отвечает за генерацию идентификаторов для постоянных запросов, используя GET-запросы для хешированных запросов и выполняя повторную отправку запросов при необходимости.

Затем добавляем эту ссылку в любое место цепочки ссылок перед ссылкой для отправки ответа:

```js
import { ApolloClient, InMemoryCache, HttpLink } from "@apollo/client"
import { createPersistedQueryLink } from "@apollo/client/link/persisted-queries"
import { sha256 } from 'crypto-hash'

const linkChain = createPersistedQueryLink({ sha256 }).concat(
  new HttpLink({ uri: "http://localhost:4000/graphql" }))

const client = new ApolloClient({
  cache: new InMemoryCache()
  link: linkChain
})
```

__Настройка кеша__

По умолчанию Сервер хранит реестр постоянных запросов в памяти. Если мы передаем в конструктор `ApolloServer` другой `cache`, тогда для хранения реестра будет использоваться этот `cache`.

Для постоянных запросов можно определить отдельный кеш. Для этого в конструктор `ApolloServer` в объект `persistedQueries` передается настройка `cache`. В настоящее время поддерживаются следующие хранилища данных:

- локальный кеш, хранящийся в памяти - `InMemoryLRUCache` (`apollo-server-caching`)
- Memcached - `MemcachedCache` (`apollo-server-cache-memcached`)
- Redis (один экземпляр) - `RedisCache` (`apollo-server-cache-redis`)
- Redis Cluster - `RedisClusterCache` (`apollo-server-cache-redis`)

_Пример использования `Memcached`_

```js
const { MemcachedCache } = require('apollo-server-cache-memcached')
const { ApolloServer } = require('apollo-server')

const server = new ApolloServer({
  typeDefs,
  resolvers,

  persistedQueries: {
    cache: new MemcachedCache(
      ['memcached-1.local', 'memcached-2.local', 'memcached-3.local'],
      { retries: 10, retry: 10000 } // Настройки
    )
  }
})
```

_Пример использования `Redis` (один экземпляр)_

```js
const { BaseRedisCache } = require('apollo-server-cache-redis')
const Redis = require('ioredis')

const server = new ApolloServer({
  typeDefs,
  resolvers,

  persistedQueries: {
    cache: new BaseRedisCache({
      client: new Redis({
        host: 'redis-server',
      })
    })
  }
})
```

_Пример использования `Redis` (`Sentinel`)_

```js
const { BaseRedisCache } = require('apollo-server-cache-redis')
const Redis = require('ioredis')

const server = new ApolloServer({
  typeDefs,
  resolvers,

  persistedQueries: {
    cache: new BaseRedisCache({
      client: new Redis({
        sentinels: [{
          host: 'sentinel-host-01',
          port: 26379
         }],
        password: 'my_password',
        name: 'service_name',
      })
    })
  }
})
```

__Настройка времени жизни кеша__

Время жизни (time-to-live, TTL) кеша - это время, в течение которого зарегистрированный постоянный запрос хранится в кеше. Если время жизни истекло и запрос был очищен, он повторно регистрируется при следующей отправке клиентом.

Кеш, хранящийся в памяти, не имеет времени жизни. Для поддерживаемых хранилищ данных время жизни по умолчанию составляет `300` секунд. Это время можно изменить:

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  persistedQueries: {
    ttl: 900 // 15 минут
  }
})
```

Присвоение настройке `ttl` значения `null` отключает `TLL`:

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  persistedQueries: {
    ttl: null
  }
})
```

__Отключение постоянных запросов__

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,

  persistedQueries: false
})
```

## Аутентификация и авторизация

Как правило, нам нужен какой-то способ управлять тем, какие пользователи могут видеть и взаимодействовать с данными.

- Аутентификация - процесс определения того, выполнил ли пользователь вход в систему, а также определение того, что это за пользователь.
- Авторизация - процесс определения того, какими правами обладает пользователь.

__Помещение аутентифицированного юзера в `context`__

Существует множество способов выполнить аутентификацию пользователя.

В следующем примере мы извлекаем токен пользователя из HTTP-заголовка `Authorization`, включенного в каждый запрос. Затем мы получаем объект пользователя на основе токена и добавляем этот объект в контекст, который передается каждому резолверу. Резолвер может использовать этот объект для определения полномочий пользователя:

```js
const { ApolloServer } = require('apollo-server')

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    // Обратите внимание: в данном примере для доступа к заголовкам используется аргумент `req` (`Express`),
    // однако в других интеграциях (`Koa`, `Lambda` и т.п.) этот аргумент может быть иным

    // Получаем токен из заголовка
    const token = req.headers.authorization || ''

    // Пытаемся получить юзера по токену
    const user = getUser(token)

    // Добавляем юзера в контекст
    return { user }
  }
})

server.listen().then(({ url }) => {
 console.log(`🚀 Сервер запущен по адресу: ${url}`)
})
```

Иногда можно ограничиться помещением в контекст чего-то вроде `{ loggedIn: true }`, но чаще нам требуется некоторая информация о пользователе, например, `{ user: { id: 12345, roles: ['user', 'admin'] } }`.

__Методы авторизации__

_На уровне интерфейса_

Модифицируем функцию контекста:

```js
context: ({ req }) => {
 const token = req.headers.authorization || ''

 const user = getUser(token)

 // Блокируем пользователя
 // Здесь мы также можем проверять роль/полномочия пользователя
 if (!user) throw new AuthenticationError('Вы должны выполнить вход в систему')

 return { user }
}
```

_На уровне резолверов_

Авторизация на уровне резолверов позволяет определять, какие поля являются открытыми, а какие - закрытыми.

В качестве первого примера создадим резолвера, который доступен только валидному юзеру:

```js
users: (parent, args, context) => {
 // В данном случае мы имитируем отсутствие данных для
 // неавторизованного пользователя. Другим вариантом является
 // вызов ошибки
 if (!context.user) return null

 return ['Alice', 'Bob']
}
```

Для блокировки доступа мы можем возвращать `null` или `[]` из резолвера. Вызов ошибки также означает запрет доступа.

Расширим приведенный пример. Теперь для доступа к списку пользователей юзер должен иметь статус администратора:

```js
users: (parent, args, context) => {
  // Проверяем, что пользователь авторизован и имеет статус администратора
  if (!context.user || !context.user.roles.includes('admin')) return null
  return context.models.User.getAll()
}
```

_На уровне моделей_

Вы обратили внимание, что мы заменили массив пользователей на `context.models.User.getAll()`? Это связано с тем, что в идеале логика получения и обработки данных должна содержаться в источниках данных или объектах моделей, а не в резолверах.

Например, модель `User` может включать логику работы с юзерами и выглядеть так:

```js
export const User = {
 getAll: () => { /* логика получения/обработки всех пользователей */ },
 getById: (id) => { /* логика получения/обработки одного пользователя */ },
 getByGroupId: (id) => { /* логика получения/обработки группы пользователей */ }
}
```

Соответствующая схема может выглядеть так:

```gql
type Query {
 user (id: ID!): User
 article (id: ID!): Article
}

type Article {
 author: User
}

type User {
 id: ID!
 name: String!
}
```

Как и логика получения данных, авторизация может быть делегирована модели.

_Делегирование авторизации моделям_

Модель может быть добавлена в контекст точно также, как и пользователь:

```js
context: ({ req }) => {
 const token = req.headers.authentication || ''

 const user = getUser(token);

 if (!user) throw new AuthenticationError('Вы должны выполнить вход в систему')

 // Добавляем пользователя и модели в контекст
 return {
   user,
   models: {
     User: generateUserModel({ user }),
     // ...
   }
 }
}
```

Отрефакторим модель `User`:

```js
export const generateUserModel = ({ user }) => ({
 getAll: () => {},
 getById: (id) => {},
 getByGroupId: (id) => {}
})
```

Теперь любой метод модели имеет доступ к `user`, что позволяет реализовать логику определения полномочий пользователя на уровне модели:

```js
getAll: () => {
 if(!user || !user.roles.includes('admin')) return null
 return fetch('http://myurl.com/users')
}
```

_C помощью кастомной директивы_

```gql
const typeDefs = `
  directive @auth(requires: Role = ADMIN) on OBJECT | FIELD_DEFINITION

  enum Role {
    ADMIN
    REVIEWER
    USER
  }

  type User @auth(requires: USER) {
    name: String
    banned: Boolean @auth(requires: ADMIN)
    canPost: Boolean @auth(requires: REVIEWER)
  }
`
```

Директива `@auth` может вызываться на типе или на полях, если мы хотим ограничить доступ к определенным полям, как в приведенном выше примере. Логика авторизации скрыта в реализации директивы.

Одним из способов реализовать директиву `@auth` является использование настройки `schemaTransforms` функции `makeExecutableSchema` из проекта [`graphql-tools`](https://www.graphql-tools.com/). После реализации `schemaTransforms` сервер создается с помощью `new ApolloServer({ schema: makeExecutableSchema({ typeDefs, resolvers, schemaTransforms }) })`.

_За пределами `GraphQL`_

Если у нас имеется `REST API` со встроенной авторизацией, мы можем сразу передать объект запроса в модель:

```js
// src/server.js
context: ({ req }) => {
 // Передаем объект запроса в модель
 return {
   user,
   models: {
     User: generateUserModel({ req }),
     // ...
   }
 }
}

// src/models/user.js
export const generateUserModel = ({ req }) => ({
 getAll: () => {
   return fetch('http://myurl.com/users', { headers: req.headers })
 }
})
```