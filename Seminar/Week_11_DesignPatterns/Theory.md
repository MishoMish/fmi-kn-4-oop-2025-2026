# Шаблони за дизайн (Design Patterns) — Въведение

## Въведение

Шаблоните за дизайн (Design Patterns) са **доказани решения на повтарящи се проблеми** в софтуерния дизайн. Те не са готов код, а по-скоро **рецепти** — описание на проблем и подход за решаването му.

Въведени от „Бандата на четиримата" (GoF) в книгата *Design Patterns: Elements of Reusable Object-Oriented Software* (1994).

### Категории

| Категория | Описание | Примери |
|-----------|----------|---------|
| **Creational** | Как създаваме обекти | Factory, Singleton, Prototype |
| **Structural** | Как организираме класове | Adapter, Decorator, Composite |
| **Behavioral** | Как обектите комуникират | Strategy, Observer, Iterator |

---

## 1. Factory (Фабрика)

### Проблем
Клиентският код не трябва да знае конкретния тип обект, който създава. Решението кой тип да се създаде се взема на едно централно място.

### Решение

```cpp
class Shape {
public:
    virtual void draw() const = 0;
    virtual ~Shape() {}
};

class Circle : public Shape {
public:
    void draw() const override { std::cout << "○" << std::endl; }
};

class Rectangle : public Shape {
public:
    void draw() const override { std::cout << "□" << std::endl; }
};

class Triangle : public Shape {
public:
    void draw() const override { std::cout << "△" << std::endl; }
};

// Фабричен метод
Shape* createShape(const char* type) {
    if (std::strcmp(type, "circle") == 0) return new Circle();
    if (std::strcmp(type, "rectangle") == 0) return new Rectangle();
    if (std::strcmp(type, "triangle") == 0) return new Triangle();
    return nullptr;
}
```

### Употреба

```cpp
Shape* s = createShape("circle");
s->draw();  // ○
delete s;
```

> 💡 Когато добавим нова фигура, променяме **само** фабричния метод, а не клиентския код.

---

## 2. Strategy (Стратегия)

### Проблем
Обект трябва да използва различни алгоритми за една и съща операция, избрани по време на изпълнение.

### Решение

```cpp
// Стратегия (интерфейс)
class SortStrategy {
public:
    virtual void sort(int arr[], int n) = 0;
    virtual ~SortStrategy() {}
};

class BubbleSort : public SortStrategy {
public:
    void sort(int arr[], int n) override {
        for (int i = 0; i < n - 1; i++)
            for (int j = 0; j < n - i - 1; j++)
                if (arr[j] > arr[j+1])
                    std::swap(arr[j], arr[j+1]);
    }
};

class SelectionSort : public SortStrategy {
public:
    void sort(int arr[], int n) override {
        for (int i = 0; i < n - 1; i++) {
            int minIdx = i;
            for (int j = i + 1; j < n; j++)
                if (arr[j] < arr[minIdx]) minIdx = j;
            std::swap(arr[i], arr[minIdx]);
        }
    }
};

// Контекст
class Sorter {
    SortStrategy* strategy;
public:
    Sorter(SortStrategy* s) : strategy(s) {}
    
    void setStrategy(SortStrategy* s) { strategy = s; }
    
    void sort(int arr[], int n) {
        strategy->sort(arr, n);
    }
};
```

### Употреба

```cpp
BubbleSort bubble;
SelectionSort selection;

Sorter sorter(&bubble);
int arr[] = {5, 2, 8, 1};
sorter.sort(arr, 4);          // Използва BubbleSort

sorter.setStrategy(&selection);
sorter.sort(arr, 4);          // Сега използва SelectionSort
```

---

## 3. Singleton

### Проблем
Трябва да има **точно един** екземпляр от даден клас в цялата програма.

### Решение

```cpp
class Logger {
    Logger() {}                             // Частен конструктор
    Logger(const Logger&) = delete;         // Без копиране
    Logger& operator=(const Logger&) = delete;
    
    static Logger* instance;
    
public:
    static Logger& getInstance() {
        if (!instance) {
            instance = new Logger();
        }
        return *instance;
    }
    
    void log(const char* message) {
        std::cout << "[LOG] " << message << std::endl;
    }
};

Logger* Logger::instance = nullptr;
```

### Модерен вариант (C++11, thread-safe)

```cpp
class Logger {
    Logger() {}
    Logger(const Logger&) = delete;
    Logger& operator=(const Logger&) = delete;
    
public:
    static Logger& getInstance() {
        static Logger instance;  // Thread-safe от C++11
        return instance;
    }
    
    void log(const char* message) {
        std::cout << "[LOG] " << message << std::endl;
    }
};
```

> ⚠️ Singleton е **антипатърн** в много случаи — създава глобално състояние и затруднява тестването. Използвайте с мярка.

---

## 4. Prototype (Прототип)

### Проблем
Създаване на нов обект чрез **копиране** на съществуващ, без да знаем конкретния тип.

### Решение
Това е шаблонът зад `clone()` метода, който вече познаваме:

```cpp
class Document {
public:
    virtual Document* clone() const = 0;
    virtual void print() const = 0;
    virtual ~Document() {}
};

class TextDocument : public Document {
    char content[256];
public:
    TextDocument(const char* text) { std::strncpy(content, text, 255); content[255] = '\0'; }
    Document* clone() const override { return new TextDocument(*this); }
    void print() const override { std::cout << "Text: " << content << std::endl; }
};

class SpreadsheetDocument : public Document {
    int rows, cols;
public:
    SpreadsheetDocument(int r, int c) : rows(r), cols(c) {}
    Document* clone() const override { return new SpreadsheetDocument(*this); }
    void print() const override { std::cout << "Sheet: " << rows << "x" << cols << std::endl; }
};
```

### Prototype Registry (каталог с прототипи)

```cpp
class DocumentRegistry {
    Document* prototypes[10];
    int count = 0;
public:
    void registerPrototype(Document* proto) {
        if (count < 10) prototypes[count++] = proto;
    }
    
    Document* create(int index) const {
        if (index >= 0 && index < count)
            return prototypes[index]->clone();
        return nullptr;
    }
    
    ~DocumentRegistry() {
        for (int i = 0; i < count; i++) delete prototypes[i];
    }
};
```

---

## 5. Observer (Наблюдател) — бонус

### Проблем
Обект трябва да уведомява множество други обекти, когато състоянието му се промени.

```cpp
class Observer {
public:
    virtual void update(int newValue) = 0;
    virtual ~Observer() {}
};

class Subject {
    Observer* observers[20];
    int count = 0;
    int value = 0;

public:
    void attach(Observer* o) { if (count < 20) observers[count++] = o; }
    
    void setValue(int v) {
        value = v;
        notify();
    }
    
    void notify() {
        for (int i = 0; i < count; i++)
            observers[i]->update(value);
    }
};
```

---

## Кога кой шаблон?

| Ситуация | Шаблон |
|----------|--------|
| Не знаем конкретния тип → създаваме обект | **Factory** |
| Различни алгоритми за една задача | **Strategy** |
| Точно един екземпляр в програмата | **Singleton** |
| Копиране на обект без да знаем типа | **Prototype** (clone) |
| Уведомяване при промяна | **Observer** |

---

## 🧠 Дискусия

1. Singleton наистина ли е нужен? Кога е по-добре да предадем зависимости чрез конструктора (Dependency Injection)?
2. Може ли Factory методът да се направи по-разширяем (без `if`-ове)?
3. Strategy vs. обикновена виртуална функция — каква е разликата?
