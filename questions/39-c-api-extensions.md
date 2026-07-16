---
topic: c-api-extensions
block: "V. Внутренности CPython"
number: 39
status: todo
---

# Собес Python — C API и расширения

## БАЗА

1. Что такое `PyObject` и какие поля у него обязательны в non-GC объекте?
2. Разница между `Py_INCREF` и `Py_XINCREF`? Когда обязателен второй?
3. Что значит «функция возвращает borrowed reference» и почему это опасно? Приведи пример API, который так делает.
4. `PyList_GetItem` vs `PyList_GET_ITEM` — в чём разница и когда второй нельзя использовать?
5. Что означает соглашение «steals a reference»? Назови две функции C API, которые крадут ссылку.
6. Как расширение возвращает `None`? Почему `return Py_None;` — баг (до 3.12)?
7. Что такое `METH_VARARGS`, `METH_KEYWORDS`, `METH_O`, `METH_NOARGS` в `PyMethodDef`?
8. Как парсить аргументы: `PyArg_ParseTuple` vs `PyArg_ParseTupleAndKeywords`? Что означают форматы `s`, `s#`, `y*`, `O!`, `O&`?
9. Чем ctypes отличается от cffi по подходу? Что такое ABI-mode и API-mode в cffi?
10. Что даёт Cython поверх чистого C-расширения? Разница между `def`, `cdef`, `cpdef` функциями?
11. Что делают макросы `Py_BEGIN_ALLOW_THREADS` / `Py_END_ALLOW_THREADS` и во что они разворачиваются?
12. Как расширение сигнализирует об ошибке? Почему `return NULL;` без установленного исключения — баг?
13. Что такое `PyModuleDef` и почему у модуля есть `m_size`? Что значит `m_size = -1`?
14. Как из C проверить тип объекта: `PyLong_Check` vs `PyLong_CheckExact` — когда разница критична?

## СРЕДНИЙ УРОВЕНЬ

15. PEP 384 — Stable ABI. Что именно стабилизировали, что запретили и почему `Py_LIMITED_API` ломает `PyObject_HEAD` inline-доступ?
16. Что такое abi3-wheel? Какие ограничения по версии CPython и почему нельзя использовать статические типы (`PyTypeObject` в статике) в Limited API?
17. PEP 489 — multi-phase init. Зачем `PyModuleDef_Slot`, `Py_mod_exec`, `Py_mod_create`? Какую проблему это решило для субинтерпретаторов?
18. PEP 573 — как из метода C-типа получить состояние своего модуля без глобальных переменных? Что делает `PyType_GetModuleState`?
19. Как правильно поддержать GC в C-типе: `Py_TPFLAGS_HAVE_GC`, `tp_traverse`, `tp_clear`, `PyObject_GC_Track/UnTrack`. Что случится, если забыть `tp_traverse`?
20. Почему в `tp_dealloc` GC-типа нужен `PyObject_GC_UnTrack` до освобождения полей?
21. Что такое `Py_TPFLAGS_BASETYPE` и `Py_TPFLAGS_DEFAULT`? Почему без первого твой тип нельзя наследовать?
22. `PyType_FromSpec` vs статический `PyTypeObject` — что даёт heap type и какую цену платим?
23. Как работает `tp_new` vs `tp_init` vs `tp_alloc`? Кто из них вызывается при `copy.copy` / `pickle`?
24. Что такое буферный протокол (PEP 3118)? Что должен заполнить `bf_getbuffer` и зачем нужен `bf_releasebuffer`?
25. Как ctypes определяет `restype`/`argtypes` и что произойдёт, если их не задать на 64-бит платформе для функции, возвращающей указатель?
26. `ctypes.CDLL` vs `ctypes.PyDLL` — в чём разница относительно GIL?
27. Cython: `nogil` блок, `with nogil:`, директива `boundscheck=False`. Что можно и чего категорически нельзя делать внутри `nogil`?
28. Cython memoryview (`double[:, ::1]`) vs `np.ndarray` в сигнатуре — что быстрее и почему? Что означает `::1`?
29. Как в C API корректно вызвать Python-код из чужого (не-Python) потока? Что делают `PyGILState_Ensure` / `PyGILState_Release`?
30. Что такое `PyErr_Fetch`/`PyErr_Restore` (и `PyErr_GetRaisedException` в 3.12+)? Зачем сохранять исключение в `tp_dealloc`?
31. `PyMem_Malloc` vs `PyObject_Malloc` vs `malloc` — три аллокатора. Когда какой и что ломается при смешивании?

## ЗАДАЧИ (напиши код)

32. Напиши C-расширение с одной функцией `sum_ints(seq)`, принимающей произвольный iterable и возвращающей сумму как Python int. Обработай ошибки итерации и не-int элементы. Где здесь утечка, если написать наивно?
33. Через `ctypes` вызови `qsort` из libc для массива `int`, передав Python-функцию как компаратор. Замерь сложность вызова и объясни, почему это медленнее, чем `list.sort()`.
34. Напиши Cython-функцию, считающую скалярное произведение двух `double[::1]` в `nogil`-блоке. Покажи, как отпустить GIL и как проверить, что он реально отпущен.
35. Реализуй C-тип `Counter` с полем `PyObject *data` (dict). Напиши `tp_traverse`, `tp_clear`, `tp_dealloc` так, чтобы цикл `c.data['self'] = c` собирался GC. Докажи, что собирается.
36. Через `cffi` в API-mode оберни функцию `int parse(const char *s, size_t len, double *out)`. Верни `float` или подними `ValueError`. Как избежать копирования при передаче `bytes`?
37. Напиши функцию C API, принимающую `buffer` (PEP 3118) и возвращающую CRC32 без копирования данных, отпуская GIL на время расчёта. Что нужно проверить перед `Py_BEGIN_ALLOW_THREADS`?
38. Реализуй в C потокобезопасный счётчик (инкремент из нескольких потоков) двумя способами: под GIL и с атомиками. Покажи, где GIL уже не спасает.

---

# 20 ВОПРОСОВ НА СЕНЬОРНОСТЬ

**39.** `PyGILState_Ensure` из потока, созданного pthread'ом, в приложении с субинтерпретаторами (PEP 554/734) — почему это исторически ломалось и что изменил `PyThreadState`-per-interpreter? Почему `PyGILState_Ensure` привязан к main interpreter?

**40.** PEP 703 (free-threading, 3.13 `--disable-gil`). Что такое biased reference counting и deferred refcounting? Почему `Py_INCREF` в nogil-сборке стал дороже, но масштабируется лучше? Какие C-расширения молча ломаются?

**41.** В free-threaded сборке модуль обязан объявить `Py_mod_gil = Py_MOD_GIL_NOT_USED`. Что произойдёт при импорте расширения без этого слота и почему дефолт именно такой?

**42.** Твой C-тип держит `PyObject *cache` и не реализует `tp_traverse`. Тесты зелёные, но RSS растёт на 40 МБ/час в проде. Объясни механику утечки и почему `gc.collect()` не помогает. Как поймать это через `gc.get_referrers` / `PYTHONDEBUG`?

**43.** `Py_DECREF` внутри `tp_dealloc` может рекурсивно вызвать твой `tp_dealloc` для 1M-элементного связного списка → stack overflow. Как CPython решает это для `list`/`tuple` (`Py_TRASHCAN_BEGIN`)? Почему trashcan небезопасен для heap types до 3.9?

**44.** Бенчмарк: ctypes-вызов `libc.abs(-1)` ≈ 1.5 µs, cffi API-mode ≈ 90 нс, Cython `cdef extern` ≈ 15 нс. Разложи разницу по слоям: где libffi, где ffi_call, где генерация C-кода на этапе компиляции.

**45.** Ты пишешь расширение с Limited API (abi3, `Py_LIMITED_API=0x03090000`). Почему нельзя написать `((PyVarObject*)obj)->ob_size` и как получить длину? Что теряешь по перфу и какова реальная цена (числа)?

**46.** Функция C API `PyDict_GetItem` не устанавливает исключение и подавляет ошибки в `__hash__`/`__eq__`. Почему это ошибка дизайна, что предлагает `PyDict_GetItemRef` (3.13, PEP 741-эпоха) и какой класс багов это порождало в реальных расширениях?

**47.** Borrowed reference из `PyList_GetItem`, затем вызов произвольного Python-кода (например, `__eq__` элемента), который делает `lst.clear()`. Объясни use-after-free по шагам. Почему `PySequence_GetItem` дороже, но безопаснее?

**48.** `Py_BEGIN_ALLOW_THREADS` вокруг блока, где ты трогаешь `PyObject*` (даже просто читаешь `ob_refcnt`). Почему это UB даже в GIL-сборке, если объект может быть освобождён другим потоком? Что можно делать в nogil-регионе?

**49.** Cython: `cdef class` с `__dealloc__` и циклической ссылкой на Python-объект. Почему Cython по умолчанию генерирует `tp_traverse`, а `freelist` через `@cython.freelist(8)` его ломает? Когда freelist даёт реальный выигрыш (числа)?

**50.** PEP 3118: `PyBuffer_FillInfo` vs ручное заполнение `Py_buffer` для non-contiguous массива. Что означают `strides`, `suboffsets`, `PyBUF_ND` vs `PyBUF_STRIDES` vs `PyBUF_FULL`? Кто обязан вызвать `PyBuffer_Release` и что произойдёт, если не вызвать?

**51.** NumPy до 2.0 держал ABI-совместимость через `PyArray_API`-таблицу указателей и `import_array()`. Почему сборка расширения против NumPy 1.26 работает на 1.20 (forward), но не наоборот? Что сломала NumPy 2.0 и зачем `oldest-supported-numpy` существовал?

**52.** Security: `ctypes.cast(id(obj), ctypes.py_object)` даёт доступ к любому объекту по адресу. Как через ctypes изменить содержимое immutable `str` или `int`-кэш `[-5, 256]`? Почему это не считается CVE в CPython и где граница sandbox-модели?

**53.** Твой C-код вызывает `PyErr_SetString` и делает `return NULL`, но выше по стеку ты вызвал `Py_DECREF`, который дёрнул `__del__`, который сам поднял исключение. Что стало с твоим исключением? Как `PyErr_WriteUnraisable` и `sys.unraisablehook` (PEP 3151-эпоха, `unraisablehook` в 3.8) меняют картину?

**54.** Fork в процессе с C-расширением, держащим pthread-mutex, захваченный чужим потоком в момент fork. Классический deadlock. Почему `os.register_at_fork` не спасает чужой код и почему в 3.14 дефолт `multiprocessing` на Linux сменили на `forkserver`?

**55.** Архитектура: 500M строк CSV, 16 ГБ RAM, нужен парсинг + агрегация. Варианты: Cython + memoryview, C-расширение с mmap + буферный протокол, Rust/PyO3, Arrow C Data Interface (zero-copy). Выбери и защити — где GIL, где копии, где cache locality, какова цена maintenance?

**56.** Java JNI требует явный `env` в каждом вызове и GetPrimitiveArrayCritical, Go cgo платит ~50 нс за переключение стека и запрещает хранить Go-указатели в C, Rust/PyO3 даёт `Python<'py>` токен как compile-time доказательство удержания GIL. Что из этого CPython C API не может дать в принципе и почему?

**57.** Что было в Python 2: `PyString_AsString` возвращал внутренний буфер `str` — borrowed, valid до смерти объекта. В py3 `PyUnicode_AsUTF8` тоже возвращает указатель на кэш внутри объекта. Почему в 3.10 добавили `PyUnicode_AsUTF8AndSize` в Limited API только в 3.10, и когда этот указатель инвалидируется?

**58.** Trade-off без правильного ответа: библиотека с горячим числовым ядром. ctypes (ноль сборки, медленно), cffi (нужен компилятор в API-mode), Cython (свой язык, крупный build), pure-C + abi3 (одно колесо на все версии, максимум боли), PyO3 (Rust-тулчейн у контрибьюторов). Ты мейнтейнер OSS с 3 контрибьюторами и 50k загрузок/день. Твой выбор и что ты сознательно приносишь в жертву?

---

**Бонус-киллер:** `sys.getrefcount(obj)` всегда возвращает на 1 больше «настоящего» из-за аргумента. В free-threaded сборке 3.13+ для immortal-объектов (PEP 683) он возвращает `4294967295`. Объясни: как immortal-объекты обходят refcount, почему `Py_INCREF` для них — не no-op, а проверка, и что это сделало с перфом обычных объектов в GIL-сборке?

Хочешь разбор ответов? Скажи номера.
