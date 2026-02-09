# Седмица 13 — Примери

## Пример 1: SRP — Разделяне на отговорности

```cpp
#include <iostream>
#include <fstream>
#include <cstring>

// Модел — само данни и бизнес логика
class Student {
    char name[100];
    double grades[10];
    int gradeCount;
    
public:
    Student(const char* n) : gradeCount(0) {
        std::strncpy(name, n, 99);
        name[99] = '\0';
    }
    
    void addGrade(double g) {
        if (gradeCount < 10) grades[gradeCount++] = g;
    }
    
    const char* getName() const { return name; }
    
    double average() const {
        if (gradeCount == 0) return 0;
        double sum = 0;
        for (int i = 0; i < gradeCount; i++) sum += grades[i];
        return sum / gradeCount;
    }
    
    int getGradeCount() const { return gradeCount; }
    double getGrade(int i) const { return (i >= 0 && i < gradeCount) ? grades[i] : 0; }
};

// Persistence — само запис/четене
class StudentRepository {
public:
    void saveToFile(const Student& s, const char* filename) const {
        std::ofstream f(filename);
        f << s.getName() << std::endl;
        f << s.getGradeCount() << std::endl;
        for (int i = 0; i < s.getGradeCount(); i++) {
            f << s.getGrade(i) << " ";
        }
        f << std::endl;
    }
};

// Presentation — само визуализация
class StudentPrinter {
public:
    void printSummary(const Student& s) const {
        std::cout << "Студент: " << s.getName() << std::endl;
        std::cout << "Среден успех: " << s.average() << std::endl;
        std::cout << "Оценки: ";
        for (int i = 0; i < s.getGradeCount(); i++) {
            std::cout << s.getGrade(i) << " ";
        }
        std::cout << std::endl;
    }
};

int main() {
    Student s("Иван Иванов");
    s.addGrade(5.5);
    s.addGrade(6.0);
    s.addGrade(4.5);
    
    StudentPrinter printer;
    printer.printSummary(s);
    
    StudentRepository repo;
    repo.saveToFile(s, "student.txt");
    std::cout << "Записан във файл." << std::endl;
    
    return 0;
}
```

---

## Пример 2: OCP + Strategy — Разширяемо сортиране

```cpp
#include <iostream>

// Абстракция — критерий за сравнение
class Comparator {
public:
    virtual bool compare(int a, int b) const = 0;
    virtual ~Comparator() {}
};

class AscendingComparator : public Comparator {
public:
    bool compare(int a, int b) const override { return a > b; }
};

class DescendingComparator : public Comparator {
public:
    bool compare(int a, int b) const override { return a < b; }
};

class AbsComparator : public Comparator {
    int abs(int x) const { return x < 0 ? -x : x; }
public:
    bool compare(int a, int b) const override { return abs(a) > abs(b); }
};

// Контекст — НЕ се променя при добавяне на нов Comparator (OCP)
void bubbleSort(int arr[], int n, const Comparator& cmp) {
    for (int i = 0; i < n - 1; i++)
        for (int j = 0; j < n - i - 1; j++)
            if (cmp.compare(arr[j], arr[j+1]))
                std::swap(arr[j], arr[j+1]);
}

void print(int arr[], int n) {
    for (int i = 0; i < n; i++) std::cout << arr[i] << " ";
    std::cout << std::endl;
}

int main() {
    int arr[] = {-3, 7, -1, 4, -8, 2};
    int n = 6;
    int copy1[6], copy2[6], copy3[6];
    for (int i = 0; i < n; i++) copy1[i] = copy2[i] = copy3[i] = arr[i];
    
    AscendingComparator asc;
    bubbleSort(copy1, n, asc);
    std::cout << "Ascending: "; print(copy1, n);
    
    DescendingComparator desc;
    bubbleSort(copy2, n, desc);
    std::cout << "Descending: "; print(copy2, n);
    
    AbsComparator absCmp;
    bubbleSort(copy3, n, absCmp);
    std::cout << "By |value|: "; print(copy3, n);
    
    return 0;
}
```

---

## Пример 3: DIP — Notification Service

```cpp
#include <iostream>
#include <cstring>

// Абстракция
class INotifier {
public:
    virtual void send(const char* to, const char* message) = 0;
    virtual ~INotifier() {}
};

// Конкретни имплементации
class EmailNotifier : public INotifier {
public:
    void send(const char* to, const char* message) override {
        std::cout << "[Email → " << to << "] " << message << std::endl;
    }
};

class SMSNotifier : public INotifier {
public:
    void send(const char* to, const char* message) override {
        std::cout << "[SMS → " << to << "] " << message << std::endl;
    }
};

class ConsoleNotifier : public INotifier {
public:
    void send(const char* to, const char* message) override {
        std::cout << "[Console] За " << to << ": " << message << std::endl;
    }
};

// Високо ниво — зависи САМО от абстракцията INotifier
class OrderService {
    INotifier* notifier;
    
public:
    OrderService(INotifier* n) : notifier(n) {}
    
    void placeOrder(const char* customerEmail) {
        std::cout << "Поръчката е създадена." << std::endl;
        notifier->send(customerEmail, "Поръчката ви е приета!");
    }
};

int main() {
    // Лесно заменяме имплементацията
    EmailNotifier email;
    OrderService service1(&email);
    service1.placeOrder("ivan@mail.bg");
    
    std::cout << std::endl;
    
    SMSNotifier sms;
    OrderService service2(&sms);
    service2.placeOrder("+359888123456");
    
    return 0;
}
```

> 💡 `OrderService` **не знае** дали изпраща email или SMS. Може утре да добавим `PushNotifier` без да променим `OrderService`.
