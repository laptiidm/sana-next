Ось як я розумію твою ідею MVP для презентації проекту **Time-Slotter**:

---

## 🎯 **Time-Slotter — MVP-версія**

### 📌 **Ідея:**

> Time-Slotter — це простий вебінструмент для візуального управління часом, де користувачі можуть переглядати й бронювати вільні часові слоти, а адміністратори — створювати та редагувати доступний розклад.

---

## 👥 **Цільова аудиторія:**

* **Фрілансери, майстри, інструктори**, які хочуть швидко поділитися своїм розкладом на день/тиждень.
* **Малі бізнеси** — наприклад, салони, репетитори, ремонтники.
* Будь-хто, хто хоче **простий і швидкий спосіб надавати та приймати бронювання**.

---

## 🔑 **Основні можливості MVP:**

### 👨‍💼 Для адміністратора (власник слоту/графіка):

* Візуальна сітка слотів (на день або годину)
* Позначення, які слоти доступні
* Вибір тривалості доступних блоків: 15 / 30 / 45 / 60 хв
* Можливість вказати коментар до слоту

### 👤 Для користувача:

* Перегляд сітки доступного часу
* Бронювання одного або кількох слотів
* Введення власного коментаря
* Подача "заявки на бронювання" (опціонально)

---

## 🧩 **Технічна структура MVP:**

| Компонент      | Технологія                           |
| -------------- | ------------------------------------ |
| UI (інтерфейс) | React                                |
| стан           | Redux                                |
| side-effects   | rxjs                                 |
| Бекенд API     | ASP.NET Core (GraphQL)               |
| База даних     | SQL Server (Express)                 |
| Деплой         | AWS EC2 / Docker / Railway (для MVP) |

---

## ⚙️ **Приклад сценарію використання:**

1. Адміністратор входить в систему (MVP без авторизації — локальна роль).
2. На тижневій сітці позначає доступні години роботи.
3. Користувач заходить на публічну сторінку, бачить сітку.
4. Вибирає вільний слот і надсилає заявку з коментарем.
5. Заявка фіксується в базі або надсилається адміну.

---

## 💡 **Ключова цінність:**

> Замість важких CRM-систем — надається **мінімалістичний, інтуїтивний візуальний інструмент** для керування часом, який легко інтегрувати у свій сайт або використовувати окремо.

---


---

## 📂 Базова структура проєкту з орієнтацією на розширення

---

### 🧱 **Фронтенд (React + Redux + RxJS)**

```
/src
  /api                    # Запити до бекенду (GraphQL/REST)
  /app
    store.ts              # Конфігурація Redux store + RxJS middleware/епіки
    rootReducer.ts
  /components             # Загальні UI-компоненти (кнопки, модалки, поля вводу)
  /features
    /admin
      AdminDashboard.tsx
      adminSlice.ts       # Redux slice (сторінка/логіка)
      adminEpics.ts       # RxJS-епіки для побічних ефектів
      components/         # Локальні компоненти для адміна
    /booking
      BookingPage.tsx
      bookingSlice.ts
      bookingEpics.ts
      components/
    /auth                  # Авторизація (якщо буде)
  /routes
    AppRoutes.tsx          # React Router конфігурація
  /utils                   # Утиліти, допоміжні функції
  /styles                  # Глобальні стилі, теми
  index.tsx                # Вхідний файл
```

---

### 🧩 **Бекенд (ASP.NET Core + GraphQL)**

```
/src
  /API
    Controllers            # Якщо REST, або GraphQL контролери/різолвери
    GraphQL
      Schemas             # GraphQL схеми (typedefs)
      Resolvers           # Логіка GraphQL резолверів
  /Application
    Services              # Бізнес-логіка (сервіси)
    Interfaces            # Інтерфейси для репозиторіїв, сервісів
  /Domain
    Entities              # Основні сутності (Slot, Booking, User, Resource)
    ValueObjects          # Допоміжні класи (наприклад, Duration)
    Exceptions            # Кастомні помилки
  /Infrastructure
    Persistence           # Репозиторії, доступ до БД
    Data                  # Контекст БД (EF Core, налаштування)
    ExternalServices      # Взаємодія з іншими API
  /Common                  # Спільний код (утиліти, константи)
  Startup.cs               # Конфігурація аплікації
  Program.cs               # Точка входу
```

---

## 🛠️ Пояснення

* **Redux slices + RxJS epics**:
  Слайси зберігають локальний стан, епіки слухають екшени і виконують асинхронні дії (наприклад, запити до API, таймери, вебсокети).

* **GraphQL резолвери** розділені по сутностях — зручно додавати нові функції без хаосу.

* **Domain-driven design (DDD)** підхід на бекенді — це для чіткості логіки та легкої підтримки.

---

# DB

```dbml
// Project: Time-Slotter MVP
// DBML syntax for dbdiagram.io

Table Roles {
  Id int [pk, increment]
  Name nvarchar
}

Table Users {
  Id int [pk, increment]
  Username nvarchar
  RoleId int [ref: > Roles.Id]
  CreatedAt datetime
}

Table SlotSets {
  Id int [pk, increment]
  Name nvarchar
  AdminId int [ref: > Users.Id]
  CreatedAt datetime
  IsActive bit
}

Table Slots {
  Id int [pk, increment]
  SlotSetId int [ref: > SlotSets.Id]
  Position int
  IsEnabled bit
}

Table SlotSegments {
  Id int [pk, increment]
  SlotId int [ref: > Slots.Id]
  Index int
  IsBooked bit
  BookedBy int [ref: > Users.Id, null]
  BookedAt datetime [note: 'Nullable until confirmed']
}

Table BookingStatuses {
  Id int [pk, increment]
  Name nvarchar // e.g. 'pending', 'approved', 'rejected'
}

Table BookingRequests {
  Id int [pk, increment]
  SlotSegmentId int [ref: > SlotSegments.Id]
  UserId int [ref: > Users.Id]
  Comment nvarchar
  StatusId int [ref: > BookingStatuses.Id]
  CreatedAt datetime
  ConfirmedAt datetime [null]
  ConfirmedBy int [ref: > Users.Id, null]
}
```

---

### ✅ Пояснення:

* `Roles` — ролі (`admin`, `user`)
* `Users` — користувачі з FK на `Roles`
* `SlotSets` — набір слотів, прив’язаний до `AdminId`
* `Slots` — 12 слотів у кожному слот-сеті
* `SlotSegments` — по 3 в кожному слоті, бронюються після підтвердження
* `BookingRequests` — заявки на бронювання від юзерів
* `BookingStatuses` — таблиця зі статусами: `pending`, `approved`, `rejected`

---




