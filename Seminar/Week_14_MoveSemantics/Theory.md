# Move семантика и Правилото на петте

## Въведение

C++11 въведе **move семантика** — механизъм за „преместване" на ресурси от един обект в друг, вместо скъпо копиране. Това е особено важно при работа с динамична памет, файлове и други тежки ресурси.

---

## 1. Lvalue и Rvalue

### Lvalue
Израз, който има **адрес в паметта** — може да стои от лявата страна на `=`:

```cpp
int x = 5;      // x е lvalue
int& ref = x;   // OK — може да вземем lvalue референция
```

### Rvalue
Израз, който е **временен** — няма постоянен адрес:

```cpp
int y = 3 + 4;  // (3 + 4) е rvalue
// int& ref = 3 + 4;  // ❌ Грешка! Не можем да вземем lvalue ref на rvalue
int&& rref = 3 + 4;  // ✅ Rvalue референция (C++11)
```

### Защо е важно?

```cpp
String getName() { return String("Иван"); }  // Връща временен обект

String s = getName();  
// Без move: конструира временен → копира в s → разрушава временния
// С move: конструира временен → ПРЕМЕСТВА в s (без копиране!)
```

---

## 2. Move конструктор

```cpp
class String {
    char* data;
    int length;
    
public:
    // Обикновен конструктор
    String(const char* str) {
        length = std::strlen(str);
        data = new char[length + 1];
        std::strcpy(data, str);
        std::cout << "  [Ctor] " << data << std::endl;
    }
    
    // Copy конструктор (скъп — заделя памет и копира)
    String(const String& other) : length(other.length) {
        data = new char[length + 1];
        std::strcpy(data, other.data);
        std::cout << "  [Copy] " << data << std::endl;
    }
    
    // Move конструктор (евтин — „краде" ресурсите)
    String(String&& other) noexcept : data(other.data), length(other.length) {
        other.data = nullptr;   // Важно! Оставяме source в валидно състояние
        other.length = 0;
        std::cout << "  [Move] " << data << std::endl;
    }
    
    ~String() {
        if (data) {
            std::cout << "  [Dtor] " << data << std::endl;
        }
        delete[] data;
    }
};
```

### Как работи move конструкторът?

```
ПРЕДИ move:
  source: data → [И|в|а|н|\0]    other: data → ???
  
СЛЕД move:
  source: data → nullptr          other: data → [И|в|а|н|\0]
```

Не се копират данни — само се **прехвърлят указатели**. O(1) вместо O(n)!

---

## 3. Move оператор за присвояване

```cpp
String& operator=(String&& other) noexcept {
    if (this != &other) {
        delete[] data;           // Освобождаваме текущите ресурси
        
        data = other.data;       // Крадем ресурсите
        length = other.length;
        
        other.data = nullptr;    // Оставяме source валиден
        other.length = 0;
    }
    std::cout << "  [Move=]" << std::endl;
    return *this;
}
```

---

## 4. `std::move` — каст към rvalue

`std::move` **не премества** нищо — просто превръща lvalue в rvalue референция:

```cpp
String a("Hello");
String b = std::move(a);  // a се третира като rvalue → извиква move ctor

// ВНИМАНИЕ: след std::move, 'a' е в "moved-from" състояние!
// Не го използвайте, освен за присвояване или разрушаване.
```

---

## 5. Правилото на петте (Rule of Five)

Ако клас дефинира **един** от следните, трябва да дефинира **всичките пет**:

| # | Специален метод | Сигнатура |
|---|-----------------|-----------|
| 1 | Деструктор | `~T()` |
| 2 | Copy конструктор | `T(const T&)` |
| 3 | Copy operator= | `T& operator=(const T&)` |
| 4 | Move конструктор | `T(T&&) noexcept` |
| 5 | Move operator= | `T& operator=(T&&) noexcept` |

### Пълен пример

```cpp
class DynamicArray {
    int* data;
    int size;
    
public:
    // Конструктор
    DynamicArray(int n) : size(n), data(new int[n]()) {}
    
    // 1. Деструктор
    ~DynamicArray() { delete[] data; }
    
    // 2. Copy конструктор
    DynamicArray(const DynamicArray& other) 
        : size(other.size), data(new int[other.size]) {
        for (int i = 0; i < size; i++) data[i] = other.data[i];
    }
    
    // 3. Copy operator=
    DynamicArray& operator=(const DynamicArray& other) {
        if (this != &other) {
            int* newData = new int[other.size];
            for (int i = 0; i < other.size; i++) newData[i] = other.data[i];
            delete[] data;
            data = newData;
            size = other.size;
        }
        return *this;
    }
    
    // 4. Move конструктор
    DynamicArray(DynamicArray&& other) noexcept 
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }
    
    // 5. Move operator=
    DynamicArray& operator=(DynamicArray&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
};
```

---

## 6. Copy-and-Swap идиом (елегантно решение)

```cpp
class DynamicArray {
    int* data;
    int size;
    
    void swap(DynamicArray& other) noexcept {
        std::swap(data, other.data);
        std::swap(size, other.size);
    }
    
public:
    DynamicArray(int n) : size(n), data(new int[n]()) {}
    ~DynamicArray() { delete[] data; }
    
    DynamicArray(const DynamicArray& other)
        : size(other.size), data(new int[other.size]) {
        for (int i = 0; i < size; i++) data[i] = other.data[i];
    }
    
    DynamicArray(DynamicArray&& other) noexcept
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }
    
    // Един operator= за copy И move!
    DynamicArray& operator=(DynamicArray other) {  // По стойност!
        swap(other);
        return *this;
    }
};
```

> 💡 Параметърът `other` се конструира по стойност — ако подадем lvalue, се извиква copy ctor; ако подадем rvalue, се извиква move ctor. Универсално и exception-safe!

---

## 7. Кога се извиква move?

| Ситуация | Извиква |
|----------|---------|
| `String b = a;` | Copy ctor |
| `String b = std::move(a);` | Move ctor |
| `String b = createString();` | Move ctor (или RVO) |
| `b = a;` | Copy operator= |
| `b = std::move(a);` | Move operator= |
| `b = createString();` | Move operator= (или RVO) |

---

## 8. Return Value Optimization (RVO)

Компилаторът може да **елиминира** копирането/преместването при връщане от функция:

```cpp
String createGreeting() {
    String s("Здравей!");
    return s;  // RVO — може да конструира директно в caller-а
}

String g = createGreeting();  // Може да не се извика нито copy, нито move ctor
```

> 💡 Не пишете `return std::move(s)` — това **пречи** на RVO!

---

## Обобщение

| Концепция | Описание |
|-----------|----------|
| Lvalue | Именуван обект с адрес |
| Rvalue | Временен обект без постоянен адрес |
| `T&&` | Rvalue референция |
| `std::move()` | Каст към rvalue (не премества!) |
| Move ctor | Прехвърля ресурси от временен обект |
| Move operator= | Присвояване чрез преместване |
| Rule of Five | Деструктор + Copy + Move (по 2) |
| Copy-and-swap | Елегантен operator= за copy и move |
| RVO | Компилаторна оптимизация — елиминира копиране |
| `noexcept` | Move операциите трябва да са noexcept |
