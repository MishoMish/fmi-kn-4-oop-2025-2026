# Седмица 12 — Примери

## Пример 1: Безопасен масив с изключения

```cpp
#include <iostream>
#include <stdexcept>
#include <cstring>

class SafeArray {
    int* data;
    int size;
    
public:
    SafeArray(int n) : size(n) {
        if (n <= 0) throw std::invalid_argument("Размерът трябва да е положителен");
        data = new int[n]();
    }
    
    ~SafeArray() { delete[] data; }
    
    SafeArray(const SafeArray& other) : size(other.size), data(new int[other.size]) {
        std::memcpy(data, other.data, size * sizeof(int));
    }
    
    SafeArray& operator=(const SafeArray& other) {
        if (this != &other) {
            int* newData = new int[other.size];
            std::memcpy(newData, other.data, other.size * sizeof(int));
            delete[] data;
            data = newData;
            size = other.size;
        }
        return *this;
    }
    
    int& operator[](int index) {
        if (index < 0 || index >= size) {
            throw std::out_of_range("Индексът е извън границите");
        }
        return data[index];
    }
    
    const int& operator[](int index) const {
        if (index < 0 || index >= size) {
            throw std::out_of_range("Индексът е извън границите");
        }
        return data[index];
    }
    
    int getSize() const { return size; }
};

int main() {
    try {
        SafeArray arr(5);
        arr[0] = 10;
        arr[1] = 20;
        arr[2] = 30;
        
        std::cout << "arr[2] = " << arr[2] << std::endl;
        
        // Тест: достъп извън границите
        std::cout << arr[10] << std::endl;
        
    } catch (const std::out_of_range& e) {
        std::cerr << "Out of range: " << e.what() << std::endl;
    } catch (const std::invalid_argument& e) {
        std::cerr << "Invalid: " << e.what() << std::endl;
    }
    
    // Тест: невалиден размер
    try {
        SafeArray bad(-5);
    } catch (const std::exception& e) {
        std::cerr << "Грешка: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## Пример 2: RAII file wrapper

```cpp
#include <iostream>
#include <fstream>
#include <stdexcept>

class FileRAII {
    std::fstream file;
    char filename[256];
    
public:
    FileRAII(const char* name, std::ios::openmode mode) {
        std::strncpy(filename, name, 255);
        filename[255] = '\0';
        file.open(name, mode);
        if (!file.is_open()) {
            throw std::runtime_error(
                std::string("Не мога да отворя файл: ") + name
            );
        }
        std::cout << "[RAII] Файлът '" << name << "' е отворен" << std::endl;
    }
    
    ~FileRAII() {
        if (file.is_open()) {
            file.close();
            std::cout << "[RAII] Файлът '" << filename << "' е затворен" << std::endl;
        }
    }
    
    std::fstream& get() { return file; }
    
    FileRAII(const FileRAII&) = delete;
    FileRAII& operator=(const FileRAII&) = delete;
};

void processFile(const char* name) {
    FileRAII f(name, std::ios::out);  // RAII — отваря файла
    
    f.get() << "Hello, RAII!" << std::endl;
    f.get() << "Файлът се затваря автоматично." << std::endl;
    
    // Дори при изключение, деструкторът ще затвори файла
}

int main() {
    try {
        processFile("test_raii.txt");
        
        // Прочитане
        FileRAII reader("test_raii.txt", std::ios::in);
        char line[100];
        while (reader.get().getline(line, 100)) {
            std::cout << "> " << line << std::endl;
        }
        
    } catch (const std::exception& e) {
        std::cerr << "Грешка: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## Пример 3: Собствена йерархия на изключения

```cpp
#include <iostream>
#include <stdexcept>
#include <cstring>

// Базов клас за грешки на приложението
class AppError : public std::runtime_error {
public:
    AppError(const char* msg) : std::runtime_error(msg) {}
};

class ValidationError : public AppError {
    char field[100];
public:
    ValidationError(const char* fieldName, const char* msg) 
        : AppError(msg) {
        std::strncpy(field, fieldName, 99);
        field[99] = '\0';
    }
    const char* getField() const { return field; }
};

class DatabaseError : public AppError {
public:
    DatabaseError(const char* msg) : AppError(msg) {}
};

// Модел
class User {
    char name[100];
    int age;
    
public:
    User(const char* n, int a) {
        if (std::strlen(n) == 0)
            throw ValidationError("name", "Името не може да е празно");
        if (a < 0 || a > 150)
            throw ValidationError("age", "Невалидна възраст");
        
        std::strncpy(name, n, 99);
        name[99] = '\0';
        age = a;
    }
    
    void print() const {
        std::cout << name << " (възраст: " << age << ")" << std::endl;
    }
};

int main() {
    // Тест 1: Валиден потребител
    try {
        User u1("Иван", 25);
        u1.print();
    } catch (const AppError& e) {
        std::cerr << "Грешка: " << e.what() << std::endl;
    }
    
    // Тест 2: Невалидна възраст
    try {
        User u2("Мария", -5);
    } catch (const ValidationError& e) {
        std::cerr << "Валидация [" << e.getField() << "]: " << e.what() << std::endl;
    }
    
    // Тест 3: Празно име
    try {
        User u3("", 20);
    } catch (const ValidationError& e) {
        std::cerr << "Валидация [" << e.getField() << "]: " << e.what() << std::endl;
    }
    
    return 0;
}
```
