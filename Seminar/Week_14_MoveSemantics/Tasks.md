# Седмица 14 — Задачи за затвърждаване

## Задача 1: `UniquePtr` (опростен)

Имплементирайте шаблонен клас `UniquePtr<T>`, който:
- Пази указател `T*`
- **Забранява** копиране (`= delete`)
- Поддържа **move** конструктор и move operator=
- Има методи: `T* get()`, `T& operator*()`, `T* operator->()`
- Деструкторът освобождава паметта

```cpp
UniquePtr<int> p1(new int(42));
// UniquePtr<int> p2 = p1;  // ❌ Компилаторна грешка
UniquePtr<int> p2 = std::move(p1);  // ✅ OK
```

---

## Задача 2: `String` с пълно Rule of Five

Имплементирайте клас `String`:
- Всички 5 специални метода (деструктор, copy ctor, copy=, move ctor, move=)
- Оператор `+` за конкатенация (hint: връща нов обект → move)
- Оператор `<<` за изход
- Добавете `std::cout` в тялото на всеки специален метод, за да проследите кой се извиква

Тествайте:
```cpp
String a("Hello");
String b(" World");
String c = a + b;  // Кои методи се извикват?
```

---

## Задача 3: Move-enabled контейнер

Допълнете `DynamicArray` от примерите с:
- Метод `emplace_back(int)` — добавя елемент (като `push_back`)
- Метод `shrink_to_fit()` — намалява `capacity` до `size`
- Всички методи трябва да са exception-safe

---

## Задача 4: Анализ на извиквания

Без да компилирате, определете за всеки ред кой специален метод ще бъде извикан:

```cpp
String createString() {
    String s("temp");
    return s;
}

int main() {
    String a("Hello");        // ?
    String b = a;             // ?
    String c = std::move(a);  // ?
    String d("World");        // ?
    d = b;                    // ?
    d = std::move(b);         // ?
    String e = createString();// ?
}
```

След това компилирайте и проверете (компилирайте без `-O2`, за да видите всички извиквания).
