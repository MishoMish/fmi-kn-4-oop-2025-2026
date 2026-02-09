# SOLID принципи

## Въведение

SOLID е акроним за пет фундаментални принципа на обектно-ориентирания дизайн, формулирани от Robert C. Martin. Те правят кода **по-разбираем, по-гъвкав и по-лесен за поддръжка**.

| Буква | Принцип | Кратко |
|-------|---------|--------|
| **S** | Single Responsibility | Един клас — една отговорност |
| **O** | Open/Closed | Отворен за разширяване, затворен за промяна |
| **L** | Liskov Substitution | Наследникът може да замени базата |
| **I** | Interface Segregation | Много малки интерфейси > един голям |
| **D** | Dependency Inversion | Зависи от абстракции, не от конкретни класове |

---

## 1. Single Responsibility Principle (SRP)

> Клас трябва да има **една и само една причина** да бъде променен.

### ❌ Нарушение

```cpp
class Employee {
    char name[100];
    double salary;
    
public:
    // Бизнес логика
    double calculateBonus() const { return salary * 0.1; }
    
    // Работа с файлове
    void saveToFile(const char* filename) const {
        std::ofstream f(filename);
        f << name << " " << salary;
    }
    
    // Генериране на отчет
    void printReport() const {
        std::cout << "=== Report ===" << std::endl;
        std::cout << name << ": " << salary << std::endl;
    }
};
```

Три причини за промяна: промяна в бизнес логиката, промяна на формата на файла, промяна на формата на отчета.

### ✅ Правилно

```cpp
class Employee {
    char name[100];
    double salary;
public:
    Employee(const char* n, double s);
    const char* getName() const { return name; }
    double getSalary() const { return salary; }
    double calculateBonus() const { return salary * 0.1; }
};

class EmployeeRepository {
public:
    void save(const Employee& e, const char* filename) const;
    Employee load(const char* filename) const;
};

class EmployeeReportGenerator {
public:
    void printReport(const Employee& e) const;
};
```

---

## 2. Open/Closed Principle (OCP)

> Класовете трябва да са **отворени за разширяване**, но **затворени за модификация**.

### ❌ Нарушение

```cpp
double calculateArea(Shape* s) {
    if (s->type == "circle")
        return 3.14 * s->radius * s->radius;
    else if (s->type == "rectangle")
        return s->width * s->height;
    // Трябва да добавяме нов else if за всяка нова фигура!
}
```

### ✅ Правилно — полиморфизъм

```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() {}
};

class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area() const override { return 3.14159 * r * r; }
};

// Добавяне на нова фигура НЕ изисква промяна на съществуващ код
class Hexagon : public Shape {
    double side;
public:
    Hexagon(double s) : side(s) {}
    double area() const override { return 2.598 * side * side; }
};
```

---

## 3. Liskov Substitution Principle (LSP)

> Ако `S` е подтип на `T`, обекти от тип `T` могат да бъдат **заменени** с обекти от тип `S`, без да се наруши коректността.

### ❌ Класическо нарушение: квадрат наследява правоъгълник

```cpp
class Rectangle {
protected:
    int width, height;
public:
    virtual void setWidth(int w) { width = w; }
    virtual void setHeight(int h) { height = h; }
    int area() const { return width * height; }
};

class Square : public Rectangle {
public:
    void setWidth(int w) override { width = w; height = w; }
    void setHeight(int h) override { width = h; height = h; }
};

// Тази функция очаква Rectangle поведение
void resize(Rectangle& r) {
    r.setWidth(5);
    r.setHeight(10);
    // За Rectangle: area = 50 ✅
    // За Square: area = 100 ❌ (неочаквано!)
}
```

### ✅ Решение: отделни класове

```cpp
class Shape {
public:
    virtual int area() const = 0;
    virtual ~Shape() {}
};

class Rectangle : public Shape {
    int width, height;
public:
    Rectangle(int w, int h) : width(w), height(h) {}
    int area() const override { return width * height; }
};

class Square : public Shape {
    int side;
public:
    Square(int s) : side(s) {}
    int area() const override { return side * side; }
};
```

---

## 4. Interface Segregation Principle (ISP)

> Клиентите не трябва да зависят от интерфейси, които **не използват**.

### ❌ Нарушение: „тлъст" интерфейс

```cpp
class IWorker {
public:
    virtual void work() = 0;
    virtual void eat() = 0;
    virtual void sleep() = 0;
    virtual ~IWorker() {}
};

class Robot : public IWorker {
public:
    void work() override { /* OK */ }
    void eat() override { /* Роботът не яде?! */ }
    void sleep() override { /* Роботът не спи?! */ }
};
```

### ✅ Правилно: малки, фокусирани интерфейси

```cpp
class IWorkable {
public:
    virtual void work() = 0;
    virtual ~IWorkable() {}
};

class IFeedable {
public:
    virtual void eat() = 0;
    virtual ~IFeedable() {}
};

class ISleepable {
public:
    virtual void sleep() = 0;
    virtual ~ISleepable() {}
};

class Human : public IWorkable, public IFeedable, public ISleepable {
public:
    void work() override { /* ... */ }
    void eat() override { /* ... */ }
    void sleep() override { /* ... */ }
};

class Robot : public IWorkable {
public:
    void work() override { /* ... */ }
    // Не имплементира eat/sleep — не му трябват!
};
```

---

## 5. Dependency Inversion Principle (DIP)

> Модулите от високо ниво не трябва да зависят от модули от ниско ниво. И двата трябва да зависят от **абстракции**.

### ❌ Нарушение

```cpp
class MySQLDatabase {
public:
    void save(const char* data) { /* MySQL логика */ }
};

class UserService {
    MySQLDatabase db;  // ← Директна зависимост от конкретен клас!
public:
    void createUser(const char* name) {
        db.save(name);
    }
};
```

### ✅ Правилно: зависимост от абстракция

```cpp
class IDatabase {
public:
    virtual void save(const char* data) = 0;
    virtual ~IDatabase() {}
};

class MySQLDatabase : public IDatabase {
public:
    void save(const char* data) override { /* MySQL логика */ }
};

class PostgresDatabase : public IDatabase {
public:
    void save(const char* data) override { /* Postgres логика */ }
};

class UserService {
    IDatabase* db;  // ← Зависимост от абстракция
public:
    UserService(IDatabase* database) : db(database) {}
    
    void createUser(const char* name) {
        db->save(name);
    }
};
```

---

## 🧠 Дискусия

1. Кой SOLID принцип е най-лесно нарушим на практика?
2. Как OCP се свързва с полиморфизма?
3. Може ли спазването на SOLID да усложни кода? Кога е „прекалено"?

---

## Обобщение

| Принцип | Симптом при нарушение | Решение |
|---------|----------------------|---------|
| **SRP** | Клас с 500+ реда, прави „всичко" | Раздели на малки класове |
| **OCP** | Дълги `if`/`switch` по тип | Полиморфизъм |
| **LSP** | Наследник нарушава очакванията | Ревизирай йерархията |
| **ISP** | Празни имплементации | Раздели интерфейса |
| **DIP** | Конкретни типове навсякъде | Въведи абстракции |
