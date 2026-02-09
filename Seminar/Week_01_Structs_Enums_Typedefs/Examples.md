# Седмица 1 — Примери

## Пример 1: Моделиране на точка в равнината

```cpp
#include <iostream>
#include <cmath>

struct Point {
    double x;
    double y;
};

// Функция, приемаща структура по const референция
double distanceToOrigin(const Point& p) {
    return std::sqrt(p.x * p.x + p.y * p.y);
}

// Функция, връщаща нова структура
Point midpoint(const Point& a, const Point& b) {
    return {(a.x + b.x) / 2.0, (a.y + b.y) / 2.0};
}

int main() {
    Point a = {3.0, 4.0};
    Point b = {7.0, 1.0};
    
    std::cout << "Разстояние от A до началото: " << distanceToOrigin(a) << std::endl;  // 5.0
    
    Point m = midpoint(a, b);
    std::cout << "Среда: (" << m.x << ", " << m.y << ")" << std::endl;  // (5.0, 2.5)
    
    return 0;
}
```

> 💡 **Забележете:** Предаваме `Point` по `const&`, за да избегнем излишно копиране. Връщаме по стойност — компилаторът оптимизира чрез RVO.

---

## Пример 2: Цветове с `enum class`

```cpp
#include <iostream>

enum class Color { Red, Green, Blue, Yellow, White, Black };

// Функция, работеща с enum class
const char* colorToString(Color c) {
    switch (c) {
        case Color::Red:    return "Червен";
        case Color::Green:  return "Зелен";
        case Color::Blue:   return "Син";
        case Color::Yellow: return "Жълт";
        case Color::White:  return "Бял";
        case Color::Black:  return "Черен";
        default:            return "Неизвестен";
    }
}

int main() {
    Color favorite = Color::Blue;
    std::cout << "Любим цвят: " << colorToString(favorite) << std::endl;
    
    // Enum class предотвратява случайни сравнения:
    // if (favorite == 2) { ... }  // ❌ Компилаторна грешка!
    
    return 0;
}
```

> 💡 `enum class` не позволява имплицитна конверсия до `int`. Това предотвратява цял клас от бъгове.

---

## Пример 3: Вариантен тип с `union`

```cpp
#include <iostream>
#include <cstring>

enum class ValueType { Int, Float, Text };

struct ConfigValue {
    char name[64];
    ValueType type;
    union {
        int intVal;
        float floatVal;
        char textVal[128];
    };
};

void printConfig(const ConfigValue& cfg) {
    std::cout << cfg.name << " = ";
    switch (cfg.type) {
        case ValueType::Int:   std::cout << cfg.intVal; break;
        case ValueType::Float: std::cout << cfg.floatVal; break;
        case ValueType::Text:  std::cout << cfg.textVal; break;
    }
    std::cout << std::endl;
}

int main() {
    ConfigValue maxRetries;
    std::strcpy(maxRetries.name, "max_retries");
    maxRetries.type = ValueType::Int;
    maxRetries.intVal = 5;
    
    ConfigValue threshold;
    std::strcpy(threshold.name, "threshold");
    threshold.type = ValueType::Float;
    threshold.floatVal = 0.75f;
    
    printConfig(maxRetries);  // max_retries = 5
    printConfig(threshold);   // threshold = 0.75
    
    return 0;
}
```

> ⚠️ **Обърнете внимание** на tagged union шаблона: enum + union. Полето `type` ни казва кой член на union-а е валиден.

---

## Пример 4: Организация с `namespace`

```cpp
#include <iostream>
#include <cmath>

namespace Math {
    const double PI = 3.14159265358979;
    
    double degreesToRadians(double degrees) {
        return degrees * PI / 180.0;
    }
    
    namespace Trigonometry {
        double sinDeg(double degrees) {
            return std::sin(degreesToRadians(degrees));
        }
        double cosDeg(double degrees) {
            return std::cos(degreesToRadians(degrees));
        }
    }
}

namespace Units {
    using Meters = double;
    using Kilograms = double;
    using Seconds = double;
    
    Meters toMeters(double feet) {
        return feet * 0.3048;
    }
}

int main() {
    std::cout << "sin(30°) = " << Math::Trigonometry::sinDeg(30) << std::endl;  // ~0.5
    
    Units::Meters height = Units::toMeters(6.0);  // 6 фута в метри
    std::cout << "Височина: " << height << " m" << std::endl;
    
    return 0;
}
```

> 💡 **Забележете** как `using` създава смислени псевдоними (`Meters`, `Kilograms`), които подобряват четимостта без да добавят runtime overhead.
