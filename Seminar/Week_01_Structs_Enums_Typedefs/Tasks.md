# Седмица 1 — Задачи за затвърждаване

## Задача 1: Автомобилен каталог

Дефинирайте структура `Car` с полета: марка (`char[50]`), модел (`char[50]`), година, цена. Дефинирайте `enum class FuelType { Petrol, Diesel, Electric, Hybrid }` и го добавете като поле.

Напишете функции:
- `void printCar(const Car& car)` — отпечатва информацията
- `Car findCheapest(const Car cars[], int count)` — връща най-евтината кола
- `int countByFuel(const Car cars[], int count, FuelType fuel)` — брои колите по вид гориво

---

## Задача 2: Вложени структури — Университет

Дефинирайте:
- `struct Address` с полета: град (`char[50]`), улица (`char[100]`), номер (`int`)
- `struct Student` с полета: име (`char[100]`), факултетен номер (`int`), среден успех (`double`), адрес (`Address`)

Напишете функция `void printStudentCard(const Student& s)`, която отпечатва цялата информация за студента, включително адреса.

---

## Задача 3: Псевдоними с `using`

Дефинирайте следните псевдоними:
```cpp
using Score = int;
using StudentID = unsigned int;
using GradeList = Score[100];
```

Използвайте ги, за да напишете функция:
```cpp
double averageScore(const GradeList& grades, int count);
```

> 💡 Целта е да се види как `using` подобрява четимостта на кода, без да променя логиката.

---

## Задача 4: Организация с `namespace`

Създайте два namespace-а:

- `namespace Converter` — с функции `double celsiusToFahrenheit(double)` и `double fahrenheitToCelsius(double)`
- `namespace Validator` — с функции `bool isPositive(int)`, `bool isInRange(int value, int min, int max)`

Демонстрирайте употребата им в `main()`, като извикате функции от двата namespace-а.

> 🧠 **Помислете:** Какво би станало, ако и двата namespace-а имаха функция `convert()`? Как бихте разрешили конфликта?
