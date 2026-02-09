# Полиморфизъм и абстрактни класове

## Въведение

Полиморфизмът е способността един и същ интерфейс да се държи различно в зависимост от **типа на обекта**, а не от типа на указателя. Това е може би **най-мощната концепция** в ООП — позволява ни да пишем код, който работи с бъдещи типове, без да бъде променян.

---

## 1. Статично vs. динамично свързване

### Статично свързване (без `virtual`)

```cpp
class Animal {
public:
    void speak() { std::cout << "..." << std::endl; }
};

class Dog : public Animal {
public:
    void speak() { std::cout << "Бау!" << std::endl; }
};

Animal* ptr = new Dog();
ptr->speak();  // "..." ← Извиква Animal::speak()!
delete ptr;
```

Компилаторът решава **по типа на указателя** (`Animal*`) коя функция да извика.

### Динамично свързване (с `virtual`)

```cpp
class Animal {
public:
    virtual void speak() { std::cout << "..." << std::endl; }
    virtual ~Animal() {}
};

class Dog : public Animal {
public:
    void speak() override { std::cout << "Бау!" << std::endl; }
};

Animal* ptr = new Dog();
ptr->speak();  // "Бау!" ← Извиква Dog::speak()!
delete ptr;    // Правилен деструктор благодарение на virtual ~Animal()
```

Решението коя функция да се извика се взема **по време на изпълнение**, въз основа на реалния тип на обекта.

---

## 2. Виртуални функции — как работят (vtable)

Когато клас има `virtual` метод, компилаторът създава **виртуална таблица (vtable)** — масив от указатели към функции:

```
Animal vtable:        Dog vtable:
┌─────────────────┐   ┌─────────────────┐
│ speak → Animal::│   │ speak → Dog::   │
│ ~Animal         │   │ ~Dog            │
└─────────────────┘   └─────────────────┘
```

Всеки обект с виртуални методи има скрит указател `vptr` към своята vtable. При извикване на `ptr->speak()`:

1. Следва `vptr` → vtable
2. Намира `speak` в таблицата
3. Извиква правилната функция

> 💡 Цената на полиморфизма: един допълнителен указател на обект + индиректно извикване. Пренебрежимо в повечето случаи.

---

## 3. Чисто виртуални функции и абстрактни класове

### Чисто виртуална функция

Функция **без имплементация** в базовия клас:

```cpp
class Shape {
public:
    virtual double area() const = 0;       // Чисто виртуална
    virtual void draw() const = 0;         // Чисто виртуална
    virtual ~Shape() {}
};

// Shape s;  // ❌ Грешка! Shape е абстрактен клас
```

### Абстрактен клас

Клас с поне една чисто виртуална функция. **Не може да се инстанцира**, но може да се използва като тип на указател/референция.

### Конкретен наследник

```cpp
class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    
    double area() const override { return 3.14159 * radius * radius; }
    void draw() const override { std::cout << "○ (r=" << radius << ")" << std::endl; }
};

class Rectangle : public Shape {
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    
    double area() const override { return width * height; }
    void draw() const override { std::cout << "□ (" << width << "x" << height << ")" << std::endl; }
};
```

---

## 4. Интерфейси

Клас **само** с чисто виртуални функции е де факто **интерфейс**:

```cpp
class Printable {
public:
    virtual void print(std::ostream& os) const = 0;
    virtual ~Printable() {}
};

class Serializable {
public:
    virtual void save(const char* filename) const = 0;
    virtual void load(const char* filename) = 0;
    virtual ~Serializable() {}
};
```

> 💡 C++ няма ключова дума `interface` (за разлика от Java). Конвенцията е клас с всички методи `pure virtual`.

---

## 5. Виртуален деструктор

```cpp
class Base {
public:
    ~Base() { std::cout << "~Base" << std::endl; }  // ❌ НЕ е virtual
};

class Derived : public Base {
    int* data;
public:
    Derived() : data(new int[100]) {}
    ~Derived() { delete[] data; std::cout << "~Derived" << std::endl; }
};

Base* ptr = new Derived();
delete ptr;  // ❌ Извиква САМО ~Base()! ~Derived() НЕ се извиква → MEMORY LEAK!
```

**Правило:** Ако клас има поне една `virtual` функция, деструкторът **задължително** трябва да бъде `virtual`.

---

## 6. `override` и `final`

### `override` (C++11)

Казва на компилатора: „Искам да предефинирам виртуален метод от базата":

```cpp
class Animal {
public:
    virtual void speak() const { std::cout << "..." << std::endl; }
};

class Dog : public Animal {
public:
    void speak() const override { std::cout << "Бау!" << std::endl; }
    // void speek() const override { ... }  // ❌ Компилаторна грешка! Няма speek() в Animal
};
```

### `final`

Предотвратява по-нататъшно наследяване или предефиниране:

```cpp
class Animal {
public:
    virtual void speak() const { ... }
};

class Dog final : public Animal {  // Никой не може да наследи Dog
    void speak() const final { ... }  // Никой не може да предефинира speak()
};
```

---

## 7. Хетерогенни колекции (вектори от `Base*`)

```cpp
void printAllShapes(Shape* shapes[], int count) {
    for (int i = 0; i < count; i++) {
        shapes[i]->draw();
        std::cout << "  Лице: " << shapes[i]->area() << std::endl;
    }
}

int main() {
    Shape* shapes[3];
    shapes[0] = new Circle(5);
    shapes[1] = new Rectangle(3, 4);
    shapes[2] = new Circle(2);
    
    printAllShapes(shapes, 3);
    
    for (int i = 0; i < 3; i++) delete shapes[i];
}
```

---

## 8. Методът `clone()` — виртуален конструктор

Копиращият конструктор не е виртуален. За дълбоко копиране на полиморфен обект използваме `clone()`:

```cpp
class Shape {
public:
    virtual Shape* clone() const = 0;
    virtual ~Shape() {}
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    Shape* clone() const override { return new Circle(*this); }
};
```

---

## Обобщение

| Концепция | Описание |
|-----------|----------|
| `virtual` | Активира динамично свързване |
| `= 0` | Чисто виртуална функция (абстрактен клас) |
| `override` | Проверка от компилатора за правилно предефиниране |
| `final` | Забрана за наследяване/предефиниране |
| vtable | Таблица с указатели към виртуалните функции |
| `virtual ~Destructor` | Задължителен при полиморфизъм |
| `clone()` | Виртуален конструктор за дълбоко копиране |
| Хетерогенна колекция | Масив от `Base*` с различни наследници |
