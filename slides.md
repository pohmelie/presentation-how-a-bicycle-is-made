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
    f = open("yoba.txt", "w")
    f.write("foo")

# Presenter Notes
что не так?

кто-то забудет close и не заметит, потому что при уничтожении вызовется close

в этом случае повезло, но в подавляющем большинстве случаев явный close никто не вызывает неявно из `__del__`

---

    !python
    f = open("yoba.txt", "w")
    f.write("foo")
    f.close()

# Presenter Notes
достаточно ли это хорошее решение с точки зрения api?

если есть возможность забыть вызвать метод, значит это обязательно случится

---

    !python
    import os
    import signal

    f = open("yoba.txt", "w")
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


    with File("yoba.txt", "w") as f:
        f.write("foo")

# Presenter Notes
первые магические методы

три(!) магических метода

меня сделают core-dev благодаря такому контекстному менеджеру?

---

# Context manager

    !python
    with open("yoba.txt", "w") as f:
        f.write("foo")

# Presenter Notes
боюсь что нет, потому что объект, который возвращает open уже умеет всё сам

но есть вариант ещё лучше для этой конкретной задачи...

---

# No context manager :'(

    !python
    from pathlib import Path
    Path("yoba.txt").write_text("foo")

* read_text
* write_text
* read_bytes
* write_bytes

# Presenter Notes
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

# Presenter Notes
чтобы использовать инстанс/объект как функцию

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)

# Presenter Notes
мимикрировать под коллекцию

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)

# Presenter Notes
когда property недостаточно

динамические цепочки для api

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
* comparison (`__lt__`, `__eq__`, ...)

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
* comparison (`__lt__`, `__eq__`, ...)
* descriptors (`__get__`, `__set__`, `__set_name__`)

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

---

# Обзор magic methods
* callable (`__call__`)
* item access (`__getitem__`, `__setitem__`, ...)
* attribute access (`__getattr__`, `__setattr__`, ...)
* comparison (`__lt__`, `__eq__`, ...)
* descriptors (`__get__`, `__set__`, `__set_name__`)
* arithmetic/binary (`__and__`, `__or__`, `__add__`, `__sub__`, ...)
* metaclasses (`__new__`, `__init_subclass__`, ...)

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

# Presenter Notes
интерфейсы для builtin функций, но не только

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
