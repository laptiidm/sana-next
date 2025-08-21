### 🔑 Робота з гостями та реєстрацією

1. **Ідентифікація гостя**

   * При першому вході генерується `GuestId (GUID)`
   * Зберігається у `localStorage`/cookies
   * Усі дані (`MealLog`, `Settings`) прив’язуються до `GuestId`

2. **Збереження цільових калорій**

   * Для гостя: `TargetCalories` зберігається по `GuestId`
   * Для зареєстрованого: по `UserId`

3. **Перехід до акаунта**

   * Після реєстрації всі записи з `GuestId` переносяться на `UserId`
   * `GuestId` можна деактивувати

4. **Особливості**

   * Якщо юзер чистить localStorage → дані губляться
   * Кілька пристроїв = кілька `GuestId` (злиття можна додати пізніше)

---

```sql
-- Користувачі
CREATE TABLE Users (
    UserId UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Email NVARCHAR(255) UNIQUE NULL,
    PasswordHash NVARCHAR(255) NULL,
    CreatedAt DATETIME2 DEFAULT SYSDATETIME()
);

-- Гості (для тих, хто ще не зареєстрований)
CREATE TABLE Guests (
    GuestId UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    CreatedAt DATETIME2 DEFAULT SYSDATETIME()
);

-- Цільові калорії (налаштування)
CREATE TABLE UserSettings (
    SettingId UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    UserId UNIQUEIDENTIFIER NULL,
    GuestId UNIQUEIDENTIFIER NULL,
    TargetCalories INT NOT NULL DEFAULT 2000,
    FOREIGN KEY (UserId) REFERENCES Users(UserId),
    FOREIGN KEY (GuestId) REFERENCES Guests(GuestId)
);

-- Журнал прийомів їжі
CREATE TABLE MealLogs (
    MealId UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    UserId UNIQUEIDENTIFIER NULL,
    GuestId UNIQUEIDENTIFIER NULL,
    FoodName NVARCHAR(255) NOT NULL,
    Calories INT NOT NULL,
    CreatedAt DATETIME2 DEFAULT SYSDATETIME(),
    FOREIGN KEY (UserId) REFERENCES Users(UserId),
    FOREIGN KEY (GuestId) REFERENCES Guests(GuestId)
);

-- (Опційно) Інгредієнти/деталі страви
CREATE TABLE MealItems (
    ItemId UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    MealId UNIQUEIDENTIFIER NOT NULL,
    IngredientName NVARCHAR(255) NOT NULL,
    Calories INT NOT NULL,
    FOREIGN KEY (MealId) REFERENCES MealLogs(MealId)
);
```

### 🔑 Логіка

* **Гість**: усі записи йдуть через `GuestId`.
* **Зареєстрований**: йдуть через `UserId`.
* **При реєстрації**: робимо `UPDATE MealLogs/UserSettings SET UserId = @NewUserId WHERE GuestId = @GuestId`.

---


