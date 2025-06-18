
---

# 🧠 Todo List API (GraphQL на ASP.NET Core)

## ✅ Загальний опис

Це GraphQL API-проєкт для керування задачами (todo), побудований на ASP.NET Core з використанням бібліотеки [GraphQL.NET](https://github.com/graphql-dotnet/graphql-dotnet).
Сховище даних обирається динамічно між **XML** та **базою даних (SQL Server)**.

---

## ⚙️ Технологічні особливості

* **Code-first GraphQL API** (схема створюється через C# класи)
* **Сховище обирається через заголовок HTTP** (`X-Storage-Type`)
* **Без використання GraphQL middleware** (`MapGraphQL()` не застосовується)
* **Класичний endpoint** `/graphql`, який приймає POST-запити
* **Серіалізація**: `GraphQL.NewtonsoftJson`
* **Використовується DI (Dependency Injection)** для всіх компонентів
* **Підтримка сесій** (для веб-інтерфейсу)

---

## 📦 Основні сутності

| Клас / Інтерфейс    | Роль                                                                     |
| ------------------- | ------------------------------------------------------------------------ |
| **Controllers**     |                                                                          |
| `GraphQLController` | Обробляє POST-запити до `/graphql`, викликає схему, серіалізує відповідь |
| `TasksController`   | MVC контролер для web UI (Razor), працює з `IStorageSelectionService`    |

\| **GraphQL**                          |      |
\| `TaskSchema`                        | Основна схема: реєструє `Query` і `Mutation` |
\| `TaskQuery`                         | Описує GraphQL-запити: `activeTasks`, `completedTasks` |
\| `TaskMutation`                      | Описує мутації: `addTask`, `markTaskAsDone` |
\| `TaskType`                          | GraphQL тип для задачі (`TaskModel`) |
\| `TaskInputType`                     | Вхідний тип для мутацій |
\| `GraphQLStorageType`               | Enum у вигляді `EnumerationGraphType` (XML або Database) |
\| `GraphQLQuery`                     | DTO для отримання запиту з клієнта |
\| `IGraphQLTextSerializer`           | Інтерфейс серіалізатора (реалізується `GraphQLSerializer`) |

\| **Repositories**                    |      |
\| `ITaskRepository`                  | Інтерфейс для роботи зі сховищем |
\| `XmlTaskRepository`               | Зберігає задачі у XML |
\| `DbTaskRepository`                | Працює з SQL Server через ADO.NET |
\| `IXmlRepositorySettingsProvider`  | Надає шлях до XML з конфігурації |
\| `IDatabaseRepositorySettingsProvider` | Надає connection string до бази даних |

\| **Services**                        |      |
\| `IStorageSelectionService`        | Вибирає потрібне сховище (`ITaskRepository`) залежно від заголовку `X-Storage-Type` |
\| `StorageSelectionService`         | Реалізація логіки вибору сховища |

\| **Models**                          |      |
\| `TaskModel`                        | Модель задачі |
\| `TasksViewModel`                  | ViewModel для Razor-сторінки |

\| **Enums**                           |      |
\| `StorageType`                     | Перелічення типів сховищ: `Xml`, `Database` |

\| **Configurations**                 |      |
\| `StorageOptions`                  | Конфігурація шляху до XML та рядка з'єднання для БД (з `appsettings.json`) |

---

## 📂 Конфігурація (appsettings.json)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=...;Database=...;"
  },
  "StorageOptions": {
    "XmlFilePath": "Storage/tasks.xml",
    "DefaultStorageType": "Database"
  }
}
```

---

## 🧩 Архітектура обробки запиту (GraphQL)

1. **Клієнт** надсилає POST-запит на `/graphql` з тілом `{"query": "...", "variables": {...}}` та заголовком `X-Storage-Type: Xml|Database`
2. `GraphQLController` читає `X-Storage-Type`, виконує `IDocumentExecuter.ExecuteAsync(...)`
3. В `UserContext` передається тип сховища
4. `TaskQuery` / `TaskMutation` через `IStorageSelectionService` отримують відповідний репозиторій
5. Результат серіалізується в JSON і повертається клієнту

---

## 🧪 Як тестувати

* Тестовий GraphQL-запит:

```json
POST /graphql
Headers: 
  Content-Type: application/json
  X-Storage-Type: Database

{
  "query": "query($type: StorageType!) { activeTasks(storageType: $type) { taskId description } }",
  "variables": {
    "type": "Database"
  }
}
```

---

## 🛠 Подальший розвиток

* Винесення вибору сховища в middleware або context accessor
* Відмова від дублювання `storageType` в змінних
* Можлива міграція на Minimal API з GraphQL.Server

---

