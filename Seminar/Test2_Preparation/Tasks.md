# Подготовка за Тест 2

> Обхват: Седмици 8–11 (Итератори, Композиция, Наследяване, Полиморфизъм, Design Patterns)

---

## Задача 1: Полиморфна колекция

Създайте абстрактен клас `Media` с:
- Чисто виртуални методи: `play()`, `getInfo()`, `getDuration()`, `clone()`
- Виртуален деструктор

Наследници:
- `Song` (заглавие, изпълнител, продължителност)
- `Podcast` (заглавие, водещ, брой епизоди, средна продължителност)
- `Audiobook` (заглавие, автор, продължителност, текущо позиция)

Създайте `Playlist` — хетерогенна колекция от `Media*` с Голяма четворка (използвайте `clone()`).

---

## Задача 2: Design Pattern — Factory + Strategy

Създайте система за компресиране на данни:

**Strategy:**
- `CompressionStrategy` (интерфейс) с `compress(const char*, char*, int&)` и `decompress(...)`
- `RLECompression` — Run-Length Encoding
- `XORCompression` — XOR с ключ

**Factory:**
- `CompressionFactory::create(const char* type)` — създава стратегия по име

**Контекст:**
- `FileCompressor` — приема `CompressionStrategy*` и има `compressFile()` / `decompressFile()`

---

## Задача 3: Итератор за свързан списък

Имплементирайте `LinkedList<int>` с:
- Вложен клас `Iterator` с `operator*`, `operator++`, `operator!=`
- Методи `begin()` и `end()`
- Поддръжка на range-based for цикъл

```cpp
LinkedList list;
list.push_back(1); list.push_back(2); list.push_back(3);
for (int val : list) {
    std::cout << val << " ";
}
```

---

## Задача 4: Йерархия с Diamond

Създайте:
- `Vehicle` (базов) — марка, година
- `LandVehicle` : virtual `Vehicle` — брой колела
- `WaterVehicle` : virtual `Vehicle` — водоизместимост
- `AmphibiousVehicle` : `LandVehicle`, `WaterVehicle`

Демонстрирайте правилната работа с виртуално наследяване. Създайте масив от `Vehicle*` с различни видове превозни средства.

---

## Задача 5: Observer Pattern

Имплементирайте `StockMarket` система:
- `Stock` — име, цена
- `StockObserver` — интерфейс с `onPriceChange(const char* name, double oldPrice, double newPrice)`
- `AlertObserver` — известява при промяна над 5%
- `LogObserver` — записва всяка промяна в масив

Когато цената се промени, всички наблюдатели се уведомяват.
