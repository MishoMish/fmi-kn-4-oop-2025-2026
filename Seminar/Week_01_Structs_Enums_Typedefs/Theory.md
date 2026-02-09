# Структури, Enum, Union, Typedef и Namespace в C++

## Въведение

Преди да навлезем в класовете и обектно-ориентираното програмиране, трябва да разберем основните средства за дефиниране на потребителски типове данни в C++. Тази седмица покрива `struct`, `enum`, `union`, `typedef`/`using` и `namespace` — градивните елементи, върху които стъпва ООП.

---

## 1. Структури (`struct`)

### Какво е структура?

Структурата е колекция от свързани данни, групирани под едно име. Тя може да съдържа членове от различни типове — числа, символи, масиви, дори други структури.

```cpp
struct Point {
    int x;
    int y;
};
```

### Деклариране и инициализация

```cpp
// Инициализация с агрегатен инициализатор
Point p1 = {10, 20};

// C++11: uniform initialization
Point p2{3, 4};

// Достъп до членове
std::cout << "X: " << p1.x << ", Y: " << p1.y << std::endl;
```

### Разлика между `struct` и `class`

| Характеристика | `struct` | `class` |
|----------------|----------|---------|
| Достъп по подразбиране | `public` | `private` |
| Наследяване по подразбиране | `public` | `private` |
| Конвенционална употреба | Прости данни (POD) | Обекти с поведение |

> 💡 **Важно:** Технически `struct` и `class` в C++ са почти идентични. Разликата е само в подразбиращия се достъп. По конвенция `struct` се използва за прости групи от данни, а `class` — за обекти с капсулация.

### Структури и функции

Структурите могат да се предават на функции по стойност или по референция:

```cpp
void printPoint(const Point& p) {
    std::cout << "(" << p.x << ", " << p.y << ")" << std::endl;
}

Point movePoint(Point p, int dx, int dy) {
    p.x += dx;
    p.y += dy;
    return p;
}
```

### Вложени структури

```cpp
struct Engine {
    int horsepower;
    double displacement;
};

struct Car {
    char brand[50];
    int year;
    Engine engine;  // Вложена структура
};

Car myCar = {"Toyota", 2023, {150, 2.0}};
std::cout << myCar.engine.horsepower;  // 150
```

---

## 2. Изброими типове (`enum`)

### Класически `enum`

Изброимите типове дефинират множество от именувани константи. Те правят кода по-четим:

```cpp
enum Color { RED, GREEN, BLUE };  // RED=0, GREEN=1, BLUE=2

Color c = GREEN;
if (c == GREEN) {
    std::cout << "Зелен цвят" << std::endl;
}
```

**Проблем:** Класическите `enum` стойности „изтичат" в обхващащия scope:

```cpp
enum Color { RED, GREEN, BLUE };
enum TrafficLight { RED, YELLOW, GREEN };  // ❌ Грешка! RED и GREEN вече съществуват
```

### `enum class` (C++11)

`enum class` решава проблема с обхвата и предотвратява имплицитни конверсии:

```cpp
enum class Color { Red, Green, Blue };
enum class TrafficLight { Red, Yellow, Green };  // ✅ Без конфликт

Color c = Color::Green;        // Изисква квалифициране
// int n = c;                  // ❌ Не се конвертира имплицитно
int n = static_cast<int>(c);   // ✅ Явна конверсия
```

### Задаване на конкретни стойности

```cpp
enum class HttpStatus {
    OK = 200,
    NotFound = 404,
    InternalError = 500
};
```

> 🧠 **Дискусия:** Кога бихте използвали `enum` вместо поредица от `const int` константи? Какви предимства дава `enum class` пред обикновен `enum`?

---

## 3. Обединения (`union`)

### Какво е union?

Union е тип, при който всички членове споделят **едно и също място в паметта**. Размерът на union е равен на размера на най-големия му член. В даден момент само един член съдържа валидна стойност.

```cpp
union Data {
    int intVal;
    float floatVal;
    char charVal;
};

Data d;
d.intVal = 42;
std::cout << d.intVal;    // 42
d.floatVal = 3.14f;       // Презаписва intVal!
std::cout << d.intVal;    // ❌ Undefined behavior
```

### Кога се използва union?

- Когато искаме да спестим памет (вариантни типове)
- За представяне на данни, които могат да бъдат от различен тип в различни моменти

### Пример: Вариантен тип

```cpp
enum class ValueType { Int, Float, String };

struct Variant {
    ValueType type;
    union {
        int intVal;
        float floatVal;
        char strVal[32];
    };
};
```

> ⚠️ **Внимание:** В съвременния C++ вместо ръчни `union` конструкции се препоръчва `std::variant` (C++17), който е тип-безопасна алтернатива.

---

## 4. Псевдоними на типове (`typedef` и `using`)

### `typedef` (класически подход)

```cpp
typedef unsigned long ulong;
typedef int (*FuncPtr)(int, int);  // Указател към функция

ulong bigNumber = 4294967295UL;
```

### `using` (модерен подход — C++11)

```cpp
using ulong = unsigned long;
using FuncPtr = int(*)(int, int);
using StringVector = std::vector<std::string>;
```

> 💡 **Препоръка:** Предпочитайте `using` пред `typedef` — по-четим е и поддържа шаблони:

```cpp
template <typename T>
using Vec = std::vector<T>;  // ✅ Работи с using

// typedef std::vector<T> Vec;  // ❌ Не работи с typedef директно
```

---

## 5. Пространства от имена (`namespace`)

### Проблемът

Когато множество библиотеки дефинират функции или класове с еднакви имена, възникват конфликти. `namespace` решава този проблем.

### Дефиниране и използване

```cpp
namespace Geometry {
    struct Point {
        double x, y;
    };
    
    double distance(const Point& a, const Point& b) {
        double dx = a.x - b.x;
        double dy = a.y - b.y;
        return std::sqrt(dx * dx + dy * dy);
    }
}

namespace Physics {
    struct Point {
        double x, y, z;  // Друг Point с различна структура
        double mass;
    };
}

int main() {
    Geometry::Point g = {1.0, 2.0};
    Physics::Point p = {1.0, 2.0, 3.0, 5.0};
    
    // using директива (внимавайте с конфликти!)
    using namespace Geometry;
    Point g2 = {3.0, 4.0};  // Geometry::Point
    
    return 0;
}
```

### Вложени namespace (C++17)

```cpp
namespace Company::Department::Team {
    void doWork() { /* ... */ }
}
// Еквивалентно на:
// namespace Company { namespace Department { namespace Team { ... } } }
```

### Анонимни namespace

```cpp
namespace {
    int helperFunction() { return 42; }  // Видим само в текущия файл
}
```

> 🧠 **Дискусия:** Защо `using namespace std;` се счита за лоша практика в header файлове? Какви проблеми може да причини?

---

## Обобщение

| Конструкция | Предназначение | Ключова идея |
|-------------|---------------|-------------|
| `struct` | Групиране на свързани данни | Потребителски съставен тип |
| `enum` / `enum class` | Именувани константи | Четимост и тип-безопасност |
| `union` | Споделена памет за различни типове | Спестяване на памет |
| `typedef` / `using` | Псевдоними на типове | Четимост на сложни типове |
| `namespace` | Организация на кода | Избягване на конфликти |

Тези конструкции са фундаментът, върху който ще изграждаме класове и обектно-ориентиран дизайн в следващите седмици.
