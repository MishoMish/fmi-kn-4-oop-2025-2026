# Седмица 5 — Примери

## Пример 1: Клас `String` с пълна Голяма четворка

```cpp
#include <iostream>
#include <cstring>

class String {
    char* data;
    int length;
    
    void copyFrom(const String& other) {
        length = other.length;
        data = new char[length + 1];
        std::strcpy(data, other.data);
    }
    
    void free() {
        delete[] data;
        data = nullptr;
        length = 0;
    }
    
public:
    // Конструктор
    String(const char* str = "") {
        length = std::strlen(str);
        data = new char[length + 1];
        std::strcpy(data, str);
        std::cout << "  [Конструктор] \"" << data << "\"" << std::endl;
    }
    
    // Копиращ конструктор
    String(const String& other) {
        copyFrom(other);
        std::cout << "  [Copy ctor] \"" << data << "\"" << std::endl;
    }
    
    // operator=
    String& operator=(const String& other) {
        if (this != &other) {
            free();
            copyFrom(other);
        }
        std::cout << "  [operator=] \"" << data << "\"" << std::endl;
        return *this;
    }
    
    // Деструктор
    ~String() {
        std::cout << "  [Деструктор] \"" << (data ? data : "null") << "\"" << std::endl;
        free();
    }
    
    const char* c_str() const { return data; }
    int getLength() const { return length; }
};

int main() {
    std::cout << "--- Създаване ---" << std::endl;
    String a("Hello");         // Конструктор
    
    std::cout << "--- Копиране ---" << std::endl;
    String b = a;              // Copy constructor
    
    std::cout << "--- Присвояване ---" << std::endl;
    String c("World");         // Конструктор
    c = a;                     // operator=
    
    std::cout << "--- Край на main ---" << std::endl;
    return 0;
    // Деструктори се извикват в обратен ред: c, b, a
}
```

> 💡 Този пример трасира всяко извикване, за да видите точно кога какво се случва.

---

## Пример 2: Проблемът с shallow copy

```cpp
#include <iostream>
#include <cstring>

class BrokenString {
    char* data;
public:
    BrokenString(const char* str) {
        data = new char[std::strlen(str) + 1];
        std::strcpy(data, str);
    }
    
    // НЯМА копиращ конструктор и operator= → shallow copy!
    
    ~BrokenString() { delete[] data; }
    
    void print() const { std::cout << data << std::endl; }
    
    void modify() { data[0] = 'X'; }  // Модифицира първия символ
};

int main() {
    BrokenString a("Hello");
    BrokenString b = a;  // Shallow copy! b.data == a.data
    
    b.modify();          // Променя b, но...
    a.print();           // "Xello" — a също е променено! 😱
    
    // При изход: double delete на data → CRASH 💥
    return 0;
}
```

> ⚠️ **Това е класически бъг.** Без deep copy, два обекта споделят една памет. Промяна на единия засяга другия. Унищожаването причинява double delete.

---

## Пример 3: `DynamicArray` с push/pop

```cpp
#include <iostream>

class DynamicArray {
    int* data;
    int size;
    int capacity;
    
    void resize() {
        capacity *= 2;
        int* newData = new int[capacity];
        for (int i = 0; i < size; i++) {
            newData[i] = data[i];
        }
        delete[] data;
        data = newData;
    }
    
    void copyFrom(const DynamicArray& other) {
        size = other.size;
        capacity = other.capacity;
        data = new int[capacity];
        for (int i = 0; i < size; i++) {
            data[i] = other.data[i];
        }
    }
    
    void free() {
        delete[] data;
    }
    
public:
    DynamicArray() : data(new int[4]), size(0), capacity(4) {}
    
    DynamicArray(const DynamicArray& other) { copyFrom(other); }
    
    DynamicArray& operator=(const DynamicArray& other) {
        if (this != &other) {
            free();
            copyFrom(other);
        }
        return *this;
    }
    
    ~DynamicArray() { free(); }
    
    void push(int val) {
        if (size >= capacity) resize();
        data[size++] = val;
    }
    
    int pop() {
        if (size == 0) return -1;
        return data[--size];
    }
    
    void print() const {
        std::cout << "[";
        for (int i = 0; i < size; i++) {
            if (i > 0) std::cout << ", ";
            std::cout << data[i];
        }
        std::cout << "] (size=" << size << ", cap=" << capacity << ")" << std::endl;
    }
};

int main() {
    DynamicArray arr;
    for (int i = 1; i <= 10; i++) {
        arr.push(i * 10);
    }
    arr.print();  // [10, 20, 30, 40, 50, 60, 70, 80, 90, 100] (size=10, cap=16)
    
    DynamicArray copy = arr;   // Deep copy
    copy.push(110);
    copy.print();              // [10, ..., 100, 110] (size=11, cap=16)
    arr.print();               // [10, ..., 100] (size=10) — непроменен!
    
    return 0;
}
```

> 💡 `resize()` удвоява капацитета — стратегия за амортизирана O(1) сложност при `push`.
