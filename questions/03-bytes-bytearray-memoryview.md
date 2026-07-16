---
topic: bytes-bytearray-memoryview
block: "I. Типы данных и модель объекта"
number: 3
status: todo
---

# Собес Python — bytes / bytearray / memoryview

## БАЗА

1. `bytes` vs `bytearray` — в чём разница на уровне модели объекта? Какой из них можно положить в `set` и почему?
2. Что вернёт `b"abc"[0]` и что вернёт `b"abc"[0:1]`? Почему типы разные?
3. Как из `str` получить `bytes` и обратно? Почему `bytes(str)` без аргументов падает?
4. Что делает `bytes(10)` и чем отличается от `bytes([10])`?
5. Какие значения принимает литерал `b"..."`? Что произойдёт с `b"привет"`?
6. Сложность `bytearray.append`, `bytearray.insert(0, x)`, `bytes + bytes` — назови O() для каждой.
7. Что такое `memoryview` одним предложением? Зачем он нужен, если есть срезы?
8. `mv = memoryview(bytearray(b"hello"))`; `mv[0] = 65` — сработает? А если внутри `bytes`?
9. Что покажут `mv.readonly`, `mv.format`, `mv.itemsize`, `mv.nbytes` для `memoryview(b"abc")`?
10. Почему `b"abc" * 3` дешевле, чем цикл с `+=` по `bytes`? А по `bytearray`?
11. Что такое `bytes.hex()` и `bytes.fromhex()`? Что будет с пробелами внутри hex-строки?
12. Какие методы `str` есть у `bytes` и какие принципиально не могут быть? Что с `bytes.format`?
13. Что вернёт `b"abc" == "abc"` и почему это не `TypeError`? Что меняет флаг `-b`?
14. Как прочитать файл в существующий буфер без лишней копии — какой метод у file object?

## СРЕДНИЙ УРОВЕНЬ

15. Опиши структуру `PyBytesObject` в CPython. Почему `sys.getsizeof(b"")` = 33, а `sys.getsizeof(b"a")` = 34?
16. Почему `bytes` — variable-size объект с inline-данными, а `bytearray` держит `ob_bytes` отдельным указателем? Как это меняет цену `realloc`?
17. Что такое `ob_exports` в `bytearray`? Что случится с `ba.resize()`/`ba.append()`, пока жив `memoryview(ba)`?
18. Как работает over-allocation в `bytearray`? Какой коэффициент роста и как он соотносится с list?
19. `bytearray` имеет `ob_start` (offset). Зачем? Какая операция становится O(1) благодаря ему?
20. Что такое buffer protocol (PEP 3118)? Назови роль `Py_buffer`, `bf_getbuffer`, `bf_releasebuffer`.
21. Флаги `PyBUF_SIMPLE`, `PyBUF_WRITABLE`, `PyBUF_C_CONTIGUOUS`, `PyBUF_STRIDES` — кто их запрашивает и что означает отказ?
22. Что такое non-contiguous memoryview? Как получить его в чистом Python без numpy и что сломается при `.tobytes()` vs `bytes(mv)`?
23. `mv.cast('I')` — какие ограничения? Почему нельзя cast для non-C-contiguous?
24. Почему `memoryview` нужно закрывать (`.release()` / `with`)? Что конкретно течёт, если не закрыть?
25. Интернирование: почему `b"" is b""` → True, а `bytes([1]) is bytes([1])` → False? Что кэширует CPython для однобайтовых `bytes`?
26. Чем `bytes(mv)` отличается от `mv.tobytes()` и от `mv.toreadonly()`? Где копия, где нет?
27. Зачем нужны `socket.recv_into` / `readinto` / `os.preadv`, если есть `recv`? Что они принимают на вход по протоколу?
28. Как `struct.pack_into` / `unpack_from` работают с writable буфером? Чем отличаются от `struct.pack`?
29. `%`-форматирование для `bytes` вернули в 3.5 (PEP 461). Почему сначала выпилили в py3.0 и почему пришлось вернуть?
30. Что такое `array.array` в контексте buffer protocol? Чем `memoryview(array('i', ...))` отличается от `memoryview(bytearray(...))` по `format`/`itemsize`?
31. Хэш `bytes` — сиквенс-хэш или тот же, что у `str`? Что делает `PYTHONHASHSEED` с `bytes`?

## ЗАДАЧИ (напиши код)

32. Реализуй парсер length-prefixed сообщений из потока (`4 байта big-endian длина + payload`), не создавая копий payload. Приёмный буфер — `bytearray`, наружу отдаёшь `memoryview`. Что делать с хвостом? Назови сложность на M сообщений.
33. Напиши `read_exactly(sock, n)` на `recv_into` + `memoryview`, без `+=` и без промежуточных `bytes`. Обработай `recv_into` → 0. Сколько копий делает твоя версия и сколько наивная?
34. Реализуй in-place XOR двух буферов одинаковой длины максимально быстро на чистом Python. Дай два способа (побайтово и через `int.from_bytes`) и сравни по сложности и по аллокациям.
35. Напиши функцию `chunks(buf, n)`, отдающую zero-copy окна буфера, включая неполный последний кусок. Что вернуть, если `buf` — readonly? Тест на границах.
36. Реализуй ring buffer фиксированного размера на `bytearray` с `write(data)`/`read(n)`. Wrap-around — без конкатенации. Как отдать read-результат одним куском, если он пересекает границу?
37. Напиши constant-time сравнение двух `bytes` вручную (без `hmac.compare_digest`) и объясни, где твоя реализация всё равно течёт по времени.
38. Реализуй `hexdump(buf, width=16)` — офсет, hex-байты, ASCII-колонка — работая через `memoryview`, без материализации `bytes`.

---

# 20 ВОПРОСОВ НА СЕНЬОРНОСТЬ

**39.** `b = bytearray(10**8); mv = memoryview(b); del b` — сколько памяти освободилось? Теперь `mv.release()`. Что произошло с `ob_exports` и когда реально сработал `tp_dealloc`?

**40.** `data[1:] ` на `bytes` в 100MB в цикле парсера даёт O(N²). Классический фикс — `memoryview`. Но `mv = mv[1:]` в цикле на 10M итераций тоже медленный и жрёт память. Почему? Что происходит с цепочкой `Py_buffer` и `mv.obj`?

**41.** CPython кэширует `bytes` длины 0 и все 256 однобайтовых (`characters[256]` в `bytesobject.c`). Почему для `str` аналогичный кэш ведёт себя иначе на не-ASCII? Что даёт этот кэш в реальном парсере и чем он опасен при работе с `id()`?

**42.** PEP 3118 ввёл `Py_buffer` со `strides`, `suboffsets`, `shape`. `suboffsets` — для чего они, кто их реально использует и почему `memoryview` в CPython бросает на них исключение в большинстве операций?

**43.** `bytes(memoryview_non_contiguous)` работает, а `struct.unpack_from(fmt, mv)` — нет. Какой именно флаг `PyBUF_*` запрашивает каждый и почему `struct` не хочет делать implicit copy?

**44.** `hmac.compare_digest(a, b)` для `bytes` и для `str` работает по-разному. Что происходит при `str` с не-ASCII? Почему это не просто «медленнее», а security-баг?

**45.** `int.from_bytes(b, 'big')` на 1MB буфере: какая сложность? Почему `int.to_bytes` обратно — не симметрично по цене? Что поменял CPython 3.11 с лимитом на конвертацию int↔str и почему bytes-конвертация под лимит не попала?

**46.** У тебя `bytearray` растёт через `+=` до 1GB. RSS процесса — 2.4GB. Объясни цифру: over-allocation, `realloc` без `mremap`, фрагментация pymalloc? Что из этого реально виновато и как проверить?

**47.** `bytearray.__iadd__` vs `bytearray.extend` vs `bytearray.append` в цикле на 10M байт — расставь по скорости и объясни разницу через байткод (`dis`) и число вызовов `resize`.

**48.** В Go `[]byte` и `string` конвертируются с копией, но компилятор её убирает в известных паттернах (`map[string(b)]`). В Rust есть `&[u8]`/`Vec<u8>`/`Bytes` с borrow checker. Что в Python играет роль borrow checker для `memoryview` и в чём фундаментальная разница в гарантиях?

**49.** Ты пишешь C-extension и получаешь `Py_buffer` через `PyArg_ParseTuple` с форматом `y*` vs `s*` vs `y#`. Разница? Кто обязан звать `PyBuffer_Release` и что случится, если забыть?

**50.** `mv = memoryview(b); mv2 = mv.cast('B', shape=[2, 3])` — как cast меняет `ndim` и почему `mv[0]` после этого возвращает `memoryview`, а не `int`? Что вернёт `mv.tolist()` и во сколько это дороже `.tobytes()`?

**51.** Ты держишь 500M коротких бинарных ключей (по ~20 байт) в памяти, RAM 16GB. Список из `bytes` не влезает. Опиши архитектуру: один большой `bytearray` + индекс офсетов? `array('Q')`? mmap? Посчитай оверхед каждого варианта.

**52.** `mmap.mmap` поддерживает buffer protocol. `memoryview(mmap_obj)[:1000]` — сколько байт прочитано с диска? Что произойдёт при truncate файла под живым memoryview и почему это SIGBUS, а не Python-исключение?

**53.** Два потока: один пишет `ba[i] = x`, другой читает `ba[i]`. Это atomic под GIL? А `ba += b"..."`? А `ba.extend(...)` из двух потоков — какие гарантии даёт GIL и какие нет?

**54.** `pickle` протокол 5 (PEP 574) ввёл out-of-band buffers и `PickleBuffer`. Какую конкретную проблему это решило для `bytes`/`memoryview` и почему `buffer_callback` обязателен на стороне сериализации?

**55.** В Python 2 `str` = `bytes`, а `bytearray` появился в 2.6 как бэкпорт. Какая именно боль py2 привела к жёсткому разделению в py3? Приведи пример кода, который в py2 работал «правильно» на ASCII и молча портил данные на не-ASCII.

**56.** Zlib/hashlib принимают buffer protocol и отпускают GIL. `hashlib.sha256(mv).hexdigest()` на non-contiguous `mv` — что произойдёт внутри, отпустится ли GIL и сохранится ли zero-copy?

**57.** `b"a" in b"abcdef"` использует не наивный поиск. Какой алгоритм в CPython (`stringlib/fastsearch.h`), с какого момента он включается и какой у него worst case? Можно ли устроить DoS на подстрочном поиске в `bytes`?

**58.** Ты отдаёшь наружу `memoryview` на внутренний `bytearray` пула. Клиент сохраняет его и читает через 10 секунд — данные уже перезаписаны. Как задизайнить API, чтобы это было невозможно, не делая копию на каждый вызов? Разбери варианты и цену каждого.

---

**Бонус-киллер:** `ba = bytearray(b"abc"); mv = memoryview(ba); ba.append(0x64)` — падает с `BufferError`. Но `ba[0] = 0x41` при живом `mv` проходит, и `mv[0]` тоже станет `0x41`. А теперь: `mv = memoryview(ba)[1:]; del ba` — объект жив, данные читаются. Кто держит ссылку, что лежит в `mv.obj`, и почему `BufferError` — это не про мутацию, а про адрес?

Хочешь разбор ответов? Скажи номера.
