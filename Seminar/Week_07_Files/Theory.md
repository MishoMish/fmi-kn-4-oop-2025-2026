# Работа с файлове: текстови и двоични

## Въведение

Файловете са основният начин за **персистиране на данни** — съхраняване на информация, която да надживее изпълнението на програмата. В C++ работим с файлове чрез **потоци** (streams) — същата концепция като `std::cin` и `std::cout`, но насочена към файлове.

> 📌 **Тази седмица е посветена изцяло на файлове** — тема, критично важна за сериализация на обекти и за изпитите.

---

## 1. Видове файлове

| Характеристика | Текстов файл | Двоичен файл |
|----------------|-------------|-------------|
| Формат | Четим от човек | Сурови байтове |
| Разделители | Нов ред, интервали | Няма — фиксирана структура |
| Размер | По-голям | По-малък |
| Преносимост | По-добра | Зависи от платформата |
| Пример | `.txt`, `.csv`, `.json` | `.bin`, `.dat`, изображения |

---

## 2. Потоци за файлове

```cpp
#include <fstream>

std::ifstream  // Вход (четене от файл)
std::ofstream  // Изход (писане във файл)
std::fstream   // И двете
```

### Отваряне и затваряне

```cpp
// Начин 1: Чрез конструктор
std::ifstream file("data.txt");
if (!file.is_open()) {
    std::cerr << "Грешка при отваряне!" << std::endl;
    return;
}
// ... работа с файла ...
file.close();  // Не е задължително — деструкторът затваря

// Начин 2: Чрез open()
std::ifstream file;
file.open("data.txt");
```

### Флагове за отваряне

```cpp
std::ofstream file("log.txt", std::ios::app);     // Добавяне в края
std::ofstream file("data.bin", std::ios::binary);  // Двоичен режим
std::fstream file("data.txt", std::ios::in | std::ios::out);  // Четене и писане
```

| Флаг | Описание |
|------|----------|
| `ios::in` | Отваряне за четене |
| `ios::out` | Отваряне за писане (изтрива съдържанието!) |
| `ios::app` | Добавяне в края |
| `ios::binary` | Двоичен режим |
| `ios::trunc` | Изтрива съдържанието при отваряне |
| `ios::ate` | Поставя позицията в края при отваряне |

---

## 3. Текстови файлове

### Писане

```cpp
#include <fstream>

void writeStudents() {
    std::ofstream file("students.txt");
    if (!file) return;
    
    file << "Иван" << " " << 5.50 << std::endl;
    file << "Мария" << " " << 6.00 << std::endl;
    file << "Петър" << " " << 4.75 << std::endl;
}
```

### Четене с `>>`

```cpp
void readStudents() {
    std::ifstream file("students.txt");
    if (!file) return;
    
    char name[100];
    double grade;
    
    while (file >> name >> grade) {
        std::cout << name << ": " << grade << std::endl;
    }
}
```

### Четене ред по ред с `getline`

```cpp
void readLines() {
    std::ifstream file("notes.txt");
    if (!file) return;
    
    char line[256];
    int lineNum = 1;
    
    while (file.getline(line, 256)) {
        std::cout << lineNum++ << ": " << line << std::endl;
    }
}
```

### Проверка за грешки

```cpp
std::ifstream file("data.txt");

file.good();   // Няма грешки
file.eof();    // Достигнат е краят на файла
file.fail();   // Грешка при операция (грешен формат и т.н.)
file.bad();    // Сериозна грешка (хардуерна)

// Идиоматичен начин за проверка:
if (!file) {
    std::cerr << "Грешка!" << std::endl;
}
```

---

## 4. Двоични файлове

### Защо двоични файлове?

- По-бързи за четене/писане (няма форматиране)
- По-компактни
- Позволяват **директен достъп** (random access)
- Идеални за сериализация на структури

### Писане

```cpp
struct Student {
    char name[50];
    int age;
    double grade;
};

void writeBinary() {
    Student s = {"Иван", 21, 5.50};
    
    std::ofstream file("students.bin", std::ios::binary);
    if (!file) return;
    
    file.write(reinterpret_cast<const char*>(&s), sizeof(Student));
}
```

### Четене

```cpp
void readBinary() {
    Student s;
    
    std::ifstream file("students.bin", std::ios::binary);
    if (!file) return;
    
    file.read(reinterpret_cast<char*>(&s), sizeof(Student));
    std::cout << s.name << ", " << s.age << ", " << s.grade << std::endl;
}
```

### Масив от структури

```cpp
void writeArray(const Student students[], int count) {
    std::ofstream file("all_students.bin", std::ios::binary);
    if (!file) return;
    
    // Записваме броя първо
    file.write(reinterpret_cast<const char*>(&count), sizeof(int));
    // После масива
    file.write(reinterpret_cast<const char*>(students), count * sizeof(Student));
}

int readArray(Student students[], int maxCount) {
    std::ifstream file("all_students.bin", std::ios::binary);
    if (!file) return 0;
    
    int count;
    file.read(reinterpret_cast<char*>(&count), sizeof(int));
    if (count > maxCount) count = maxCount;
    file.read(reinterpret_cast<char*>(students), count * sizeof(Student));
    
    return count;
}
```

---

## 5. Позициониране във файл

```cpp
// Позиция за четене (get)
file.seekg(0, std::ios::beg);    // Начало на файла
file.seekg(0, std::ios::end);    // Край на файла
file.seekg(100, std::ios::beg);  // 100 байта от началото
file.seekg(-50, std::ios::cur);  // 50 байта назад от текущата позиция
file.tellg();                     // Текуща позиция

// Позиция за писане (put)
file.seekp(0, std::ios::beg);
file.tellp();
```

### Определяне на размера на файл

```cpp
std::ifstream file("data.bin", std::ios::binary);
file.seekg(0, std::ios::end);
int fileSize = file.tellg();
file.seekg(0, std::ios::beg);  // Връщаме се в началото

int recordCount = fileSize / sizeof(Student);
```

### Достъп до конкретен запис

```cpp
void readStudent(int index) {
    std::ifstream file("students.bin", std::ios::binary);
    
    // Прескачаме int (броя) + index * sizeof(Student)
    file.seekg(sizeof(int) + index * sizeof(Student), std::ios::beg);
    
    Student s;
    file.read(reinterpret_cast<char*>(&s), sizeof(Student));
    std::cout << s.name << std::endl;
}
```

---

## 6. C++ Casts (Кратко въведение)

При работа с файлове срещаме `reinterpret_cast`. Нека разберем четирите вида cast в C++:

| Cast | Предназначение | Пример |
|------|---------------|--------|
| `static_cast` | Безопасни конверсии между свързани типове | `static_cast<int>(3.14)` → `3` |
| `dynamic_cast` | Полиморфно кастване (с проверка) | `dynamic_cast<Derived*>(basePtr)` |
| `reinterpret_cast` | Преинтерпретиране на битовете | Файлов I/O: `char*` ↔ `struct*` |
| `const_cast` | Премахване/добавяне на `const` | `const_cast<char*>(constStr)` |

> ⚠️ `reinterpret_cast` е „опасен" — казва на компилатора „виж тези байтове като друг тип". Използвайте го **само** за двоичен I/O и подобни нисконивови операции.

---

## 7. Сериализация на обекти

### Проблемът с динамичната памет

```cpp
// ❌ Не работи директно за обекти с указатели!
class Person {
    char* name;  // Указател — записва се адресът, НЕ съдържанието!
    int age;
};
```

### Решение: Ръчна сериализация

```cpp
void savePerson(std::ofstream& file, const Person& p) {
    int nameLen = std::strlen(p.getName());
    file.write(reinterpret_cast<const char*>(&nameLen), sizeof(int));
    file.write(p.getName(), nameLen);
    int age = p.getAge();
    file.write(reinterpret_cast<const char*>(&age), sizeof(int));
}

Person loadPerson(std::ifstream& file) {
    int nameLen;
    file.read(reinterpret_cast<char*>(&nameLen), sizeof(int));
    char* name = new char[nameLen + 1];
    file.read(name, nameLen);
    name[nameLen] = '\0';
    int age;
    file.read(reinterpret_cast<char*>(&age), sizeof(int));
    Person p(name, age);
    delete[] name;
    return p;
}
```

---

## Обобщение

| Тема | Ключова идея |
|------|-------------|
| `ifstream` / `ofstream` | Файлови потоци, аналогични на `cin`/`cout` |
| Текстови файлове | `<<`, `>>`, `getline()` |
| Двоични файлове | `write()`, `read()`, `reinterpret_cast` |
| Позициониране | `seekg()`, `seekp()`, `tellg()`, `tellp()` |
| Сериализация | Запис и четене на обекти — внимание с указатели! |
| C++ Casts | `static_cast`, `dynamic_cast`, `reinterpret_cast`, `const_cast` |

> 🧠 **Ключов takeaway:** Файловете са „врата" към дълготрайно съхранение. Разбирането на разликата между текстов и двоичен режим и правилната сериализация е основа за бази данни, конфигурации и мрежова комуникация.
