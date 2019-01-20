![bicycle](bicycle.png)

# Presenter Notes
* что кроется за синтаксическим сахаром в пифоне
* некоторые best practice для новичков и не только
* магические методы
* базовое понимание асинхронного программирования в питоне

---

# Обо мне

Никита Мелентьев. Ведущий инженер-программист КТБС.

[https://github.com/pohmelie](https://github.com/pohmelie)

[https://github.com/aio-libs/aioftp](https://github.com/aio-libs/aioftp)

# Presenter Notes
* big pulls: micropython, cerberus
* 40 pulls on github

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

![dino](динозавр-философ.jpg)

---

    !python
    f = open("yoba.txt", "w")
    f.write("foo")

# Presenter Notes
* что не так?
* ленивый оставит так, потому что при уничтожении будет close и всё запишется
* явное лучше неявного

---

    !python
    f = open("yoba.txt", "w")
    f.write("foo")
    f.close()

# Presenter Notes
* достаточно ли это хорошее решение?
* если есть возможность забыть вызвать метод, значит это обязательно случится

---

    !python
    import os
    import signal

    f = open("yoba.txt", "w")
    f.write("foo")
    # time
    os.kill(os.getpid(), signal.SIGKILL)

# Presenter Notes
* time - какая-то работа программы
* текст не попадёт в файл из-за буферизации
* какие идеи?
* нужен сахар!

---

# Context manager

# Presenter Notes
создатели решили, да, это частая задача, инициализировать ресурс, использовать и финализировать

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

---

# Context manager

    !python
    with open("yoba.txt", "w") as f:
        f.write("foo")

# Presenter Notes
благо уже написано, но есть вариант ещё лучше для этой конкретной задачи...

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
* pathlib отличная библиотека для работы с путями, файлами, итерации по директориям и т.д.
* идеальна для one shot задач: json, yaml, xml, small binaries etc. всё, что влезет в память
