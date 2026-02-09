# Седмица 11 — Задачи за затвърждаване

## Задача 1: Animal Factory

Създайте `AnimalFactory`, който по подаден низ (`"dog"`, `"cat"`, `"bird"`) създава съответния наследник на `Animal`. Всеки клас трябва да има метод `void makeSound() const`.

Напишете програма, в която потребителят въвежда видове животни и програмата ги създава, запазва в масив и накрая отпечатва звуците на всички.

---

## Задача 2: Strategy за форматиране

Създайте интерфейс `Formatter` с метод `void format(const char* text) const`.

Имплементирайте:
- `UpperCaseFormatter` — отпечатва с главни букви
- `LowerCaseFormatter` — отпечатва с малки букви
- `TitleCaseFormatter` — първата буква на всяка дума е главна

Създайте клас `TextEditor`, който приема `Formatter*` и има метод `void displayFormatted(const char* text)`.

---

## Задача 3: Event System (Observer)

Имплементирайте система за събития:

- `EventListener` — интерфейс с `virtual void onEvent(const char* eventName) = 0`
- `EventEmitter` — клас с методи `subscribe(EventListener*)`, `emit(const char* eventName)`

Тествайте с:
- `LogListener` — записва събитието в конзолата
- `CountListener` — брои колко пъти е получил събитие

---

## Задача 4: Prototype Registry

Създайте `ShapeRegistry`:
- Пази прототипи на различни `Shape` наследници, асоциирани с имена (напр. `"circle"`, `"rect"`)
- Метод `Shape* create(const char* name)` — клонира прототипа

Тествайте: регистрирайте `Circle(1)`, `Rectangle(2,3)` като прототипи. Създайте нови обекти от регистъра и проверете, че те са независими копия.
