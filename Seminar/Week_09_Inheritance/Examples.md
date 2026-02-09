# Седмица 9 — Примери

## Пример 1: `Person` → `Student` йерархия

```cpp
#include <iostream>
#include <cstring>

class Person {
protected:
    char name[100];
    int age;
    
public:
    Person(const char* n, int a) : age(a) {
        std::strncpy(name, n, 99);
        name[99] = '\0';
        std::cout << "Person::Person(\"" << name << "\")" << std::endl;
    }
    
    ~Person() {
        std::cout << "Person::~Person(\"" << name << "\")" << std::endl;
    }
    
    const char* getName() const { return name; }
    int getAge() const { return age; }
    
    void print() const {
        std::cout << name << ", " << age << " г." << std::endl;
    }
};

class Student : public Person {
    int facultyNumber;
    double grade;
    
public:
    Student(const char* n, int a, int fn, double g)
        : Person(n, a), facultyNumber(fn), grade(g)
    {
        std::cout << "Student::Student(ФН: " << fn << ")" << std::endl;
    }
    
    ~Student() {
        std::cout << "Student::~Student(ФН: " << facultyNumber << ")" << std::endl;
    }
    
    int getFN() const { return facultyNumber; }
    double getGrade() const { return grade; }
    
    // Override на print()
    void print() const {
        Person::print();  // Извикваме базовия print
        std::cout << "  ФН: " << facultyNumber << ", Оценка: " << grade << std::endl;
    }
};

int main() {
    std::cout << "=== Създаване ===" << std::endl;
    Student s("Иван Петров", 21, 62300, 5.50);
    
    std::cout << "\n=== Извикване на print ===" << std::endl;
    s.print();
    
    std::cout << "\n=== Край ===" << std::endl;
    return 0;
}
// Изход:
// === Създаване ===
// Person::Person("Иван Петров")
// Student::Student(ФН: 62300)
//
// === Извикване на print ===
// Иван Петров, 21 г.
//   ФН: 62300, Оценка: 5.5
//
// === Край ===
// Student::~Student(ФН: 62300)
// Person::~Person("Иван Петров")
```

> 💡 Обърнете внимание на **реда**: конструктори (Base → Derived), деструктори (Derived → Base).

---

## Пример 2: `Device` йерархия с override

```cpp
#include <iostream>

class Device {
protected:
    char name[50];
    bool on;
    
public:
    Device(const char* n) : on(false) {
        std::strncpy(name, n, 49);
        name[49] = '\0';
    }
    
    void turnOn() { on = true; }
    void turnOff() { on = false; }
    
    void status() const {
        std::cout << name << " е " << (on ? "включен" : "изключен") << std::endl;
    }
};

class Phone : public Device {
    int batteryPercent;
    
public:
    Phone(const char* n, int battery)
        : Device(n), batteryPercent(battery) {}
    
    void status() const {
        Device::status();  // Базов status
        std::cout << "  Батерия: " << batteryPercent << "%" << std::endl;
    }
    
    void call(const char* number) const {
        if (!on) {
            std::cout << "Телефонът е изключен!" << std::endl;
            return;
        }
        std::cout << name << " звъни на " << number << std::endl;
    }
};

class Laptop : public Device {
    int ramGB;
    
public:
    Laptop(const char* n, int ram)
        : Device(n), ramGB(ram) {}
    
    void status() const {
        Device::status();
        std::cout << "  RAM: " << ramGB << " GB" << std::endl;
    }
};

int main() {
    Phone iphone("iPhone 15", 87);
    Laptop macbook("MacBook Pro", 16);
    
    iphone.turnOn();
    iphone.status();
    iphone.call("0888123456");
    
    macbook.status();  // Изключен
    
    return 0;
}
```

---

## Пример 3: Диамантеният проблем

```cpp
#include <iostream>

class Animal {
public:
    int legs;
    Animal(int l) : legs(l) {
        std::cout << "Animal(" << l << ")" << std::endl;
    }
};

// БЕЗ virtual — D ще има ДВЕ копия на Animal
// class FlyingAnimal : public Animal { ... };
// class SwimmingAnimal : public Animal { ... };

// С virtual — D ще има ЕДНО копие на Animal
class FlyingAnimal : virtual public Animal {
public:
    FlyingAnimal(int l) : Animal(l) {
        std::cout << "FlyingAnimal" << std::endl;
    }
};

class SwimmingAnimal : virtual public Animal {
public:
    SwimmingAnimal(int l) : Animal(l) {
        std::cout << "SwimmingAnimal" << std::endl;
    }
};

class Duck : public FlyingAnimal, public SwimmingAnimal {
public:
    // При virtual наследяване, най-производният клас
    // инициализира virtual базата!
    Duck() : Animal(2), FlyingAnimal(2), SwimmingAnimal(2) {
        std::cout << "Duck" << std::endl;
    }
};

int main() {
    Duck d;
    std::cout << "Крака: " << d.legs << std::endl;  // 2 (еднозначно!)
    return 0;
}
```

> 💡 При `virtual` наследяване, **най-производният клас** (Duck) е отговорен за инициализацията на виртуалната база (Animal).
