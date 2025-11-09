# Учебное занятие — Длинная арифметика (BigIntegers)

*Материал основан на вашей рабочей тетради.* 

**Цель:** научиться хранить и выполнять базовые операции с целыми числами произвольной длины (сложение, вычитание, умножение, деление), понять идею Карацубы и ускоренное побитовое сложение.

---

## План занятия

1. Представление больших чисел
2. Сравнение
3. Сложение и вычитание (в столбик)
4. Умножение: школьный алгоритм и Карацуба
5. Деление (школьный алгоритм)
6. Ускоренное двоичное сложение (идейно)
7. Примеры кода и тесты
8. Задания

---

## Формат представления

* В этом уроке `BigInt` представлен как `std::vector<int>` в десятичной системе (база = 10).
* **LSB-first** — младший разряд (единицы) хранится в `digits[0]`, старший — в конце. Такой порядок удобен для операций с переносом/заимствованием.

---

## Полный пример кода (готово для копирования в `main.cpp`)

```cpp
// main.cpp
// Пример реализации операций для больших целых чисел (base = 10).
// Компиляция: g++ -std=c++17 main.cpp -O2 -o main

#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <cassert>
#include <ctime>
#include <cstdlib>

using namespace std;

// Тип для больших целых (каждый элемент 0..9, LSB в index 0)
using BigInt = vector<int>;

// ---------- Утилиты ----------
void trim(BigInt &a) {
    // Удалить ведущие нули (старшие разряды)
    while (a.size() > 1 && a.back() == 0) a.pop_back();
}

BigInt fromString(const string &s) {
    // Создать BigInt из строки "12345"
    BigInt a;
    for (int i = (int)s.size() - 1; i >= 0; --i) {
        if (!isdigit(s[i])) continue;
        a.push_back(s[i] - '0');
    }
    if (a.empty()) a.push_back(0);
    trim(a);
    return a;
}

string toString(const BigInt &a) {
    // Преобразовать BigInt в строку "12345"
    string s;
    for (int i = (int)a.size() - 1; i >= 0; --i) s.push_back(char('0' + a[i]));
    if (s.empty()) return "0";
    return s;
}

BigInt randomBigInt(int n) {
    // Генерация случайного числа длины n (старшая цифра != 0)
    assert(n >= 1);
    BigInt a(n);
    for (int i = 0; i < n; ++i) a[i] = rand() % 10;
    if (a.back() == 0) a.back() = 1 + (rand() % 9);
    trim(a);
    return a;
}

// ---------- Сравнение ----------
int compare(const BigInt &a, const BigInt &b) {
    // Возвращает: 1 если a>b, 0 если a==b, -1 если a<b
    if (a.size() != b.size()) return a.size() > b.size() ? 1 : -1;
    for (int i = (int)a.size() - 1; i >= 0; --i)
        if (a[i] != b[i]) return (a[i] > b[i]) ? 1 : -1;
    return 0;
}

// ---------- Сложение ----------
BigInt add(const BigInt &a, const BigInt &b) {
    int n = max(a.size(), b.size());
    BigInt res;
    res.reserve(n + 1);
    int carry = 0;
    for (int i = 0; i < n; ++i) {
        int da = (i < (int)a.size()) ? a[i] : 0;
        int db = (i < (int)b.size()) ? b[i] : 0;
        int sum = da + db + carry;
        res.push_back(sum % 10);
        carry = sum / 10;
    }
    if (carry) res.push_back(carry);
    trim(res);
    return res;
}

// ---------- Вычитание (a >= b) ----------
BigInt sub(const BigInt &a, const BigInt &b) {
    BigInt res;
    res.reserve(a.size());
    int borrow = 0;
    for (int i = 0; i < (int)a.size(); ++i) {
        int da = a[i];
        int db = (i < (int)b.size()) ? b[i] : 0;
        int cur = da - db - borrow;
        if (cur < 0) { cur += 10; borrow = 1; } else borrow = 0;
        res.push_back(cur);
    }
    trim(res);
    return res;
}

// Удобная обёртка: возвращает (результат, знак)
pair<BigInt,int> subSigned(const BigInt &a, const BigInt &b) {
    int cmp = compare(a,b);
    if (cmp == 0) return {BigInt{0}, 0};
    if (cmp > 0) return {sub(a,b), +1};
    else return {sub(b,a), -1};
}

// ---------- Школьное умножение O(n*m) ----------
BigInt multiplySchoolbook(const BigInt &a, const BigInt &b) {
    BigInt res(a.size() + b.size(), 0);
    for (size_t i = 0; i < a.size(); ++i) {
        int carry = 0;
        for (size_t j = 0; j < b.size() || carry; ++j) {
            long long cur = res[i + j] + 1LL * a[i] * (j < b.size() ? b[j] : 0) + carry;
            res[i + j] = int(cur % 10);
            carry = int(cur / 10);
        }
    }
    trim(res);
    return res;
}

// ---------- Вспомогательные для Карацубы ----------
BigInt shiftDecimal(const BigInt &a, int k) {
    // Умножение на 10^k — добавляем k нулей перед цифрами (LSB-first)
    if ((a.size() == 1 && a[0] == 0) || k == 0) return a;
    BigInt res(k, 0);
    res.insert(res.end(), a.begin(), a.end());
    return res;
}

// ---------- Карацуба (рекурсивно) ----------
BigInt karatsuba(const BigInt &a, const BigInt &b) {
    int n = max(a.size(), b.size());
    if (n <= 32) return multiplySchoolbook(a, b);

    int k = n / 2;
    BigInt a0(a.begin(), a.begin() + min((int)a.size(), k));
    BigInt a1 = ( (int)a.size() > k ? BigInt(a.begin() + k, a.end()) : BigInt{0} );
    BigInt b0(b.begin(), b.begin() + min((int)b.size(), k));
    BigInt b1 = ( (int)b.size() > k ? BigInt(b.begin() + k, b.end()) : BigInt{0} );

    trim(a0); trim(a1); trim(b0); trim(b1);

    BigInt z0 = karatsuba(a0, b0);
    BigInt z2 = karatsuba(a1, b1);
    BigInt sumA = add(a0, a1);
    BigInt sumB = add(b0, b1);
    BigInt z1 = karatsuba(sumA, sumB);
    // z1 = (a0+a1)*(b0+b1) - z2 - z0
    z1 = sub(sub(z1, z2), z0);

    BigInt res = add(add(shiftDecimal(z2, 2*k), shiftDecimal(z1, k)), z0);
    trim(res);
    return res;
}

// ---------- Умножение на цифру (0..9) ----------
BigInt mulByDigit(const BigInt &a, int d) {
    if (d == 0) return BigInt{0};
    BigInt res;
    res.reserve(a.size() + 2);
    int carry = 0;
    for (size_t i = 0; i < a.size(); ++i) {
        int cur = a[i] * d + carry;
        res.push_back(cur % 10);
        carry = cur / 10;
    }
    while (carry) {
        res.push_back(carry % 10);
        carry /= 10;
    }
    trim(res);
    return res;
}

// ---------- Деление (школьный метод) ----------
// Возвращает (quotient, remainder)
pair<BigInt, BigInt> divide(const BigInt &a, const BigInt &b) {
    assert(!(b.size() == 1 && b[0] == 0)); // деление на ноль запрещено
    if (compare(a,b) < 0) return {BigInt{0}, a};

    BigInt cur; // текущая часть (LSB-first)
    BigInt quotient;
    quotient.resize(a.size(), 0);

    for (int i = (int)a.size() - 1; i >= 0; --i) {
        // сдвигаем cur влево на одну десятичную цифру и добавляем текущую цифру
        cur.insert(cur.begin(), 0);
        cur[0] = a[i];
        trim(cur);

        // ищем максимальную q в [0..9], такой что b*q <= cur
        int l = 0, r = 9, x = 0;
        while (l <= r) {
            int mid = (l + r) >> 1;
            BigInt prod = mulByDigit(b, mid);
            if (compare(prod, cur) <= 0) {
                x = mid;
                l = mid + 1;
            } else r = mid - 1;
        }
        quotient[i] = x;
        BigInt prod = mulByDigit(b, x);
        cur = sub(cur, prod);
    }

    trim(quotient);
    trim(cur);
    return {quotient, cur};
}

// ---------- Ускоренное двоичное сложение (идея) ----------
using BitVec = vector<int>; // каждый элемент 0/1, LSB-first

BitVec bitwiseXor(const BitVec &a, const BitVec &b) {
    int n = max(a.size(), b.size());
    BitVec r(n,0);
    for (int i = 0; i < n; ++i) {
        int da = i < (int)a.size() ? a[i] : 0;
        int db = i < (int)b.size() ? b[i] : 0;
        r[i] = da ^ db;
    }
    return r;
}
BitVec bitwiseAnd(const BitVec &a, const BitVec &b) {
    int n = max(a.size(), b.size());
    BitVec r(n,0);
    for (int i = 0; i < n; ++i) {
        int da = i < (int)a.size() ? a[i] : 0;
        int db = i < (int)b.size() ? b[i] : 0;
        r[i] = da & db;
    }
    return r;
}
BitVec shiftLeftOne(const BitVec &a) {
    BitVec r(1,0);
    r.insert(r.end(), a.begin(), a.end());
    return r;
}

BitVec addBinaryAccelerated(BitVec a, BitVec b) {
    BitVec sum = bitwiseXor(a,b);
    BitVec carry = shiftLeftOne(bitwiseAnd(a,b));
    while (true) {
        bool allZero = true;
        for (int x : carry) if (x) { allZero = false; break; }
        if (allZero) break;
        BitVec tmp = bitwiseXor(sum, carry);
        BitVec newCarry = shiftLeftOne(bitwiseAnd(sum, carry));
        sum = tmp;
        carry = newCarry;
    }
    while (sum.size() > 1 && sum.back() == 0) sum.pop_back();
    return sum;
}

// ---------- Тесты и демонстрация ----------
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    srand((unsigned)time(nullptr));

    // Пример: ввод двух чисел вручную
    cout << "Пример: введите два целых числа (в одной строке через пробел):\n";
    string sa, sb;
    if (!(cin >> sa >> sb)) return 0;
    BigInt A = fromString(sa);
    BigInt B = fromString(sb);

    cout << "A = " << toString(A) << "\n";
    cout << "B = " << toString(B) << "\n";

    cout << "A + B = " << toString(add(A,B)) << "\n";

    auto [diff, sign] = subSigned(A,B);
    cout << "A - B = " << (sign < 0 ? "-" : "") << toString(diff) << "\n";

    cout << "A * B (schoolbook) = " << toString(multiplySchoolbook(A,B)) << "\n";
    cout << "A * B (karatsuba)   = " << toString(karatsuba(A,B)) << "\n";

    auto [Q,R] = divide(A,B);
    cout << "A / B = " << toString(Q) << ", remainder = " << toString(R) << "\n";

    // Короткий бинарный пример: 13 + 7
    BitVec a_bits = {1,0,1,1}; // 13 (1101) LSB-first
    BitVec b_bits = {1,1,1};   // 7  (111)
    BitVec s_bits = addBinaryAccelerated(a_bits, b_bits);
    int val = 0;
    for (int i = (int)s_bits.size()-1; i >=0; --i) val = val*2 + s_bits[i];
    cout << "13 + 7 (binary accelerated) = " << val << "\n";

    return 0;
}
```

---

## Как запустить (на любом компьютере с `g++`)

1. Создайте файл `main.cpp` и вставьте код.
2. Компиляция:

```bash
g++ -std=c++17 main.cpp -O2 -o main
```

3. Запуск:

```bash
./main
```

Программа попросит ввести два целых числа (в одну строку) и покажет результаты операций.

---

## Пояснения (важные детали для студентов)

* **`LSB-first`** означает, что вектор хранит число в обратном порядке: `123 -> {3,2,1}`. Это упрощает обработку переноса.
* Обратите внимание на функцию `trim` — удаление ведущих нулей; без неё сравнения и вывод могут быть некорректны.
* Для очень больших чисел (тысячи цифр) эффективнее использовать более крупный базис, например `base = 10^4` или `base = 10^9`, где каждый элемент вектора хранит не одну, а несколько цифр — это уменьшит количество операций.
* **Карацуба** полезна при очень больших длинах (обычно сотни и более цифр). Здесь показана упрощённая рекурсивная реализация, чтобы понять идею (уменьшение числа умножений с 4 до 3 при разбиении).

---

## Упражнения (для самостоятельной работы)

1. **Базис 10000.** Реализовать `BigInt` в базе `base = 10000` (каждая ячейка хранит 4 цифры) и адаптировать все операции. Сравнить скорость с текущей реализацией (замерить время на больших случайных числах).
2. **Отрицательные числа.** Добавить поддержку отрицательных чисел (класс `SignedBigInt`) и реализовать сравнение/сложение/умножение с учётом знака.
3. **Нормализованное деление.** Реализовать нормализованный алгоритм деления по Кнуту и исследовать разницу в скорости на больших числах.
4. **Тесты.** Написать набор автоматических тестов: генерировать случайные числа, сравнивать результаты операций с эталонными (например, с использованием `std::stoll`/`__int128` для небольших длин) либо с языком, где есть BigInt (Python), чтобы убедиться в корректности.
