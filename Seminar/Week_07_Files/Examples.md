# Седмица 7 — Примери

## Пример 1: Запис и четене на числа от текстов файл

```cpp
#include <iostream>
#include <fstream>

void writeNumbers(const char* filename, const int arr[], int size) {
    std::ofstream file(filename);
    if (!file) {
        std::cerr << "Грешка при отваряне за писане!" << std::endl;
        return;
    }
    
    for (int i = 0; i < size; i++) {
        file << arr[i];
        if (i < size - 1) file << " ";
    }
    file << std::endl;
    
    std::cout << "Записани " << size << " числа в " << filename << std::endl;
}

int readNumbers(const char* filename, int arr[], int maxSize) {
    std::ifstream file(filename);
    if (!file) {
        std::cerr << "Грешка при отваряне за четене!" << std::endl;
        return 0;
    }
    
    int count = 0;
    while (count < maxSize && file >> arr[count]) {
        count++;
    }
    
    std::cout << "Прочетени " << count << " числа от " << filename << std::endl;
    return count;
}

int main() {
    int data[] = {42, 17, 93, 5, 68, 31};
    writeNumbers("numbers.txt", data, 6);
    
    int result[100];
    int count = readNumbers("numbers.txt", result, 100);
    
    for (int i = 0; i < count; i++) {
        std::cout << result[i] << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## Пример 2: Сериализация на структура в двоичен файл

```cpp
#include <iostream>
#include <fstream>
#include <cstring>

struct Student {
    char name[50];
    int facultyNumber;
    double grade;
};

void saveStudents(const char* filename, const Student students[], int count) {
    std::ofstream file(filename, std::ios::binary);
    if (!file) return;
    
    // Записваме броя студенти
    file.write(reinterpret_cast<const char*>(&count), sizeof(int));
    
    // Записваме масива
    file.write(reinterpret_cast<const char*>(students), count * sizeof(Student));
    
    std::cout << "Записани " << count << " студенти в " << filename << std::endl;
}

int loadStudents(const char* filename, Student students[], int maxCount) {
    std::ifstream file(filename, std::ios::binary);
    if (!file) return 0;
    
    int count;
    file.read(reinterpret_cast<char*>(&count), sizeof(int));
    
    if (count > maxCount) count = maxCount;
    file.read(reinterpret_cast<char*>(students), count * sizeof(Student));
    
    return count;
}

int main() {
    Student students[] = {
        {"Иван Петров", 62100, 5.50},
        {"Мария Георгиева", 62101, 6.00},
        {"Петър Стоянов", 62102, 4.75}
    };
    
    saveStudents("students.bin", students, 3);
    
    Student loaded[10];
    int count = loadStudents("students.bin", loaded, 10);
    
    std::cout << "\nПрочетени студенти:" << std::endl;
    for (int i = 0; i < count; i++) {
        std::cout << loaded[i].name << " (ФН: " << loaded[i].facultyNumber 
                  << ") — " << loaded[i].grade << std::endl;
    }
    
    return 0;
}
```

> 💡 **Забележете:** Тъй като `Student` е POD тип (без указатели), можем да го запишем директно с `write`. За обекти с `char*` или динамична памет, трябва ръчна сериализация.

---

## Пример 3: Четене на CSV файл (текстов, структуриран)

```cpp
#include <iostream>
#include <fstream>
#include <cstring>

struct Product {
    char name[50];
    double price;
    int quantity;
};

int loadCSV(const char* filename, Product products[], int maxCount) {
    std::ifstream file(filename);
    if (!file) return 0;
    
    char line[256];
    int count = 0;
    
    // Пропускаме header реда
    file.getline(line, 256);
    
    while (count < maxCount && file.getline(line, 256)) {
        // Парсваме CSV: name,price,quantity
        char* token = std::strtok(line, ",");
        if (!token) continue;
        std::strncpy(products[count].name, token, 49);
        
        token = std::strtok(nullptr, ",");
        if (!token) continue;
        products[count].price = std::atof(token);
        
        token = std::strtok(nullptr, ",");
        if (!token) continue;
        products[count].quantity = std::atoi(token);
        
        count++;
    }
    
    return count;
}

int main() {
    // Първо, създаваме CSV файл за демонстрация
    std::ofstream csv("products.csv");
    csv << "name,price,quantity" << std::endl;
    csv << "Лаптоп,1999.99,15" << std::endl;
    csv << "Мишка,29.99,200" << std::endl;
    csv << "Клавиатура,79.50,85" << std::endl;
    csv.close();
    
    // Четем го обратно
    Product products[100];
    int count = loadCSV("products.csv", products, 100);
    
    std::cout << "Продукти:" << std::endl;
    for (int i = 0; i < count; i++) {
        std::cout << "  " << products[i].name 
                  << " — " << products[i].price << " лв."
                  << " (x" << products[i].quantity << ")" << std::endl;
    }
    
    return 0;
}
```

> 💡 CSV (Comma-Separated Values) е популярен текстов формат. Парсването изисква внимание с `strtok` или ръчна обработка.
