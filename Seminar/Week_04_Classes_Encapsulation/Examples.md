# Седмица 4 — Примери

## Пример 1: Клас `BankAccount` с валидация

```cpp
#include <iostream>
#include <cstring>

class BankAccount {
    char owner[100];
    double balance;
    
    // Private helper — не е част от публичния интерфейс
    bool isValidAmount(double amount) const {
        return amount > 0;
    }
    
public:
    // Конструктор (ще разгледаме детайлно следващата седмица)
    BankAccount(const char* ownerName, double initialBalance) {
        std::strncpy(owner, ownerName, 99);
        owner[99] = '\0';
        balance = (initialBalance > 0) ? initialBalance : 0;
    }
    
    bool deposit(double amount) {
        if (!isValidAmount(amount)) return false;
        balance += amount;
        return true;
    }
    
    bool withdraw(double amount) {
        if (!isValidAmount(amount)) return false;
        if (amount > balance) return false;
        balance -= amount;
        return true;
    }
    
    // Const методи — не променят обекта
    const char* getOwner() const { return owner; }
    double getBalance() const { return balance; }
    
    void print() const {
        std::cout << "Сметка на " << owner 
                  << ", баланс: " << balance << " лв." << std::endl;
    }
};

int main() {
    BankAccount acc("Иван Петров", 1000.0);
    acc.print();                    // Сметка на Иван Петров, баланс: 1000 лв.
    
    acc.deposit(500);
    acc.print();                    // баланс: 1500 лв.
    
    acc.withdraw(2000);             // Неуспешно (недостатъчен баланс)
    acc.print();                    // баланс: 1500 лв. (непроменен)
    
    acc.deposit(-100);              // Неуспешно (невалидна сума)
    acc.print();                    // баланс: 1500 лв. (непроменен)
    
    return 0;
}
```

> 💡 **Обърнете внимание:** `isValidAmount` е `private` — вътрешен helper. Потребителят не трябва да знае за него.

---

## Пример 2: `Student` с private данни и публичен интерфейс

```cpp
#include <iostream>
#include <cstring>

class Student {
    char name[100];
    int facultyNumber;
    double grades[20];
    int gradeCount;
    
public:
    Student(const char* n, int fn) : facultyNumber(fn), gradeCount(0) {
        std::strncpy(name, n, 99);
        name[99] = '\0';
    }
    
    bool addGrade(double grade) {
        if (grade < 2.0 || grade > 6.0) return false;  // Валидация
        if (gradeCount >= 20) return false;             // Капацитет
        grades[gradeCount++] = grade;
        return true;
    }
    
    double averageGrade() const {
        if (gradeCount == 0) return 0;
        double sum = 0;
        for (int i = 0; i < gradeCount; i++) {
            sum += grades[i];
        }
        return sum / gradeCount;
    }
    
    const char* getName() const { return name; }
    int getFacultyNumber() const { return facultyNumber; }
    int getGradeCount() const { return gradeCount; }
    
    void printReport() const {
        std::cout << "Студент: " << name << " (ФН: " << facultyNumber << ")" << std::endl;
        std::cout << "Оценки (" << gradeCount << "): ";
        for (int i = 0; i < gradeCount; i++) {
            std::cout << grades[i] << " ";
        }
        std::cout << std::endl;
        std::cout << "Среден успех: " << averageGrade() << std::endl;
    }
};

int main() {
    Student s("Мария Георгиева", 62300);
    
    s.addGrade(5.50);
    s.addGrade(6.00);
    s.addGrade(4.75);
    s.addGrade(1.00);   // Невалидна — няма да се добави
    
    s.printReport();
    // Студент: Мария Георгиева (ФН: 62300)
    // Оценки (3): 5.5 6 4.75
    // Среден успех: 5.41667
    
    return 0;
}
```

---

## Пример 3: Method chaining с `this`

```cpp
#include <iostream>

class QueryBuilder {
    char table[50];
    char condition[200];
    int limitVal;
    
public:
    QueryBuilder() : limitVal(-1) {
        table[0] = '\0';
        condition[0] = '\0';
    }
    
    QueryBuilder& from(const char* t) {
        std::strncpy(table, t, 49);
        table[49] = '\0';
        return *this;
    }
    
    QueryBuilder& where(const char* cond) {
        std::strncpy(condition, cond, 199);
        condition[199] = '\0';
        return *this;
    }
    
    QueryBuilder& limit(int n) {
        limitVal = n;
        return *this;
    }
    
    void execute() const {
        std::cout << "SELECT * FROM " << table;
        if (condition[0] != '\0') {
            std::cout << " WHERE " << cond;
        }
        if (limitVal > 0) {
            std::cout << " LIMIT " << limitVal;
        }
        std::cout << ";" << std::endl;
    }
};

int main() {
    QueryBuilder()
        .from("students")
        .where("grade > 5.0")
        .limit(10)
        .execute();
    // SELECT * FROM students WHERE grade > 5.0 LIMIT 10;
    
    return 0;
}
```

> 💡 Method chaining е възможно, защото всеки метод връща `*this` — референция към обекта. Използва се много в builder шаблона.
