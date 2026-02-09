# Седмица 6 — Примери

## Пример 1: `Complex` число с аритметика и поток

```cpp
#include <iostream>
#include <cmath>

class Complex {
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    Complex& operator+=(const Complex& other) {
        real += other.real;
        imag += other.imag;
        return *this;
    }
    
    Complex operator+(const Complex& other) const {
        Complex result(*this);
        result += other;
        return result;
    }
    
    bool operator==(const Complex& other) const {
        const double eps = 1e-9;
        return std::abs(real - other.real) < eps && 
               std::abs(imag - other.imag) < eps;
    }
    
    bool operator!=(const Complex& other) const {
        return !(*this == other);
    }
    
    double modulus() const {
        return std::sqrt(real * real + imag * imag);
    }
    
    friend std::ostream& operator<<(std::ostream& os, const Complex& c);
    friend std::istream& operator>>(std::istream& is, Complex& c);
};

std::ostream& operator<<(std::ostream& os, const Complex& c) {
    os << "(" << c.real;
    if (c.imag >= 0) os << "+";
    os << c.imag << "i)";
    return os;
}

std::istream& operator>>(std::istream& is, Complex& c) {
    std::cout << "Реална: ";
    is >> c.real;
    std::cout << "Имагинерна: ";
    is >> c.imag;
    return is;
}

int main() {
    Complex a(3, 4);
    Complex b(1, -2);
    
    std::cout << a << " + " << b << " = " << (a + b) << std::endl;
    // (3+4i) + (1-2i) = (4+2i)
    
    std::cout << "|" << a << "| = " << a.modulus() << std::endl;
    // |(3+4i)| = 5
    
    a += b;
    std::cout << "След +=: " << a << std::endl;
    // След +=: (4+2i)
    
    return 0;
}
```

---

## Пример 2: `Vector2D` с оператори

```cpp
#include <iostream>
#include <cmath>

class Vector2D {
    double x, y;
    
public:
    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}
    
    Vector2D operator+(const Vector2D& v) const { return {x + v.x, y + v.y}; }
    Vector2D operator-(const Vector2D& v) const { return {x - v.x, y - v.y}; }
    
    // Скаларно произведение (dot product)
    double operator*(const Vector2D& v) const { return x * v.x + y * v.y; }
    
    // Умножение по скалар
    Vector2D operator*(double s) const { return {x * s, y * s}; }
    
    double length() const { return std::sqrt(x * x + y * y); }
    
    bool operator==(const Vector2D& v) const {
        return std::abs(x - v.x) < 1e-9 && std::abs(y - v.y) < 1e-9;
    }
    
    friend std::ostream& operator<<(std::ostream& os, const Vector2D& v) {
        return os << "(" << v.x << ", " << v.y << ")";
    }
    
    // Позволява: 2.0 * vec (скалар отляво)
    friend Vector2D operator*(double s, const Vector2D& v) {
        return v * s;
    }
};

int main() {
    Vector2D a(3, 4);
    Vector2D b(1, 2);
    
    std::cout << a << " + " << b << " = " << (a + b) << std::endl;
    std::cout << a << " · " << b << " = " << (a * b) << std::endl;
    std::cout << "2 * " << a << " = " << (2.0 * a) << std::endl;
    std::cout << "|" << a << "| = " << a.length() << std::endl;
    
    return 0;
}
```

> 💡 `friend operator*(double, Vector2D)` позволява `2.0 * vec` — без него само `vec * 2.0` работи.

---

## Пример 3: `Matrix` с `operator[]`

```cpp
#include <iostream>

class Matrix {
    int** data;
    int rows, cols;
    
public:
    Matrix(int r, int c) : rows(r), cols(c) {
        data = new int*[rows];
        for (int i = 0; i < rows; i++) {
            data[i] = new int[cols]();  // Инициализирано с 0
        }
    }
    
    ~Matrix() {
        for (int i = 0; i < rows; i++) delete[] data[i];
        delete[] data;
    }
    
    // operator[] връща ред — позволява matrix[i][j]
    int* operator[](int row) { return data[row]; }
    const int* operator[](int row) const { return data[row]; }
    
    friend std::ostream& operator<<(std::ostream& os, const Matrix& m) {
        for (int i = 0; i < m.rows; i++) {
            for (int j = 0; j < m.cols; j++) {
                os << m[i][j] << "\t";
            }
            os << std::endl;
        }
        return os;
    }
};

int main() {
    Matrix m(3, 3);
    
    // Запълване — използваме m[i][j] като обикновен масив
    for (int i = 0; i < 3; i++)
        for (int j = 0; j < 3; j++)
            m[i][j] = i * 3 + j + 1;
    
    std::cout << m;
    // 1  2  3
    // 4  5  6
    // 7  8  9
    
    return 0;
}
```

> 💡 `operator[]` връща `int*` (указател към ред), а вторият `[]` е обикновен достъп до масив. Така получаваме `matrix[i][j]` синтаксис.
