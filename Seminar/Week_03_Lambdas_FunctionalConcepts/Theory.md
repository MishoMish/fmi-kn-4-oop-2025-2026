# Ламбда функции и функционални концепции

## Въведение

Ламбда функциите (въведени в C++11) са **анонимни функции**, които могат да бъдат дефинирани на място — точно там, където са нужни. Те са изключително полезни за кратки операции, callbacks и функционален стил на програмиране.

> 💡 **Нова седмица** спрямо 2024–2025: Ламбдите бяха обединени с шаблоните миналата година. Тази година ги разглеждаме отделно, за да дадем достатъчно време за captures, mutable ламбди и функционални шаблони.

---

## 1. Синтаксис на ламбда функции

### Пълен синтаксис

```
[capture_list](parameter_list) mutable -> return_type { body }
```

| Част | Задължителна | Описание |
|------|-------------|----------|
| `[capture_list]` | ✅ Да | Какви външни променливи да улови |
| `(parameter_list)` | ❌ (може да се пропусне ако няма параметри) | Параметри на ламбдата |
| `mutable` | ❌ | Позволява промяна на captured-by-value променливи |
| `-> return_type` | ❌ (автоматично се извежда) | Явен тип на връщане |
| `{ body }` | ✅ Да | Тялото на функцията |

### Прости примери

```cpp
// Най-простата ламбда
auto hello = []() { std::cout << "Hello!" << std::endl; };
hello();  // Hello!

// С параметри
auto add = [](int a, int b) { return a + b; };
std::cout << add(3, 4);  // 7

// С явен тип на връщане
auto divide = [](double a, double b) -> double {
    if (b == 0) return 0;
    return a / b;
};
```

---

## 2. Captures — Улавяне на външни променливи

Captures са онова, което отличава ламбдите от обикновените функции. Те позволяват на ламбдата да „вижда" променливи от обхващащата среда.

### Видове captures

```cpp
int x = 10;
int y = 20;

auto f1 = [x]()    { return x; };        // Capture x по стойност
auto f2 = [&x]()   { x++; };             // Capture x по референция
auto f3 = [=]()     { return x + y; };    // Всички по стойност
auto f4 = [&]()     { x++; y++; };        // Всички по референция
auto f5 = [=, &x]() { x++; return y; };   // Всички по стойност, x по референция
auto f6 = [&, x]()  { y++; return x; };   // Всички по референция, x по стойност
```

### Разлика между capture по стойност и по референция

```cpp
int counter = 0;

// Capture по стойност — копие, не се променя оригиналът
auto byValue = [counter]() {
    // counter++;  // ❌ Грешка! Captured by value е const
    std::cout << counter << std::endl;
};

// Capture по референция — работим с оригинала
auto byRef = [&counter]() {
    counter++;
    std::cout << counter << std::endl;
};

byRef();  // counter = 1
byRef();  // counter = 2
byValue(); // Отпечатва 0 (копието от момента на capture)
```

> ⚠️ **Внимание:** При capture по референция, ако оригиналната променлива излезе от обхват преди ламбдата да бъде извикана, имаме **dangling reference** — undefined behavior!

### Кога кое да използваме?

| Capture | Кога | Пример |
|---------|------|--------|
| `[x]` (по стойност) | Когато не искаме промяна, или ламбдата живее по-дълго от `x` | Callback за бъдещо изпълнение |
| `[&x]` (по референция) | Когато искаме промяна, и `x` гарантирано е жив | Брояч, акумулатор |
| `[=]` | Малък брой малки стойности | Прости предикати |
| `[&]` | Много променливи, всички жизнеспособни | Вътрешни helpers |

---

## 3. `mutable` ламбди

По подразбиране, captured-by-value променливите са `const`. Ключовата дума `mutable` позволява промяна на *копието*:

```cpp
int seed = 0;
auto counter = [seed]() mutable {
    return ++seed;  // Променяме копието, не оригинала
};

std::cout << counter() << std::endl;  // 1
std::cout << counter() << std::endl;  // 2
std::cout << counter() << std::endl;  // 3
std::cout << seed << std::endl;       // 0 (оригиналът не е променен)
```

> 🧠 **Дискусия:** Mutable ламбдата има *състояние*. Тя е подобна на функтор (обект с `operator()`). Компилаторът всъщност генерира точно такъв клас за всяка ламбда!

---

## 4. Ламбди и `std::function`

### Съхраняване в `std::function`

```cpp
#include <functional>

std::function<int(int, int)> operation;

operation = [](int a, int b) { return a + b; };
std::cout << operation(3, 4);  // 7

operation = [](int a, int b) { return a * b; };
std::cout << operation(3, 4);  // 12
```

### Предаване като параметър

```cpp
void repeat(int n, std::function<void()> action) {
    for (int i = 0; i < n; i++) {
        action();
    }
}

int count = 0;
repeat(5, [&count]() { count++; std::cout << count << " "; });
// 1 2 3 4 5
```

---

## 5. Генерични ламбди (C++14)

В C++14 ламбдите могат да имат `auto` параметри, което ги прави шаблонни:

```cpp
auto print = [](const auto& value) {
    std::cout << value << std::endl;
};

print(42);           // int
print(3.14);         // double
print("hello");      // const char*
```

### Полезен пример: генеричен компаратор

```cpp
auto less = [](const auto& a, const auto& b) { return a < b; };
auto greater = [](const auto& a, const auto& b) { return a > b; };

// Може да се използва с различни типове
std::cout << less(3, 5) << std::endl;     // true
std::cout << less(3.0, 2.5) << std::endl; // false
```

---

## 6. Функционални шаблони с ламбди

### Композиция на функции

```cpp
template <typename F, typename G>
auto compose(F f, G g) {
    return [f, g](auto x) { return f(g(x)); };
}

auto doubleIt = [](int x) { return x * 2; };
auto addOne = [](int x) { return x + 1; };

auto doubleThenAdd = compose(addOne, doubleIt);
std::cout << doubleThenAdd(5) << std::endl;  // (5 * 2) + 1 = 11

auto addThenDouble = compose(doubleIt, addOne);
std::cout << addThenDouble(5) << std::endl;  // (5 + 1) * 2 = 12
```

### Pipeline (тръбопровод)

```cpp
template <typename T>
T pipeline(T value) { return value; }  // Базов случай

template <typename T, typename F, typename... Fs>
auto pipeline(T value, F first, Fs... rest) {
    return pipeline(first(value), rest...);
}

int result = pipeline(5,
    [](int x) { return x * 2; },     // 10
    [](int x) { return x + 3; },     // 13
    [](int x) { return x * x; }      // 169
);
```

---

## Обобщение

| Концепция | Описание |
|-----------|----------|
| Ламбда | Анонимна функция, дефинирана на място |
| Capture `[=]` | Улавяне по стойност (копие) |
| Capture `[&]` | Улавяне по референция |
| `mutable` | Позволява промяна на captured-by-value |
| `std::function` | Контейнер за произволен извикваем обект |
| Генерични ламбди | `auto` параметри (C++14) |
| Композиция | Комбиниране на функции `f(g(x))` |

> 🧠 **Ключов takeaway:** Ламбдите са „анонимни функтори". Компилаторът генерира клас с `operator()` за всяка ламбда. Captures стават полета на този клас. Разбирането на това е ключово за ООП.
