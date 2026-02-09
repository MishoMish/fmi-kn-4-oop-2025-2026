# Класове и капсулация

## Въведение

Класовете са сърцето на обектно-ориентираното програмиране. Те обединяват **данни** и **поведение** в една единица, позволявайки ни да моделираме реалния свят чрез абстракции. Тази седмица се фокусираме върху основните концепции: дефиниране на класове, модификатори за достъп, капсулация и указателят `this`.

---

## 1. Какво е клас?

Класът е **потребителски дефиниран тип**, който групира:
- **Полета (data members)** — данните, които обектът притежава
- **Методи (member functions)** — операциите, които обектът може да извършва

```cpp
class Rectangle {
    double width;   // поле
    double height;  // поле
    
public:
    double area() { return width * height; }      // метод
    double perimeter() { return 2 * (width + height); }  // метод
};
```

### Обект = Инстанция на клас

```cpp
Rectangle r;       // r е обект от тип Rectangle
r.area();          // Извикваме метод на обекта
```

> 🧠 **Аналогия:** Класът е *чертеж*, обектът е *постройката* по този чертеж. Може да имаме много обекти от един клас, всеки с различни стойности на полетата.

---

## 2. Модификатори за достъп

C++ предлага три нива на достъп:

| Модификатор | Достъп |
|-------------|--------|
| `public` | Достъпен от навсякъде |
| `private` | Достъпен само от методите на класа |
| `protected` | Достъпен от класа и неговите наследници |

```cpp
class Person {
private:          // Скрити данни
    char name[100];
    int age;
    
protected:        // Достъпно за наследници
    char id[20];
    
public:           // Публичен интерфейс
    void setName(const char* n);
    const char* getName() const;
    void setAge(int a);
    int getAge() const;
};
```

### Подразбиращ се достъп

- `class` → `private` по подразбиране
- `struct` → `public` по подразбиране

---

## 3. Капсулация (Encapsulation)

### Идеята

Капсулацията означава **скриване на вътрешното представяне** на обекта и предоставяне на контролиран достъп чрез публичен интерфейс.

### Защо е важна?

```cpp
// ❌ БЕЗ капсулация — директен достъп
struct BankAccount {
    double balance;
};

BankAccount acc;
acc.balance = -1000;  // Невалидно, но никой не спира!
```

```cpp
// ✅ С капсулация — контролиран достъп
class BankAccount {
    double balance;
    
public:
    BankAccount() : balance(0) {}
    
    bool deposit(double amount) {
        if (amount <= 0) return false;  // Валидация!
        balance += amount;
        return true;
    }
    
    bool withdraw(double amount) {
        if (amount <= 0 || amount > balance) return false;
        balance -= amount;
        return true;
    }
    
    double getBalance() const { return balance; }
};
```

### Принципи на капсулацията

1. **Полетата са `private`** — не се достъпват директно отвън
2. **Публичният интерфейс** е стабилен — вътрешната имплементация може да се промени
3. **Валидация** — методите гарантират, че данните са винаги валидни (**инварианти**)
4. **Getter/Setter** — контролиран достъп, когато е нужен

> 🧠 **Дискусия:** „Просто сложи getter и setter за всяко поле" е ли добра капсулация? Какво е разликата между `getBalance()` и `balance` като `public`?

### Инварианти на класа

**Инвариант** е условие, което *винаги* трябва да е вярно за обекта:

```cpp
class Circle {
    double radius;  // Инвариант: radius > 0
    
public:
    Circle(double r) {
        if (r <= 0) throw std::invalid_argument("Radius must be positive");
        radius = r;
    }
    
    void setRadius(double r) {
        if (r <= 0) throw std::invalid_argument("Radius must be positive");
        radius = r;
    }
    
    double getRadius() const { return radius; }
    double area() const { return 3.14159 * radius * radius; }
};
```

---

## 4. Указателят `this`

Всеки метод има имплицитен параметър `this` — указател към обекта, върху който е извикан:

```cpp
class Counter {
    int value;
    
public:
    Counter() : value(0) {}
    
    // this е имплицитен — не го пишем в параметрите
    void increment() {
        this->value++;  // Еквивалентно на: value++;
    }
    
    // Полезен при конфликт на имена
    void setValue(int value) {
        this->value = value;  // this->value е полето, value е параметърът
    }
    
    // Позволява method chaining
    Counter& add(int n) {
        value += n;
        return *this;  // Връщаме самия обект
    }
    
    int getValue() const { return value; }
};

int main() {
    Counter c;
    c.add(5).add(3).add(2);  // Method chaining!
    std::cout << c.getValue();  // 10
}
```

> 💡 `this` е от тип `ClassName*` (или `const ClassName*` в `const` метод).

---

## 5. `const` методи

Метод, маркиран с `const`, обещава да **не променя** обекта:

```cpp
class Student {
    char name[100];
    double grade;
    
public:
    // const метод — може да се извика на const обект
    const char* getName() const { return name; }
    double getGrade() const { return grade; }
    
    // Не-const метод — променя обекта
    void setGrade(double g) { grade = g; }
};

void printStudent(const Student& s) {
    std::cout << s.getName();    // ✅ getName() е const
    // s.setGrade(6.0);          // ❌ setGrade() не е const
}
```

### Правило

- Маркирайте като `const` **всеки метод, който не променя обекта**
- `const` обект може да извика **само `const` методи**
- Компилаторът проверява това по време на компилация

---

## Обобщение

| Концепция | Описание | Ключова дума |
|-----------|----------|-------------|
| Клас | Тип с данни + поведение | `class` |
| Капсулация | Скриване на данни, контролиран достъп | `private`, `public` |
| Инвариант | Условие, винаги вярно за обекта | Валидация в setter/constructor |
| `this` | Указател към текущия обект | Имплицитен параметър |
| `const` метод | Не променя обекта | `const` след параметрите |
| Method chaining | Верижно извикване на методи | `return *this;` |

Класовете са фундаментът на ООП. В следващите седмици ще разгледаме как обектите се създават, копират и унищожават (Голямата четворка).
