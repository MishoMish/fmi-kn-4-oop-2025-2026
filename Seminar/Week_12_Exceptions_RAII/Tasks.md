# Седмица 12 — Задачи за затвърждаване

## Задача 1: `SafeString`

Създайте клас `SafeString`, който:
- Пази динамичен символен низ
- Хвърля `std::invalid_argument` при подаден `nullptr` в конструктора
- `operator[]` хвърля `std::out_of_range` при невалиден индекс
- Имплементира RAII — деструкторът освобождава паметта

Тествайте с `try`/`catch` блокове за различните видове грешки.

---

## Задача 2: RAII Logger

Създайте клас `ScopedLogger`, който:
- В конструктора отваря файл за записване и записва `"[START] <message>"`
- В деструктора записва `"[END] <message>"` и затваря файла
- Дори при изключение в обхвата, `[END]` се записва автоматично

```cpp
void riskyOperation() {
    ScopedLogger log("operations.log", "riskyOperation");
    // ... код, който може да хвърли изключение
}
```

---

## Задача 3: Йерархия от банкови изключения

Имплементирайте:
- `BankError` ← `std::runtime_error`
- `InsufficientFundsError` с поле `amount` (колко липсват)
- `AccountLockedError` с поле `reason`

Клас `BankAccount` с методи `deposit()`, `withdraw()`, `transfer()`, които хвърлят съответните изключения.

---

## Задача 4: Exception-safe `resize`

Имплементирайте метод `resize(int newSize)` за `SafeArray` (от Пример 1), който:
- Заделя нова памет
- Копира данните
- Ако `new` хвърли `std::bad_alloc`, старият масив остава непроменен (strong exception guarantee)

Подсказка: Използвайте copy-and-swap идиома.
