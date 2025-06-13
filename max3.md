Ось приклад списку основних сутностей (класів і інтерфейсів) твого проєкту з коротким описом їх ролі, згрупованих по папках:

---

### Configurations

* **StorageOptions** — клас для зчитування конфігурації шляху до XML та рядка підключення до бази даних.

---

### Controllers

* **GraphQLController** — API-контролер, який приймає GraphQL-запити, виконує їх через схему і повертає результат у JSON.
* **TasksController** (ймовірно є) — клас для веб-інтерфейсу чи інших HTTP-запитів, пов’язаних із задачами.

---

### Enums

* **StorageType** — enum, який визначає тип сховища: XML або Database.
* **GraphQLStorageType** — GraphQL-обгортка enum StorageType для використання в GraphQL-схемі.

---

### GraphQL

* **TaskQuery** — клас GraphQL-запитів (query), який містить поля для отримання активних та виконаних задач.
* **TaskMutation** — клас GraphQL-мутацій (mutation), що реалізує додавання і оновлення задач.
* **TaskType** — опис GraphQL-типу, який відповідає моделі TaskModel.
* **TaskInputType** — опис типу введення для мутацій (наприклад, для створення нової задачі).
* **TaskSchema** — GraphQL-схема, що зв’язує Query і Mutation.
* **GraphQLQuery** — клас для десеріалізації вхідного JSON-запиту (query, variables).
* **IDocumentExecuter** — сервіс, який виконує GraphQL-запит за схемою.
* **IDocumentWriter** — сервіс, який серіалізує результат виконання запиту у JSON.
* **DocumentExecuter** — стандартна реалізація IDocumentExecuter.
* **DocumentWriter** — (за потребою) реалізація IDocumentWriter, яка відповідає за серіалізацію результатів.

---

### Models

* **TaskModel** — модель задачі, яка містить властивості: TaskId, Description, DueDate, IsDone тощо.
* **CategoryModel** (можливо) — модель категорії задач.
* (інші моделі, якщо є)

---

### Services

* **IStorageSelectionService** — інтерфейс для вибору активного сховища.
* **StorageSelectionService** — реалізація сервісу вибору сховища.
* **ITaskRepository** — інтерфейс для роботи з колекцією задач.
* **XmlTaskRepository** — реалізація сховища задач на базі XML-файлу.
* **DbTaskRepository** — реалізація сховища задач на базі бази даних.

---

Якщо потрібно, можу допомогти структурувати цей список детальніше або додати конкретні описи методів і властивостей.
