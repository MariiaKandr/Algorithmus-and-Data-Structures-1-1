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

# Пузырьковая сортировка (Bubble Sort) — задание 1

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

# Шейкерная сортировка (Cocktail / Shaker) — задание 2*

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

# Сортировка вставками (Insertion Sort) — задание 3

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

# Сортировка выбором (Selection Sort) — задание 4

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

# Сортировка Шелла (Shell Sort) — задание 5*

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

# Гномья сортировка (Gnome Sort) — задание 6

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

# Функции + измерения времени (задание 7*)

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

# Бинарная вставка — улучшение Insertion (задание 8*)

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
