# Седмица 10 — Примери

## Пример 1: `Shape` абстрактен клас с хетерогенна колекция

```cpp
#include <iostream>
#include <cmath>

class Shape {
public:
    virtual double area() const = 0;
    virtual double perimeter() const = 0;
    virtual void print() const = 0;
    virtual Shape* clone() const = 0;
    virtual ~Shape() {}
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    
    double area() const override { return M_PI * radius * radius; }
    double perimeter() const override { return 2 * M_PI * radius; }
    void print() const override {
        std::cout << "Кръг (r=" << radius << "), лице=" << area() << std::endl;
    }
    Shape* clone() const override { return new Circle(*this); }
};

class Rectangle : public Shape {
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    
    double area() const override { return width * height; }
    double perimeter() const override { return 2 * (width + height); }
    void print() const override {
        std::cout << "Правоъгълник (" << width << "x" << height 
                  << "), лице=" << area() << std::endl;
    }
    Shape* clone() const override { return new Rectangle(*this); }
};

class Triangle : public Shape {
    double a, b, c;
public:
    Triangle(double a, double b, double c) : a(a), b(b), c(c) {}
    
    double area() const override {
        double s = (a + b + c) / 2;
        return std::sqrt(s * (s-a) * (s-b) * (s-c));
    }
    double perimeter() const override { return a + b + c; }
    void print() const override {
        std::cout << "Триъгълник (" << a << ", " << b << ", " << c 
                  << "), лице=" << area() << std::endl;
    }
    Shape* clone() const override { return new Triangle(*this); }
};

int main() {
    const int N = 4;
    Shape* shapes[N];
    shapes[0] = new Circle(5);
    shapes[1] = new Rectangle(3, 4);
    shapes[2] = new Triangle(3, 4, 5);
    shapes[3] = new Circle(2);
    
    // Полиморфно отпечатване
    std::cout << "=== Всички фигури ===" << std::endl;
    for (int i = 0; i < N; i++) {
        shapes[i]->print();
    }
    
    // Намиране на максимално лице
    double maxArea = 0;
    int maxIdx = 0;
    for (int i = 0; i < N; i++) {
        if (shapes[i]->area() > maxArea) {
            maxArea = shapes[i]->area();
            maxIdx = i;
        }
    }
    std::cout << "\nНай-голямо лице: ";
    shapes[maxIdx]->print();
    
    // Дълбоко копиране с clone()
    Shape* copy = shapes[0]->clone();
    std::cout << "\nКопие: ";
    copy->print();
    
    // Почистване
    delete copy;
    for (int i = 0; i < N; i++) delete shapes[i];
    
    return 0;
}
```

---

## Пример 2: Employee йерархия с виртуален `getSalary()`

```cpp
#include <iostream>
#include <cstring>

class Employee {
protected:
    char name[100];
    
public:
    Employee(const char* n) {
        std::strncpy(name, n, 99);
        name[99] = '\0';
    }
    
    virtual double getSalary() const = 0;
    virtual void describe() const = 0;
    virtual Employee* clone() const = 0;
    virtual ~Employee() {}
};

class Developer : public Employee {
    double baseSalary;
    int experienceYears;
    
public:
    Developer(const char* n, double base, int exp)
        : Employee(n), baseSalary(base), experienceYears(exp) {}
    
    double getSalary() const override {
        return baseSalary * (1 + 0.05 * experienceYears);
    }
    
    void describe() const override {
        std::cout << "Developer: " << name << ", заплата: " << getSalary() << std::endl;
    }
    
    Employee* clone() const override { return new Developer(*this); }
};

class Manager : public Employee {
    double baseSalary;
    int teamSize;
    
public:
    Manager(const char* n, double base, int team)
        : Employee(n), baseSalary(base), teamSize(team) {}
    
    double getSalary() const override {
        return baseSalary + 200 * teamSize;  // Бонус за всеки подчинен
    }
    
    void describe() const override {
        std::cout << "Manager: " << name << " (екип: " << teamSize 
                  << "), заплата: " << getSalary() << std::endl;
    }
    
    Employee* clone() const override { return new Manager(*this); }
};

double totalPayroll(Employee* employees[], int count) {
    double total = 0;
    for (int i = 0; i < count; i++) {
        total += employees[i]->getSalary();
    }
    return total;
}

int main() {
    Employee* team[3];
    team[0] = new Developer("Иван", 3000, 5);
    team[1] = new Developer("Мария", 3500, 3);
    team[2] = new Manager("Петър", 4000, 2);
    
    for (int i = 0; i < 3; i++) team[i]->describe();
    
    std::cout << "\nОбщ фонд: " << totalPayroll(team, 3) << std::endl;
    
    for (int i = 0; i < 3; i++) delete team[i];
    return 0;
}
```

> 💡 `totalPayroll` не знае **нищо** за `Developer` и `Manager` — работи само с `Employee*`. Ако утре добавим `Intern`, функцията не се променя.
