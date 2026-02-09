# 📚 Семинар по Обектно-ориентирано програмиране

**Курс:** Обектно-ориентирано програмиране (ООП)  
**Семестър:** Летен, 2025/2026  
**Факултет:** Факултет по Математика и Информатика (ФМИ)  
**Специалност:** Компютърни Науки  
**Група:** КН-4

---

## 📖 За този семинар

Този семинар е **теоретично-ориентиран**. Всяка седмица е структурирана с фокус върху:

1. 🧠 **Теория и обяснения** — основната и доминиращата част
2. 💻 **Илюстративни примери** — код с коментари и дискусия
3. 📝 **Задачи за затвърждаване** — малък брой задачи, подкрепящи теорията

> ⚠️ Това **НЕ** е практикум. Задачите не са фокусът — те подпомагат разбирането.

---

## 🗓️ Седмичен план

### Част I — Основи на C++ (Седмици 1–3)

<details>
<summary><h3>📅 Седмица 1 — Структури, Enum, Union, Typedef, Namespace</h3></summary>

#### 📌 Тема: Потребителски типове данни и организация на кода

#### 📚 Теория (Theory.md)
- **Структури (`struct`):** дефиниция, инициализация, достъп до членове
- **Разлика между `struct` и `class`** — подразбиращ се достъп
- **Изброими типове (`enum` и `enum class`):** символични константи, тип-безопасност
- **Обединения (`union`):** споделена памет, кога се използват
- **Псевдоними на типове (`typedef` / `using`):** четимост на кода
- **Пространства от имена (`namespace`):** организация и избягване на конфликти

#### 💻 Примери (Examples.md)
- Моделиране на точка (`Point`) със `struct`
- Цветове с `enum class`
- Вариантен тип с `union` за данни на различни типове
- `namespace` за разделяне на модули

#### 📝 Задачи (Tasks.md)
- 4 задачи: кола с `struct`+`enum`, вложени структури, `typedef` за псевдоними, `namespace` организация

#### 📂 Файлове:
- [`Theory.md`](Week_01_Structs_Enums_Typedefs/Theory.md)
- [`Examples.md`](Week_01_Structs_Enums_Typedefs/Examples.md)
- [`Tasks.md`](Week_01_Structs_Enums_Typedefs/Tasks.md)

</details>

<details>
<summary><h3>📅 Седмица 2 — Шаблони на функции и функции от по-висок ред</h3></summary>

#### 📌 Тема: Обобщено програмиране и функционални концепции

#### 📚 Теория (Theory.md)
- **Шаблони на функции (`template`):** синтаксис, извеждане на типове, инстанциране
- **Шаблони с множество параметри** и ограничения
- **Указатели към функции:** синтаксис, предаване като аргумент
- **`std::function`:** универсален обвиващ тип за извикваеми обекти
- **Функции от по-висок ред:** `map`, `filter`, `reduce` концепции

#### 💻 Примери (Examples.md)
- Шаблонна `maxValue` за произволен тип
- `filter` с указател към функция
- `transform` с `std::function`

#### 📝 Задачи (Tasks.md)
- 4 задачи: `swap` шаблон, `findMin`, сортиране с компаратор, `forEach` с функционален обект

#### 📂 Файлове:
- [`Theory.md`](Week_02_Templates_HigherOrderFunctions/Theory.md)
- [`Examples.md`](Week_02_Templates_HigherOrderFunctions/Examples.md)
- [`Tasks.md`](Week_02_Templates_HigherOrderFunctions/Tasks.md)

</details>

<details>
<summary><h3>📅 Седмица 3 — Ламбда функции и функционални концепции</h3></summary>

#### 📌 Тема: Анонимни функции, captures и функционален стил

#### 📚 Теория (Theory.md)
- **Ламбда функции:** синтаксис `[capture](params) -> return_type { body }`
- **Видове captures:** `[=]`, `[&]`, `[x]`, `[&x]`, `[this]`, смесени
- **Mutable ламбди** и състояние
- **Ламбди и `std::function`** — съхраняване и предаване
- **Генерични ламбди (C++14):** `auto` параметри
- **Функционален стил:** композиция, chaining, pipeline

#### 💻 Примери (Examples.md)
- Ламбда за сортиране по критерий
- Capture по стойност vs по референция
- Ламбда фабрика (функция, връщаща ламбда)

#### 📝 Задачи (Tasks.md)
- 4 задачи: филтриране с ламбда, трансформация на масив, ламбда компаратор, pipeline от операции

#### 📂 Файлове:
- [`Theory.md`](Week_03_Lambdas_FunctionalConcepts/Theory.md)
- [`Examples.md`](Week_03_Lambdas_FunctionalConcepts/Examples.md)
- [`Tasks.md`](Week_03_Lambdas_FunctionalConcepts/Tasks.md)

</details>

---

### Част II — Класове и обектен дизайн (Седмици 4–6)

<details>
<summary><h3>📅 Седмица 4 — Класове и капсулация</h3></summary>

#### 📌 Тема: Обектно-ориентирано мислене — класове, достъп, this

#### 📚 Теория (Theory.md)
- **Какво е клас:** данни + поведение, абстракция на реалния свят
- **Членове:** полета (data members) и методи (member functions)
- **Модификатори за достъп:** `public`, `private`, `protected`
- **Капсулация:** скриване на данните, getter/setter, инварианти
- **Указателят `this`:** имплицитен параметър, кога е нужен
- **`const` методи:** семантика на неизменимост

#### 💻 Примери (Examples.md)
- Клас `BankAccount` с валидация
- `Student` с `private` данни и `public` интерфейс
- Демонстрация на `this` при конфликт на имена

#### 📝 Задачи (Tasks.md)
- 4 задачи: клас `Car`, клас `Rectangle`, клас `Employee` с капсулирани полета

#### 📂 Файлове:
- [`Theory.md`](Week_04_Classes_Encapsulation/Theory.md)
- [`Examples.md`](Week_04_Classes_Encapsulation/Examples.md)
- [`Tasks.md`](Week_04_Classes_Encapsulation/Tasks.md)

</details>

<details>
<summary><h3>📅 Седмица 5 — Жизнен цикъл на обект: Конструктори, деструктори, Голямата четворка</h3></summary>

#### 📌 Тема: Създаване, копиране и унищожаване на обекти

#### 📚 Теория (Theory.md)
- **Конструктори:** подразбиращ се, с параметри, списък за инициализация
- **Деструктори:** почистване на ресурси, ред на извикване
- **Копиращ конструктор:** кога се извиква, shallow vs deep copy
- **Оператор `operator=`:** семантика на присвояване, self-assignment
- **Голямата четворка (Big Four):** правилото — ако дефинираш едно, дефинирай всички
- **Динамична памет в класове:** `new`/`delete`, отговорност за ресурсите

#### 💻 Примери (Examples.md)
- Клас `String` с пълна Голяма четворка
- `Point` — copy constructor и destructor
- Демонстрация на проблема с shallow copy

#### 📝 Задачи (Tasks.md)
- 3 задачи: `DynamicArray`, `Book` + `Library`, `BankAccount` с динамична памет

#### 📂 Файлове:
- [`Theory.md`](Week_05_ObjectLifecycle_BigFour/Theory.md)
- [`Examples.md`](Week_05_ObjectLifecycle_BigFour/Examples.md)
- [`Tasks.md`](Week_05_ObjectLifecycle_BigFour/Tasks.md)

</details>

<details>
<summary><h3>📅 Седмица 6 — Предефиниране на оператори</h3></summary>

#### 📌 Тема: Оператори като функции — синтактичен захар за класове

#### 📚 Теория (Theory.md)
- **Какво е предефиниране на оператори:** смисъл и ограничения
- **Аритметични оператори:** `+`, `-`, `*`, `/`
- **Оператори за сравнение:** `==`, `!=`, `<`, `>`, `<=`, `>=`
- **Оператори за поток:** `<<` и `>>` (като `friend` функции)
- **Оператор за присвояване:** `operator=` (revisited)
- **Оператор за индексиране:** `operator[]`
- **Инкремент/Декремент:** `++` и `--` (prefix vs postfix)
- **Член-функция vs. външна функция** — кога кое е подходящо

#### 💻 Примери (Examples.md)
- `Complex` число с `+`, `==`, `<<`
- `Vector2D` с аритметични оператори
- `Matrix` с `operator[]`

#### 📝 Задачи (Tasks.md)
- 4 задачи: `Fraction` с оператори, `Date` с сравнение, `BigInteger` с `+` и `<<`

#### 📂 Файлове:
- [`Theory.md`](Week_06_OperatorOverloading/Theory.md)
- [`Examples.md`](Week_06_OperatorOverloading/Examples.md)
- [`Tasks.md`](Week_06_OperatorOverloading/Tasks.md)

</details>

---

### Част III — Файлове и структури от данни (Седмици 7–8)

<details>
<summary><h3>📅 Седмица 7 — Работа с файлове: текстови и двоични</h3></summary>

#### 📌 Тема: Четене и писане на данни — файлов вход/изход

#### 📚 Теория (Theory.md)
- **Видове файлове:** текстови vs двоични
- **Потоци:** `ifstream`, `ofstream`, `fstream`
- **Методи на потоците:** `open()`, `close()`, `is_open()`, `eof()`, `fail()`
- **Флагове за отваряне:** `ios::in`, `ios::out`, `ios::app`, `ios::binary`, `ios::trunc`
- **Четене от текстов файл:** `>>`, `getline()`, ред по ред
- **Писане в текстов файл:** `<<`, форматиране
- **Двоични файлове:** `read()`, `write()`, `seekg()`, `seekp()`, `tellg()`, `tellp()`
- **Сериализация на структури:** запис и четене на обекти
- **C++ Casting:** `static_cast`, `dynamic_cast`, `reinterpret_cast`, `const_cast`

#### 💻 Примери (Examples.md)
- Запис и четене на числа от текстов файл
- Сериализация на `Student` в двоичен файл
- Масив от структури в двоичен файл

#### 📝 Задачи (Tasks.md)
- 5 задачи: обръщане на ред, филтриране на студенти, броене на думи, двоичен масив I/O, сортиране от файл

#### 📂 Файлове:
- [`Theory.md`](Week_07_Files/Theory.md)
- [`Examples.md`](Week_07_Files/Examples.md)
- [`Tasks.md`](Week_07_Files/Tasks.md)

</details>

<details>
<summary><h3>📅 Седмица 8 — Итератори и композиция</h3></summary>

#### 📌 Тема: Обхождане на колекции и обектна композиция

#### 📚 Теория (Theory.md)
- **Какво е итератор:** абстракция за обхождане на колекции
- **`begin()` и `end()`:** конвенция в C++
- **Оператори на итератор:** `*`, `++`, `--`, `!=`
- **Създаване на собствен итератор** за потребителски клас
- **Range-based `for` цикъл:** как работи отвътре
- **Композиция (has-a):** обект съдържа друг обект
- **Разлика между композиция и наследяване**
- **Жизнен цикъл при композиция**

#### 💻 Примери (Examples.md)
- `ArrayIterator` за собствен масив
- `IntArray` с `begin()`/`end()` за range-based for
- Композиция: `University` съдържа `Department` обекти

#### 📝 Задачи (Tasks.md)
- 4 задачи: `RangeIterator`, итератор за `Vector`, обратен итератор, `Schedule` с композиция

#### 📂 Файлове:
- [`Theory.md`](Week_08_Iterators_Composition/Theory.md)
- [`Examples.md`](Week_08_Iterators_Composition/Examples.md)
- [`Tasks.md`](Week_08_Iterators_Composition/Tasks.md)

</details>

---

### Част IV — Наследяване и полиморфизъм (Седмици 9–11)

<details>
<summary><h3>📅 Седмица 9 — Наследяване</h3></summary>

#### 📌 Тема: Йерархии от класове — специализация и повторно използване на код

#### 📚 Теория (Theory.md)
- **Идеята за наследяване:** специализация, обобщение
- **`is-a` vs. `has-a`** — кога наследяване, кога композиция
- **Синтаксис на наследяване** в C++
- **Видове наследяване:** `public`, `protected`, `private`
- **Достъп до членове** при различните видове
- **Жизнен цикъл:** ред на конструиране и унищожаване
- **Предефиниране (overriding)** на методи в наследника
- **Скриване на имена (name hiding)**
- **Диамантен проблем** и `virtual` наследяване

#### 💻 Примери (Examples.md)
- `Person` → `Student` йерархия
- `Device` → `Phone`, `Laptop` с override
- Диамантен проблем с `virtual` inheritance

#### 📝 Задачи (Tasks.md)
- 4 задачи: `Animal` йерархия, `Shape` с `area()`, `Vehicle` с `fuelEfficiency()`, диамантен проблем

#### 📂 Файлове:
- [`Theory.md`](Week_09_Inheritance/Theory.md)
- [`Examples.md`](Week_09_Inheritance/Examples.md)
- [`Tasks.md`](Week_09_Inheritance/Tasks.md)

</details>

<details>
<summary><h3>📅 Седмица 10 — Полиморфизъм и абстрактни класове</h3></summary>

#### 📌 Тема: Виртуални функции, vtable и абстрактни интерфейси

#### 📚 Теория (Theory.md)
- **Статичен vs. динамичен полиморфизъм**
- **Виртуални функции (`virtual`):** vtable механизъм
- **Чисто виртуални функции (`= 0`):** абстрактни класове
- **Интерфейси** в C++ (класове само с чисто виртуални методи)
- **Виртуален деструктор:** защо е задължителен
- **`override` и `final`** ключови думи
- **Вектори от `Base*`** — работа с хетерогенни колекции
- **`clone()` метод:** виртуален конструктор за копиране

#### 💻 Примери (Examples.md)
- `Shape` абстрактен клас с `Circle`, `Rectangle`
- Вектор от `Shape*` с полиморфно поведение
- `clone()` за дълбоко копиране на полиморфни обекти

#### 📝 Задачи (Tasks.md)
- 4 задачи: `Employee` йерархия с `getSalary()`, хетерогенна колекция, абстрактен `Drawable`, `clone()` имплементация

#### 📂 Файлове:
- [`Theory.md`](Week_10_Polymorphism/Theory.md)
- [`Examples.md`](Week_10_Polymorphism/Examples.md)
- [`Tasks.md`](Week_10_Polymorphism/Tasks.md)

</details>

<details>
<summary><h3>📅 Седмица 11 — Шаблони за дизайн (Design Patterns) — Въведение</h3></summary>

#### 📌 Тема: Повтарящи се решения на често срещани проблеми

#### 📚 Теория (Theory.md)
- **Какво са шаблоните за дизайн:** история, GoF, класификация
- **Категории:** Creational, Structural, Behavioral
- **Factory Pattern:** създаване на обекти без указване на конкретен клас
- **Strategy Pattern:** алгоритъм като обект, подменяем по време на изпълнение
- **Singleton Pattern:** единствена инстанция, глобален достъп
- **Prototype Pattern:** клониране на обекти с `clone()`
- **Кога да ползваме шаблони** и кога не — anti-patterns

#### 💻 Примери (Examples.md)
- `AnimalFactory` — Factory за създаване на животни от низ
- `SortStrategy` — Strategy за различни алгоритми за сортиране
- `Logger` — Singleton за логване

#### 📝 Задачи (Tasks.md)
- 4 задачи: Factory за фигури, Strategy за форматиране, подобрена Factory с регистрация, Prototype клониране

#### 📂 Файлове:
- [`Theory.md`](Week_11_DesignPatterns/Theory.md)
- [`Examples.md`](Week_11_DesignPatterns/Examples.md)
- [`Tasks.md`](Week_11_DesignPatterns/Tasks.md)

</details>

---

### Част V — Напреднали теми (Седмици 12–14)

<details>
<summary><h3>📅 Седмица 12 — Обработка на изключения и RAII</h3></summary>

#### 📌 Тема: Надеждно управление на грешки и ресурси

#### 📚 Теория (Theory.md)
- **Проблеми на традиционната обработка** на грешки (error codes, errno)
- **Изключения:** `throw`, `try`, `catch` — синтаксис и семантика
- **Стандартна йерархия на изключения:** `std::exception`, `runtime_error`, `logic_error`
- **Потребителски изключения:** наследяване от `std::exception`
- **RAII (Resource Acquisition Is Initialization):** автоматично управление на ресурси
- **`noexcept`:** декларация за липса на изключения
- **Добри практики:** кога да хвърляме, кога да хващаме, re-throwing
- **Изключения и деструктори** — защо деструкторите не трябва да хвърлят

#### 💻 Примери (Examples.md)
- Потребителско изключение `FileOpenError`
- RAII файлов wrapper (`SafeFile`)
- Йерархия на изключения за бази данни

#### 📝 Задачи (Tasks.md)
- 4 задачи: валидация на вход, RAII обвивка, йерархия за магазин, обработка на числа от файл

#### 📂 Файлове:
- [`Theory.md`](Week_12_Exceptions_RAII/Theory.md)
- [`Examples.md`](Week_12_Exceptions_RAII/Examples.md)
- [`Tasks.md`](Week_12_Exceptions_RAII/Tasks.md)

</details>

<details>
<summary><h3>📅 Седмица 13 — SOLID принципи</h3></summary>

#### 📌 Тема: Пет принципа за добър обектно-ориентиран дизайн

#### 📚 Теория (Theory.md)
- **SRP (Single Responsibility Principle):** един клас — една отговорност
- **OCP (Open/Closed Principle):** отворен за разширение, затворен за модификация
- **LSP (Liskov Substitution Principle):** наследниците заменят базовия без проблем
- **ISP (Interface Segregation Principle):** малки, специализирани интерфейси
- **DIP (Dependency Inversion Principle):** зависимост от абстракции, не от конкретни класове
- **Връзка с Design Patterns** — как шаблоните имплементират SOLID
- **Често срещани грешки** и как да ги избягваме

#### 💻 Примери (Examples.md)
- Нарушаване и спазване на всеки принцип
- Рефакториране на „лош" код стъпка по стъпка

#### 📝 Задачи (Tasks.md)
- 5 задачи: по една за всеки SOLID принцип — намерете грешката и рефакторирайте

#### 📂 Файлове:
- [`Theory.md`](Week_13_SOLID/Theory.md)
- [`Examples.md`](Week_13_SOLID/Examples.md)
- [`Tasks.md`](Week_13_SOLID/Tasks.md)

</details>

<details>
<summary><h3>📅 Седмица 14 — Move семантика и Правилото на петте</h3></summary>

#### 📌 Тема: Ефективно прехвърляне на ресурси

#### 📚 Теория (Theory.md)
- **Lvalue vs. Rvalue:** какво е адрес, какво е временна стойност
- **Lvalue референции (`&`) vs. Rvalue референции (`&&`)**
- **Move конструктор:** прехвърляне вместо копиране
- **Move `operator=`:** семантика на преместващо присвояване
- **`std::move`:** кастване към rvalue
- **Правилото на петте (Rule of Five):** Big Four + move операции
- **Кога компилаторът генерира move операции**
- **`emplace_back` vs. `push_back`:** практическа разлика

#### 💻 Примери (Examples.md)
- `NumberArray` с пълно Rule of Five
- Демонстрация на `std::move` при swap
- `Tracer` клас за визуализация на move vs copy

#### 📝 Задачи (Tasks.md)
- 4 задачи: `ResourceHolder` с Rule of Five, move-only клас, `UniquePtr` шаблон, профилиране с `Tracer`

#### 📂 Файлове:
- [`Theory.md`](Week_14_MoveSemantics/Theory.md)
- [`Examples.md`](Week_14_MoveSemantics/Examples.md)
- [`Tasks.md`](Week_14_MoveSemantics/Tasks.md)

</details>

---

### 🧪 Подготовка за тестове и изпити

<details>
<summary><h3>📝 Подготовка за Тест 1</h3></summary>

**Покрити теми:** Седмици 1–6 (Структури, шаблони, ламбди, класове, Big Four, оператори)

- [`Tasks.md`](Test1_Preparation/Tasks.md) — Примерни задачи за първия тест

</details>

<details>
<summary><h3>📝 Подготовка за Тест 2</h3></summary>

**Покрити теми:** Седмици 7–11 (Файлове, итератори, наследяване, полиморфизъм, Design Patterns)

- [`Tasks.md`](Test2_Preparation/Tasks.md) — Примерни задачи за втория тест

</details>

<details>
<summary><h3>🎓 Подготовка за Изпит</h3></summary>

**Покрити теми:** Всички седмици (1–14)

- [`Tasks.md`](Exam_Preparation/Tasks.md) — Комплексни задачи за финален изпит

</details>

<details>
<summary><h3>🔄 Подготовка за Поправка</h3></summary>

**Покрити теми:** Всички седмици (1–14)

- [`Tasks.md`](Retake_Preparation/Tasks.md) — Задачи за поправителен изпит

</details>

---

## 📊 Обобщение

| Седмица | Тема | Фокус |
|---------|------|-------|
| **1** | Структури, Enum, Union, Typedef, Namespace | Потребителски типове |
| **2** | Шаблони и функции от по-висок ред | Обобщено програмиране |
| **3** | Ламбда функции и функционални концепции | Анонимни функции |
| **4** | Класове и капсулация | ООП основи |
| **5** | Конструктори, деструктори, Голямата четворка | Жизнен цикъл |
| **6** | Предефиниране на оператори | Синтактичен захар |
| **7** | Работа с файлове (текстови и двоични) | Файлов I/O |
| **8** | Итератори и композиция | Обхождане и has-a |
| **9** | Наследяване | Йерархии |
| **10** | Полиморфизъм и абстрактни класове | Виртуални функции |
| **11** | Шаблони за дизайн (Design Patterns) | Factory, Strategy, Singleton |
| **12** | Обработка на изключения и RAII | Грешки и ресурси |
| **13** | SOLID принципи | Добър дизайн |
| **14** | Move семантика и Правилото на петте | Ефективност |

---

## 🎯 Как да учите

1. **Четете Theory.md** — Разберете концепциите преди да пишете код
2. **Разгледайте Examples.md** — Анализирайте примерите, опитайте ги сами
3. **Решете Tasks.md** — Затвърдете с малък брой целенасочени задачи
4. **Дискутирайте** — Задавайте въпроси, обсъждайте алтернативи

---

## 📚 Допълнителни ресурси

| Ресурс | Линк |
|--------|------|
| C++ Reference | [cppreference.com](https://en.cppreference.com/) |
| Compiler Explorer | [godbolt.org](https://godbolt.org/) |
| C++ Insights | [cppinsights.io](https://cppinsights.io/) |
| Refactoring Guru (Design Patterns) | [refactoring.guru](https://refactoring.guru/design-patterns) |
| ISO C++ FAQ | [isocpp.org/faq](https://isocpp.org/wiki/faq) |

---

> 📌 **Забележка:** Този семинар е разработен на базата на семинарите от 2024–2025 г. с подобрена структура, по-ясни обяснения и по-добра организация.
