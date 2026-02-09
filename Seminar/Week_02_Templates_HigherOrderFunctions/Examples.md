# Седмица 2 — Примери

## Пример 1: Шаблонна функция `maxValue`

```cpp
#include <iostream>
#include <cstring>

template <typename T>
T maxValue(T a, T b) {
    return (a > b) ? a : b;
}

// Специализация за C-низове (const char*)
template <>
const char* maxValue<const char*>(const char* a, const char* b) {
    return (std::strcmp(a, b) > 0) ? a : b;
}

int main() {
    std::cout << maxValue(10, 20) << std::endl;          // 20 (int)
    std::cout << maxValue(3.14, 2.71) << std::endl;      // 3.14 (double)
    std::cout << maxValue('A', 'Z') << std::endl;        // Z (char)
    std::cout << maxValue("apple", "banana") << std::endl; // banana (специализация)
    
    return 0;
}
```

> 💡 **Забележете:** За `const char*` операторът `>` сравнява *адреси*, не съдържание. Затова е нужна специализация с `strcmp`.

---

## Пример 2: `filter` с указател към функция

```cpp
#include <iostream>

bool isEven(int x) { return x % 2 == 0; }
bool isPositive(int x) { return x > 0; }

int filter(const int arr[], int size, int result[], bool (*predicate)(int)) {
    int count = 0;
    for (int i = 0; i < size; i++) {
        if (predicate(arr[i])) {
            result[count++] = arr[i];
        }
    }
    return count;
}

void printArray(const int arr[], int size) {
    for (int i = 0; i < size; i++) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;
}

int main() {
    int data[] = {-3, 1, 4, -1, 5, -9, 2, 6};
    int result[8];
    
    int count = filter(data, 8, result, isEven);
    std::cout << "Четни: ";
    printArray(result, count);  // 4 2 6
    
    count = filter(data, 8, result, isPositive);
    std::cout << "Положителни: ";
    printArray(result, count);  // 1 4 5 2 6
    
    return 0;
}
```

> 💡 Една и съща функция `filter` с различни предикати дава различен резултат. Поведението се инжектира отвън.

---

## Пример 3: `transform` с `std::function`

```cpp
#include <iostream>
#include <functional>

template <typename T>
void transform(T arr[], int size, std::function<T(const T&)> func) {
    for (int i = 0; i < size; i++) {
        arr[i] = func(arr[i]);
    }
}

int square(int x) { return x * x; }

int main() {
    int numbers[] = {1, 2, 3, 4, 5};
    
    // С обикновена функция
    transform<int>(numbers, 5, square);
    // numbers = {1, 4, 9, 16, 25}
    
    // С ламбда (без capture)
    transform<int>(numbers, 5, [](const int& x) { return x + 1; });
    // numbers = {2, 5, 10, 17, 26}
    
    // С ламбда (с capture)
    int offset = 100;
    transform<int>(numbers, 5, [offset](const int& x) { return x + offset; });
    // numbers = {102, 105, 110, 117, 126}
    
    for (int i = 0; i < 5; i++) {
        std::cout << numbers[i] << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

> 💡 `std::function` приема обикновени функции, ламбди и функтори — универсален интерфейс за извикваеми обекти.
