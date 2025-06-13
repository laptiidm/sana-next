
---

### Як прикручено GraphQL у твоєму проєкті

* **Сервіси GraphQL реєструються через DI (Dependency Injection)**:

  * Реєструються GraphQL типи (наприклад, `TaskType`, `TaskInputType`, `GraphQLStorageType`).
  * Реєструються запити та мутації (`TaskQuery`, `TaskMutation`).
  * Реєструється схема (`ISchema`), яка агрегує запити та мутації.
  * Реєструються інтерфейси для виконання запитів та формування відповіді (`IDocumentExecuter`, `IDocumentWriter`).

* **У тебе немає GraphQL middleware** на рівні конвеєра ASP.NET (наприклад, `app.UseGraphQL()`), бо замість цього всі запити йдуть через звичайний MVC контролер — `GraphQLController`.

* **`GraphQLController` — це API-контролер, який приймає HTTP POST на `/graphql`,** приймає тіло запиту, виконує GraphQL через `IDocumentExecuter`, серіалізує результат через `IDocumentWriter` і повертає JSON.

---

### Що саме реєструється у `Program.cs`

```csharp
// GraphQL типи
builder.Services.AddSingleton<TaskType>();
builder.Services.AddSingleton<TaskInputType>();
builder.Services.AddSingleton<GraphQLStorageType>();

// GraphQL запити та мутації
builder.Services.AddScoped<TaskQuery>();
builder.Services.AddScoped<TaskMutation>();

// Схема, яка об'єднує запити і мутації
builder.Services.AddScoped<ISchema, TaskSchema>();

// Виконавець запитів і серіалізатор результату
builder.Services.AddSingleton<IDocumentExecuter, DocumentExecuter>();
builder.Services.AddSingleton<IDocumentWriter, DocumentWriter>();
```

---

### Як це працює під час запиту

1. Клієнт робить HTTP POST на `/graphql` (через контролер).

2. Контролер отримує запит, викликає `IDocumentExecuter.ExecuteAsync()`, передаючи схему, запит і параметри.

3. `DocumentExecuter` виконує резолвери (твої методи у `TaskQuery`, `TaskMutation`), отримує дані із потрібного сховища.

4. Результат серіалізується у JSON через `IDocumentWriter`.

5. JSON відправляється клієнту.

---

### Чому так?

* Такий підхід дає повний контроль над HTTP-інтерфейсом (через контролер).
* Не потрібно додатково підключати middleware.
* Легко налаштовувати авторизацію, логування, валідацію через стандартні ASP.NET механізми.

---

