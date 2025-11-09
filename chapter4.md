# Алгоритмы сортировки — методический материал (Markdown для GitHub)

Материал оформлен на основе рабочей тетради «Алгоритмы — Рабочая тетрадь 4» (семестр 1, 2025–2026). 

---

## Содержание

1. [Введение — базовые понятия](#введение---базовые-понятия)
2. [Таблица сложности и свойства](#таблица-сложности-и-свойства)
3. [Операция `swap` — пример и измерения (Пример 1)](#операция-swap---пример-и-измерения-пример-1)
4. [Пузырьковая сортировка (Bubble Sort) — задание 1](#пузырьковая-сортировка-bubble-sort---задание-1)
5. [Шейкерная сортировка (Cocktail / Shaker) — задание 2*](#шейкерная-сортировка-cocktail--shaker---задание-2)
6. [Сортировка вставками (Insertion Sort) — задание 3](#сортировка-вставками-insertion-sort---задание-3)
7. [Сортировка выбором (Selection Sort) — задание 4](#сортировка-выбором-selection-sort---задание-4)
8. [Сортировка Шелла (Shell Sort) — задание 5*](#сортировка-шелла-shell-sort---задание-5)
9. [Гномья сортировка (Gnome Sort) — задание 6](#гномья-сортировка-gnome-sort---задание-6)
10. [Функции + измерения времени (задание 7*)](#функции--измерения-времени-задание-7)
11. [Бинарная вставка — улучшение Insertion (задание 8*)](#бинарная-вставка----улучшение-insertion-задание-8)
12. [Как оформить на GitHub / структура репозитория](#как-оформить-на-github--структура-репозитория)
13. [Примечания и рекомендации для студентов](#примечания-и-рекомендации-для-студентов)

---

# Введение — базовые понятия

**Алгоритм сортировки** — способ упорядочить элементы массива по возрастанию (или убыванию). В практических лабораторных задачах обычно нужно:

* реализовать алгоритм (часто на C++),
* оформить в виде функции,
* измерить время выполнения на маленьких и больших массивах,
* проанализировать сложность.

**Важные свойства алгоритмов:**

* *in-place*: использует O(1) дополнительной памяти (кроме стека и небольших переменных);
* *стабильность*: остаётся ли относительный порядок равных элементов;
* число сравнений/перестановок (ключевой ресурс).

---

# Таблица сложности и свойства

| Алгоритм          |                     Среднее время | Худшее время | Память | Стабильность | Примечание                       |
| ----------------- | --------------------------------: | -----------: | -----: | -----------: | -------------------------------- |
| Bubble            |                             O(n²) |        O(n²) |   O(1) |           Да | прост и учебен                   |
| Cocktail (Shaker) |                             O(n²) |        O(n²) |   O(1) |           Да | Bi-directional bubble            |
| Insertion         |                             O(n²) |        O(n²) |   O(1) |           Да | хорошо для почти отсортированных |
| Selection         |                             O(n²) |        O(n²) |   O(1) |          Нет | минимизирует число обменов       |
| Shell             | зависит от последовательности gap |            — |   O(1) |          Нет | намного быстрее на средних n     |
| Gnome             |                             O(n²) |        O(n²) |   O(1) |           Да | реализация ↔ один цикл (можно)   |



---


## 1. Что такое сортировка

**Сортировка** — алгоритм, упорядочивающий набор элементов по некоторому критерию (обычно по ключу: возрастание/убывание). В программировании сортировки — одна из базовых операций: поиска, слияния, индексации, удаления дубликатов и т.д.

Ключевые вопросы: корректность, сложность (время и память), стабильность и пригодность для конкретного набора данных.

---

## 2. Классификация алгоритмов сортировки 

1. **По модели сравнений**

   * *Сравнительные*: Quicksort, Mergesort, Heapsort, Insertion, Selection, Bubble и т.д. Нижняя граница сравнений — Ω(n log n) для сортировки произвольных ключей.
   * *Немного сравнивающие / не сравнивающие*: Counting sort, Radix sort — дают лучше O(n) при дополнительных предположениях (малограда или целые ключи в ограниченном диапазоне).

2. **По месту использования памяти**

   * *In-place* (на месте): Quicksort (обычно), Heapsort,Insertion — O(1) дополнительной памяти.
   * *Не in-place*: Mergesort (классический) — требует O(n) доп. памяти (хотя есть in-place варианты с усложнением).

3. **Стабильность**

   * *Стабильные*: Mergesort, Insertion, Bubble (в простых реализациях), std::stable_sort.
   * *Нестабильные*: Quicksort (обычные реализации), Heapsort, Selection.

4. **Внутренние vs внешние**

   * *Внутренние* — весь массив помещается в ОЗУ (обычные алгоритмы).
   * *Внешние* — данные слишком велики для ОЗУ — используются внешние алгоритмы (external merge sort).

---

## 3. Какие сортировки реально используют на практике

* **std::sort (C++)** — реализация Introsort: быстрая комбинация QuickSort + HeapSort + Insertion; работает in-place, нестабильна, среднее O(n log n), худшее O(n log n) (гарантируется переходом на HeapSort).
* **std::stable_sort** — обычно реализован через Merge Sort; стабильный, требует O(n) доп. памяти, O(n log n) время.
* **Quicksort** — быстрый на среднем, простая реализация, но надо правильно выбирать pivot (и иногда использовать случайный pivot или медиану).
* **Merge sort** — стабильный и предсказуемый; часто используется для `stable_sort` и внешних сортировок.
* **Heapsort** — гарантированное O(n log n) и in-place; редко используется как основная реализация std::sort, но полезен если нужно ограничение памяти и худший случай O(n log n).
* **Counting / Radix sort** — для ключей типа int/string при ограниченном диапазоне — линейные по времени в практических сценариях.
* **Hybrid алгоритмы (Introsort)** — реальны и широко используются (например — std::sort).

---

## 4. Где готовые реализации в C++

* `#include <algorithm>`

  * `std::sort(begin, end, comp)` — Introsort (обычно).
  * `std::stable_sort(begin, end, comp)` — стабильный (merge).
  * `std::partial_sort`, `std::nth_element`, `std::is_sorted`, `std::sort_heap`, `std::make_heap`, `std::push_heap`, `std::pop_heap`.
* `#include <iterator>` — для работы с итераторами (RandomAccessIterator требуется для std::sort).
* `#include <functional>` — для предикатов/функторов.

---

## 5. Три типа «указателей» в C++ и их влияние на код

Пользователи часто выделяют **3 классических подхода** к доступу/ссылкам на объекты:

1. **Raw pointers (`T*`)**

   * Могут быть `nullptr`, копируются по адресу.
   * Нужны для низкоуровневого управления памятью, C-совместимости.
   * Требуют заботы об владении/освобождении памяти (опасность утечек / двойного удаления).
   * Влияние на сортировки: сортировка указателей (обмен адресов) — очень дешёвая операция; но сортируя указатели, нужно помнить о сроке жизни объектов.

2. **References (`T&` / `T&&`)** (не совсем указатель, но pointer-like)

   * Не могут быть `nullptr` (если только не использовать ссылку на невалидный объект).
   * Удобны для передачи объектов в функции без копирования.
   * Не хранятся в контейнерах (нельзя `std::vector<T&>`), поэтому для контейнерной сортировки обычно используются указатели или smart pointers.

3. **Smart pointers (`std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`)**

   * `unique_ptr<T>` — единоличное владение; перемещаемый, не копируемый.
   * `shared_ptr<T>` — подсчёт ссылок; копирование инкрементирует счётчик; возможно циклическое удержание объектов (требует weak_ptr для разрыва циклов).
   * `weak_ptr<T>` — не владеет объектом; проверяется через `.lock()`.
   * Влияние: сортировка контейнера `std::vector<std::unique_ptr<T>>` требует использования `std::move` и компаратора, работающего по `*ptr` или по ключу; сортировка smart pointers дешевле по копированию адреса, но `shared_ptr` несёт накладные расходы на атомарный/неатомарный инкремент/декремент счётчика (в многопоточных сценариях — дополнительная стоимость).

Дополнительно:

* **const correctness**: `T* const` vs `const T*` — влияет на то, что можно менять при сортировке.
* **Pointer stability**: сортировка ссылок/указателей не меняет объекты, а только их порядок — это полезно, если объекты тяжёлые для копирования.

---

## 6. Пошаговые примеры кода 

---

### Уровень 1 — (очень просто) пузырёк на `vector<int>` — сложность O(n²)

```cpp
// level1_bubble.cpp
#include <vector>
#include <iostream>

// Простая пузырьковая сортировка — учебный пример.
// Время: O(n*n), память: O(1), стабильная.
void bubble_sort(std::vector<int>& a) {
    int n = (int)a.size();
    for (int i = 0; i < n - 1; ++i) {
        bool swapped = false;
        for (int j = 0; j < n - 1 - i; ++j) {
            if (a[j] > a[j+1]) {
                std::swap(a[j], a[j+1]); // обмен соседних
                swapped = true;
            }
        }
        if (!swapped) break; // уже отсортировано
    }
}

int main() {
    std::vector<int> v = {5, 2, 9, 1, 5};
    bubble_sort(v);
    for (int x : v) std::cout << x << ' ';
    // Вывод: 1 2 5 5 9
}
```
---

### Уровень 2 — (универсально) шаблонная Insertion сортировка (generic + стабильность)

```cpp
// level2_insertion_template.cpp
#include <vector>
#include <iostream>
#include <functional> // std::less

// Вставки — стабильный алгоритм, хорош для почти отсортированных данных.
template<typename RandomIt, typename Compare = std::less<>>
void insertion_sort(RandomIt first, RandomIt last, Compare comp = Compare()) {
    for (RandomIt it = first + 1; it != last; ++it) {
        auto key = *it;               // копия значения
        RandomIt j = it;
        while (j != first && comp(key, *(j - 1))) {
            *j = *(j - 1);           // сдвигаем вправо
            --j;
        }
        *j = key;                    // вставляем
    }
}

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};
    insertion_sort(v.begin(), v.end());
    for (int x : v) std::cout << x << ' ';
}
```

`RandomIt` — итератор произвольного доступа; `Compare` — компаратор. Обсудите где эта сортировка эффективна (почти-сортированные массивы).

---

### Уровень 3 — (практично) использование стандартных реализаций `std::sort` и `std::stable_sort`

```cpp
// level3_std_sort.cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, -2, 33, 7, 7, 2};
    // std::sort: быстрый Introsort, требует RandomAccessIterator
    std::sort(v.begin(), v.end()); // по возрастанию

    // stable сортировка: сохраняет относительный порядок равных элементов
    // std::stable_sort(v.begin(), v.end());

    for (int x : v) std::cout << x << ' ';
    // Вывод: -2 2 7 7 10 33
}
```

`std::sort` — быстрый, но нестабилен; `std::stable_sort` — стабилен, но требует доп. памяти.

---

### Уровень 4 — (сложнее) сортировка структур по ключу

```cpp
// level4_structs.cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;
};

int main() {
    std::vector<Person> people = {
        {"Alice", 30}, {"Bob", 20}, {"Charlie", 30}, {"Dave", 25}
    };

    // Если хочется сохранить порядок людей с одинаковым age, используем stable_sort
    std::stable_sort(people.begin(), people.end(),
        [](const Person& a, const Person& b){ return a.age < b.age; });

    for (auto &p : people) std::cout << p.name << '(' << p.age << ") ";
    // Выведет: Bob(20) Dave(25) Alice(30) Charlie(30)
    // Alice и Charlie сохранили исходный порядок среди равных age.
}
```

Стабильность нужна, когда у вас вторичный важный критерий — например, сохранение первоначального порядка.

---

### Уровень 5 — (эффективность) сортировка тяжёлых объектов: сортируем по ключу, используем move semantics

```cpp
// level5_move_semantics.cpp
#include <algorithm>
#include <vector>
#include <string>
#include <iostream>

// "Тяжёлый" объект — например, большой std::string.
struct Big {
    std::string data;
    int key;
    Big(std::string d, int k) : data(std::move(d)), key(k) {}
    // допускаем перемещения и копирование
};

int main() {
    std::vector<Big> arr;
    arr.emplace_back(std::string(1000, 'a'), 3);
    arr.emplace_back(std::string(1000, 'b'), 1);
    arr.emplace_back(std::string(1000, 'c'), 2);

    // Сортируем по ключу, но минимизируем лишние копии:
    std::sort(arr.begin(), arr.end(),
        [](const Big& A, const Big& B){ return A.key < B.key; });

    // Если хотим перемещать элементы вручную в кастомном алгоритме — используем std::move
    // В данном случае std::sort использует swaps/assignments, которые используют move, если он доступен.

    for (auto &x : arr) std::cout << x.key << ' ';
}
```

Современные контейнерные алгоритмы часто используют move-операции (если доступны) — это снижает стоимость копирования тяжёлых объектов.

---

### Уровень 6 — (указатели и владение) сортировка контейнера `vector<unique_ptr<T>>`

```cpp
// level6_unique_ptr.cpp
#include <algorithm>
#include <vector>
#include <memory>
#include <iostream>

struct Item {
    int key;
    Item(int k) : key(k) {}
};

int main() {
    std::vector<std::unique_ptr<Item>> items;
    items.push_back(std::make_unique<Item>(5));
    items.push_back(std::make_unique<Item>(2));
    items.push_back(std::make_unique<Item>(8));

    // Сортируем по значению *ptr->key; smart pointers перемещать/копировать не нужно
    std::sort(items.begin(), items.end(),
        [](const std::unique_ptr<Item>& a, const std::unique_ptr<Item>& b){
            return a->key < b->key;
        });

    for (auto &p : items) std::cout << p->key << ' ';
    // Вывод: 2 5 8
}
```

Важное: `unique_ptr` — нельзя копировать в вектор, можно только перемещать; сортируя контейнер указателей, вы меняете только адреса (быстро) — объекты остаются.

---

### Уровень 7 — (когда O(n log n) слишком мало) Counting / Radix sort — пример counting для non-negative ints

```cpp
// level7_counting_sort.cpp
#include <vector>
#include <iostream>
#include <algorithm>

// Counting sort: O(n + k), где k = max_value - min_value + 1
// Подходит если ключи — целые в небольшом диапазоне.
void counting_sort(std::vector<int>& a, int max_value) {
    std::vector<int> cnt(max_value + 1, 0);
    for (int x : a) ++cnt[x];
    int idx = 0;
    for (int val = 0; val <= max_value; ++val) {
        while (cnt[val]--) {
            a[idx++] = val;
        }
    }
}

int main() {
    std::vector<int> v = {3,1,2,3,0,2,1};
    counting_sort(v, 3);
    for (int x : v) std::cout << x << ' ';
    // Вывод: 0 1 1 2 2 3 3
}
```

Для больших диапазонов counting sort неэффективен по памяти; radix sort — последовательное применение counting по разрядам.

---

# Операция `swap` — пример и измерения (Пример 1)

Ниже — два способа обмена двух элементов: использование `std::swap` и собственная функция `my_swap`. Также пример замера времени.

```cpp
// swap_examples.cpp
#include <iostream>
#include <vector>
#include <algorithm> // std::swap
#include <chrono>
#include <random>

void my_swap(int &a, int &b) {
    int t = a;
    a = b;
    b = t;
}

template<typename F>
long long measure_swap(F swap_func, std::vector<std::pair<int,int>> &pairs, int iterations=1) {
    auto start = std::chrono::high_resolution_clock::now();
    for (int it=0; it<iterations; ++it) {
        for (auto &p : pairs) {
            int x = p.first;
            int y = p.second;
            swap_func(x, y);
            // prevent optimizer removing code:
            if (x == -1) std::cout << "";
        }
    }
    auto end = std::chrono::high_resolution_clock::now();
    return std::chrono::duration_cast<std::chrono::microseconds>(end-start).count();
}

int main() {
    std::mt19937 rng(123);
    std::uniform_int_distribution<int> dist(0, 1'000'000);
    std::vector<std::pair<int,int>> pairs(1'000'00);
    for (auto &p : pairs) p = {dist(rng), dist(rng)};

    auto t1 = measure_swap([](int &a, int &b){ std::swap(a,b); }, pairs, 50);
    auto t2 = measure_swap([](int &a, int &b){ int tmp=a; a=b; b=tmp; }, pairs, 50);

    std::cout << "std::swap: " << t1 << " us\n";
    std::cout << "my_swap:   " << t2 << " us\n";
    return 0;
}
```

**Примечание:** Разница часто минимальна, компилятор может инлайнить `std::swap` или оптимизировать пользовательский код — измеряйте корректно (чтобы не оптимизировалось в пустоту).

---

# Пузырьковая сортировка (Bubble Sort)

**Идея:** многократные проходы по массиву, при которых соседние элементы сравниваются и при необходимости меняются местами. После каждого прохода последний элемент — максимальный среди неотсортированных.

**Псевдокод:**

```
for i = 0 to n-2
  swapped = false
  for j = 0 to n-2-i
    if a[j] > a[j+1] then swap(a[j], a[j+1]); swapped = true
  if not swapped then break
```

**C++ реализация:**

```cpp
void bubble_sort(std::vector<int>& a) {
    int n = a.size();
    for (int i = 0; i < n-1; ++i) {
        bool swapped = false;
        for (int j = 0; j < n-1-i; ++j) {
            if (a[j] > a[j+1]) {
                std::swap(a[j], a[j+1]);
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}
```

**Пример:**
Input: `[5, 1, 4, 2, 8]` → Output: `[1,2,4,5,8]`

---

# Шейкерная сортировка (Cocktail / Shaker) 

**Идея:** двунаправленный проход — сначала слева направо (как пузырёк), затем справа налево; границы рабочей части сужаются на основе последнего обмена.

**Псевдокод:**

```
left = 0; right = n-1
while left < right:
  last = 0
  for i = left to right-1:
    if a[i] > a[i+1]: swap; last = i
  right = last
  for i = right downto left+1:
    if a[i-1] > a[i]: swap; last = i
  left = last
```

**C++ реализация:**

```cpp
void shaker_sort(std::vector<int>& a) {
    int left = 0, right = (int)a.size() - 1;
    while (left < right) {
        int last = 0;
        for (int i = left; i < right; ++i) {
            if (a[i] > a[i+1]) { std::swap(a[i], a[i+1]); last = i; }
        }
        right = last;
        for (int i = right; i > left; --i) {
            if (a[i-1] > a[i]) { std::swap(a[i-1], a[i]); last = i; }
        }
        left = last;
    }
}
```

---

# Сортировка вставками (Insertion Sort)

**Идея:** строим отсортированную часть слева; каждый следующий элемент вставляем в нужное место в уже отсортированной части.

**Псевдокод:**

```
for i = 1 to n-1
  key = a[i]
  j = i-1
  while j >= 0 and a[j] > key
    a[j+1] = a[j]; j = j-1
  a[j+1] = key
```

**C++ реализация:**

```cpp
void insertion_sort(std::vector<int>& a) {
    for (int i = 1; i < (int)a.size(); ++i) {
        int key = a[i];
        int j = i - 1;
        while (j >= 0 && a[j] > key) {
            a[j+1] = a[j];
            --j;
        }
        a[j+1] = key;
    }
}
```

**Примечание:** годится для почти отсортированных данных.

---

# Сортировка выбором (Selection Sort) 

**Идея:** на каждой итерации выбираем минимальный элемент в неотсортированной части и ставим его на следующую позицию.

**Псевдокод:**

```
for i = 0 to n-2
  min_idx = i
  for j = i+1 to n-1
    if a[j] < a[min_idx] then min_idx = j
  swap(a[i], a[min_idx])
```

**C++ реализация:**

```cpp
void selection_sort(std::vector<int>& a) {
    int n = a.size();
    for (int i = 0; i < n-1; ++i) {
        int min_idx = i;
        for (int j = i+1; j < n; ++j)
            if (a[j] < a[min_idx]) min_idx = j;
        std::swap(a[i], a[min_idx]);
    }
}
```

**Примечание:** количество обменов минимальное (≤ n), но сравнений — O(n²). Нестабильна.

---

# Сортировка Шелла (Shell Sort)

**Идея:** улучшение вставки: элементы сравниваются на расстоянии `gap`, затем gap уменьшается, завершается `gap=1` (обычная вставка).

**Простая стратегия gap:** `gap = n/2; while (gap > 0) { ...; gap /= 2; }`

**C++ реализация (простая версия):**

```cpp
void shell_sort(std::vector<int>& a) {
    int n = a.size();
    for (int gap = n/2; gap > 0; gap /= 2) {
        for (int i = gap; i < n; ++i) {
            int tmp = a[i];
            int j = i;
            while (j >= gap && a[j-gap] > tmp) {
                a[j] = a[j-gap];
                j -= gap;
            }
            a[j] = tmp;
        }
    }
}
```

**Примечание:** эффективность сильно зависит от последовательности gap (на практике используют Хиббарда, Седжвика и др.).

---

# Гномья сортировка (Gnome Sort) 

**Идея:** элемент «шагает» влево до своего места, при нарушении порядка — обмен и шаг назад; иначе шаг вперед. Можно реализовать одним циклом `while` — это условие задания (не более одного цикла).

**Одноцикловая реализация (соответствует требованию «не более одного цикла»):**

```cpp
void gnome_sort(std::vector<int>& a) {
    int n = a.size();
    int i = 1;
    while (i < n) {
        if (i == 0 || a[i-1] <= a[i]) {
            ++i;
        } else {
            std::swap(a[i-1], a[i]);
            --i;
        }
    }
}
```

**Примечание:** это действительно один цикл `while` — внутри присутствуют только условные переходы и один обмен.

---

# Функции + измерения времени 

**Требование:** оформить алгоритмы 1–6 как функции и измерить время выполнения каждой на одном и том же исходном массиве (для маленького и большого размера).

Рекомендации по измерениям:

* Используйте `std::chrono::high_resolution_clock`.
* Для честности измеряйте на копиях одного и того же исходного массива (чтобы каждый алгоритм получал одинаковые входные данные).
* Повторяйте измерение несколько раз и усредняйте (для уменьшения шума).
* Генерация данных:

  * маленький: `n = 100` — для медленных O(n²) алгоритмов —
  * большой: `n = 10000` (или меньше, если эти алгоритмы слишком медленные на вашей машине).
* Для Shell и т.п. можно использовать большие n — они лучше масштабируют.

**Примерный измерительный каркас:**

```cpp
#include <chrono>
#include <random>
#include <iostream>
#include <vector>
#include <functional>

long long time_run(std::function<void(std::vector<int>&)> f, std::vector<int> a, int repeats=3) {
    long long sum = 0;
    for (int r=0; r<repeats; ++r) {
        auto copy = a;
        auto start = std::chrono::high_resolution_clock::now();
        f(copy);
        auto end = std::chrono::high_resolution_clock::now();
        sum += std::chrono::duration_cast<std::chrono::milliseconds>(end-start).count();
    }
    return sum / repeats;
}

int main() {
    // generate base array
    std::mt19937 rng(42);
    std::uniform_int_distribution<int> d(0, 1'000'000);
    int n_small = 200, n_large = 10000;
    std::vector<int> small(n_small), large(n_large);
    for (int &x : small) x = d(rng);
    for (int &x : large) x = d(rng);

    // e.g. call time_run(bubble_sort, small)
}
```

---

# Бинарная вставка — улучшение Insertion 

**Идея:** при вставке элемента в отсортированную часть искать позицию бинарным поиском (O(log n)), но смещение элементов всё ещё O(n), поэтому сложность остаётся O(n²) в целом, но число сравнений уменьшается.

**Псевдокод (для поиска позиции):**

```
for i = 1..n-1:
  key = a[i]
  pos = binary_search_position(a[0..i-1], key)
  shift a[pos..i-1] right by 1
  a[pos] = key
```

**C++ реализация (binary insertion):**

```cpp
int binary_search_pos(const std::vector<int>& a, int key, int high) {
    int low = 0;
    while (low < high) {
        int mid = (low + high) / 2;
        if (a[mid] <= key) low = mid + 1;
        else high = mid;
    }
    return low;
}

void binary_insertion_sort(std::vector<int>& a) {
    int n = a.size();
    for (int i = 1; i < n; ++i) {
        int key = a[i];
        int pos = binary_search_pos(a, key, i);
        // shift right
        for (int j = i; j > pos; --j) a[j] = a[j-1];
        a[pos] = key;
    }
}
```

**Сравнение:** меньше сравнений — полезно, если сравнения дорогие; но смещения остаются.


# Примечания и рекомендации 

* Всегда тестируйте алгоритмы на различных входах: случайных, уже отсортированных, обратном порядке, с большим количеством одинаковых элементов.
* Для замеров используйте компиляцию с оптимизациями (`-O2`/`-O3`) и фиксированный сид для генератора случайных чисел.
* Обращайте внимание на стабильность алгоритмов, если сортируете структуры с ключом и связанной информацией.
* Для больших данных используйте эффективные алгоритмы: `std::sort` (обычно быстрый introsort) или `std::stable_sort` (слияние).
* Оформляйте код комментариями, поясняющими ключевые шаги алгоритма (особенно там, где логика не тривиальна).
Скажите, что предпочитаете — дам готовые файлы/архив или вставлю весь набор кодов прямо здесь.
