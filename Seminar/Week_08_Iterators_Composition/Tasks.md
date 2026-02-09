# Седмица 8 — Задачи за затвърждаване

## Задача 1: `RangeIterator`

Създайте клас `Range`, който поддържа range-based for за обхождане на числа от `start` до `end` (exclusive):

```cpp
for (int val : Range(1, 6)) {
    std::cout << val << " ";  // 1 2 3 4 5
}
```

Разширете с опционална стъпка:
```cpp
for (int val : Range(0, 10, 2)) {
    std::cout << val << " ";  // 0 2 4 6 8
}
```

---

## Задача 2: Итератор за `DynamicArray`

Вземете класа `DynamicArray` от Седмица 5 и добавете му:
- Вложен клас `Iterator` с `*`, `++`, `!=`
- `begin()` и `end()` методи
- Const версии

Тествайте с range-based for цикъл.

---

## Задача 3: Обратен итератор

Създайте клас `ReversibleArray`, който поддържа и нормално, и обратно обхождане:

```cpp
ReversibleArray arr(5);
// ... запълване ...

for (int val : arr) { ... }           // Нормален ред
for (int val : arr.reversed()) { ... } // Обратен ред
```

> 💡 `reversed()` може да връща помощен обект с различни `begin()`/`end()`.

---

## Задача 4: Композиция — `Schedule`

Създайте система за разписание:
- `TimeSlot` с начален и краен час (`int` часове и минути)
- `Lecture` с предмет, преподавател и `TimeSlot` (композиция)
- `DaySchedule` с масив от `Lecture` обекти (композиция)
- Методи: `addLecture()`, `print()`, `hasConflict()` (проверка за припокриване)
