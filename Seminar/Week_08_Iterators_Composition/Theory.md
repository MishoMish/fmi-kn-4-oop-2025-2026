# Итератори и композиция

## Въведение

Итераторите предоставят **унифициран начин за обхождане** на колекции от елементи, без да знаем вътрешната структура на колекцията. Композицията пък е ключова ООП техника, при която обект **съдържа** друг обект като свое поле.

---

## 1. Какво е итератор?

Итераторът е обект, който знае как да:
- Сочи към **текущ елемент** от колекцията
- **Премести се** напред (или назад) в колекцията
- **Сравни се** с друг итератор (за край на обхождането)

### Конвенция в C++

```cpp
container.begin()  // Итератор към ПЪРВИЯ елемент
container.end()    // Итератор СЛЕД последния елемент (one-past-end)
```

```
begin()                              end()
  ↓                                    ↓
  [elem0] [elem1] [elem2] [elem3]  [ ? ]
```

### Range-based `for` цикъл

Когато пишем:
```cpp
for (auto& elem : container) { ... }
```

Компилаторът го транслира до:
```cpp
{
    auto __begin = container.begin();
    auto __end = container.end();
    for (; __begin != __end; ++__begin) {
        auto& elem = *__begin;
        // ... body ...
    }
}
```

> 💡 За да поддържа range-based for, типът трябва да има `begin()` и `end()` методи, а итераторът — `*`, `++` и `!=` оператори.

---

## 2. Операции на итератор

Минималните оператори, които итераторът трябва да поддържа:

| Оператор | Описание |
|----------|----------|
| `*it` | Достъп до текущия елемент (dereference) |
| `++it` | Преместване към следващия елемент |
| `it != other` | Сравнение — различни ли са два итератора |

### За двупосочен (bidirectional) итератор, добавяме:
| Оператор | Описание |
|----------|----------|
| `--it` | Преместване към предишния елемент |

---

## 3. Създаване на собствен итератор

### Стъпка по стъпка

```cpp
class IntArray {
    int* data;
    int size;
    
public:
    IntArray(int n) : size(n) { data = new int[n]; }
    ~IntArray() { delete[] data; }
    
    int& operator[](int i) { return data[i]; }
    int getSize() const { return size; }
    
    // Вложен клас — итератор
    class Iterator {
        int* ptr;
    public:
        Iterator(int* p) : ptr(p) {}
        
        int& operator*() { return *ptr; }
        Iterator& operator++() { ++ptr; return *this; }
        bool operator!=(const Iterator& other) const { return ptr != other.ptr; }
    };
    
    // begin() и end() методи
    Iterator begin() { return Iterator(data); }
    Iterator end() { return Iterator(data + size); }
};
```

### Използване

```cpp
IntArray arr(5);
for (int i = 0; i < 5; i++) arr[i] = (i + 1) * 10;

// Range-based for — работи благодарение на begin()/end()
for (int& val : arr) {
    std::cout << val << " ";  // 10 20 30 40 50
}
```

---

## 4. Const итератор

За `const` обекти трябва `const` версия на итератора:

```cpp
class IntArray {
    // ...
    
    class ConstIterator {
        const int* ptr;
    public:
        ConstIterator(const int* p) : ptr(p) {}
        
        const int& operator*() const { return *ptr; }
        ConstIterator& operator++() { ++ptr; return *this; }
        bool operator!=(const ConstIterator& other) const { return ptr != other.ptr; }
    };
    
    ConstIterator begin() const { return ConstIterator(data); }
    ConstIterator end() const { return ConstIterator(data + size); }
};

void printArray(const IntArray& arr) {
    for (const int& val : arr) {  // Извиква const begin()/end()
        std::cout << val << " ";
    }
}
```

---

## 5. Композиция (has-a релация)

### Какво е композиция?

Композицията е когато един клас **съдържа** обект от друг клас като поле. Това моделира „has-a" релация:

```
University has-a Department
Department has-a Professor
Car has-a Engine
```

```cpp
class Engine {
    int horsepower;
    char type[20];  // "V6", "Electric", etc.
public:
    Engine(int hp, const char* t) : horsepower(hp) {
        std::strncpy(type, t, 19);
        type[19] = '\0';
    }
    void start() const { std::cout << type << " двигател стартиран (" << horsepower << "к.с.)" << std::endl; }
};

class Car {
    char brand[50];
    Engine engine;  // Композиция — Car HAS-A Engine
    
public:
    Car(const char* b, int hp, const char* engineType)
        : engine(hp, engineType)
    {
        std::strncpy(brand, b, 49);
        brand[49] = '\0';
    }
    
    void start() const {
        std::cout << brand << ": ";
        engine.start();
    }
};
```

### Жизнен цикъл при композиция

При създаване на `Car`:
1. Първо се конструира `Engine` (вътрешният обект)
2. После тялото на конструктора на `Car`

При унищожаване — обратен ред:
1. Деструкторът на `Car`
2. После деструкторът на `Engine`

---

## 6. Композиция vs. Наследяване

| Аспект | Композиция (has-a) | Наследяване (is-a) |
|--------|--------------------|--------------------|
| Релация | „Съдържа" | „Е вид от" |
| Гъвкавост | По-гъвкаво (може да се подмени) | По-ригидно |
| Coupling | По-слабо свързване | По-силно свързване |
| Пример | `Car has-a Engine` | `Dog is-a Animal` |

> 🧠 **Правило:** Предпочитайте композиция пред наследяване (Composition over Inheritance). Използвайте наследяване само когато „is-a" е наистина правилната релация.

---

## Обобщение

| Концепция | Описание |
|-----------|----------|
| Итератор | Обект за обхождане на колекция |
| `begin()` / `end()` | Конвенция за начало и „един-след-край" |
| `*`, `++`, `!=` | Минимални оператори на итератор |
| Range-based for | Захар над `begin()`/`end()` + итератор |
| Композиция | Обект съдържа друг обект (has-a) |
| Composition > Inheritance | Предпочитайте композиция за по-гъвкав дизайн |
