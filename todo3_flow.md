Ось таблиця, яка підсумовує реалізовані функціональності у вашому ASP.NET Core MVC проєкті на основі нашого діалогу:

| **Що реалізовано**                               | **Як це реалізовано**                                                                                                                                                  |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Перемикання між сховищами (DB/XML)**        | Через `StorageSelectionService`, який зберігає вибір користувача в `HttpContext.Session` і динамічно створює репозиторій (`XmlTaskRepository` або `DBTaskRepository`). |
| **2. Інтерфейс `ITaskRepository`**               | Визначає спільний інтерфейс для всіх репозиторіїв: `GetActiveTasks`, `GetCompletedTasks`, `AddTask`, `MarkTaskAsDone`.                                                 |
| **3. Реалізація `XmlTaskRepository`**            | Частково реалізований: додано **тимчасову реалізацію `GetActiveTasks()`**, яка повертає тестові задачі. Інші методи поки викликають `NotImplementedException`.         |
| **4. Реалізація `DBTaskRepository`**             | Аналогічно, додано **тимчасову реалізацію `GetActiveTasks()`** з тестовими даними. Конструктор приймає рядок з'єднання, але поки не підключено до БД.                  |
| **5. Відображення задач у `TasksController`**    | Метод `Index()` отримує активні задачі через `StorageSelectionService.GetCurrentRepository().GetActiveTasks()` і передає їх у `TasksViewModel`.                        |
| **6. Модель `TaskModel`**                        | Містить всі потрібні поля: `TaskId`, `Description`, `DueDate`, `IsDone`, `CreatedAt`, `CompletedAt`, `CategoryId`, `CategoryName`.                                     |
| **7. Модель `TasksViewModel`**                   | Містить: `List<TaskModel> ActiveTasks`, `List<TaskModel> CompletedTasks`, `TaskModel? NewTask`. Використовується у View для відображення задач.                        |
| **8. Перевірка обраного типу зберігання**        | Через `StorageSelectionService.GetCurrentStorageType()` — перевіряє сесію, або повертає `DefaultStorageType` з `StorageOptions`.                                       |
| **9. Підтримка сесій (Session)**                 | Увімкнено використання сесій для збереження `StorageTypeSessionKey`. Працює через `IHttpContextAccessor`.                                                              |
| **10. Підготовка до інтеграції з БД/XML-файлом** | Репозиторії створюються вручну в `StorageSelectionService` на основі обраного типу зберігання.                                                                         |

---


