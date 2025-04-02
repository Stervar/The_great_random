# Великий random

## Даты
2 марта, 2019 - 16 октября, 2019

## Введение
Генераторы случайных чисел (аббр. ГСЧ или RNG) можно разделить на псевдослучайные генераторы (pseudo random number generator – PRNG) и настоящие генераторы (true random number generator – TRNG). Настоящее случайное число может быть получено, например, честным бросанием (без мухлежа) игрального кубика. Но цифровая техника, в т.ч. и компьютер — вещь точная и детерминированная. И нет так очевидно, где нам там брать случайные числа. Да, бывают аппаратные ГСЧ, построенные на аналоговых шумах или квантовых эффектах, но они не всегда доступны простым пользователям. Однако математики разработали алгоритмы, по которым можно с помощью простых и точных операций (типа сложения и деления) получать «иллюзию» случайности.

## Линейный конгруэнтный метод
Давайте для начала рассмотрим линейный конгруэнтный метод и попробуем сконструировать свой рандом. Все начинается с зерна (seed). 

```python
x[0] = seed
x[i + 1] = (a * x[i] + b) mod c
```

Каждое из них будет в пределах [0..c). Вот реализация:

```python
class MyRandom:
    def __init__(self, seed=42):
        self._state = seed

    def random(self):
        self._state = (5 * self._state + 9) % 17
        return self._state

r = MyRandom(42)
print([r.random() for _ in range(10)])  # [15, 16, 4, 12, 1, 14, 11, 13, 6, 5]

r2 = MyRandom(24)
print([r2.random() for _ in range(10)])  # [10, 8, 15, 16, 4, 12, 1, 14, 11, 13]

r3 = MyRandom(42)
print([r3.random() for _ in range(10)])  # [15, 16, 4, 12, 1, 14, 11, 13, 6, 5]
```

Первое. Последовательности кажутся случайными, но на самом деле качество их невелико. Через некоторое время числа начинают повторяться. Последовательность периодична. Второе. Наш псевдослучайный генератор выдает одинаковые последовательности для одинаковых seed. Алгоритм детерминирован. Последнее свойство бывает вредно и полезно. 

## Инициализация генератора
Представим, что вы проводите эксперимент. Допустим, учите нейросеть. Инициализировав веса случайными числами, вы получаете какой-то результат. Далее вы меняете что-то в архитектуре сети и запускаете снова, и получаете иной результат. Но как убедиться, повлияли ли ваши изменения в коде, или просто иная случайная инициализация изменила результат? Имеет смысл зафиксировать seed генератора случайных чисел константой в начале программы. При следующем запуске мы получим точно такую же инициализацию сети, как и в предыдущем.

Но, если мы не хотим повторяемости, то можно инициализировать генератор какой-то меняющейся от запуска к запуску переменной (например, временем):

```python
import time
r4 = MyRandom(int(time.time()))
print([r4.random() for _ in range(10)])  # [3, 7, 10, 8, 15, 16, 4, 12, 1, 14]
```

## Способы получения случайных величин в Python
Для получения случайных величин в Python есть несколько способов. Мы рассмотрим следующие:

- Встроенный модуль `random`
- `numpy.random` из библиотеки NumPy
- Функцию `os.urandom`
- Встроенный модуль `secrets`
- Встроенный модуль `uuid`

## Модуль random
Самый популярный вариант: модель встроенный random. Модуль random предоставляет набор функций для генерации псевдослучайных чисел. Реализована генерация на языке Си (исходник) по более хитрому алгоритму «вихрь Мерсенна», разработанному в 1997 году. Он дает более «качественные» псевдослучайные числа. Но они по-прежнему получаются из начального зерна (seed) путем совершения математических операций. Зная seed и алгоритм можно воспроизвести последовательность случайных чисел; более того существуют алгоритмы позволяющие вычислить из последовательности чисел её seed. Поэтому такие алгоритмы не пригодны для генерации конфиденциальных данных: паролей, и ключей доступа. Но он вполне сгодится для генерации случайностей в играх (не азартных) и прочих приложений, где не страшно, если кто-то сможет воспроизвести и продолжить последовательностей случайных чисел. Воспроизводимость случайностей поможет вам в задачах статистики, в симуляциях различных процессов.

## Примеры использования
```python
import random
random.seed(new_seed)  # сброс ГСЧ с новым seed

random.seed(4242)
print(random.random())  # 0.8624508153567833
print(random.random())  # 0.41569372364698065

random.seed(4242)
print(random.random())  # 0.8624508153567833
print(random.random())  # 0.41569372364698065
```

Когда мы второй раз задали тот же seed, ГСЧ выдает точно такие же случайные числа. Если мы не задаем seed, то ГСЧ будет скорее всего инициализирован системным временем, и значения будут отличаться от запуска к запуску.

```python
print(random.randint(5, 8))  # случайное целое число от 5 до 8 (включительно)
print([random.randint(5, 8) for _ in range(10)])  # [6, 8, 5, 8, 6, 6, 8, 5, 5, 6]
```



random.randint(a, b) – случайное целое число от a до b (включительно):

>>> random.randint(5, 8)
5
>>> [random.randint(5, 8) for _ in range(10)]
[6, 8, 5, 8, 6, 6, 8, 5, 5, 6]
random.randrange(a, b, step) – случайное целое число от a до b (не включая b) с шагом step. Аргументы имеют такой же смысл, как у функции range. Если мы зададим только a, получим число в [0, a) с шагом 1; если задаем a и b, то в число будет в диапазоне [a, b):

 range (диапазон)
Добавлено в Python 3.0
Диапазон — неизменяемая последовательность целых чисел.
range(stop) | range(start, stop[, step]) -> range
 start=0 : Целое число, которое должно явиться началом последовательности.

 stop : Целое число, на котором должно завершиться формирование последовательности. Не входит в последовательность.

 step=1 : Целое число — шаг, с которым должна формироваться последовательность. При попытке задать нуль, возбуждается ValueError.
    type(range(3))  # class 'range'

    list(range(5))  # [0, 1, 2, 3, 4]
    list(range(1, 5))  # [1, 2, 3, 4]
    list(range(0, 10, 3))  # [0, 3, 6, 9]
    list(range(0, -5, -1))  # [0, -1, -2, -3, -4]
    list(range(0))  # []
    list(range(1, 0))  # []

Преимуществом данного типа по сравнению с обычным списком или кортежем, является то, что он занимает всегда одинаковое небольшое количество памяти вне зависимости от того, какой длины диапазон представляет. Это возможно благодаря тому, что в памяти хранятся только параметры, а значения вычисляются по мере необходимости.

Последовательности реализуют интерфейс collections.abc.Sequence ABC, и предоставляют такие возможности как проверка вхождения, поиск по индексу, срезы и отрицательную индексацию.

Внимание
Диапазоны, содержащие значения превышающие sys.maxsize поддерживаются, однако некоторые методы (например, len()) могут вызывать OverflowError.

Внимание
Проверка диапазонов на равенство при помощи == и != сравнивает их как последовательности. Это означает, что два диапазона равны, если они представляют одинаковую последовательность значений. Пример: range(0) == range(2, 1, 3) или range(0, 3, 2) == range(0, 4, 2).


Python 2
Вместо типа существует функция range(), возвращающая список.
Существует тип xrange, последователем дела которого является тип range из Python 3.

    type(range(3))  # type 'list'
    type(xrange(3))  # type 'xrange'
