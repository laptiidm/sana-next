
---

# Основні сутності в `GraphQLController`

### 1. **Клас контролера**

* `GraphQLController` — API контролер, який обробляє GraphQL-запити по маршруту `/graphql`.
* Наслідується від `ControllerBase` (API-only, без підтримки представлень).

---

### 2. **Залежності (через конструктор)**

* `ISchema _schema` — GraphQL схема, що описує структуру API (типи, запити, мутації).
* `IDocumentExecuter _executer` — сервіс для виконання GraphQL-запитів.
* `IGraphQLTextSerializer _serializer` — сервіс для серіалізації та десеріалізації JSON (GraphQL результатів і вхідних змінних).

---

### 3. **Метод обробки запиту**

* `[HttpPost] Post([FromBody] GraphQLQuery query)` — асинхронний метод, який приймає GraphQL-запит у форматі JSON через тіло HTTP POST.

---

### 4. **Вхідна модель**

* `GraphQLQuery query` — DTO (модель даних), що містить:

  * `Query` — текст GraphQL-запиту.
  * `OperationName` — ім’я операції (якщо їх кілька в запиті).
  * `Variables` — змінні для GraphQL-запиту.

---

### 5. **Обробка заголовків**

* Зчитування заголовку HTTP `X-Storage-Type` через `HttpContext.Request.Headers`.
* Передача його у `userContext` — словник, який доступний у резолверах GraphQL.

---

### 6. **Обробка змінних**

* Якщо вхідні `Variables` не null:

  * Серіалізуємо змінні у JSON.
  * Десеріалізуємо у `Inputs` — тип GraphQL для змінних запиту.
* Якщо немає — використовуємо `Inputs.Empty`.

---

### 7. **Виконання GraphQL-запиту**

* Виклик `_executer.ExecuteAsync(...)` з параметрами:

  * `Schema`, `Query`, `OperationName`, `Variables`.
  * `UserContext` (для передачі додаткових даних у резолвери).
  * `RequestServices` — для інжекції додаткових сервісів у GraphQL.
  * `ThrowOnUnhandledException` — викидає помилки при їх виникненні.

---

### 8. **Обробка результату**

* Якщо у результаті є помилки (`result.Errors`):

  * Повертаємо HTTP 400 BadRequest з описом помилок у JSON.
* Інакше:

  * Серіалізуємо результат у JSON і повертаємо його клієнту з контент-типом `application/json`.

---

### 9. **HTTP відповіді**

* `BadRequest(string message)` — повертає код 400 при відсутності або некоректності запиту.
* `Content(string json, string contentType)` — повертає JSON-дані з потрібним `Content-Type`.

---

# Підсумок у вигляді списку сутностей:

1. **GraphQLController** — контролер для GraphQL API
2. **ISchema** (`_schema`) — схема GraphQL
3. **IDocumentExecuter** (`_executer`) — виконавець запитів
4. **IGraphQLTextSerializer** (`_serializer`) — серіалізатор JSON
5. **GraphQLQuery** (`query`) — модель вхідного запиту
6. **HttpContext.Request.Headers\["X-Storage-Type"]** — заголовок для вибору сховища
7. **userContext** — словник для передачі додаткових даних у резолвери
8. **Inputs** — GraphQL-об'єкт змінних запиту
9. **ExecuteAsync(...)** — асинхронне виконання GraphQL-запиту
10. **GraphQLResult** (`result`) — результат виконання з даними та помилками
11. **IActionResult** — результат HTTP відповіді: 400 або 200 з JSON

---

