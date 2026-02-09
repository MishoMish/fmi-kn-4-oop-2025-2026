# Наследяване

## Въведение

Наследяването е механизъм, чрез който нов клас (наследник/derived) се създава на базата на съществуващ (базов/base), **наследявайки** неговите членове и добавяйки нови. Това моделира **„is-a" релация**: „Кучето **е** Животно", „Студентът **е** Човек".

---

## 1. Идеята за наследяване

### Проблемът: дублиране на код

```cpp
class Student {
    char name[100]; int age; double grade; int fn;
    // ... методи за name, age ...
};
class Teacher {
    char name[100]; int age; double salary; char subject[50];
    // ... СЪЩИТЕ методи за name, age, ПЛЮС нови ...
};
```

### Решението: обща база

```cpp
class Person {
protected:
    char name[100];
    int age;
public:
    Person(const char* n, int a);
    const char* getName() const;
    int getAge() const;
};

class Student : public Person {
    double grade;
    int fn;
public:
    Student(const char* n, int a, double g, int f);
    double getGrade() const;
};

class Teacher : public Person {
    double salary;
    char subject[50];
public:
    Teacher(const char* n, int a, double s, const char* subj);
};
```

---

## 2. `is-a` vs. `has-a`

| Въпрос | Отговор | Техника |
|--------|---------|---------|
| „X е вид от Y?" | Да → is-a | Наследяване |
| „X съдържа Y?" | Да → has-a | Композиция |

**Примери:**
- Куче **is-a** Животно ✅ → наследяване
- Кола **has-a** Двигател ✅ → композиция
- Кола **is-a** Двигател ❌ → **не** наследяване!

> 🧠 **Дискусия:** Квадратът **is-a** Правоъгълник? Математически да, но в ООП може да създаде проблеми (нарушаване на LSP). Ще обсъдим в Седмица 13.

---

## 3. Синтаксис на наследяване

```cpp
class Base {
protected:
    int x;
public:
    void doSomething() { std::cout << "Base" << std::endl; }
};

class Derived : public Base {
    int y;
public:
    void doMore() {
        x = 10;        // ✅ protected е достъпен в наследника
        doSomething();  // ✅ public методи също
    }
};
```

---

## 4. Видове наследяване и достъп

| Член в Base | `public` наследяване | `protected` наследяване | `private` наследяване |
|-------------|---------------------|------------------------|----------------------|
| `public` | остава `public` | става `protected` | става `private` |
| `protected` | остава `protected` | остава `protected` | става `private` |
| `private` | ❌ недостъпен | ❌ недостъпен | ❌ недостъпен |

> 💡 В практиката почти винаги се използва **`public`** наследяване. Останалите са рядкост.

---

## 5. Жизнен цикъл при наследяване

### Конструиране

При създаване на `Derived` обект:
1. **Първо** се извиква конструкторът на `Base`
2. **После** конструкторът на `Derived`

```cpp
class Base {
public:
    Base() { std::cout << "Base()" << std::endl; }
    Base(int x) { std::cout << "Base(" << x << ")" << std::endl; }
    ~Base() { std::cout << "~Base()" << std::endl; }
};

class Derived : public Base {
public:
    // Извикваме конкретен конструктор на Base чрез initializer list
    Derived(int x) : Base(x) {
        std::cout << "Derived(" << x << ")" << std::endl;
    }
    ~Derived() { std::cout << "~Derived()" << std::endl; }
};

Derived d(42);
// Base(42)
// Derived(42)
// ~Derived()
// ~Base()
```

### Унищожаване

Обратен ред: **първо** деструкторът на `Derived`, **после** на `Base`.

---

## 6. Предефиниране (Overriding)

Наследникът може да **предефинира** метод на базовия клас:

```cpp
class Animal {
public:
    void speak() { std::cout << "..." << std::endl; }
};

class Dog : public Animal {
public:
    void speak() { std::cout << "Бау!" << std::endl; }
};

class Cat : public Animal {
public:
    void speak() { std::cout << "Мяу!" << std::endl; }
};

Dog d;
d.speak();  // "Бау!"

// НО:
Animal* ptr = &d;
ptr->speak();  // "..." ← Извиква Animal::speak()! Не Dog::speak()!
```

> ⚠️ Без `virtual`, методът се избира по **типа на указателя/референцията**, не по **типа на обекта**. Това е статично свързване (static binding). Полиморфизмът (Седмица 10) решава този проблем.

---

## 7. Скриване на имена (Name Hiding)

Ако наследникът дефинира метод със **същото име** (дори с различни параметри), той **скрива** всички версии от базовия клас:

```cpp
class Base {
public:
    void func(int x) { std::cout << "Base::func(int)" << std::endl; }
    void func(double x) { std::cout << "Base::func(double)" << std::endl; }
};

class Derived : public Base {
public:
    void func(int x) { std::cout << "Derived::func(int)" << std::endl; }
    // Base::func(double) е СКРИТ!
};

Derived d;
d.func(42);    // Derived::func(int)
// d.func(3.14);  // Грешка или неочаквано — Base::func(double) е скрит

// Решение: using
class DerivedFixed : public Base {
public:
    using Base::func;  // „Разкриваме" Base::func
    void func(int x) { std::cout << "DerivedFixed::func(int)" << std::endl; }
};
```

---

## 8. Диамантеният проблем

При множествено наследяване може да възникне **двойно наследяване** на база:

```cpp
class A { public: int value; };
class B : public A {};
class C : public A {};
class D : public B, public C {};  // D има ДВЕ копия на A::value!

D d;
// d.value = 5;  // ❌ Двусмислено! B::value или C::value?
d.B::value = 5;  // Явно уточняване
```

### Решение: `virtual` наследяване

```cpp
class A { public: int value; };
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};  // D има ЕДНО копие на A::value

D d;
d.value = 5;  // ✅ Еднозначно
```

---

## Обобщение

| Концепция | Описание |
|-----------|----------|
| Наследяване | Клас получава членовете на друг клас |
| `is-a` | Релация за наследяване |
| `public` наследяване | Запазва достъпа (най-често) |
| Initializer list | Избор на базов конструктор |
| Overriding | Предефиниране на метод в наследника |
| Name hiding | Наследник скрива едноименни методи от базата |
| Диамантен проблем | Множествено наследяване → дубликати; решение: `virtual` |

> 🧠 **Следващата седмица:** Полиморфизъм — как `virtual` функции позволяват динамично избиране на метод по време на изпълнение.
