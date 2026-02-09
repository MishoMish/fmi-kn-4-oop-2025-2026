# Подготовка за поправителен изпит

> Обхват: Всички теми (Седмици 1–14) — фокус върху типични грешки и слабости

---

## Теоретични въпроси за самоподготовка

Преди задачите, уверете се, че можете да отговорите на тези въпроси:

1. Каква е разликата между `struct` и `class` в C++?
2. Какво е `vtable` и кога се създава?
3. Защо деструкторът трябва да е `virtual` при полиморфизъм?
4. Какво е RAII и защо е важно?
5. Обяснете разликата между shallow copy и deep copy.
6. Кога се извиква move конструкторът вместо copy конструктора?
7. Какво е „slicing" при наследяване?
8. Обяснете Liskov Substitution Principle с пример.
9. Какво е разликата между `override` и `final`?
10. Кога да използваме композиция вместо наследяване?

---

## Задача 1: Намерете грешките

Следният код съдържа **поне 5 грешки**. Намерете ги и обяснете:

```cpp
class Base {
    int* data;
public:
    Base(int n) { data = new int[n]; }
    ~Base() { delete data; }
    
    Base(const Base& other) {
        data = other.data;
    }
};

class Derived : public Base {
    char* name;
public:
    Derived(int n, const char* str) : Base(n) {
        name = new char[strlen(str)];
        strcpy(name, str);
    }
    
    ~Derived() { delete name; }
};

void process(Base b) {
    // ...
}

int main() {
    Base* ptr = new Derived(5, "hello");
    process(*ptr);
    delete ptr;
}
```

---

## Задача 2: `StringVector`

Имплементирайте `StringVector` — динамичен масив от `String` обекти:
- **Rule of Five** (включително move)
- `push_back(const String&)` и `push_back(String&&)` (move overload)
- `operator[]` с bounds checking
- `find(const char*)` — връща индекса
- Запис/четене от файл

---

## Задача 3: Полиморфна йерархия с SOLID

Създайте система за превозни средства:

**Интерфейси (ISP):**
- `IDriveable` — `drive()`, `getFuelLevel()`
- `IElectric` — `charge()`, `getBatteryLevel()`
- `IDisplayable` — `display()`

**Класове:**
- `Car` : `IDriveable`, `IDisplayable`
- `ElectricCar` : `IDriveable`, `IElectric`, `IDisplayable`
- `Bicycle` : `IDisplayable` (не е `IDriveable` — няма гориво!)

**Factory:** `VehicleFactory::create(const char* type)`

**Колекция:** `Garage` — пази `IDisplayable*` масив, показва всички

---

## Задача 4: Exception Handling

Допълнете класа `BankAccount`:

```cpp
class BankAccount {
    char owner[100];
    double balance;
    bool locked;
public:
    void deposit(double amount);   // хвърля при amount <= 0 или locked
    void withdraw(double amount);  // хвърля при недостатъчен баланс или locked
    void transfer(BankAccount& to, double amount);  // exception-safe!
};
```

Изискване: `transfer` трябва да има **strong exception guarantee** — ако нещо се обърка, и двата акаунта остават непроменени.

---

## Задача 5: Цялостна задача

Имплементирайте система за управление на зоопарк:

- `Animal` (абстрактен) — име, вид, възраст
  - `Mammal` — тегло, дали е домашно
  - `Bird` — размах на крилата, дали лети
  - `Reptile` — дължина, дали е отровно

- `Zoo`:
  - Хетерогенна колекция с Rule of Five
  - `addAnimal()`, `removeByName()`, `findBySpecies()`
  - `getOldestAnimal()`, `getTotalCount()`
  - Factory за създаване на животни от потребителски вход
  - Сериализация в бинарен файл
  - `operator<<` за отпечатване на целия зоопарк
  - Обработка на грешки с custom exceptions
