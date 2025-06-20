
---

## План презентації: Як працює GraphQL API у моєму проєкті

### 1. Вступ

* Мета - керування задачами з двома типами сховищ — база даних і XML
* Основна точка входу — endpoint `/graphql` для обробки всіх запитів ([Route("graphql")])

### 2. Прийом запиту

* Клієнт надсилає POST-запит з GraphQL-запитом у форматі JSON ([ApiController] - очікує JSON як вхідний формат за замовчуванням, хоча не тільки це визначає вхідний формат)
* Формат тіла запиту: поля `query`, `variables`, `operationName` (defined in GraphQLQuery.cs)
* Тип сховища передається не у запиті, а у заголовку HTTP `X-Storage-Type` (HttpContext.Request.Headers["X-Storage-Type"].FirstOrDefault() ... HttpContext as a field from ControllerBase)

### 3. Обробка запиту в GraphQLController

* Серіалізація і десеріалізація JSON через `IGraphQLTextSerializer` (використання Newtonsoft.Json)
* Читання заголовка `X-Storage-Type` і передача його в контекст користувача (GraphQLController : userContext, every query, additional info)
* Виклик `IDocumentExecuter.ExecuteAsync` — основний метод виконання запиту з передачею схеми, змінних, userContext, DI-сервісів

### 4. Схема та резолвери

* Визначення схеми GraphQL (`TaskSchema`), що містить Query і Mutation (TaskSchema.cs inherits Schema)
* Резолвери використовують `IStorageSelectionService`, щоб вибрати правильне сховище (XML чи база даних) на основі `X-Storage-Type` 
    (`.ResolveAsync(async context => ... var storageSelectionService = context.RequestServices.GetRequiredService<IStorageSelectionService>();`)
* Через інтерфейс `ITaskRepository` звернення до сховища абстраговані від конкретної реалізації

### 5. Вибір сховища

* `StorageSelectionService` читає тип сховища із заголовка або сесії
* Через DI отримує конкретний репозиторій (`XmlTaskRepository` або `DbTaskRepository`)
* Забезпечує гнучкість і масштабованість (легко додати нові сховища)

### 6. Асинхронність

* Використання асинхронних методів `ExecuteAsync`, `GetActiveTasksAsync` і т.д. (impr!!!)
* Переваги: не блокується сервер, підтримка одночасних запитів

### 7. Серіалізація результату

* Повернення результату у вигляді JSON, сформованого через `IGraphQLTextSerializer` (`var json = _serializer.Serialize(result);`)
* Обробка помилок у запитах і їх відправка у відповіді

### 8. Відсутність middleware

* Логіка обробки GraphQL-запитів реалізована без додаткових middleware, через контролер
* Простота та контроль

### 9. Підсумок

* Гнучке API з вибором сховища на рівні запиту через заголовки
* Чітка структура: контролер → виконавець → схема → репозиторії
* Легка підтримка і розширення

---

