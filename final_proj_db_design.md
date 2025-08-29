### 🔹 Користувачі та профілі

* Створення акаунту через email або сторонніх провайдерів (Google, Apple, тощо).
* Збереження профілю користувача (`user_profiles`): ім’я, стать, дата народження, зріст, вага, рівень активності, часовий пояс, країна.
* Статус акаунту (`active`, `disabled`, `deleted`).

---

### 🔹 Цілі харчування

* Встановлення та збереження цілей по калоріях, білках, жирах і вуглеводах (`nutrition_goals`).
* Періоди дії цілей (`start_date`, `end_date`).

---

### 🔹 Продукти та нутрієнти

* Збереження продуктів (`foods`) з брендами, одиницями (`units`) і порціями (`food_portions`).
* Можливість додавати кастомні продукти (`is_custom`, `created_by_user_id`).
* Підтримка баркодів (`barcodes`).
* Зв’язок продуктів з нутрієнтами (`nutrients` і `food_nutrients`) для підрахунку калорій та макро-/мікронутрієнтів.

---

### 🔹 Рецепти та страви

* Збереження рецептів/страв користувачів (`recipes`).
* Зв’язок рецептів з інгредієнтами (`recipe_items`), включаючи кількість та одиниці виміру.
* Можливість обчислювати калорійність страви на основі інгредієнтів.

---

### 🔹 Щоденник харчування

* Ведення щоденника (`diary_days`) з нотатками та відміткою `planned` (плановані/фактичні дні).
* Прийоми їжі (`meals`) і елементи прийому (`meal_items`) — продукти або рецепти, кількість і одиниця виміру.

---

### 🔹 Фізична активність

* Ведення вправ (`exercises`) з датою, назвою, тривалістю та спаленими калоріями.
* Логи ваги (`body_weight_logs`) для відстеження змін маси тіла.

---

Ця схема покриває **MVP для апки з підрахунку калорій**, ведення щоденника харчування, базових вправ та персональних цілей, а також авторизацію через провайдерів.

---

---

### 📋 Таблиці та їх призначення

| Таблиця                | Призначення                                                           | Функціонал, що спирається                             |
| ---------------------- | --------------------------------------------------------------------- | ----------------------------------------------------- |
| **users**              | Облікові записи користувачів, логін/пароль, статус                    | Реєстрація, логін, автентифікація, авторизація        |
| **user\_profiles**     | Особисті дані та параметри (ім’я, стать, зріст, вага, активність, TZ) | Персоналізація, розрахунок калорій та норм, аналітика |
| **user\_providers**    | Зв’язок з соцмережами (Google, Apple, Facebook)                       | Логін через зовнішніх провайдерів (SSO)               |
| **nutrition\_goals**   | Цільові показники харчування                                          | Відстеження прогресу, рекомендації, планування        |
| **brands**             | Бренди харчових продуктів                                             | Каталогізація продуктів, пошук по бренду              |
| **units**              | Одиниці виміру (грам, штука, склянка…)                                | Вибір порцій у щоденнику/рецептах                     |
| **foods**              | База продуктів (назва, бренд, чи користувацький)                      | Каталог їжі, власні продукти користувачів             |
| **barcodes**           | Штрихкоди, прив’язка до продуктів                                     | Пошук продукту по штрихкоду (MVP можна без цього)     |
| **food\_portions**     | Варіанти порцій для продукту (1 шматок = 30 г)                        | Гнучке введення кількості їжі                         |
| **recipes**            | Страви, створені користувачами                                        | Збереження власних рецептів                           |
| **recipe\_items**      | Інгредієнти рецепту (продукт + кількість + одиниця)                   | Розрахунок калорійності/складу страви                 |
| **diary\_days**        | Журнал харчування по днях                                             | Щоденний трекінг, планування, аналітика               |
| **meals**              | Прийоми їжі в межах дня (сніданок, обід…)                             | Організація харчового щоденника                       |
| **meal\_items**        | Конкретні продукти/рецепти у прийомі їжі                              | Ведення харчового щоденника                           |
| **body\_weight\_logs** | Логи ваги користувача                                                 | Трекінг прогресу, графіки змін ваги                   |

---

👉 Таким чином, ядро (без якого немає сенсу) — `users`, `user_profiles`, `foods`, `units`, `food_portions`, `diary_days`, `meals`, `meal_items`.
Другорядні, які можна винести "на потім" — `brands`, `barcodes`, `recipes`, `recipe_items`, `nutrition_goals`, `body_weight_logs`.
---
---

## 📝 Опис БД

### 1. **Користувачі (users, user\_profiles, user\_providers)**

* **Навіщо:** основа багатокористувацької системи, збереження акаунтів, профілів та сторонніх входів.
* **Реальний сценарій:**

  * Новий користувач реєструється → запис у `users`.
  * Він додає стать, зріст, вагу → дані йдуть у `user_profiles`.
  * Якщо він заходить через Google → зберігається зв’язок у `user_providers`.

---

### 2. **Бренди, одиниці, поживні речовини (brands, units, nutrients)**

* **Навіщо:** стандартизація даних про продукти, щоб уникнути хаосу.
* **Реальний сценарій:**

  * Користувач додає в свій раціон йогурт Danone → зберігається бренд "Danone".
  * Для ваги йогурту обирає "грам" чи "порція" з таблиці `units`.
  * Калорійність і білки вказуються у `nutrients`.

---

### 3. **Харчові продукти (foods, food\_portions, food\_nutrients, barcodes)**

* **Навіщо:** опис кожного продукту, як його міряти і які нутрієнти він містить.
* **Реальний сценарій:**

  * У магазині користувач сканує штрихкод шоколадки → знаходимо її в `barcodes`, і одразу бачимо калорійність із `food_nutrients`.
  * Якщо вказати порцію «плитка» → через `food_portions` система знає, що це, наприклад, 90 г.

---

### 4. **Рецепти (recipes, recipe\_items)**

* **Навіщо:** дозволяє групувати продукти в одну страву.
* **Реальний сценарій:**

  * Користувач створює рецепт «Омлет» → запис у `recipes`.
  * Додає яйця, масло, молоко → кожен рядок йде в `recipe_items`.
  * Пізніше він додає «Омлет» до щоденника як одну страву.

---

### 5. **Щоденник (diary\_days, meals, meal\_items)**

* **Навіщо:** зберігає щоденний раціон і прогрес користувача.
* **Реальний сценарій:**

  * Користувач відкриває день «27 серпня» → створюється запис у `diary_days`.
  * Додає прийоми їжі: сніданок, обід, вечеря → таблиця `meals`.
  * До сніданку додає хліб і сир → `meal_items`.
  * Система автоматично рахує калорії за день.

---

### 6. **Цілі і прогрес (nutrition\_goals, body\_weight\_logs)**

* **Навіщо:** фіксуємо мету користувача і відстежуємо зміни у вазі.
* **Реальний сценарій:**

  * Користувач ставить собі ціль: 2000 ккал/день, 120 г білка → запис у `nutrition_goals`.
  * Щотижня він записує вагу → `body_weight_logs`.
  * Система будує графік і показує прогрес.

---

---

### **Єдиний T-SQL скрипт для створення схеми даних**


```sql
-- Створення таблиць, пов'язаних з користувачами
CREATE TABLE users (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    email NVARCHAR(255) UNIQUE,
    password_hash NVARCHAR(MAX),
    status NVARCHAR(50) NOT NULL DEFAULT 'active',
    created_at DATETIMEOFFSET NOT NULL DEFAULT GETDATE()
);

CREATE TABLE user_profiles (
    user_id BIGINT PRIMARY KEY,
    display_name NVARCHAR(255),
    sex NVARCHAR(50),
    birth_date DATE,
    height_cm NUMERIC(5, 2),
    weight_kg NUMERIC(6, 3),
    activity_level NVARCHAR(50),
    timezone NVARCHAR(100),
    country NVARCHAR(100),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE TABLE user_providers (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    provider NVARCHAR(50) NOT NULL,
    provider_user_id NVARCHAR(255) NOT NULL,
    created_at DATETIMEOFFSET NOT NULL DEFAULT GETDATE(),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT UQ_user_providers UNIQUE (provider, provider_user_id)
);

-- Створення таблиць для брендів, одиниць та поживних речовин (незалежні)
CREATE TABLE brands (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE units (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(100) NOT NULL UNIQUE,
    abbr NVARCHAR(20) NOT NULL UNIQUE,
    grams_per_unit NUMERIC(10, 4) NOT NULL
);

CREATE TABLE nutrients (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(100) NOT NULL UNIQUE,
    unit NVARCHAR(20) NOT NULL
);

-- Створення таблиць, що залежать від користувачів
CREATE TABLE nutrition_goals (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE,
    target_calories INT,
    protein_g NUMERIC(7, 2),
    fat_g NUMERIC(7, 2),
    carbs_g NUMERIC(7, 2),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE body_weight_logs (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    date DATE NOT NULL,
    weight_kg NUMERIC(6, 3) NOT NULL,
    note NVARCHAR(MAX),
    FOREIGN KEY (user_id) REFERENCES users(id),
    CONSTRAINT UQ_body_weight_logs UNIQUE (user_id, date)
);

CREATE TABLE recipes (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    name NVARCHAR(255) NOT NULL,
    servings NUMERIC(8, 3) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id),
    CONSTRAINT UQ_recipes UNIQUE (user_id, name)
);

-- Створення таблиць, що залежать від брендів, одиниць та користувачів
CREATE TABLE foods (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    brand_id BIGINT,
    name NVARCHAR(255) NOT NULL,
    default_serving_g NUMERIC(10, 4),
    is_custom BIT NOT NULL DEFAULT 0,
    created_by_user_id BIGINT,
    FOREIGN KEY (brand_id) REFERENCES brands(id),
    FOREIGN KEY (created_by_user_id) REFERENCES users(id),
    CONSTRAINT UQ_foods UNIQUE (brand_id, name)
);

-- Створення таблиць, що залежать від продуктів
CREATE TABLE barcodes (
    code NVARCHAR(255) PRIMARY KEY,
    food_id BIGINT NOT NULL,
    source NVARCHAR(255),
    FOREIGN KEY (food_id) REFERENCES foods(id)
);

CREATE TABLE food_portions (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    food_id BIGINT NOT NULL,
    unit_id BIGINT NOT NULL,
    grams NUMERIC(10, 4) NOT NULL,
    FOREIGN KEY (food_id) REFERENCES foods(id),
    FOREIGN KEY (unit_id) REFERENCES units(id),
    CONSTRAINT UQ_food_portions UNIQUE (food_id, unit_id)
);

CREATE TABLE food_nutrients (
    food_id BIGINT NOT NULL,
    nutrient_id BIGINT NOT NULL,
    amount_per_100g NUMERIC(12, 6) NOT NULL,
    PRIMARY KEY (food_id, nutrient_id),
    FOREIGN KEY (food_id) REFERENCES foods(id),
    FOREIGN KEY (nutrient_id) REFERENCES nutrients(id)
);

-- Створення таблиць для щоденника
CREATE TABLE diary_days (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    date DATE NOT NULL,
    note NVARCHAR(MAX),
    planned BIT NOT NULL DEFAULT 0,
    calories_burned INT DEFAULT 0,
    FOREIGN KEY (user_id) REFERENCES users(id),
    CONSTRAINT UQ_diary_days UNIQUE (user_id, date)
);

CREATE TABLE meals (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    diary_day_id BIGINT NOT NULL,
    meal_type NVARCHAR(50) NOT NULL,
    name NVARCHAR(255),
    FOREIGN KEY (diary_day_id) REFERENCES diary_days(id) ON DELETE CASCADE
);

-- Створення таблиць, що залежать від продуктів, рецептів та щоденника
CREATE TABLE recipe_items (
    recipe_id BIGINT NOT NULL,
    line_no INT NOT NULL,
    food_id BIGINT NOT NULL,
    quantity NUMERIC(12, 4) NOT NULL,
    unit_id BIGINT NOT NULL,
    PRIMARY KEY (recipe_id, line_no),
    FOREIGN KEY (recipe_id) REFERENCES recipes(id),
    FOREIGN KEY (food_id) REFERENCES foods(id),
    FOREIGN KEY (unit_id) REFERENCES units(id)
);

CREATE TABLE meal_items (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    meal_id BIGINT NOT NULL,
    food_id BIGINT,
    recipe_id BIGINT,
    quantity NUMERIC(12, 4) NOT NULL,
    unit_id BIGINT NOT NULL,
    grams_override NUMERIC(12, 4),
    FOREIGN KEY (meal_id) REFERENCES meals(id) ON DELETE CASCADE,
    FOREIGN KEY (food_id) REFERENCES foods(id),
    FOREIGN KEY (recipe_id) REFERENCES recipes(id),
    FOREIGN KEY (unit_id) REFERENCES units(id),
    CONSTRAINT CK_meal_items CHECK ((food_id IS NOT NULL AND recipe_id IS NULL) OR (food_id IS NULL AND recipe_id IS NOT NULL))
);
```



