![bicycle](images/bicycle.png)

# Presenter Notes
что кроется за синтаксическим сахаром в пифоне

некоторые best practice для новичков и старичков

магические методы

базовое понимание асинхронного программирования в питоне

---

# Обо мне

Никита Мелентьев. Ведущий инженер-программист КТБС.

[https://github.com/pohmelie](https://github.com/pohmelie)

[https://github.com/aio-libs/aioftp](https://github.com/aio-libs/aioftp)

# Presenter Notes
big pulls: micropython, cerberus

40 pulls on github

---

# Обо мне

Никита Мелентьев. Ведущий инженер-программист КТБС.

[https://github.com/pohmelie](https://github.com/pohmelie)

[https://github.com/aio-libs/aioftp](https://github.com/aio-libs/aioftp)

В КТБС...

---

# Обо мне

Никита Мелентьев. Ведущий инженер-программист КТБС.

[https://github.com/pohmelie](https://github.com/pohmelie)

[https://github.com/aio-libs/aioftp](https://github.com/aio-libs/aioftp)

В КТБС... делаю скриншоты!

![dino](images/динозавр-философ.jpg)

---

    !python
    f = open("file.txt", "w")
    f.write("foo")

# Presenter Notes
что не так?

кто-то забудет close и не заметит, потому что при уничтожении вызовется close

в этом случае повезло, но в подавляющем большинстве случаев явный close никто не вызывает неявно из `__del__`

---

    !python
    f = open("file.txt", "w")
    f.write("foo")
    f.close()

# Presenter Notes
достаточно ли это хорошее решение с точки зрения api?

если есть возможность забыть вызвать метод, значит это обязательно случится

---

    !python
    import os
    import signal

    f = open("file.txt", "w")
    f.write("foo")
    # time
    os.kill(os.getpid(), signal.SIGKILL)

# Presenter Notes
time - какая-то работа программы

текст не попадёт в файл из-за буферизации

какие идеи?

---

# ОТКЛЮЧИТЬ БУФЕРИЗАЦИЮ!
![idea](images/idea.png)

# Presenter Notes
нужен сахар!

---

# Context manager

# Presenter Notes
создатели решили не отключать буферизацию а

"да, это частая задача, инициализировать ресурс, использовать и финализировать"

---

# Context manager

    !python
    with context(...) as resource:
        # do something with resource

---

# Context manager

    !python
    class File:
        def __init__(self, name, mode="r"):
            self.name = name
            self.mode = mode

        def __enter__(self):
            self.f = open(self.name, self.mode)
            return self.f

        def __exit__(self, *exc_info):
            self.f.close()


    with File("file.txt", "w") as f:
        f.write("foo")

# Presenter Notes
первые магические методы

три(!) магических метода

меня сделают core-dev благодаря такому контекстному менеджеру?

---

# Context manager

    !python
    with open("file.txt", "w") as f:
        f.write("foo")

# Presenter Notes
боюсь что нет, потому что объект, который возвращает open уже умеет всё сам

но есть вариант ещё лучше для этой конкретной задачи...

---

# No context manager :'(

    !python
    from pathlib import Path
    Path("file.txt").write_text("foo")

* read_text
* write_text
* read_bytes
* write_bytes

# Presenter Notes
появилась в 3.4

эти функции идеальны для сохранения/чтения json, yaml, xml, small binaries etc. всё, что влезет в память

pathlib отличная библиотека для работы с путями, итерации по директориям и т.д.

не парсите пути сами!

не объединяйте пути сами!

не отделяйте расширение файла строковыми функциями!

---

# Decorator

---

# Decorator

    !python
    import time
    import random

    def foo():
        time.sleep(random.random())

    foo()

# Presenter Notes
хотим измерить время выполнения функции

первая идея: измерить непосредственно где вызывается

---

# Decorator

    !python
    import time
    import random

    def foo():
        time.sleep(random.random())

    start = time.time()
    foo()
    print(time.time() - start)

# Presenter Notes
неудобно: одна функция может вызываться сотни раз в разных местах и когда появится 101 место кто-нибудь да забудет
добавить измерение

идея: измерять внутри функции

---

# Decorator

    !python
    import time
    import random

    def foo():
        start = time.time()
        time.sleep(random.random())
        print(time.time() - start)

    foo()

# Presenter Notes
хорошо, когда это две строки, но(!) это измерение времени не относится к логике функции, той задаче,
которую решает функция это измерение не нужно

неудобно: много функций, если захотим изменить способ измерения времени придётся менять в сотне мест

идея: модифицировать функцию не модифицируя её! сделать обёртку

---

# Decorator

    !python
    import time
    import random

    def bar(f):
        def timed():
            start = time.time()
            f()
            print(time.time() - start)
        return timed

    def foo():
        time.sleep(random.random())
    foo = bar(foo)

    foo()

# Presenter Notes
цель достигнута! но некрасиво

для этого в питоне есть сахар!

---

# Decorator

    !python
    import time
    import random
    import functools

    def bar(f):
        @functools.wraps(f)
        def timed():
            start = time.time()
            f()
            print(time.time() - start)
        return timed

    @bar
    def foo():
        time.sleep(random.random())

    foo()

# Presenter Notes
wraps? @bar?

легко добавлять/убирать

не мешается с кодом самой функции

некоторые боятся

следующая тема...

---

# Шеукфещкы

---

# Iterator

# Presenter Notes
все привыкли итерировать контейнеры

что делать если хочется кастомного поведения?

magic methods! :tada:

---

# Iterator

    !python
    class Countdown:
        def __init__(self, n):
            self.n = n

        def __iter__(self):
            return self

        def __next__(self):
            if self.n == 0:
                raise StopIteration()
            v = self.n
            self.n -= 1
            return v

# Presenter Notes
iter возвращает объект у которого будет вызываться next

StopIteration скажет питону что мы всё

---

# Iterator

    !python
    class Countdown:
        def __init__(self, n):
            self.n = n

        def __iter__(self):
            return self

        def __next__(self):
            if self.n == 0:
                raise StopIteration()
            v = self.n
            self.n -= 1
            return v

    for x in Countdown(5):
        # work

---

# Обзор magic methods

# Presenter Notes
чтобы не цитировать документацию просто пробежимся по группам magic methods

---

# Обзор magic methods
* callable (`__call__`)
#
    !python
    class A:
        def __call__(self):
            return "foobar"

    a = A()
    assert a() == "foobar"

# Presenter Notes
чтобы использовать инстанс/объект как функцию

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
#
    !python
    class A:
        def __getitem__(self, key):
            return key * 2

    a = A()
    assert a[2] == 4

# Presenter Notes
мимикрировать под коллекцию

некоторые magic методы имеют фиксированные аргументы, некоторые нет

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
#
    !python
    class A:
        def __getattr__(self, name):
            return name * 2

    a = A()
    assert a.foo == "foofoo"

# Presenter Notes
когда property недостаточно

динамические цепочки для api

очень динамичная вещь, в духе питона, но ide не сможет помочь и документацию не сгенерировать

aptlier

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
* comparison (`__lt__`, `__eq__`, ...)
#
    !python
    class A:
        def __init__(self, x)
            self.x = x

        def __eq__(self, other):
            return self.x == other.x

    a = A(1)
    b = A(1)
    assert a == b

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
* comparison (`__lt__`, `__eq__`, ...)
* descriptors (`__get__`, `__set__`, `__set_name__`)
#
    !python
    class D:
        def __set_name__(self, owner, name):
            self.name = name

        def __get__(self):
            return self.name

    class A:
        var = D()

    a = A()
    assert a.var == "var"

# Presenter Notes
как property, только есть свой объект-хранилище

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
* comparison (`__lt__`, `__eq__`, ...)
* descriptors (`__get__`, `__set__`, `__set_name__`)
* arithmetic/binary (`__and__`, `__or__`, `__add__`, `__sub__`, ...)
#
    !python
    class A:
        def __init__(self, x):
            self.x = x

        def __add__(self, other):
            return self.__class__(self.x + other.x)

    a = A(1) + A(2)
    assert a.x == 3

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
* comparison (`__lt__`, `__eq__`, ...)
* descriptors (`__get__`, `__set__`, `__set_name__`)
* arithmetic/binary (`__and__`, `__or__`, `__add__`, `__sub__`, ...)
* metaclasses (`__new__`, `__init_subclass__`, ...)


![hedgehog](images/hedgehog.jpg)

# Presenter Notes
кастомизация создания класса, нужно очень редко

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
* comparison (`__lt__`, `__eq__`, ...)
* descriptors (`__get__`, `__set__`, `__set_name__`)
* arithmetic/binary (`__and__`, `__or__`, `__add__`, `__sub__`, ...)
* metaclasses (`__new__`, `__init_subclass__`, ...)
* function-correspond (`__len__`, `__repr__`, `__str__`, `__hash__`, `__bool__`, ...)
#
    !python
    class A:
        def __len__(self):
            return 42

    a = A()
    assert len(a) == 42

# Presenter Notes
интерфейсы для builtin функций, но не только

например, `__bool__` вызывается в if

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
* comparison (`__lt__`, `__eq__`, ...)
* descriptors (`__get__`, `__set__`, `__set_name__`)
* arithmetic/binary (`__and__`, `__or__`, `__add__`, `__sub__`, ...)
* metaclasses (`__new__`, `__init_subclass__`, ...)
* function-correspond (`__len__`, `__repr__`, `__str__`, `__hash__`, `__bool__`, ...)
* https://docs.python.org/3/reference/datamodel.html#special-method-names

---

# Generator

---

# Generator

    !python
    def fib(n):
        ns = []
        a, b = 0, 1
        for _ in range(n):
            ns.append(a)
            a, b = b, a + b
        return ns

    for x in fib(10):
        print(x)

# Presenter Notes
n-первых чисел фибоначчи

классический энергичный/неленивый подход

недостатки: получим все числа сразу, память,
вызывающий заблокирован пока не будет составлен весь список чисел

---

# Generator

    !python
    class Fib:
        def __init__(self, n):
            self.n = n
            self.a, self.b = 0, 1

        def __iter__(self):
            return self

        def __next__(self):
            if not self.n:
                raise StopIteration()
            self.n -= 1
            v = self.a
            self.a, self.b = self.b, self.a + self.b
            return v


    for x in Fib(10):
        print(x)

# Presenter Notes
можно написать итератор

нет проблем с памятью, нет блокировки

но... некрасиво, шумно, логика не очень хорошо читается

можно заменить 5 строчками...

---

# Generator

    !python
    def fib(n):
        a, b = 0, 1
        for _ in range(n):
            yield a
            a, b = b, a + b

    for x in fib(10):
        print(x)

# Presenter Notes
объяснить флоу

это пример генератора, который используется как итератор, но у генератора есть дополнительные
свойства, которых нет у итератора

---

# Generator

    !python
    def gen():  # <-
        s = 0
        while True:
            x = yield s
            s += x

    g = gen()  # <generator object gen at 0x7fb8cfbfb7d8>


# Presenter Notes
объект-генератор создан, но код ещё не выполнялся

`__next__`, `send`, `throw`, `close`

---

# Generator

    !python
    def gen():
        s = 0
        while True:
            x = yield s  # <-
            s += x

    g = gen()
    assert g.send(None) == 0

# Presenter Notes
первый send не попадёт в x, а только дойдёт до yield

---

# Generator

    !python
    def gen():
        s = 0
        while True:
            x = yield s  # <-
            s += x

    g = gen()
    assert g.send(None) == 0
    assert g.send(1) == 1
    assert g.send(2) == 3
    assert g.send(3) == 6

# Presenter Notes
флоу

---

# Generator

    !python
    def gen():
        s = 0
        while True:
            x = yield s  # <-
            s += x

    g = gen()
    assert g.send(None) == 0
    assert g.send(1) == 1
    assert g.send(2) == 3
    assert g.send(3) == 6
    g.close()

# Presenter Notes
генератор можно остановить (бросить исключение GeneratorExit внутрь)

---

# Generator

    !python
    def gen():
        s = 0
        while True:
            x = yield s  # <-
            s += x

    g = gen()
    assert g.send(None) == 0
    assert g.send(1) == 1
    assert g.send(2) == 3
    assert g.send(3) == 6
    g.throw(Exception("FOO"))

# Presenter Notes
можно бросить кастомной исключение

---

# Generator

    !python
    def gen():
        s = 0
        while True:
            try:
                x = yield s
            except Exception:
                yield "error"
                break
            s += x

    g = gen()
    assert g.send(None) == 0
    assert g.send(1) == 1
    assert g.send(2) == 3
    assert g.send(3) == 6
    assert g.throw(Exception("FOO")) == "error"

# Presenter Notes
генератор может обработать исключение

давайте объединим генератор, декоратор и контекстный менеджер!

---

# Generator

    !python
    import contextlib

    @contextlib.contextmanager
    def context(name, mode="r"):
        f = open(name, mode)
        try:
            yield f
        finally:
            f.close()

    with context("file.txt", "w") as f:
        f.write("foo")

# Presenter Notes
объяснить все элементы

что хорошо: понятный флоу без коллбеков, логика отделена от интерфейса контекстного менеджера декоратором

но и это ещё не всё!

---

# Generator

PEP380 — Syntax for Delegating to a Subgenerator (Python 3.3)

# Presenter Notes
в далёком 2009 году Gregory Ewing предложил то, что позволило в последствии появиться целому семейству библиотек
использующих неблокирующие операции и сейчас это целое направление в разработке на пифоне

кто-нибудь слышал о tornado?

угадайте в каком году был первый релиз... 2009

так что же он предложил?

---

# Generator

PEP380 — Syntax for Delegating to a Subgenerator (Python 3.3)

    !python
    def gen1():
        yield 1
        yield 2
        return 3

    def gen2():
        value = yield from gen1()
        assert value == 3

    for x in gen2():
        print(x)
    # 1
    # 2

# Presenter Notes
что даёт?

стек генераторов, механизм абстракции схожий с функциями

что сделает send?

---

# Generator

PEP380 — Syntax for Delegating to a Subgenerator (Python 3.3)

    !python
    def gen1():
        yield 1
        yield 2
        return 3

    def gen2():
        value = yield from gen1()
        assert value == 3

    g = gen2()
    assert g.send(None) == 1
    assert g.send(None) == 2

# Presenter Notes
всё как ожидается, значения доходят до того, кто сделал yield (gen1)

gen2 вообще ничего не yield-ит

как работает return?

---

# Generator

PEP380 — Syntax for Delegating to a Subgenerator (Python 3.3)

    !python
    def gen1():
        yield 1
        yield 2
        raise StopIteration(3)

    def gen2():
        value = yield from gen1()
        assert value == 3

    for x in gen2():
        print(x)
    # 1
    # 2

# Presenter Notes
а значит можно написать класс, который будет себя вести как генератор и который
можно использовать в yield from

генераторы — очередной очень полезный сахар, который при этом вписывается в объектную модель питона
и может быть заменён на слегка необычный класс

---

# Generator

PEP380 — Syntax for Delegating to a Subgenerator (Python 3.3)

    !python
    class gen1:
        def __init__(self):
            self.v = 1

        def __iter__(self):
            return self

        def __next__(self):
            if self.v > 2:
                raise StopIteration(3)
            v = self.v
            self.v += 1
            return v

    def gen2():
        value = yield from gen1()
        assert value == 3

    for x in gen2():
        print(x)
    # 1
    # 2

# Presenter Notes
4 строчки против 13 не очень-то читаемых magic методов

но(!) важно знать, что это всё не какая-то магия, а просто сахар

итоги: у нас есть генераторы и делегирование (pep380)

что дальше?

---

# Generator-based framework

---

# Generator-based framework

    !python
    def a():
        trace("a enter")
        time.sleep(1)
        trace("a exit")

    def b():
        trace("b enter")
        time.sleep(.5)
        trace("b middle")
        time.sleep(.5)
        trace("b exit")

    a()
    b()

    # 0.000 a enter
    # 1.001 a exit
    # 1.001 b enter
    # 1.502 b middle
    # 2.002 b exit

# Presenter Notes
a и b не могут выполняться одновременно из-за блокирующего вызова

хотя казалось бы, почему бы двум функциям не спать одновременно?
не ждать данных из сокета одновременно? и т.д.

нам нужен фреймворк, который позволит им использовать sleep конкурентно

что значит конкурентно? это значит что все будут ждать сразу, но в самих функциях не должно быть
вызовов, мы должны их переложить на того, кто сможет объеденить хотелки функций и отдавать им результат по готовности.
для этого подходят генераторы!

---

# Generator-based framework

    !python
    def a():
        trace("a enter")
        yield 1
        trace("a exit")

    def b():
        trace("b enter")
        yield .5
        trace("b middle")
        yield .5
        trace("b exit")

# Presenter Notes
вот так мы хотим

нужен кто-то, кто будет будет в нужное время инициировать очередную итерацию генератора

напишем простой loop

---

# Generator-based framework

    !python
    def a():
        trace("a enter")
        yield 1
        trace("a exit")

    def b():
        trace("b enter")
        yield .5
        trace("b middle")
        yield .5
        trace("b exit")

    def run(*gens):
        wakes = {g: 0 for g in gens}
        while wakes:
            g = min(wakes, key=lambda g: wakes[g])
            time.sleep(max(0, wakes[g] - time.time()))
            try:
                wakes[g] = time.time() + next(g)
            except StopIteration:
                wakes.pop(g)
    run(a(), b())

# Presenter Notes
принцип работы

аналогия с asyncio

---

# Generator-based framework

    !python
    def a():
        trace("a enter")
        yield 1
        trace("a exit")

    def b():
        trace("b enter")
        yield .5
        trace("b middle")
        yield .5
        trace("b exit")

    run(a(), b())

    # 0.000 a enter
    # 0.000 b enter
    # 0.500 b middle
    # 1.001 a exit
    # 1.001 b exit

# Presenter Notes
асинхронный фреймворк готов!

проверим что с yield from всё будет работать так, как мы того ожидаем

---

# Generator-based framework

    !python
    def a():
        trace("a enter")
        yield 1
        trace("a exit")
        return "a result"

    def b():
        trace("b enter")
        yield .5
        trace("b middle")
        result = yield from a()
        trace(f"a result is {result!r}")
        yield .5
        trace("b exit")
    run(b())

    # 0.000 b enter
    # 0.501 b middle
    # 0.501 a enter
    # 1.502 a exit
    # 1.502 a result is 'a result'
    # 2.002 b exit

# Presenter Notes
почему 2 секунды?

это практически asyncio!

почему в asyncio yield from а у нас тут yield?

на месте a() располагаются вызовы к вашим любимым асинхронным библиотекам: aiohttp, asyncpg, etc.

---

# Что дальше?

![what-next](images/what-next.jpg)

---

* async/await
#
    !python
    async def foo():
        await bar()

# Presenter Notes
в один прекрасный момент (python 3.5) было решено добавить ключевые слова async/await,
при этом суть не изменилась, просто более приятные и понятные названия

жёсткая пометка что это корутина, а не по наличию yield или декоратору

---

* async/await
* async iterator
#
    !python
    class AIterator:
        def __aiter__(self):
            return self

        async def __anext__(self):
            # some async code

    async for v in AIterator():
        ...

# Presenter Notes
такие же методы только с приставкой "a"

обратите внимание, что aiter это обычная функция

---

* async/await
* async iterator
* async context manager
#
    !python
    class AContext:
        async def __aenter__(self):
            # some async code

        async def __aexit__(self, *exc_info):
            # some async code

    async with AContext() as resource:
        ...

# Presenter Notes
такие же методы только с приставкой "a"

---

* async/await
* async iterator
* async context manager
* async generator (asend, athrow, aclose)
#
    !python
    async def foo():
        for i in range(10):
            v = await bar()
            yield v

    async for v in foo():
        ...

# Presenter Notes
такие же методы только с приставкой "a"

нельзя yield в finally

cleanup не может выполняться вне лупа

---

* async/await
* async iterator
* async context manager
* async generator (asend, athrow, aclose)
#
    !python
    # python 3.7+

    @contextlib.asynccontextmanager
    async def foo():
        resource = await initialize()
        try:
            yield resource
        finally:
            await finalize(resource)

    async with foo() as resource:
        ...

# Presenter Notes
в 3.7 добавили асинхронный контекстный менеджер

если добавить `yield from`, то можно будет делать фреймворки на асинхронных генераторах, иными словами
запускать loop внутри loop-а!

---

![deeper](images/deeper.jpg)

# Presenter Notes
какой в этом смысл?

---

![вам-не-понять](images/вам-не-понять.jpg)

# Presenter Notes
это конечно шутка, добавить обычный `yield from` не получится, нужна специальная версия, что-то вроде
`yield from async`, но скорее всего этого не случится

---

# Спасибо

* landslide
* Роман Силаков
* David M. Beazley and James Powell

---

# Вопросы
