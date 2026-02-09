# Седмица 13 — Задачи за затвърждаване

## Задача 1: Рефакторинг с SRP

Даден е клас `Invoice`:

```cpp
class Invoice {
    Item items[100];
    int count;
public:
    double calculateTotal();     // Бизнес логика
    void printInvoice();         // Конзолен изход  
    void saveToFile(const char*);// Работа с файлове
    void sendByEmail(const char*);// Мрежова комуникация
};
```

Рефакторирайте го, спазвайки SRP. Създайте отделни класове за всяка отговорност.

---

## Задача 2: Разширяем филтър (OCP)

Създайте система за филтриране на продукти:

```cpp
class Product {
    char name[100];
    char color[20];
    double price;
};
```

Имплементирайте `Filter` интерфейс и конкретни филтри:
- `ColorFilter` — по цвят
- `PriceRangeFilter` — по ценови диапазон
- `AndFilter` — комбинира два филтъра с AND логика

Добавянето на нов филтър не трябва да изисква промяна на съществуващ код.

---

## Задача 3: LSP проверка

Обяснете дали следната йерархия нарушава LSP и защо:

```cpp
class Bird {
public:
    virtual void fly() = 0;
};

class Penguin : public Bird {
public:
    void fly() override {
        throw std::logic_error("Пингвините не летят!");
    }
};
```

Предложете решение, което не нарушава LSP.

---

## Задача 4: Пълен SOLID пример

Проектирайте система за уведомления (`NotificationSystem`), която:
- **SRP**: Отделни класове за всяка отговорност
- **OCP**: Лесно добавяне на нов канал (email, SMS, push)
- **ISP**: Не всички канали поддържат форматиране
- **DIP**: `NotificationService` зависи от абстракция `IChannel`

Имплементирайте поне два канала и демонстрирайте, че добавянето на трети не променя съществуващия код.
