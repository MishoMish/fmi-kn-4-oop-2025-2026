# Обработка на изключения и RAII

## Въведение

Грешките се случват: невалиден вход, липсващ файл, изчерпана памет. В C използвахме кодове за грешка (`return -1`), но това замърсява логиката и е лесно да бъде пренебрегнато. C++ предлага **изключения** (exceptions) — механизъм за отделяне на нормалния поток от обработката на грешки.

---

## 1. `throw`, `try`, `catch`

```cpp
double divide(double a, double b) {
    if (b == 0) {
        throw std::runtime_error("Деление на нула!");
    }
    return a / b;
}

int main() {
    try {
        double result = divide(10, 0);
        std::cout << result << std::endl;  // Няма да се изпълни
    } catch (const std::runtime_error& e) {
        std::cerr << "Грешка: " << e.what() << std::endl;
    }
    
    std::cout << "Програмата продължава." << std::endl;
    return 0;
}
```

### Как работи?

1. `throw` **хвърля** изключение — контролът напуска текущата функция
2. Стекът се **развива** (stack unwinding) — всички локални обекти се разрушават
3. Първият `catch` блок, който съвпада с типа, **хваща** изключението

---

## 2. Стандартна йерархия на изключения

```
std::exception
├── std::logic_error
│   ├── std::invalid_argument
│   ├── std::out_of_range
│   └── std::length_error
└── std::runtime_error
    ├── std::overflow_error
    ├── std::underflow_error
    └── std::range_error
```

Всички имат метод `what()`, който връща описание:

```cpp
#include <stdexcept>

void setAge(int age) {
    if (age < 0 || age > 150) {
        throw std::invalid_argument("Невалидна възраст!");
    }
}
```

---

## 3. Множество `catch` блокове

```cpp
try {
    // ...
} catch (const std::invalid_argument& e) {
    std::cerr << "Невалиден аргумент: " << e.what() << std::endl;
} catch (const std::runtime_error& e) {
    std::cerr << "Runtime грешка: " << e.what() << std::endl;
} catch (const std::exception& e) {
    std::cerr << "Обща грешка: " << e.what() << std::endl;
} catch (...) {
    std::cerr << "Непозната грешка!" << std::endl;
}
```

> ⚠️ `catch` блоковете се проверяват **отгоре надолу**. Слагайте по-специфичните първо!

---

## 4. Собствени класове за изключения

```cpp
class FileError : public std::runtime_error {
    char filename[256];
public:
    FileError(const char* fname, const char* msg) 
        : std::runtime_error(msg) {
        std::strncpy(filename, fname, 255);
        filename[255] = '\0';
    }
    
    const char* getFilename() const { return filename; }
};

class FileNotFoundError : public FileError {
public:
    FileNotFoundError(const char* fname)
        : FileError(fname, "Файлът не е намерен") {}
};

class FilePermissionError : public FileError {
public:
    FilePermissionError(const char* fname)
        : FileError(fname, "Нямате права за достъп") {}
};
```

---

## 5. RAII (Resource Acquisition Is Initialization)

**RAII** е може би най-важният идиом в C++: **придобиването на ресурс е инициализация**, а **освобождаването е разрушаване**.

### Проблемът без RAII

```cpp
void readFile(const char* filename) {
    int* buffer = new int[1000];
    std::ifstream file(filename);
    
    if (!file.is_open()) {
        delete[] buffer;     // ← Лесно се забравя!
        throw FileNotFoundError(filename);
    }
    
    // Ако тук хвърлим изключение → buffer НЕ се освобождава!
    processData(buffer);
    
    delete[] buffer;
}
```

### Решението с RAII

```cpp
class IntBuffer {
    int* data;
    int size;
public:
    IntBuffer(int n) : data(new int[n]), size(n) {}
    ~IntBuffer() { delete[] data; }  // Автоматично освобождаване
    
    int& operator[](int i) { return data[i]; }
    int getSize() const { return size; }
    
    IntBuffer(const IntBuffer&) = delete;  // Или deep copy
    IntBuffer& operator=(const IntBuffer&) = delete;
};

void readFile(const char* filename) {
    IntBuffer buffer(1000);  // RAII — ресурсът е в обект
    std::ifstream file(filename);  // ifstream също е RAII!
    
    if (!file.is_open()) {
        throw FileNotFoundError(filename);
        // buffer и file автоматично се разрушават при stack unwinding!
    }
    
    processData(buffer);
    // Деструкторите се извикват автоматично
}
```

> 💡 При stack unwinding всички **локални обекти** се разрушават коректно. Затова обвиваме ресурси в обекти!

---

## 6. RAII обвивки за ресурси

### Файлов манипулатор

```cpp
class FileGuard {
    std::fstream file;
public:
    FileGuard(const char* name, std::ios::openmode mode) {
        file.open(name, mode);
        if (!file.is_open()) {
            throw FileNotFoundError(name);
        }
    }
    
    ~FileGuard() {
        if (file.is_open()) file.close();
    }
    
    std::fstream& get() { return file; }
    
    FileGuard(const FileGuard&) = delete;
    FileGuard& operator=(const FileGuard&) = delete;
};
```

---

## 7. `noexcept`

Маркира функция, която **гарантира**, че не хвърля изключения:

```cpp
int add(int a, int b) noexcept {
    return a + b;  // Никога не хвърля
}

void swap(int& a, int& b) noexcept {
    int temp = a;
    a = b;
    b = temp;
}
```

> 💡 Move конструкторите и move операторите трябва да бъдат `noexcept` за оптимална работа с контейнери.

---

## 8. Добри практики

| ✅ Правилно | ❌ Грешно |
|-------------|----------|
| Хвърлете по стойност | Хвърлете указатели (`throw new ...`) |
| Хващайте по `const` референция | Хващайте по стойност (slicing!) |
| Използвайте RAII за ресурси | Ръчно `delete` в `catch` блокове |
| Хвърляйте стандартни типове | Хвърляйте `int` или `const char*` |
| `noexcept` за деструктори | Хвърляне на изключение от деструктор |

> ⚠️ **Никога** не хвърляйте изключение от деструктор! Ако по време на stack unwinding деструктор хвърли изключение, програмата **терминира** (std::terminate).

---

## Обобщение

| Концепция | Описание |
|-----------|----------|
| `throw` | Хвърля изключение |
| `try`/`catch` | Улавя и обработва изключение |
| `catch(...)` | Хваща всяко изключение |
| Stack unwinding | Автоматично разрушаване на локални обекти |
| RAII | Ресурсът се обвива в обект |
| `noexcept` | Гаранция, че функцията не хвърля |
| Custom exceptions | Наследяване от `std::exception` |
