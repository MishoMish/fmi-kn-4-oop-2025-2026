# Седмица 14 — Примери

## Пример 1: String с пълно Rule of Five

```cpp
#include <iostream>
#include <cstring>
#include <utility>

class String {
    char* data;
    int length;
    
public:
    // Конструктор
    String(const char* str = "") {
        length = std::strlen(str);
        data = new char[length + 1];
        std::strcpy(data, str);
        std::cout << "  [Ctor] \"" << data << "\"" << std::endl;
    }
    
    // 1. Деструктор
    ~String() {
        std::cout << "  [Dtor] \"" << (data ? data : "null") << "\"" << std::endl;
        delete[] data;
    }
    
    // 2. Copy конструктор
    String(const String& other) : length(other.length) {
        data = new char[length + 1];
        std::strcpy(data, other.data);
        std::cout << "  [Copy Ctor] \"" << data << "\"" << std::endl;
    }
    
    // 3. Copy operator=
    String& operator=(const String& other) {
        if (this != &other) {
            char* newData = new char[other.length + 1];
            std::strcpy(newData, other.data);
            delete[] data;
            data = newData;
            length = other.length;
            std::cout << "  [Copy=] \"" << data << "\"" << std::endl;
        }
        return *this;
    }
    
    // 4. Move конструктор
    String(String&& other) noexcept 
        : data(other.data), length(other.length) {
        other.data = nullptr;
        other.length = 0;
        std::cout << "  [Move Ctor] \"" << data << "\"" << std::endl;
    }
    
    // 5. Move operator=
    String& operator=(String&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            length = other.length;
            other.data = nullptr;
            other.length = 0;
            std::cout << "  [Move=] \"" << data << "\"" << std::endl;
        }
        return *this;
    }
    
    const char* c_str() const { return data ? data : ""; }
    int len() const { return length; }
    
    friend std::ostream& operator<<(std::ostream& os, const String& s) {
        return os << s.c_str();
    }
};

String createGreeting(const char* name) {
    String result("Здравей, ");
    // В реална имплементация бихме конкатенирали
    return result;  // RVO или move
}

int main() {
    std::cout << "=== Конструиране ===" << std::endl;
    String s1("Hello");
    
    std::cout << "\n=== Copy ===" << std::endl;
    String s2 = s1;  // Copy ctor
    
    std::cout << "\n=== Move ===" << std::endl;
    String s3 = std::move(s1);  // Move ctor
    // s1 вече е "moved-from" — не го използвайте!
    
    std::cout << "\n=== Copy= ===" << std::endl;
    String s4("World");
    s4 = s2;  // Copy operator=
    
    std::cout << "\n=== Move= ===" << std::endl;
    String s5("Temp");
    s5 = std::move(s2);  // Move operator=
    
    std::cout << "\n=== Return value ===" << std::endl;
    String s6 = createGreeting("Иван");
    
    std::cout << "\n=== Край ===" << std::endl;
    return 0;
}
```

---

## Пример 2: DynamicArray с copy-and-swap

```cpp
#include <iostream>
#include <utility>
#include <algorithm>

class DynamicArray {
    int* data;
    int size;
    int capacity;
    
    void swap(DynamicArray& other) noexcept {
        std::swap(data, other.data);
        std::swap(size, other.size);
        std::swap(capacity, other.capacity);
    }
    
public:
    DynamicArray() : data(nullptr), size(0), capacity(0) {}
    
    explicit DynamicArray(int cap) 
        : data(new int[cap]()), size(0), capacity(cap) {
        std::cout << "  [Ctor] cap=" << cap << std::endl;
    }
    
    ~DynamicArray() {
        std::cout << "  [Dtor] size=" << size << std::endl;
        delete[] data;
    }
    
    // Copy ctor
    DynamicArray(const DynamicArray& other)
        : data(new int[other.capacity]), size(other.size), capacity(other.capacity) {
        for (int i = 0; i < size; i++) data[i] = other.data[i];
        std::cout << "  [Copy] size=" << size << std::endl;
    }
    
    // Move ctor
    DynamicArray(DynamicArray&& other) noexcept
        : data(other.data), size(other.size), capacity(other.capacity) {
        other.data = nullptr;
        other.size = 0;
        other.capacity = 0;
        std::cout << "  [Move] size=" << size << std::endl;
    }
    
    // Unified operator= (copy-and-swap)
    DynamicArray& operator=(DynamicArray other) {
        std::cout << "  [op=] swap" << std::endl;
        swap(other);
        return *this;
    }
    
    void push_back(int val) {
        if (size >= capacity) {
            int newCap = capacity == 0 ? 4 : capacity * 2;
            int* newData = new int[newCap];
            for (int i = 0; i < size; i++) newData[i] = data[i];
            delete[] data;
            data = newData;
            capacity = newCap;
        }
        data[size++] = val;
    }
    
    int getSize() const { return size; }
    int& operator[](int i) { return data[i]; }
    const int& operator[](int i) const { return data[i]; }
    
    void print() const {
        std::cout << "[";
        for (int i = 0; i < size; i++) {
            if (i > 0) std::cout << ", ";
            std::cout << data[i];
        }
        std::cout << "] (size=" << size << ", cap=" << capacity << ")" << std::endl;
    }
};

DynamicArray createRange(int from, int to) {
    DynamicArray arr(to - from);
    for (int i = from; i < to; i++) arr.push_back(i);
    return arr;  // RVO или move
}

int main() {
    std::cout << "=== create ===" << std::endl;
    DynamicArray a(10);
    a.push_back(1); a.push_back(2); a.push_back(3);
    a.print();
    
    std::cout << "\n=== copy ===" << std::endl;
    DynamicArray b = a;  // Copy
    b.print();
    
    std::cout << "\n=== move ===" << std::endl;
    DynamicArray c = std::move(a);  // Move
    c.print();
    // a.print();  // undefined — не правете това
    
    std::cout << "\n=== from function ===" << std::endl;
    DynamicArray d = createRange(10, 15);
    d.print();
    
    std::cout << "\n=== end ===" << std::endl;
    return 0;
}
```

---

## Пример 3: Move в контейнер

```cpp
#include <iostream>
#include <cstring>
#include <utility>

class HeavyObject {
    char* buffer;
    int size;
    
public:
    HeavyObject(int n) : size(n), buffer(new char[n]()) {
        std::cout << "  Alloc " << n << " bytes" << std::endl;
    }
    
    ~HeavyObject() { delete[] buffer; }
    
    HeavyObject(const HeavyObject& other) : size(other.size), buffer(new char[other.size]) {
        std::memcpy(buffer, other.buffer, size);
        std::cout << "  COPY " << size << " bytes (скъпо!)" << std::endl;
    }
    
    HeavyObject(HeavyObject&& other) noexcept : buffer(other.buffer), size(other.size) {
        other.buffer = nullptr;
        other.size = 0;
        std::cout << "  MOVE (евтино!)" << std::endl;
    }
    
    HeavyObject& operator=(HeavyObject other) {
        std::swap(buffer, other.buffer);
        std::swap(size, other.size);
        return *this;
    }
};

// Ръчен "vector" демонстрация
class HeavyContainer {
    HeavyObject** items;
    int count;
    int capacity;
    
public:
    HeavyContainer() : items(nullptr), count(0), capacity(0) {}
    
    ~HeavyContainer() {
        for (int i = 0; i < count; i++) delete items[i];
        delete[] items;
    }
    
    // Добавяне по lvalue (копиране)
    void add(const HeavyObject& obj) {
        ensureCapacity();
        items[count++] = new HeavyObject(obj);  // Copy
    }
    
    // Добавяне по rvalue (преместване)
    void add(HeavyObject&& obj) {
        ensureCapacity();
        items[count++] = new HeavyObject(std::move(obj));  // Move
    }
    
private:
    void ensureCapacity() {
        if (count >= capacity) {
            int newCap = capacity == 0 ? 4 : capacity * 2;
            HeavyObject** newItems = new HeavyObject*[newCap];
            for (int i = 0; i < count; i++) newItems[i] = items[i];
            delete[] items;
            items = newItems;
            capacity = newCap;
        }
    }
};

int main() {
    HeavyContainer container;
    
    std::cout << "--- Добавяне с copy ---" << std::endl;
    HeavyObject h1(1000);
    container.add(h1);  // copy — h1 продължава да е валиден
    
    std::cout << "\n--- Добавяне с move ---" << std::endl;
    HeavyObject h2(2000);
    container.add(std::move(h2));  // move — h2 вече не е валиден
    
    std::cout << "\n--- Добавяне на временен ---" << std::endl;
    container.add(HeavyObject(3000));  // move — автоматично, rvalue
    
    std::cout << "\n--- Край ---" << std::endl;
    return 0;
}
```

> 💡 При `add(HeavyObject(3000))` обектът е rvalue → извиква се move overload. Нула излишни копия!
