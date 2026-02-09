# Шаблони на функции и функции от по-висок ред

## Въведение

Когато пишем една и съща логика за различни типове данни, дублираме код. Шаблоните (`templates`) решават този проблем, като позволяват да напишем функция *веднъж* и тя да работи за *произволен тип*. А функциите от по-висок ред ни позволяват да предаваме *поведение* като аргумент.

---

## 1. Шаблони на функции (`function templates`)

### Проблемът: дублиране на код

```cpp
int maxInt(int a, int b) { return (a > b) ? a : b; }
double maxDouble(double a, double b) { return (a > b) ? a : b; }
char maxChar(char a, char b) { return (a > b) ? a : b; }
// ... и така за всеки тип
```

### Решението: шаблон

```cpp
template <typename T>
T maxValue(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    std::cout << maxValue(3, 7) << std::endl;       // T = int
    std::cout << maxValue(3.14, 2.71) << std::endl;  // T = double
    std::cout << maxValue('a', 'z') << std::endl;    // T = char
}
```

### Как работи?

1. Компилаторът **не генерира код** за шаблона, докато не бъде извикан
2. При извикване, компилаторът **извежда типа** (`T`) от аргументите
3. Генерира се **конкретна инстанция** на функцията за този тип
4. Това се случва по **време на компилация** — няма runtime overhead

### Множество шаблонни параметри

```cpp
template <typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

auto result = add(3, 4.5);  // T = int, U = double, return = double
```

### Явно задаване на типа

```cpp
template <typename T>
T convert(double value) {
    return static_cast<T>(value);
}

int n = convert<int>(3.7);       // T трябва да се зададе явно
float f = convert<float>(3.7);
```

### Ограничения на шаблоните

- Типът `T` трябва да поддържа операциите, използвани в шаблона
- Грешките се проявяват при **инстанциране**, не при дефиниране
- Шаблонният код обикновено стои в **header файлове**

> 🧠 **Дискусия:** Какво става, ако извикаме `maxValue` с тип, който не поддържа `operator>`? Кога ще получим грешка — при компилация или при изпълнение?

---

## 2. Указатели към функции

### Синтаксис

Указателят към функция съхранява **адреса** на функция и може да бъде извикан:

```cpp
int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

// Деклариране на указател към функция
int (*operation)(int, int);

operation = add;
std::cout << operation(3, 4) << std::endl;  // 7

operation = multiply;
std::cout << operation(3, 4) << std::endl;  // 12
```

### Предаване на функция като аргумент

```cpp
void applyToArray(int arr[], int size, int (*func)(int)) {
    for (int i = 0; i < size; i++) {
        arr[i] = func(arr[i]);
    }
}

int doubleIt(int x) { return x * 2; }
int negate(int x) { return -x; }

int main() {
    int arr[] = {1, 2, 3, 4, 5};
    applyToArray(arr, 5, doubleIt);   // {2, 4, 6, 8, 10}
    applyToArray(arr, 5, negate);     // {-2, -4, -6, -8, -10}
}
```

> 💡 **Ключова идея:** Функцията `applyToArray` не знае *какво* ще прави с всеки елемент — тя приема *поведението* отвън. Това е **функция от по-висок ред**.

---

## 3. `std::function` — универсален обвиващ тип

### Проблемът с указателите

Указателите към функции не могат да обгърнат ламбди с capture, функтори (обекти с `operator()`), или методи на клас. `std::function` решава това.

```cpp
#include <functional>

// Приема всичко, което може да се извика с (int, int) -> int
std::function<int(int, int)> operation;

operation = add;                            // обикновена функция
operation = [](int a, int b) { return a - b; };  // ламбда
```

### Като параметър на функция

```cpp
#include <functional>

void processArray(int arr[], int size, std::function<int(int)> transform) {
    for (int i = 0; i < size; i++) {
        arr[i] = transform(arr[i]);
    }
}

int main() {
    int arr[] = {1, 2, 3, 4, 5};
    
    // С обикновена функция
    processArray(arr, 5, doubleIt);
    
    // С ламбда
    int factor = 3;
    processArray(arr, 5, [factor](int x) { return x * factor; });
}
```

> 💡 **`std::function` vs указател към функция:**
> - `std::function` е по-гъвкав (поддържа ламбди, функтори)
> - Указателите са по-ефективни (няма heap allocation)
> - Като правило: използвайте `std::function`, когато имате нужда от гъвкавост

---

## 4. Функции от по-висок ред

Функция от по-висок ред е функция, която:
- **Приема** друга функция като аргумент, и/или
- **Връща** функция като резултат

### Класически примери

```cpp
// filter — избира елементи, отговарящи на условие
template <typename T>
int filter(const T arr[], int size, T result[], std::function<bool(const T&)> predicate) {
    int count = 0;
    for (int i = 0; i < size; i++) {
        if (predicate(arr[i])) {
            result[count++] = arr[i];
        }
    }
    return count;
}

// forEach — прилага операция върху всеки елемент
template <typename T>
void forEach(T arr[], int size, std::function<void(T&)> action) {
    for (int i = 0; i < size; i++) {
        action(arr[i]);
    }
}
```

### Използване

```cpp
int numbers[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
int evens[10];

int count = filter<int>(numbers, 10, evens, [](const int& x) { return x % 2 == 0; });
// evens = {2, 4, 6, 8, 10}, count = 5

forEach<int>(numbers, 10, [](int& x) { x *= x; });
// numbers = {1, 4, 9, 16, 25, 36, 49, 64, 81, 100}
```

---

## Обобщение

| Концепция | Какво прави | Кога да се ползва |
|-----------|------------|-------------------|
| `template` | Обобщава функция за произволен тип | Когато логиката е еднаква за различни типове |
| Указател към функция | Съхранява адрес на функция | Прости callbacks, C-съвместимост |
| `std::function` | Обгръща произволен извикваем обект | Ламбди с capture, функтори |
| Функции от по-висок ред | Приемат/връщат функции | `filter`, `forEach`, `sort` с компаратор |

> 🧠 **Ключов takeaway:** Шаблоните абстрахират **типа**, а функциите от по-висок ред абстрахират **поведението**. Заедно дават огромна изразителна сила.
