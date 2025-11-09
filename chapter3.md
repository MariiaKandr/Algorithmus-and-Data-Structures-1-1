Ниже — готовое **краткое саммари** для `README.md` (или `index.md`) и **подробные примеры кода на C++** с комментариями, понятные студенту 1-го курса. Материал основан на вашей рабочей тетради. 

---

# Краткое саммари (для md-файла)

## О чём этот учебник

Краткое руководство по реализации операций длинной арифметики (целые числа, длина >> стандартных типов):

* представление больших чисел;
* сравнение;
* сложение и вычитание (в «столбик»);
* умножение (столбиковое и Карацуба);
* деление (школьный алгоритм, идея бинарного поиска для частного);
* ускоренное сложение двоичных чисел (метод разделения суммы и переносов).

Материал даёт практические реализации в C++ и объяснения.

## Формат представления

* Большое число — массив цифр (каждый элемент — цифра в десятичной системе), с **младшим разрядом в `digits[0]`** и старшим — в конце. Это упрощает операции с переносом.
* Для удобства использованы `std::vector<int>` (динамический массив). Можно применять и «сырые» указатели, но `vector` безопаснее.

---

# Примеры кода (C++). Общие утилиты

```cpp
// big_integer_helpers.cpp — базовые утилиты
#include <bits/stdc++.h>
using namespace std;

// Удобный тип: каждая цифра — 0..9, LSB в index 0
using BigInt = vector<int>;

// Удалить ведущие нули (старшие разряды)
void trim(BigInt &a) {
    while (a.size() > 1 && a.back() == 0) a.pop_back();
}

// Создать BigInt из строки "12345"
BigInt fromString(const string &s) {
    BigInt a;
    a.reserve(s.size());
    for (int i = (int)s.size() - 1; i >= 0; --i) {
        if (!isdigit(s[i])) continue;
        a.push_back(s[i] - '0');
    }
    if (a.empty()) a.push_back(0);
    trim(a);
    return a;
}

// Преобразовать BigInt в строку "12345"
string toString(const BigInt &a) {
    string s;
    for (int i = (int)a.size() - 1; i >= 0; --i) s.push_back(char('0' + a[i]));
    if (s.empty()) return "0";
    return s;
}

// Генерация случайного большого числа длины n (старший разряд != 0)
BigInt randomBigInt(int n) {
    assert(n >= 1);
    BigInt a(n);
    for (int i = 0; i < n; ++i) a[i] = rand() % 10;
    if (a.back() == 0) a.back() = 1 + (rand() % 9); // старший разряд не ноль
    trim(a);
    return a;
}
```

---

# Сравнение двух больших чисел

Правило: сначала по длине, затем по старшим разрядам.

```cpp
// compare: возвращает 1 если a>b, 0 если a==b, -1 если a<b
int compare(const BigInt &a, const BigInt &b) {
    if (a.size() != b.size()) return a.size() > b.size() ? 1 : -1;
    for (int i = (int)a.size() - 1; i >= 0; --i) {
        if (a[i] != b[i]) return (a[i] > b[i]) ? 1 : -1;
    }
    return 0;
}

// Пример использования:
// BigInt x = fromString("12345");
// BigInt y = fromString("543");
// int c = compare(x,y); // c == 1
```

---

# Сложение (в столбик)

Алгоритм: поразрядно суммируем с учётом переноса.

```cpp
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
```

Пример: `123 + 987 = 1110` (цифры хранятся в обратном порядке внутри `BigInt`).

---

# Вычитание (в столбик)

Правило: реализуем `a - b`, предполагая `a >= b`. Если возможно `a < b`, сначала сравнить и поменять местами, сохранив знак.

```cpp
BigInt sub(const BigInt &a, const BigInt &b) {
    // Предполагаем a >= b
    BigInt res;
    res.reserve(a.size());
    int carry = 0; // тут 'borrow'
    for (int i = 0; i < (int)a.size(); ++i) {
        int da = a[i];
        int db = (i < (int)b.size()) ? b[i] : 0;
        int cur = da - db - carry;
        if (cur < 0) {
            cur += 10;
            carry = 1;
        } else {
            carry = 0;
        }
        res.push_back(cur);
    }
    trim(res);
    return res;
}

// Удобная обёртка, возвращающая пару (результат, знак)
pair<BigInt, int> subSigned(const BigInt &a, const BigInt &b) {
    int cmp = compare(a, b);
    if (cmp == 0) return {BigInt{0}, 0};
    if (cmp > 0) {
        return {sub(a, b), +1}; // положительный результат
    } else {
        return {sub(b, a), -1}; // отрицательный результат
    }
}
```

---

# Умножение «в столбик» (наивное, O(n*m))

```cpp
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
```

Сложность: O(n⋅m) (n и m — число цифр). Подходит для чисел умеренного размера.

---

# Метод Карацубы (рекурсивно)

Идея: для двух чисел разбиваем на старшую и младшую половины:
`A = a*x + b`, `B = c*x + d`, где `x = 10^k`. Тогда
`A*B = a*c*x^2 + ((a+b)*(c+d) - a*c - b*d)*x + b*d`.
Нужно посчитать только 3 произведения размеров ~n/2 (вместо 4).

Ниже — реализация для `BigInt`. Для простоты базовый случай: если длина мала (≤32), используем schoolbook.

```cpp
// Сложение BigInt, вычитание (a >= b), сдвиг на k десятичных разрядов (append zeros)
BigInt addBig(const BigInt &a, const BigInt &b) { return add(a, b); }
BigInt subBig(const BigInt &a, const BigInt &b) { return sub(a, b); }

BigInt shiftDecimal(const BigInt &a, int k) {
    if ((a.size() == 1 && a[0] == 0) || k == 0) return a;
    BigInt res(k, 0);
    res.insert(res.end(), a.begin(), a.end());
    return res;
}

BigInt karatsuba(const BigInt &a, const BigInt &b) {
    int n = max(a.size(), b.size());
    if (n <= 32) return multiplySchoolbook(a, b); // базовый случай

    int k = n / 2; // делим по цифрам
    // a = a1 * 10^k + a0
    BigInt a0(a.begin(), a.begin() + min((int)a.size(), k));
    BigInt a1;
    if ((int)a.size() > k) a1 = BigInt(a.begin() + k, a.end()); else a1 = BigInt{0};
    BigInt b0(b.begin(), b.begin() + min((int)b.size(), k));
    BigInt b1;
    if ((int)b.size() > k) b1 = BigInt(b.begin() + k, b.end()); else b1 = BigInt{0};

    trim(a0); trim(a1); trim(b0); trim(b1);

    BigInt z0 = karatsuba(a0, b0);
    BigInt z2 = karatsuba(a1, b1);
    BigInt sumA = addBig(a0, a1);
    BigInt sumB = addBig(b0, b1);
    BigInt z1 = karatsuba(sumA, sumB);
    // z1 = (a0+a1)*(b0+b1) - z2 - z0
    z1 = subBig(subBig(z1, z2), z0);

    BigInt res = addBig(addBig(shiftDecimal(z2, 2*k), shiftDecimal(z1, k)), z0);
    trim(res);
    return res;
}
```

Примечание: разбиение по десятичным цифрам (k) — простой, но не самый быстрый вариант (обычно используют большой базис, например 10^4 или 10^9, чтобы уменьшить число операций). Для учебной реализации удобно оставаться в base=10.

---

# Деление (школьный способ) — частное + остаток

Алгоритм похож на деление в столбик: идём от старшего разряда, постепенно формируем `cur` и находим сколько раз делитель умещается в `cur`. Реализуем деление `A / B`.

```cpp
// Умножение BigInt на одиночную цифру (0..9)
BigInt mulByDigit(const BigInt &a, int d) {
    if (d == 0) return BigInt{0};
    BigInt res;
    res.reserve(a.size() + 1);
    int carry = 0;
    for (size_t i = 0; i < a.size(); ++i) {
        int cur = a[i] * d + carry;
        res.push_back(cur % 10);
        carry = cur / 10;
    }
    if (carry) {
        while (carry) {
            res.push_back(carry % 10);
            carry /= 10;
        }
    }
    trim(res);
    return res;
}

// Деление: вернёт (quotient, remainder)
pair<BigInt, BigInt> divide(const BigInt &a, const BigInt &b) {
    assert(!(b.size() == 1 && b[0] == 0)); // деление на 0 запрещено
    if (compare(a, b) < 0) return {BigInt{0}, a};

    BigInt cur; // текущая «остаточная» часть, хранится в обычном порядке (LSB=0)
    BigInt quotient;
    quotient.resize(a.size(), 0);

    for (int i = (int)a.size() - 1; i >= 0; --i) {
        // сдвигаем cur влево (умножаем на 10) и добавляем текущую цифру
        cur.insert(cur.begin(), 0); // добавляем самый младший элемент, потом перепишем
        cur[0] = a[i];
        trim(cur);

        // найти максимальную цифру q (0..9) такую, что b * q <= cur
        int x = 0, l = 0, r = 9;
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
```

Комментарий: здесь мы ищем одноразрядное частное на каждом шаге (0..9), применяя бинарный поиск по цифре. Это стандартный школьный алгоритм.

---

# Ускоренное сложение в двоичной системе (идея из тетради)

Идея: использовать два вектора — `sum` (битовые XOR) и `carry` (битовые AND сдвинутый на 1). Повторять, пока `carry` не станет нулём:

`while (carry != 0) { tmp = sum ^ carry; carry = (sum & carry) << 1; sum = tmp; }`

Реализация на векторах битов:

```cpp
using BitVec = vector<int>; // каждый элемент 0 или 1, LSB в index 0

// побитовый XOR для векторов бит
BitVec bitwiseXor(const BitVec &a, const BitVec &b) {
    int n = max(a.size(), b.size());
    BitVec r(n);
    for (int i = 0; i < n; ++i) {
        int da = (i < (int)a.size()) ? a[i] : 0;
        int db = (i < (int)b.size()) ? b[i] : 0;
        r[i] = da ^ db;
    }
    return r;
}

BitVec bitwiseAnd(const BitVec &a, const BitVec &b) {
    int n = max(a.size(), b.size());
    BitVec r(n);
    for (int i = 0; i < n; ++i) {
        int da = (i < (int)a.size()) ? a[i] : 0;
        int db = (i < (int)b.size()) ? b[i] : 0;
        r[i] = da & db;
    }
    return r;
}

BitVec shiftLeftOne(const BitVec &a) {
    if (a.empty()) return BitVec{0};
    BitVec r(1, 0);
    r.insert(r.end(), a.begin(), a.end());
    return r;
}

BitVec addBinaryAccelerated(BitVec a, BitVec b) {
    BitVec sum = bitwiseXor(a, b);
    BitVec carry = shiftLeftOne(bitwiseAnd(a, b));
    while (true) {
        // проверим carry == 0
        bool allZero = true;
        for (int x : carry) if (x) { allZero = false; break; }
        if (allZero) break;
        BitVec tmp = bitwiseXor(sum, carry);
        BitVec newCarry = shiftLeftOne(bitwiseAnd(sum, carry));
        sum = tmp;
        carry = newCarry;
    }
    // убрать ведущие нули
    while (sum.size() > 1 && sum.back() == 0) sum.pop_back();
    return sum;
}
```

Комментарий: алгоритм на практике реализуется с машинными словами (широкие регистры) эффективнее, но идея понятна — перенос обрабатывается отдельно и «перекатывается» влево.

---

# Пример использования (микро-тесты в main)

```cpp
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    srand(time(nullptr));

    // Создадим два случайных больших числа длиной 10 и 8 цифр
    BigInt A = randomBigInt(10);
    BigInt B = randomBigInt(8);

    cout << "A = " << toString(A) << "\n";
    cout << "B = " << toString(B) << "\n";

    BigInt S = add(A, B);
    cout << "A + B = " << toString(S) << "\n";

    auto [Dsign, sign] = subSigned(A, B);
    cout << "A - B = " << (sign >= 0 ? "" : "-") << toString(Dsign) << "\n";

    BigInt P1 = multiplySchoolbook(A, B);
    cout << "A * B (schoolbook) = " << toString(P1) << "\n";

    BigInt P2 = karatsuba(A, B);
    cout << "A * B (karatsuba) = " << toString(P2) << "\n";

    auto [Q, R] = divide(A, B);
    cout << "A / B = " << toString(Q) << ", remainder = " << toString(R) << "\n";

    // Мини-пример ускоренного двоичного сложения: 13 + 7
    BitVec a_bits = {1,0,1,1}; // 13: 1101 -> LSB first
    BitVec b_bits = {1,1,1};   // 7: 111
    BitVec s_bits = addBinaryAccelerated(a_bits, b_bits);
    // вывести результат в десятичном виде (для показа)
    int val = 0;
    for (int i = (int)s_bits.size()-1; i >=0; --i) val = val*2 + s_bits[i];
    cout << "13 + 7 (binary accelerated) = " << val << "\n";

    return 0;
}
```

---

# Пояснения и учебные советы

* Всегда поддерживайте *trim* (удаление ведущих нулей), иначе сравнения и другие операции станут некорректными.
* Для реальной эффективности используйте основание `base = 10^k` (например `k=4` или `k=9`) — тогда каждый элемент массива хранит не одну, а `k` цифр, и умножение/сложение выполняется быстрее (меньше отдельных операций).
* Карацуба полезна для очень больших чисел (обычно > few hundreds digits), но сложнее в реализации (особенно при использовании большого основания).
* Деление лучше реализовывать школьным способом (как показано), бинарный поиск по частному — теоретическая идея, но на практике частный ищут по цифре на каждом шаге или используют нормализацию Делоня (алгоритм Knuth).

