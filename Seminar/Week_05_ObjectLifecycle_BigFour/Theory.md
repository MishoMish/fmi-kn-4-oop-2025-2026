# Жизнен цикъл на обект: Конструктори, деструктори и Голямата четворка

## Въведение

Всеки обект в C++ преминава през ясно дефиниран жизнен цикъл: **създаване → използване → унищожаване**. Разбирането на този цикъл е критично за писане на коректен C++ код, особено при управление на динамична памет.

---

## 1. Конструктори

### Какво е конструктор?

Конструкторът е специален метод, който се извиква **автоматично** при създаване на обект. Той инициализира полетата и подготвя обекта за използване.

```cpp
class Point {
    double x, y;
public:
    // Подразбиращ се конструктор (default constructor)
    Point() : x(0), y(0) {}
    
    // Конструктор с параметри
    Point(double x, double y) : x(x), y(y) {}
};

Point p1;           // Извиква Point() → (0, 0)
Point p2(3.0, 4.0); // Извиква Point(double, double) → (3.0, 4.0)
Point p3{1.0, 2.0}; // Uniform initialization (C++11)
```

### Списък за инициализация (Member Initializer List)

```cpp
class Student {
    const int id;        // const — трябва да се инициализира
    char name[100];
    double grade;
    
public:
    // Списъкът за инициализация е ПРЕДИ тялото на конструктора
    Student(int id, const char* name, double grade)
        : id(id), grade(grade)
    {
        std::strncpy(this->name, name, 99);
        this->name[99] = '\0';
    }
};
```

**Кога е задължителен списъкът за инициализация?**
- `const` полета
- Референтни полета (`&`)
- Полета от тип без подразбиращ се конструктор
- Базов клас (при наследяване)

> 💡 **Best practice:** Винаги използвайте списък за инициализация. Той е по-ефективен от присвояване в тялото.

---

## 2. Деструктори

### Какво е деструктор?

Деструкторът се извиква **автоматично** при унищожаване на обект. Той освобождава ресурсите, които обектът притежава.

```cpp
class Logger {
    char* buffer;
    
public:
    Logger(int size) {
        buffer = new char[size];
        std::cout << "Logger създаден" << std::endl;
    }
    
    ~Logger() {
        delete[] buffer;
        std::cout << "Logger унищожен" << std::endl;
    }
};
```

### Кога се извиква деструкторът?

1. **Локален обект** — при излизане от scope
2. **`delete`** — при освобождаване на динамичен обект
3. **Масив** — за всеки елемент, в обратен ред

```cpp
void example() {
    Logger a(100);          // "Logger създаден"
    Logger b(200);          // "Logger създаден"
}   // b се унищожава, после a (обратен ред!)
// "Logger унищожен"
// "Logger унищожен"
```

---

## 3. Копиращ конструктор (Copy Constructor)

### Какво е?

Създава **ново копие** на съществуващ обект:

```cpp
class String {
    char* data;
    int length;
    
public:
    // Копиращ конструктор
    String(const String& other) : length(other.length) {
        data = new char[length + 1];
        std::strcpy(data, other.data);
    }
};
```

### Кога се извиква?

```cpp
String a("hello");
String b(a);            // 1. Явно извикване
String c = a;           // 2. Инициализация (НЕ е operator=!)
void func(String s);    // 3. Предаване по стойност
func(a);
String getStr();        // 4. Връщане по стойност (може да се оптимизира)
```

### Shallow vs. Deep Copy

```cpp
// ❌ SHALLOW COPY (по подразбиране) — двата обекта сочат към ЕДНА И СЪЩА памет
class BadString {
    char* data;
public:
    BadString(const char* str) {
        data = new char[std::strlen(str) + 1];
        std::strcpy(data, str);
    }
    // Няма копиращ конструктор → компилаторът генерира shallow copy
    ~BadString() { delete[] data; }
};

BadString a("hello");
BadString b = a;  // b.data сочи към СЪЩАТА памет като a.data
// При унищожаване → double delete! 💥
```

```cpp
// ✅ DEEP COPY — всеки обект има СВОЕ копие на данните
class GoodString {
    char* data;
public:
    GoodString(const char* str) {
        data = new char[std::strlen(str) + 1];
        std::strcpy(data, str);
    }
    
    GoodString(const GoodString& other) {
        data = new char[std::strlen(other.data) + 1];
        std::strcpy(data, other.data);
    }
    
    ~GoodString() { delete[] data; }
};
```

---

## 4. Оператор за присвояване (`operator=`)

### Разлика от копиращ конструктор

```cpp
String a("hello");
String b("world");

String c = a;    // Копиращ конструктор (c не е съществувал преди)
b = a;           // operator= (b вече съществува!)
```

### Имплементация

```cpp
class String {
    char* data;
    int length;
    
public:
    String& operator=(const String& other) {
        if (this == &other) return *this;  // 1. Проверка за самоприсвояване
        
        delete[] data;                      // 2. Освобождаване на старата памет
        
        length = other.length;              // 3. Копиране на данните
        data = new char[length + 1];
        std::strcpy(data, other.data);
        
        return *this;                       // 4. Връщане на *this (за chaining)
    }
};
```

> ⚠️ **Самоприсвояване:** `a = a;` без проверката `this == &other` ще доведе до `delete` на данните, преди да бъдат копирани!

---

## 5. Голямата четворка (Big Four / Rule of Four)

### Правилото

> Ако класът ви се нуждае от **едно** от следните, вероятно се нуждае от **всички четири**:

1. **Деструктор** (`~ClassName()`)
2. **Копиращ конструктор** (`ClassName(const ClassName&)`)
3. **Оператор за присвояване** (`operator=`)
4. *(Бъдещ)* Подразбиращ се конструктор (ако е нужен)

### Кога е нужна Голямата четворка?

Когато класът управлява **ресурси**:
- Динамична памет (`new`/`delete`)
- Файлови дескриптори
- Мрежови връзки
- Всичко, което трябва да се „почисти"

### Пълен пример

```cpp
class DynamicArray {
    int* data;
    int size;
    int capacity;
    
public:
    // Конструктор
    DynamicArray(int cap = 10) : size(0), capacity(cap) {
        data = new int[capacity];
    }
    
    // Деструктор
    ~DynamicArray() {
        delete[] data;
    }
    
    // Копиращ конструктор (deep copy)
    DynamicArray(const DynamicArray& other)
        : size(other.size), capacity(other.capacity)
    {
        data = new int[capacity];
        for (int i = 0; i < size; i++) {
            data[i] = other.data[i];
        }
    }
    
    // Оператор за присвояване
    DynamicArray& operator=(const DynamicArray& other) {
        if (this == &other) return *this;
        
        delete[] data;
        
        size = other.size;
        capacity = other.capacity;
        data = new int[capacity];
        for (int i = 0; i < size; i++) {
            data[i] = other.data[i];
        }
        
        return *this;
    }
    
    void push(int value) {
        if (size >= capacity) return;  // Опростено
        data[size++] = value;
    }
    
    int get(int index) const { return data[index]; }
    int getSize() const { return size; }
};
```

---

## 6. Ред на извикване на конструктори и деструктори

```cpp
class A {
public:
    A()  { std::cout << "A()" << std::endl; }
    ~A() { std::cout << "~A()" << std::endl; }
};

class B {
    A member;
public:
    B()  { std::cout << "B()" << std::endl; }
    ~B() { std::cout << "~B()" << std::endl; }
};

int main() {
    B obj;
}
// Изход:
// A()     ← Първо се конструират членовете
// B()     ← После тялото на конструктора
// ~B()    ← Първо деструкторът на B
// ~A()    ← После се унищожават членовете (обратен ред)
```

---

## Обобщение

| Концепция | Кога се извиква | Отговорност |
|-----------|----------------|-------------|
| Конструктор | При създаване на обект | Инициализация |
| Деструктор | При унищожаване | Освобождаване на ресурси |
| Копиращ конструктор | При копиране (нов обект) | Deep copy |
| `operator=` | При присвояване (съществуващ обект) | Deep copy + cleanup |

> 🧠 **Ключов takeaway:** Ако класът ви заделя памет с `new`, **задължително** имплементирайте Голямата четворка. В противен случай ще имате memory leaks, double deletes или dangling pointers.
