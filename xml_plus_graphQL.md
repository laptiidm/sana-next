Ось запропонований детальний план архітектури твого To-Do додатка з урахуванням ADO .NET, «чистого» GraphQL та двох незалежних сховищ:

---

## 1. Вступ

* **Мета:** створити односторінковий To-Do List, у якому задачі зберігаються або в реляційній БД (через ADO .NET), або в XML-файлі.
* **Особливість:** паралельно існує GraphQL API (на базі GraphQL.NET без HotChocolate) для сторонніх клієнтів (мобільні/SPA/etc.), але **інтерфейс Razor** залишається класичним MVC.

---

## 2. Технічний стек

* **Фреймворк:** ASP .NET Core MVC
* **Data Access:** ADO .NET (SqlConnection, SqlCommand, SqlDataReader)
* **XML-сховище:** System.Xml / XmlSerializer або LINQ-to-XML
* **GraphQL:** GraphQL.NET + GraphQL.SystemTextJson + GraphQL.Server.Transports.AspNetCore
* **DI:** вбудований контейнер .NET Core
* **Збереження вибору сховища:** Session (обґрунтування — ізоляція користувача + простота, дані не видно в URL)
* **UI:** Razor Views + опціонально невеликий vanilla JS для асинхронних форм

---

## 3. Загальна схема (логічні рівні)

```text
┌───────────────────┐
│    Presentation   │
│ • Razor Controllers
│ • Razor Views
│ • (опц. vanilla JS)
└────────┬──────────┘
         │
┌────────▼──────────┐
│     Services      │
│ • TaskService     │  ← інкапсулює логіку бізнесу
└────────┬──────────┘
         │
┌────────▼──────────┐
│    Repositories   │
│ • ITaskRepository │
│ • DbTaskRepository│  ← ADO.NET
│ • XmlTaskRepository│ ← XML-CRUD
└────────┬──────────┘
         │
┌────────▼──────────┐
│   Data Storage    │
│ • SQL Server      │
│   (Tasks table)   │
│ • tasks.xml       │
└───────────────────┘
```

Паралельно до рівня Services «наскрізь» прокидається GraphQL-endpoint, що теж звертається до `TaskService`.

---

## 4. Компоненти та їх обґрунтування

### 4.1 Presentation

* **TaskController**:

  * `Index()` – читає із Session тип сховища, отримує завдання через `TaskService` і передає у View.
  * `Create(TaskDto dto)` – додає задачу через `TaskService`.
  * `Complete(int id)` – відмічає задачу як виконану.
  * `SwitchStorage(string type)` – записує `"Db"` або `"Xml"` у `HttpContext.Session["Storage"]`.

* **Razor View**:

  * Форма введення (text, date, dropdown категорій, кнопка).
  * Список активних та виконаних (CSS для перекреслених).
  * Dropdown для вибору сховища (значення з ViewBag/Model, onchange → post на `SwitchStorage`).

### 4.2 Service Layer

* **TaskService** (singleton або scoped):

  * Інжектить `IServiceProvider` або factory, щоб динамічно отримувати `ITaskRepository` на основі Session.
  * Методи: `GetActive()`, `GetCompleted()`, `Add(TodoTask)`, `Complete(int)`.

### 4.3 Repository Layer

* **ITaskRepository** (інтерфейс): стандартні CRUD-методи.
* **DbTaskRepository** (ADO .NET):

  * Використовує `SqlConnection`, `SqlCommand` з параметрами, транзакції за потреби.
  * Таблиця `Tasks` з полями: `Id, Text, DueDate, Category, IsCompleted, CreatedAt`.
* **XmlTaskRepository**:

  * Читає/записує `tasks.xml` через `XmlSerializer` або `XDocument`.
  * Формує ті самі моделі `TodoTask`.

---

## 5. Збереження поточного сховища

* **Де?** — в `HttpContext.Session["StorageType"]`.
* **Чому Session?**

  * Ізольовано під кожного користувача (дозволяє мати різний вибір одночасно в декількох вкладках).
  * Не видно в URL (на відміну від query-string).
  * Достатньо простий механізм для web-додатка без складної авторизації.
* **Як?**

  1. У `SwitchStorage(string type)` записуємо в Session:

     ```csharp
     HttpContext.Session.SetString("StorageType", type);
     ```
  2. В `TaskService` або `TaskController` читаємо:

     ```csharp
     var type = HttpContext.Session.GetString("StorageType") ?? "Db";
     var repo = type == "Xml"
         ? _serviceProvider.GetService<XmlTaskRepository>()
         : _serviceProvider.GetService<DbTaskRepository>();
     ```

---

## 6. GraphQL API (чистий GraphQL.NET)

* **Endpoint:** `/graphql` (без UI-інтеграції в MVC).
* **Схема:**

  * `TodoQuery`: поля `activeTasks`, `completedTasks`.
  * `TodoMutation`: `addTask(text, dueDate, category): TodoTask`, `completeTask(id): Boolean`.
* **Вибір сховища:**

  * На рівні резолверів Query/Mutation так само читаємо Session (через `IHttpContextAccessor`) або приймаємо аргумент `storageType`.
* **Конфігурація в `Startup`:**

  ```csharp
  services
    .AddSingleton<IHttpContextAccessor, HttpContextAccessor>()
    .AddSingleton<ISchema, TodoSchema>()
    .AddGraphQL(options => { options.EnableMetrics = false; })
    .AddSystemTextJson();
  app.UseSession();
  app.UseGraphQL<ISchema>();
  app.UseGraphQLPlayground(); 
  ```

---

## 7. Нефункціональні аспекти

* **Безпека:** захист від XSS у Razor, CSRF-токени на POST.
* **Конфігурація:** рядок з’єднання в `appsettings.json`, шлях до `tasks.xml`.
* **Логування:** Serilog / вбудоване ILogger для помилок ADO та XML.
* **Тестування:**

  * Unit-тести для `DbTaskRepository` (з in-memory DB або mock).
  * Integration-тести для XML (тимчасова папка).
  * Тести GraphQL резолверів (через `GraphQLTestServer`).

---

## 8. Міграція даних (опціонально)

* Якщо згодом захочеш переносити задачі з XML → DB, можеш написати окремий міграційний скрипт або endpoint у адмін-панелі.

---

❓ Якщо цей план підходить — можемо деталізувати кожен розділ кодом чи діаграмою клієнт-сервер.
