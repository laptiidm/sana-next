Чудово! Ось чіткий покроковий алгоритм, як реалізувати GraphQL API без middleware, щоб через один контролер можна було звертатися до двох різних сховищ (Xml і Db), враховуючи, що у тебе вже є два репозиторії і логіка вибору сховища:

---

## Алгоритм покрокової реалізації GraphQL API без middleware

---

### 1. **Визначення схеми GraphQL**

* **Створити GraphQL-схему (`Schema`)** — це головний клас, який описує типи даних і кореневі запити.
* **Визначити типи (ObjectGraphType) для задач (TaskType)**

  * Відобразити всі властивості `TaskModel`.
* **Визначити кореневий тип запитів (Query)**

  * Запити для отримання активних та виконаних задач.
  * Запити повинні приймати параметр (наприклад, тип сховища — Xml або Db).
* **Визначити мутації (Mutation)**

  * Для додавання задачі.
  * Для позначення задачі як виконаної.

---

### 2. **Реалізація резолверів (Resolvers)**

* Резолвери — це методи, які будуть обробляти запити та мутації.
* Вони звертаються до сервісу або безпосередньо до репозиторію.
* **Головне**: залежно від переданого параметру сховища (Xml або Db) — викликати відповідний репозиторій.

---

### 3. **Інтеграція сховищ у GraphQL**

* Створити сервіс або інтерфейс, який на основі параметра сховища буде повертати потрібний репозиторій (`ITaskRepository`).
* Використовувати цей сервіс в резолверах.

---

### 4. **Створення контролера GraphQL**

* Створити API контролер, наприклад `GraphQLController`.
* В контролері:

  * Приймати запити (GET/POST).
  * Вилучати GraphQL запит (query, variables, operationName).
  * Викликати `IDocumentExecuter.ExecuteAsync()` із відповідною схемою і параметрами.
  * Повернути JSON з результатом.

---

### 5. **Налаштування DI (Dependency Injection)**

* Зареєструвати схему, `IDocumentExecuter`, сервіси сховищ, сервіси вибору сховища.
* В контролері інжектити схему та потрібні сервіси.

---

### 6. **Тестування**

* Використовувати GraphiQL, Postman, Insomnia або будь-який інший GraphQL клієнт.
* Робити запити із параметром вибору сховища.
* Перевіряти, що отримуєш дані з правильного сховища.

---

## Схематичний список класів/сутностей, які треба реалізувати:

| Назва класу/сервісу            | Роль / Завдання                                                                   |
| ------------------------------ | --------------------------------------------------------------------------------- |
| `TaskType`                     | GraphQL Object Type, описує TaskModel                                             |
| `TasksQuery`                   | Кореневий Query — отримання задач (active, completed) з вибором сховища           |
| `TasksMutation`                | GraphQL Mutation — додавання задачі, позначення як виконаної                      |
| `TasksSchema`                  | GraphQL Schema, об'єднує Query та Mutation                                        |
| `IStorageSelectionService`     | Сервіс вибору сховища (є у тебе)                                                  |
| `ITaskRepository`              | Інтерфейс репозиторію (є у тебе)                                                  |
| `XmlTaskRepository`            | Репозиторій для XML (є)                                                           |
| `DbTaskRepository`             | Репозиторій для бази даних (є)                                                    |
| `GraphQLController`            | ASP.NET Core API контролер, що приймає GraphQL запити і виконує їх без middleware |
| `GraphQLService` (опціонально) | Сервіс для логіки вибору репозиторію за параметром і делегування викликів         |

---

## Послідовність дій

1. **Опиши GraphQL тип `TaskType` для TaskModel**
2. **Опиши кореневий запит `TasksQuery` з полями:**

   * `activeTasks(storage: StorageType): [TaskType]`
   * `completedTasks(storage: StorageType): [TaskType]`
3. **Опиши мутації `TasksMutation`:**

   * `addTask(storage: StorageType, taskInput: TaskInput): TaskType`
   * `markTaskAsDone(storage: StorageType, taskId: Int): TaskType`
4. **Створи схему `TasksSchema`, додай Query і Mutation**
5. **Створи сервіс (або використай вже існуючий `IStorageSelectionService`) для вибору репозиторію за параметром `storage`**
6. **Створи контролер `GraphQLController`**:

   * Приймай запити
   * Виконуй `ExecuteAsync` із переданою схемою та параметрами
   * Поверни JSON результат
7. **Зареєструй у DI всі необхідні компоненти (Schema, DocumentExecuter, репозиторії, сервіси)**
8. **Перевір у GraphiQL / Postman, що можеш робити запити з параметром вибору сховища і отримувати відповідні результати**

---
| Компонент                 | Призначення                       |
| ------------------------- | --------------------------------- |
| Типи (`TaskType`)         | Опис структури даних              |
| Запити (`Query`)          | Опис можливих операцій читання    |
| Мутації (`Mutation`)      | Опис операцій зміни даних         |
| Вхідні типи (`InputType`) | Опис формату даних для введення   |
| Схема (`Schema`)          | Збірка всіх частин у цілісний API |

---

### Простими словами:

* **Схема (Schema)** — це **колекція класів**, що описують різні типи (наприклад, `TaskType`), запити (`Query`), мутації (`Mutation`) і вхідні типи (`InputType`).

* І є **агрегатор**, клас `Schema`, який **об’єднує** ці всі компоненти в єдину GraphQL-схему, що сервер використовує для обробки запитів.

---

### У вигляді структури:

```
TaskType            // описує поля об'єкта Task
TaskInputType       // описує вхідні дані для створення/редагування Task
TaskQuery           // описує запити (отримати список задач, задачу за id)
TaskMutation        // описує мутації (додати, оновити, видалити задачу)
TaskSchema          // агрегує Query, Mutation і типи в єдину схему
```

---

### І тоді GraphQL сервер знає:

* Які **дані** доступні (через типи).
* Як їх **читати** (через `Query`).
* Як їх **змінювати** (через `Mutation`).
* І всю цю логіку збирає у єдину **схему**.

---

Ось повністю готовий файл `GraphQLController.cs`, з урахуванням твоєї структури (`TaskQuery`, `TaskMutation`, `TaskType`, `TaskInputType`, `TaskSchema`) та залежностей.

---

### 📄 `Controllers/GraphQLController.cs`

> Створи папку `Controllers`, якщо її ще нема, і додай цей файл:

```csharp
using GraphQL;
using GraphQL.Types;
using Microsoft.AspNetCore.Mvc;
using Todo_List_3.GraphQL;

namespace Todo_List_3.Controllers
{
    [Route("graphql")]
    [ApiController]
    public class GraphQLController : ControllerBase
    {
        private readonly ISchema _schema;
        private readonly IDocumentExecuter _executer;

        public GraphQLController(ISchema schema, IDocumentExecuter executer)
        {
            _schema = schema;
            _executer = executer;
        }

        [HttpPost]
        public async Task<IActionResult> Post([FromBody] GraphQLQuery query)
        {
            if (query == null || string.IsNullOrEmpty(query.Query))
                return BadRequest("GraphQL query is missing.");

            var result = await _executer.ExecuteAsync(options =>
            {
                options.Schema = _schema;
                options.Query = query.Query;
                options.OperationName = query.OperationName;
                options.Inputs = query.Variables?.ToInputs();
                options.RequestServices = HttpContext.RequestServices;
                options.ThrowOnUnhandledException = true;
            });

            if (result.Errors?.Count > 0)
                return BadRequest(result);

            return Ok(result);
        }
    }
}
```

---

### 📄 `GraphQL/GraphQLQuery.cs`

> Додай цей новий клас у папку `GraphQL`:

```csharp
using System.Collections.Generic;

namespace Todo_List_3.GraphQL
{
    public class GraphQLQuery
    {
        public string? Query { get; set; }
        public string? OperationName { get; set; }
        public Dictionary<string, object>? Variables { get; set; }
    }
}
```

---

### 🔧 Додай до `Program.cs` такі реєстрації

Перевір, що в тебе **є або додай**:

```csharp
builder.Services.AddSingleton<IDocumentExecuter, DocumentExecuter>();
builder.Services.AddSingleton<IGraphQLTextSerializer, GraphQL.SystemTextJson.DocumentWriter>();

// ці вже мають бути
builder.Services.AddSingleton<TaskType>();
builder.Services.AddSingleton<TaskInputType>();
builder.Services.AddScoped<TaskQuery>();
builder.Services.AddScoped<TaskMutation>();
builder.Services.AddScoped<ISchema, TaskSchema>();
```

---

### ✅ Тест

У Postman або Insomnia:

* **POST** на `https://localhost:{порт}/graphql`
* JSON:

```json
{
  "query": "{ tasks { id title isCompleted } }"
}
```

---

🔚 Якщо потрібно — можу також згенерувати `launchSettings.json` або допомогти додати Swagger для зручності виклику.







