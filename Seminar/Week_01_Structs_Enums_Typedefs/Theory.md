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

| Характеристика              | `struct`           | `class`            |
| --------------------------- | ------------------ | ------------------ |
| Достъп по подразбиране      | `public`           | `private`          |
| Наследяване по подразбиране | `public`           | `private`          |
| Конвенционална употреба     | Прости данни (POD) | Обекти с поведение |

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

#### Отместване по референция vs. по стойност

**По стойност** — създава копие на структурата:

```cpp
void modifyByValue(Point p) {
    p.x = 100;  // Модифицира само копието
}

Point p = {5, 10};
modifyByValue(p);
std::cout << p.x;  // Все още 5
```

**По референция** — работи директно с оригинала:

```cpp
void modifyByReference(Point& p) {
    p.x = 100;  // Модифицира оригиналния обект
}

Point p = {5, 10};
modifyByReference(p);
std::cout << p.x;  // 100
```

**По const референция** — за четене без копиране:

```cpp
int getDistance(const Point& p) {
    return std::abs(p.x) + std::abs(p.y);  // Не можем да променяме p
}
```

> 💡 **Правило:** Ако функцията трябва да модифицира структурата, използвайте некостантна референция. Ако само чета, използвайте const референция за ефективност.

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

#### По-сложен практичен пример

```cpp
struct Address {
    std::string street;
    std::string city;
    std::string zipCode;
};

struct Person {
    std::string name;
    int age;
    Address address;  // Вложена структура
};

void printPerson(const Person& p) {
    std::cout << "Name: " << p.name << std::endl;
    std::cout << "Age: " << p.age << std::endl;
    std::cout << "City: " << p.address.city << std::endl;
}

Person john = {"John Doe", 30, {"Main St", "New York", "10001"}};
printPerson(john);
```

#### Масиви от структури

```cpp
struct Student {
    std::string name;
    double gpa;
};

std::vector<Student> students = {
    {"Alice", 3.9},
    {"Bob", 3.7},
    {"Carol", 3.8}
};

for (const auto& student : students) {
    std::cout << student.name << ": " << student.gpa << std::endl;
}
```

#### Указатели към структури

```cpp
struct Point {
    int x, y;
};

Point p = {10, 20};
Point* ptr = &p;

// Достъп чрез указател
std::cout << ptr->x << std::endl;      // 10
std::cout << (*ptr).y << std::endl;    // 20 (еквивалентно)
```

#### Размер на структурата - Alignment и padding

```cpp
struct SmallStruct {
    char c;      // 1 байт
    int i;       // 4 байта
};

// Размерът НЕ е 5! Има padding
std::cout << sizeof(SmallStruct) << std::endl;  // Обикновено 8 байта

// Чертеж на паметта:
// [c][padding][padding][padding][i][i][i][i]
```

> ⚠️ **Внимание:** Компилаторът добавя "padding" за оптимизирана достъп до памет. Редът на членовете е важен за размера на структурата!

```cpp
struct OptimizedStruct {
    int i;       // 4 байта
    char c;      // 1 байт
    // 3 байта padding
};

struct WasteStruct {
    char c;      // 1 байт
    int i;       // 3 байта padding + 4 байта = 7 байта
};

// OptimizedStruct е по-малка!
std::cout << sizeof(OptimizedStruct) << std::endl;  // 8 байта
std::cout << sizeof(WasteStruct) << std::endl;      // 12 байта
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

<details>
<summary>💡 Отговор</summary>

**Защо `enum` е по-добро от константи:**

1. **Тип-безопасност** - не можете случайно да подадете произволно число
2. **Групиране** - свързани стойности са заедно
3. **Интелисенс** - IDE показва възможните стойности
4. **Документация** - по-ясно показва какви са валидните опции

**Предимства на `enum class` пред обикновен `enum`:**

1. **Няма name pollution** - стойностите не изтичат в обхващащия scope
2. **Не се конвертира имплицитно** - по-безопасно
3. **По-добра организация** - изисква квалифициране с името на enum
4. **Избягва конфликти** - може да има enum с еднакви имена на стойности

```cpp
// ❌ С enum - име污ение
enum Color { Red, Green };
enum Status { Red };  // Грешка!

// ✅ С enum class - без конфликт
enum class Color { Red, Green };
enum class Status { Red };  // OK!
```

</details>

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

<details>
<summary>💡 Отговор</summary>

**Проблемът с `using namespace std;` в headers:**

1. **Принудителна зависимост** - всеки, който включи вашия header, автоматично получава `using namespace std;`
2. **Name collisions** - може да се сблъскате с имена от std библиотеката
3. **Неочаквано поведение** - функции могат да бъдат разрешени към грешна версия
4. **Трудна диагностика** - грешките могат да се появят в други файлове

**Пример за проблем:**

```cpp
// myheader.h
#include <algorithm>
using namespace std;  // ❌ Лошо!

void process(int x);

// someotherfile.cpp
#include "myheader.h"

int count = 10;  // ❌ Конфликт с std::count!
vector<int> v;   // ✅ Работи, но случайно
```

**Правилен подход:**

```cpp
// myheader.h
#include <algorithm>

// Квалифицирайте имената
void process(std::vector<int>& data);

// Или using declaration за конкретни имена
using std::vector;
using std::string;
```

</details>

---

## Обобщение

| Конструкция           | Предназначение                     | Ключова идея               |
| --------------------- | ---------------------------------- | -------------------------- |
| `struct`              | Групиране на свързани данни        | Потребителски съставен тип |
| `enum` / `enum class` | Именувани константи                | Четимост и тип-безопасност |
| `union`               | Споделена памет за различни типове | Спестяване на памет        |
| `typedef` / `using`   | Псевдоними на типове               | Четимост на сложни типове  |
| `namespace`           | Организация на кода                | Избягване на конфликти     |

Тези конструкции са фундаментът, върху който ще изграждаме класове и обектно-ориентиран дизайн в следващите седмици.

---

## 8. Дълбоко в паметта - Практични примери

### Визуализиране на памет при структури

Когато имаме различни типове членове, важно е да разберем как те се подреждат в паметта:

```cpp
struct BadMemoryLayout {
    char a;         // 1 байт    Адрес: 0x1000
    // 3 байта padding
    int b;          // 4 байта   Адрес: 0x1004
    char c;         // 1 байт    Адрес: 0x1008
    // 7 байта padding
    double d;       // 8 байта   Адрес: 0x1010
};

// Размер: 24 байта (1 + 3 + 4 + 1 + 7 + 8)

struct GoodMemoryLayout {
    double d;       // 8 байта   Адрес: 0x1000
    int b;          // 4 байта   Адрес: 0x1008
    char a;         // 1 байт    Адрес: 0x100C
    char c;         // 1 байт    Адрес: 0x100D
    // 2 байта padding
};

// Размер: 16 байта (8 + 4 + 1 + 1 + 2)
// Спестихме 8 байта!
```

### Упражнение: Оптимизирайте структурата

```cpp
// ❌ Неоптимално
struct DatabaseRecord {
    char status;        // 1
    int userID;         // 4
    char name[100];     // 100
    double salary;      // 8
    bool isActive;      // 1
    short departmentID; // 2
};

// ✅ Оптимално
struct DatabaseRecord {
    double salary;      // 8
    char name[100];     // 100
    int userID;         // 4
    short departmentID; // 2
    char status;        // 1
    bool isActive;      // 1
};
```

---

## 9. Enum - Разширени техники

### Enum с битови операции

```cpp
enum class FilePermissions : unsigned char {
    Owner_Read = 0x1,      // 001
    Owner_Write = 0x2,     // 010
    Owner_Execute = 0x4,   // 100
    Group_Read = 0x8,      // 001000
    Group_Write = 0x10,    // 010000
    Group_Execute = 0x20   // 100000
};

// Комбиниране на права
unsigned char userPerms = FilePermissions::Owner_Read |
                          FilePermissions::Owner_Write |
                          FilePermissions::Owner_Execute;  // 111 = 7

// Проверка
if ((userPerms & static_cast<unsigned char>(FilePermissions::Owner_Write)) != 0) {
    std::cout << "Потребителят може да пише" << std::endl;
}
```

### Enum за държавни машини

```cpp
enum class ConnectionState {
    Disconnected,
    Connecting,
    Connected,
    Reconnecting,
    Failed,
    Closed
};

class NetworkConnection {
private:
    ConnectionState state;

public:
    void connect() {
        if (state == ConnectionState::Disconnected) {
            state = ConnectionState::Connecting;
            // Инициирай свързване
        }
    }

    void onConnected() {
        if (state == ConnectionState::Connecting) {
            state = ConnectionState::Connected;
        }
    }

    void disconnect() {
        state = ConnectionState::Disconnected;
    }
};
```

---

## 10. Практични примери за реални приложения

### Пример 1: Игра с персонажи

```cpp
namespace Game {
    enum class CharacterClass { Warrior, Mage, Archer, Rogue };
    enum class Faction { Alliance, Horde, Neutral };

    struct Stats {
        int health;
        int mana;
        int attack;
        int defense;
    };

    struct Character {
        std::string name;
        CharacterClass charClass;
        Faction faction;
        Stats stats;
        int level;
        int experience;

        void takeDamage(int damage) {
            int actualDamage = damage - (stats.defense / 10);
            stats.health -= actualDamage;
            if (stats.health < 0) stats.health = 0;
        }

        void levelUp() {
            level++;
            stats.health += 10;
            stats.attack += 2;
            stats.defense += 1;
        }

        bool isAlive() const {
            return stats.health > 0;
        }
    };

    struct BattleResult {
        bool playerWon;
        int damageDealt;
        int damageTaken;
        int experienceGained;
    };
}

// Използване
Game::Character player;
player.name = "Arthas";
player.charClass = Game::CharacterClass::Warrior;
player.faction = Game::Faction::Alliance;
player.stats = {100, 0, 20, 5};
player.level = 1;
```

### Пример 2: Online магазин

```cpp
namespace ECommerce {
    enum class OrderStatus { Pending, Processing, Shipped, Delivered, Cancelled };
    enum class PaymentMethod { CreditCard, PayPal, BankTransfer };

    struct Product {
        std::string id;
        std::string name;
        double price;
        int stock;
    };

    struct Customer {
        std::string id;
        std::string email;
        std::string address;
    };

    struct OrderItem {
        Product product;
        int quantity;
        double unitPrice;

        double getTotal() const {
            return quantity * unitPrice;
        }
    };

    struct Order {
        std::string id;
        Customer customer;
        std::vector<OrderItem> items;
        OrderStatus status;
        PaymentMethod paymentMethod;
        std::string createdDate;
        std::string shippedDate;

        double getTotalAmount() const {
            double total = 0;
            for (const auto& item : items) {
                total += item.getTotal();
            }
            return total;
        }

        void applyDiscount(double percentage) {
            // Приложи отстъпка
        }
    };
}
```

### Пример 3: GPS система

```cpp
namespace Navigation {
    struct Coordinates {
        double latitude;
        double longitude;
        double altitude;

        double distanceTo(const Coordinates& other) const {
            // Хаверсинова формула за разстояние на земята
            double dlat = (other.latitude - latitude) * 0.0174533;  // Convert to radians
            double dlon = (other.longitude - longitude) * 0.0174533;
            double a = sin(dlat/2) * sin(dlat/2) +
                       cos(latitude * 0.0174533) * cos(other.latitude * 0.0174533) *
                       sin(dlon/2) * sin(dlon/2);
            double c = 2 * atan2(sqrt(a), sqrt(1-a));
            return 6371 * c;  // Радиус на Земята в км
        }
    };

    enum class TransportMode { Walking, Driving, Transit, Cycling };

    struct Route {
        Coordinates start;
        Coordinates end;
        TransportMode mode;
        std::vector<Coordinates> waypoints;
        double estimatedTime;  // в минути
        double distance;       // в км

        double getAverageSpeed() const {
            if (estimatedTime == 0) return 0;
            return (distance * 60) / estimatedTime;  // км/ч
        }
    };

    struct TrackingPoint {
        Coordinates position;
        std::string timestamp;
        int speed;  // км/ч
        int bearing;  // градуси
    };
}
```

---

## 11. Частни случаи и специални техники

### Union за оптимизиране на памета в вложени структури

```cpp
enum class ValueKind { Integer, Floating, Boolean, Null };

union Value {
    int64_t intValue;
    double doubleValue;
    bool boolValue;
};

struct TypedValue {
    ValueKind kind;
    Value data;

    std::string toString() const {
        switch (kind) {
            case ValueKind::Integer:
                return std::to_string(data.intValue);
            case ValueKind::Floating:
                return std::to_string(data.doubleValue);
            case ValueKind::Boolean:
                return data.boolValue ? "true" : "false";
            case ValueKind::Null:
                return "null";
            default:
                return "unknown";
        }
    }
};
```

### Вложени typedef за сложни типове

```cpp
// Типични сложни типове
using Matrix2D = std::vector<std::vector<double>>;
using AdjacencyList = std::map<int, std::vector<int>>;
using EventCallback = std::function<void(const Event&)>;
using PropertyMap = std::map<std::string, std::string>;

// Контейнер на контейнери
using DataTable = std::vector<std::vector<std::string>>;

// Функции с намеса
using Comparator = bool (*)(int, int);
using Transformer = int (*)(int);
```

---

## 12. Дебъгване и тестване

### Проверка на типове и размери

```cpp
#include <typeinfo>
#include <iostream>

int main() {
    // Проверка на типове
    std::cout << "Type of Point: " << typeid(Point).name() << std::endl;

    // Проверка на размери
    std::cout << "sizeof(char) = " << sizeof(char) << std::endl;
    std::cout << "sizeof(int) = " << sizeof(int) << std::endl;
    std::cout << "sizeof(double) = " << sizeof(double) << std::endl;
    std::cout << "sizeof(Point) = " << sizeof(Point) << std::endl;

    // Проверка на alignment
    std::cout << "alignof(Point) = " << alignof(Point) << std::endl;

    return 0;
}
```

### Assert за валидиране

```cpp
struct Person {
    std::string name;
    int age;

    bool isValid() const {
        return !name.empty() && age > 0 && age < 150;
    }
};

void printPerson(const Person& p) {
    assert(p.isValid() && "Person must have valid data");
    std::cout << p.name << " (" << p.age << " years)" << std::endl;
}
```

---

## 13. Интерактивни упражнения за обсъждане в клас

### Упражнение 1: Анализ на памет

Дайте предположение за размера на следните структури. Защо имат точно този размер?

```cpp
struct A { char a; char b; };                    // ?
struct B { char a; short b; };                   // ?
struct C { short a; char b; };                   // ?
struct D { char a; int b; char c; };            // ?
struct E { int a; char b; short c; double d; }; // ?
```

**Отговор и обяснение:**

- `A`: 2 байта (char = 1, char = 1, no padding)
- `B`: 4 байта (char = 1, padding = 1, short = 2)
- `C`: 4 байта (short = 2, char = 1, padding = 1)
- `D`: 12 байта (char = 1, padding = 3, int = 4, char = 1, padding = 3)
- `E`: 24 байта (int = 4, padding = 4, char = 1, padding = 7, double = 8)

### Упражнение 2: Enum vs Constants

Защо `enum class` е по-добро от просто константи?

```cpp
// Вариант 1: Constants
const int STATUS_PENDING = 0;
const int STATUS_ACTIVE = 1;
const int STATUS_DONE = 2;

// Вариант 2: enum
enum class Status { Pending, Active, Done };

// Какво ще очакват функциите?
void processStatus1(int status);     // Може да приема произволно число!
void processStatus2(Status status);  // Само валидни статуси!
```

### Упражнение 3: Дизайн на структура

Проектирайте структура за **книга във библиотека**, която трябва да съхранява:

- Заглавие
- Автор/и
- ISBN
- Година на издаване
- Жанр (enum)
- Наличност
- Позиция в библиотеката

```cpp
// Ваше решение тук...
```

### Упражнение 4: Namespace конфликти

Решете конфликта:

```cpp
namespace Library {
    struct Book { std::string title; };
    void printBook(const Book& b) { /* ... */ }
}

namespace Game {
    struct Book { std::string title; int pages; };
    void printBook(const Book& b) { /* ... */ }
}

int main() {
    Library::Book b1{"1984"};
    Game::Book b2{"Manual", 500};

    // Как да използвам правилния printBook?
    printBook(b1);  // ❌ Какъв е? Кой namespace?
}
```

---

## 14. Сравнение с други езици

### C++ vs Java

```cpp
// C++
struct Point {
    int x, y;
};

// Java
class Point {
    public int x, y;
}
```

В Java всичко е класс, но в C++ можем да избираме между `struct` и `class` според нуждите.

### C++ vs Python

```cpp
// C++
enum class Season { Spring, Summer, Autumn, Winter };

// Python
class Season(Enum):
    SPRING = 1
    SUMMER = 2
    AUTUMN = 3
    WINTER = 4
```

C++ enum е компилирано време, Python enum е runtime.

---

## 15. Проверка на знанията

### Въпроси със многовариантни отговори

**Въпрос 1:** Какъв е размерът на `struct { char a; int b; }`?

- [ ] 5 байта
- [ ] 8 байта
- [ ] 4 байта
- [ ] Зависи от компилатора

<details>
<summary>📖 Обяснение</summary>

**Отговор: 8 байта**

Не е 5 байта (1 + 4), защото компилаторът добавя **padding** за alignment:

```
Памет: [char a][pad][pad][pad][int b (4 bytes)]
Байтове:  0      1    2    3     4    5   6   7
```

- `char a` заема 1 байт (байт 0)
- 3 байта padding за alignment на `int`
- `int b` заема 4 байта (байтове 4-7)
- **Общо: 8 байта**

Alignment е необходим, защото процесорът работи по-ефективно, когато типовете са подравнени на техния естествен размер.

</details>

**Въпрос 2:** Каква е разликата между `enum` и `enum class`?

- [ ] Няма разлика
- [ ] `enum class` е по-новото, по-безопасното
- [ ] `enum` е по-бързо
- [ ] `enum class` работи само в C++17

<details>
<summary>📖 Обяснение</summary>

**Отговор: `enum class` е по-новото, по-безопасното**

**Основни разлики:**

| Характеристика       | `enum`                | `enum class`                |
| -------------------- | --------------------- | --------------------------- |
| Scope                | Стойностите "изтичат" | Стойностите са в enum scope |
| Имплицитна конверсия | Да (към int)          | Не                          |
| Конфликти на имена   | Възможни              | Невъзможни                  |
| C++ стандарт         | C++98                 | C++11                       |

```cpp
// enum - проблеми
enum Color { Red, Green };
int x = Red;  // ✅ Работи (имплицитна конверсия)

// enum class - безопасност
enum class Color { Red, Green };
int x = Color::Red;  // ❌ Грешка!
int y = static_cast<int>(Color::Red);  // ✅ Явна конверсия
```

</details>

**Въпрос 3:** Какъв е истинският размер на `union { int a; double b; }`?

- [ ] 4 байта
- [ ] 8 байта
- [ ] 12 байта
- [ ] 16 байта

<details>
<summary>📖 Обяснение</summary>

**Отговор: 8 байта**

При union всички членове **споделят едно и също място в паметта**. Размерът на union е равен на размера на **най-големия член**.

```cpp
union Data {
    int a;     // 4 байта
    double b;  // 8 байта  ← Най-големият
};

sizeof(Data) = 8 байта
```

**Визуализация:**

```
Памет: [----------------------8 байта----------------------]
       [int a (4 bytes)][unused]
       [double b (8 bytes)                                  ]
```

И двата члена използват **същите** 8 байта! В даден момент само един от тях съдържа валидна стойност.

</details>

**Въпрос 4:** Защо е лошо да се използва `using namespace std;` в header файлове?

- [ ] Причинява compiler error
- [ ] Намалява производителност
- [ ] Може да причини конфликти с други библиотеки
- [ ] Няма причина, напълно е безопасно

<details>
<summary>📖 Обяснение</summary>

**Отговор: Може да причини конфликти с други библиотеки**

Когато слагате `using namespace std;` в header файл:

1. **Всеки, който го include-ва, получава автоматично using**
2. **Потребителят няма контрол** - не може да избере дали иска това
3. **Конфликти** - функции с еднакви имена могат да се сблъскат

**Пример за проблем:**

```cpp
// bad_header.h
#include <algorithm>
using namespace std;  // ❌ Засяга всички, които го include-ват!

// user_code.cpp
#include "bad_header.h"

namespace MyLib {
    void swap(int& a, int& b);  // ❌ Конфликт с std::swap!
}

void test() {
    int x = 1, y = 2;
    swap(x, y);  // Кой swap? std::swap или MyLib::swap?
}
```

**Правилен подход:**

- Използвайте `std::` квалификация
- Или `using std::vector;` за конкретни типове в .cpp файлове

</details>

---

## 16. Общи грешки и как да ги избегнем

### Грешка 1: Забравяне на размера

```cpp
// ❌ Лошо: Предполага се, че структурата е малка
char buffer[sizeof(Point)];  // Может да е по-голямо поради padding!

// ✅ Добро: Явно проверяваме размера
char buffer[sizeof(Point)];
std::cout << "Point size: " << sizeof(Point) << std::endl;
```

### Грешка 2: Неправилно копиране

```cpp
// ❌ Лошо: Копира се цялата структура (може да е бавно)
void process(Point p) { /* ... */ }

// ✅ Добро: Преминава се по референция
void process(const Point& p) { /* ... */ }
```

### Грешка 3: Конфликти на имена

```cpp
// ❌ Лошо: Конфликт
enum Color { Red, Green, Blue };
enum Status { Red, Yellow };  // ❌ Red вече съществува!

// ✅ Добро: Използвайте enum class
enum class Color { Red, Green, Blue };
enum class Status { Red, Yellow };  // ✅ Без конфликт
```

### Грешка 4: Забравяне на const

```cpp
// ❌ Лошо: Не е ясно, че функцията не модифицира структурата
void print(Point& p) { std::cout << p.x << std::endl; }

// ✅ Добро: const показва, че данните не се променят
void print(const Point& p) { std::cout << p.x << std::endl; }
```

---

## 17. Съвети за оптимизация

### Намаляване на памет със правилен ред

```cpp
// ❌ Неоптимално: 32 байта
struct Inefficient {
    bool flag1;         // 1
    int id;             // 4 + 3 padding
    bool flag2;         // 1
    double value;       // 8 + 7 padding
    char code;          // 1 + 7 padding
};

// ✅ Оптимално: 16 байта
struct Efficient {
    double value;       // 8
    int id;             // 4
    char code;          // 1
    bool flag1;         // 1
    bool flag2;         // 1
    // 1 байт padding
};

// Намаляме паметта 50%!
```

### Използване на sizeof за дебъгване

```cpp
template<typename T>
void checkSize() {
    std::cout << "Size of " << typeid(T).name() << ": "
              << sizeof(T) << " bytes" << std::endl;
}

checkSize<Point>();
checkSize<Person>();
```

---

## Финални съвети за разработчици

### 1. Структури за данни

Всякъд път, когато имате група от свързани данни, използвайте структура вместо да ги предавате поотделно.

### 2. Enum за състояния

За всяко представяне на състояние (status, mode, state), използвайте `enum class`.

### 3. Namespace за организация

Организирайте вашия код с namespaces - това значително подобрява четимостта.

### 4. const референции

Винаги передавайте големи структури по const референция, за да избегнете ненужни копия.

### 5. Памет и производителност

Разберете как вашите структури се подреждат в паметта - това е ключово за оптимизация.
