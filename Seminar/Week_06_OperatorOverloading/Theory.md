# Предефиниране на оператори (Operator Overloading)

## Въведение

Предефинирането на оператори позволява да дадем **смисъл на операторите** за наши собствени типове. Вместо `a.add(b)`, можем да пишем `a + b` — по-четимо и интуитивно.

> 💡 **Подобрение спрямо 2024–2025:** Миналата година операторите бяха обединени с Big Four. Тази година ги разглеждаме отделно, с по-голям фокус върху конвенциите и добрите практики.

---

## 1. Основни правила

### Какво може да се предефинира?

Почти всички оператори: `+`, `-`, `*`, `/`, `%`, `==`, `!=`, `<`, `>`, `<=`, `>=`, `<<`, `>>`, `[]`, `()`, `++`, `--`, `=`, `+=`, `-=` и др.

### Какво **НЕ** може?

- `::` (scope resolution)
- `.` (member access)
- `.*` (pointer-to-member)
- `?:` (ternary)
- `sizeof`
- `typeid`

### Ключово правило

> Предефинирайте оператор само ако значението му е **интуитивно** за потребителя на класа. Ако `+` не означава „събиране" в естествен смисъл, не го предефинирайте.

---

## 2. Аритметични оператори

### Като член-функция

```cpp
class Complex {
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // operator+ като член-функция
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
    
    Complex operator-(const Complex& other) const {
        return Complex(real - other.real, imag - other.imag);
    }
    
    Complex operator*(const Complex& other) const {
        return Complex(
            real * other.real - imag * other.imag,
            real * other.imag + imag * other.real
        );
    }
};
```

### Конвенция: `+=` и `+`

```cpp
class Complex {
    // ...
    
    // operator+= модифицира обекта
    Complex& operator+=(const Complex& other) {
        real += other.real;
        imag += other.imag;
        return *this;
    }
    
    // operator+ използва operator+=
    Complex operator+(const Complex& other) const {
        Complex result(*this);   // Копие
        result += other;         // Използваме +=
        return result;
    }
};
```

> 💡 **Best practice:** Имплементирайте `+=` първо, после `+` чрез `+=`. Избягва дублиране на код.

---

## 3. Оператори за сравнение

```cpp
class Complex {
    // ...
    
    bool operator==(const Complex& other) const {
        return real == other.real && imag == other.imag;
    }
    
    bool operator!=(const Complex& other) const {
        return !(*this == other);  // Чрез operator==
    }
};
```

### За подреждаеми типове

```cpp
class Date {
    int year, month, day;
    
public:
    Date(int y, int m, int d) : year(y), month(m), day(d) {}
    
    bool operator<(const Date& other) const {
        if (year != other.year) return year < other.year;
        if (month != other.month) return month < other.month;
        return day < other.day;
    }
    
    bool operator>(const Date& other) const { return other < *this; }
    bool operator<=(const Date& other) const { return !(other < *this); }
    bool operator>=(const Date& other) const { return !(*this < other); }
    bool operator==(const Date& other) const { return !(*this < other) && !(other < *this); }
    bool operator!=(const Date& other) const { return !(*this == other); }
};
```

> 💡 **Best practice:** Имплементирайте `operator<` и `operator==`, после изразете останалите чрез тях.

---

## 4. Оператори за поток (`<<` и `>>`)

### Защо `friend`?

При `std::cout << obj`, левият операнд е `std::ostream`, не нашият клас. Затова операторът **не може** да бъде член-функция на нашия клас — трябва да е свободна функция.

```cpp
class Complex {
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // friend има достъп до private членовете
    friend std::ostream& operator<<(std::ostream& os, const Complex& c);
    friend std::istream& operator>>(std::istream& is, Complex& c);
};

std::ostream& operator<<(std::ostream& os, const Complex& c) {
    os << c.real;
    if (c.imag >= 0) os << "+";
    os << c.imag << "i";
    return os;
}

std::istream& operator>>(std::istream& is, Complex& c) {
    is >> c.real >> c.imag;
    return is;
}

// Използване:
Complex z(3, -4);
std::cout << z << std::endl;  // 3-4i
```

> ⚠️ Операторите `<<` и `>>` връщат `ostream&` / `istream&`, за да поддържат **chaining**: `std::cout << a << " " << b;`

---

## 5. Оператор за индексиране (`operator[]`)

```cpp
class IntArray {
    int* data;
    int size;
    
public:
    IntArray(int n) : size(n) { data = new int[n](); }
    ~IntArray() { delete[] data; }
    
    // Non-const версия — за четене и писане
    int& operator[](int index) {
        return data[index];
    }
    
    // Const версия — само за четене
    const int& operator[](int index) const {
        return data[index];
    }
};

IntArray arr(5);
arr[0] = 42;                 // Извиква non-const operator[]
const IntArray& ref = arr;
std::cout << ref[0];         // Извиква const operator[]
```

---

## 6. Инкремент / Декремент (`++`, `--`)

```cpp
class Counter {
    int value;
    
public:
    Counter(int v = 0) : value(v) {}
    
    // Prefix ++c (връща СЛЕД инкремента)
    Counter& operator++() {
        ++value;
        return *this;
    }
    
    // Postfix c++ (връща ПРЕДИ инкремента)
    Counter operator++(int) {  // int е dummy параметър за разграничаване
        Counter old(*this);
        ++value;
        return old;
    }
    
    int getValue() const { return value; }
};
```

> 💡 **Prefix (`++c`)** е по-ефективен — не създава копие. Предпочитайте го, когато е възможно.

---

## 7. Член-функция vs. външна функция

| Оператор | Като член | Като `friend`/външна | Препоръка |
|----------|-----------|---------------------|-----------|
| `=`, `[]`, `()`, `->` | ✅ Задължително | ❌ | Член |
| `+`, `-`, `*`, `/` | ✅ | ✅ | Външна (за симетрия) |
| `==`, `!=`, `<`, `>` | ✅ | ✅ | Външна (за симетрия) |
| `<<`, `>>` | ❌ | ✅ Задължително | Friend |
| `++`, `--` | ✅ | ✅ | Член |

> 🧠 **Симетрия:** Ако `a + b` работи, потребителят очаква `b + a` също да работи. Като член-функция, `3 + complex` няма да компилира, но `complex + 3` ще. Като външна функция, и двете работят.

---

## Обобщение

| Оператор | Синтаксис | Забележка |
|----------|-----------|-----------|
| `+`, `-`, `*`, `/` | `T operator+(const T&) const` | Връща нов обект |
| `+=`, `-=` | `T& operator+=(const T&)` | Модифицира `*this` |
| `==`, `!=` | `bool operator==(const T&) const` | |
| `<`, `>`, `<=`, `>=` | `bool operator<(const T&) const` | Останалите чрез `<` |
| `<<` | `friend ostream& operator<<(ostream&, const T&)` | Външна |
| `>>` | `friend istream& operator>>(istream&, T&)` | Външна |
| `[]` | `T& operator[](int)` | Две версии (const и non-const) |
| `++` prefix | `T& operator++()` | Връща `*this` |
| `++` postfix | `T operator++(int)` | Dummy `int`, връща копие |
