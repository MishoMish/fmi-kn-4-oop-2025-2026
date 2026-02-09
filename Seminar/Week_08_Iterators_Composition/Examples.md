# Седмица 8 — Примери

## Пример 1: Пълен `IntArray` с итератор

```cpp
#include <iostream>

class IntArray {
    int* data;
    int size;
    
public:
    IntArray(int n) : size(n) { data = new int[n](); }
    
    IntArray(const IntArray& other) : size(other.size) {
        data = new int[size];
        for (int i = 0; i < size; i++) data[i] = other.data[i];
    }
    
    IntArray& operator=(const IntArray& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new int[size];
            for (int i = 0; i < size; i++) data[i] = other.data[i];
        }
        return *this;
    }
    
    ~IntArray() { delete[] data; }
    
    int& operator[](int i) { return data[i]; }
    const int& operator[](int i) const { return data[i]; }
    int getSize() const { return size; }
    
    // --- Итератор ---
    class Iterator {
        int* ptr;
    public:
        Iterator(int* p) : ptr(p) {}
        int& operator*() { return *ptr; }
        Iterator& operator++() { ++ptr; return *this; }
        Iterator operator++(int) { Iterator tmp = *this; ++ptr; return tmp; }
        bool operator!=(const Iterator& o) const { return ptr != o.ptr; }
        bool operator==(const Iterator& o) const { return ptr == o.ptr; }
    };
    
    class ConstIterator {
        const int* ptr;
    public:
        ConstIterator(const int* p) : ptr(p) {}
        const int& operator*() const { return *ptr; }
        ConstIterator& operator++() { ++ptr; return *this; }
        bool operator!=(const ConstIterator& o) const { return ptr != o.ptr; }
    };
    
    Iterator begin() { return Iterator(data); }
    Iterator end() { return Iterator(data + size); }
    ConstIterator begin() const { return ConstIterator(data); }
    ConstIterator end() const { return ConstIterator(data + size); }
};

int main() {
    IntArray arr(5);
    for (int i = 0; i < 5; i++) arr[i] = (i + 1) * 10;
    
    // Range-based for
    std::cout << "Елементи: ";
    for (int val : arr) {
        std::cout << val << " ";
    }
    std::cout << std::endl;  // 10 20 30 40 50
    
    // Модификация чрез итератор
    for (int& val : arr) {
        val *= 2;
    }
    
    std::cout << "Удвоени: ";
    for (int val : arr) {
        std::cout << val << " ";
    }
    std::cout << std::endl;  // 20 40 60 80 100
    
    return 0;
}
```

---

## Пример 2: Композиция — `University` и `Department`

```cpp
#include <iostream>
#include <cstring>

class Professor {
    char name[100];
    char field[50];
public:
    Professor(const char* n = "", const char* f = "") {
        std::strncpy(name, n, 99); name[99] = '\0';
        std::strncpy(field, f, 49); field[49] = '\0';
    }
    const char* getName() const { return name; }
    const char* getField() const { return field; }
};

class Department {
    char name[50];
    Professor professors[20];
    int profCount;
    
public:
    Department(const char* n = "") : profCount(0) {
        std::strncpy(name, n, 49); name[49] = '\0';
    }
    
    void addProfessor(const Professor& p) {
        if (profCount < 20) professors[profCount++] = p;
    }
    
    const char* getName() const { return name; }
    int getProfCount() const { return profCount; }
    
    void print() const {
        std::cout << "  Катедра: " << name << " (" << profCount << " преподаватели)" << std::endl;
        for (int i = 0; i < profCount; i++) {
            std::cout << "    - " << professors[i].getName() 
                      << " (" << professors[i].getField() << ")" << std::endl;
        }
    }
};

class University {
    char name[100];
    Department departments[10];  // University HAS-A Department (композиция)
    int deptCount;
    
public:
    University(const char* n) : deptCount(0) {
        std::strncpy(name, n, 99); name[99] = '\0';
    }
    
    void addDepartment(const Department& d) {
        if (deptCount < 10) departments[deptCount++] = d;
    }
    
    void print() const {
        std::cout << "Университет: " << name << std::endl;
        for (int i = 0; i < deptCount; i++) {
            departments[i].print();
        }
    }
};

int main() {
    University uni("Софийски университет");
    
    Department cs("Компютърни науки");
    cs.addProfessor(Professor("Проф. Иванов", "Алгоритми"));
    cs.addProfessor(Professor("Проф. Петрова", "ИИ"));
    
    Department math("Математика");
    math.addProfessor(Professor("Проф. Георгиев", "Анализ"));
    
    uni.addDepartment(cs);
    uni.addDepartment(math);
    uni.print();
    
    return 0;
}
```

> 💡 `University` → `Department` → `Professor`: тристепенна композиция. Всеки обект управлява своите „деца".

---

## Пример 3: Обратен итератор

```cpp
#include <iostream>

class ReverseRange {
    int start, finish;
public:
    ReverseRange(int from, int to) : start(from), finish(to) {}
    
    class Iterator {
        int current;
    public:
        Iterator(int v) : current(v) {}
        int operator*() const { return current; }
        Iterator& operator++() { --current; return *this; }
        bool operator!=(const Iterator& o) const { return current != o.current; }
    };
    
    Iterator begin() const { return Iterator(finish - 1); }
    Iterator end() const { return Iterator(start - 1); }
};

int main() {
    std::cout << "Обратно броене: ";
    for (int val : ReverseRange(1, 11)) {
        std::cout << val << " ";
    }
    std::cout << std::endl;  // 10 9 8 7 6 5 4 3 2 1
    
    return 0;
}
```
