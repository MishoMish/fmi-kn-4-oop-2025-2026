# Седмица 3 — Примери

## Пример 1: Ламбда за сортиране по различни критерии

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>

struct Student {
    char name[50];
    double grade;
    int age;
};

int main() {
    Student students[] = {
        {"Иван", 5.50, 21},
        {"Мария", 6.00, 20},
        {"Петър", 4.75, 22},
        {"Анна", 5.25, 19}
    };
    int n = 4;
    
    // Сортиране по оценка (низходящо)
    std::sort(students, students + n, [](const Student& a, const Student& b) {
        return a.grade > b.grade;
    });
    
    std::cout << "По оценка (низходящо):" << std::endl;
    for (int i = 0; i < n; i++) {
        std::cout << "  " << students[i].name << ": " << students[i].grade << std::endl;
    }
    
    // Сортиране по име (лексикографски)
    std::sort(students, students + n, [](const Student& a, const Student& b) {
        return std::strcmp(a.name, b.name) < 0;
    });
    
    std::cout << "\nПо име:" << std::endl;
    for (int i = 0; i < n; i++) {
        std::cout << "  " << students[i].name << std::endl;
    }
    
    return 0;
}
```

> 💡 Една и съща `std::sort` функция, различен критерий — благодарение на ламбдите.

---

## Пример 2: Capture по стойност vs. по референция

```cpp
#include <iostream>

int main() {
    int total = 0;
    int callCount = 0;
    
    // Capture total по референция (искаме да акумулираме)
    // Capture callCount по референция (искаме да броим)
    auto accumulate = [&total, &callCount](int value) {
        total += value;
        callCount++;
        std::cout << "Извикване #" << callCount 
                  << ": добавяме " << value 
                  << ", total = " << total << std::endl;
    };
    
    accumulate(10);  // #1: добавяме 10, total = 10
    accumulate(20);  // #2: добавяме 20, total = 30
    accumulate(5);   // #3: добавяме 5, total = 35
    
    std::cout << "Краен резултат: " << total << std::endl;        // 35
    std::cout << "Брой извиквания: " << callCount << std::endl;   // 3
    
    // Пример с capture по стойност
    int threshold = 15;
    auto isAboveThreshold = [threshold](int x) {
        return x > threshold;
    };
    
    threshold = 100;  // Промяната НЕ влияе на ламбдата
    
    std::cout << std::boolalpha;
    std::cout << "20 > threshold? " << isAboveThreshold(20) << std::endl;  // true (сравнява с 15!)
    
    return 0;
}
```

> ⚠️ Capture по стойност „замразява" стойността в момента на създаване на ламбдата.

---

## Пример 3: Ламбда фабрика — функция, връщаща ламбда

```cpp
#include <iostream>
#include <functional>

// Фабрика за multiplier ламбди
std::function<int(int)> makeMultiplier(int factor) {
    return [factor](int x) { return x * factor; };
}

// Фабрика за adder ламбди
std::function<int(int)> makeAdder(int offset) {
    return [offset](int x) { return x + offset; };
}

int main() {
    auto double_it = makeMultiplier(2);
    auto triple_it = makeMultiplier(3);
    auto add_ten = makeAdder(10);
    
    std::cout << double_it(5) << std::endl;   // 10
    std::cout << triple_it(5) << std::endl;   // 15
    std::cout << add_ten(5) << std::endl;     // 15
    
    // Композиция: удвой, после добави 10
    auto combined = [&](int x) { return add_ten(double_it(x)); };
    std::cout << combined(5) << std::endl;    // 20
    
    return 0;
}
```

> 💡 **Фабричната функция** връща ламбда с различен capture. Всяка ламбда „помни" своя `factor` или `offset`. Това е *closure* — функция с обвързана среда.
