# Седмица 11 — Примери

## Пример 1: Factory за фигури с потребителски вход

```cpp
#include <iostream>
#include <cstring>
#include <cmath>

class Shape {
public:
    virtual double area() const = 0;
    virtual void print() const = 0;
    virtual Shape* clone() const = 0;
    virtual ~Shape() {}
};

class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area() const override { return M_PI * r * r; }
    void print() const override { std::cout << "Circle(r=" << r << ")" << std::endl; }
    Shape* clone() const override { return new Circle(*this); }
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double w, double h) : w(w), h(h) {}
    double area() const override { return w * h; }
    void print() const override { std::cout << "Rect(" << w << "x" << h << ")" << std::endl; }
    Shape* clone() const override { return new Rectangle(*this); }
};

// Factory
Shape* createShape() {
    char type[20];
    std::cout << "Тип (circle/rect): ";
    std::cin >> type;
    
    if (std::strcmp(type, "circle") == 0) {
        double r;
        std::cout << "Радиус: ";
        std::cin >> r;
        return new Circle(r);
    }
    if (std::strcmp(type, "rect") == 0) {
        double w, h;
        std::cout << "Ширина и височина: ";
        std::cin >> w >> h;
        return new Rectangle(w, h);
    }
    return nullptr;
}

int main() {
    const int MAX = 10;
    Shape* shapes[MAX];
    int count = 0;
    
    char choice;
    do {
        Shape* s = createShape();
        if (s && count < MAX) {
            shapes[count++] = s;
        }
        std::cout << "Добави още? (y/n): ";
        std::cin >> choice;
    } while (choice == 'y');
    
    std::cout << "\n=== Фигури ===" << std::endl;
    double totalArea = 0;
    for (int i = 0; i < count; i++) {
        shapes[i]->print();
        totalArea += shapes[i]->area();
    }
    std::cout << "Общо лице: " << totalArea << std::endl;
    
    for (int i = 0; i < count; i++) delete shapes[i];
    return 0;
}
```

---

## Пример 2: Strategy за филтриране

```cpp
#include <iostream>

class FilterStrategy {
public:
    virtual bool shouldKeep(int value) const = 0;
    virtual ~FilterStrategy() {}
};

class KeepEven : public FilterStrategy {
public:
    bool shouldKeep(int value) const override { return value % 2 == 0; }
};

class KeepPositive : public FilterStrategy {
public:
    bool shouldKeep(int value) const override { return value > 0; }
};

class KeepGreaterThan : public FilterStrategy {
    int threshold;
public:
    KeepGreaterThan(int t) : threshold(t) {}
    bool shouldKeep(int value) const override { return value > threshold; }
};

class IntArray {
    int data[100];
    int size;
    
public:
    IntArray() : size(0) {}
    
    void add(int val) { if (size < 100) data[size++] = val; }
    
    IntArray filter(const FilterStrategy& strategy) const {
        IntArray result;
        for (int i = 0; i < size; i++) {
            if (strategy.shouldKeep(data[i])) {
                result.add(data[i]);
            }
        }
        return result;
    }
    
    void print() const {
        std::cout << "[";
        for (int i = 0; i < size; i++) {
            if (i > 0) std::cout << ", ";
            std::cout << data[i];
        }
        std::cout << "]" << std::endl;
    }
};

int main() {
    IntArray arr;
    arr.add(-3); arr.add(7); arr.add(4); arr.add(-1); arr.add(10); arr.add(2);
    
    std::cout << "Оригинал: ";
    arr.print();
    
    KeepEven evenFilter;
    std::cout << "Четни:    ";
    arr.filter(evenFilter).print();
    
    KeepPositive posFilter;
    std::cout << "Положителни: ";
    arr.filter(posFilter).print();
    
    KeepGreaterThan gt5(5);
    std::cout << "По-големи от 5: ";
    arr.filter(gt5).print();
    
    return 0;
}
```

---

## Пример 3: Observer — Temperature Monitor

```cpp
#include <iostream>

class TemperatureObserver {
public:
    virtual void onTemperatureChange(double temp) = 0;
    virtual ~TemperatureObserver() {}
};

class TemperatureSensor {
    TemperatureObserver* observers[10];
    int count = 0;
    double temperature = 20.0;
    
public:
    void attach(TemperatureObserver* obs) {
        if (count < 10) observers[count++] = obs;
    }
    
    void setTemperature(double t) {
        temperature = t;
        for (int i = 0; i < count; i++) {
            observers[i]->onTemperatureChange(temperature);
        }
    }
};

class DisplayUnit : public TemperatureObserver {
    char name[50];
public:
    DisplayUnit(const char* n) { std::strncpy(name, n, 49); name[49] = '\0'; }
    void onTemperatureChange(double temp) override {
        std::cout << "[" << name << "] Температура: " << temp << "°C" << std::endl;
    }
};

class AlarmUnit : public TemperatureObserver {
    double threshold;
public:
    AlarmUnit(double t) : threshold(t) {}
    void onTemperatureChange(double temp) override {
        if (temp > threshold) {
            std::cout << "🚨 АЛАРМА! Температурата (" << temp 
                      << "°C) надвишава прага (" << threshold << "°C)!" << std::endl;
        }
    }
};

int main() {
    TemperatureSensor sensor;
    
    DisplayUnit display("Дисплей 1");
    DisplayUnit display2("Дисплей 2");
    AlarmUnit alarm(30.0);
    
    sensor.attach(&display);
    sensor.attach(&display2);
    sensor.attach(&alarm);
    
    std::cout << "--- Задаваме 25°C ---" << std::endl;
    sensor.setTemperature(25.0);
    
    std::cout << "\n--- Задаваме 35°C ---" << std::endl;
    sensor.setTemperature(35.0);
    
    return 0;
}
```
