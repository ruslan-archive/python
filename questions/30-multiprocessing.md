---
topic: multiprocessing
block: "IV. Конкурентность"
number: 30
status: todo
---

# Собес Python — MULTIPROCESSING

## БАЗА

1. `multiprocessing` vs `threading` — когда какой? Почему для CPU-bound не берут потоки?
2. Какие три start method существуют? Какой дефолт на Linux, macOS, Windows?
3. Что делает `fork` на уровне ОС? Что наследует дочерний процесс от родителя?
4. Почему `if __name__ == "__main__":` обязателен при `spawn` и не нужен при `fork`? Что случится без него?
5. Чем `Process.join()` отличается от `Process.terminate()`? Что делает `kill()`?
6. `daemon=True` у процесса — что меняет? Может ли daemon-процесс порождать детей?
7. Что такое zombie-процесс? Кто его пожинает в `multiprocessing`?
8. `Queue` vs `Pipe` — разница в устройстве и в числе endpoint'ов?
9. Как передаётся объект в другой процесс? Почему lambda нельзя отправить в `Pool.map`?
10. `Pool.map` vs `Pool.imap` vs `Pool.imap_unordered` — разница в поведении и в памяти?
11. `Pool.apply_async` возвращает `AsyncResult` — что делает `.get(timeout=...)` при исключении в воркере?
12. Зачем `multiprocessing.Lock`, если у каждого процесса своя память? Что он защищает на самом деле?

## СРЕДНИЙ УРОВЕНЬ

13. `fork` не копирует память сразу — copy-on-write. Почему в CPython большой read-only список всё равно раздувает RSS в детях?
14. `forkserver` — как он устроен? Какую проблему `fork` он решает и какую цену платит?
15. Почему `fork` в процессе с потоками — UB? Что происходит с mutex'ом, захваченным чужим потоком в момент `fork`?
16. Начиная с 3.12 `os.fork()` в многопоточном процессе кидает `DeprecationWarning`. С 3.14 дефолт на Linux сменился — на что и почему?
17. `multiprocessing.Queue` внутри — что за буфер, что за feeder thread? Почему `q.put()` может вернуться до того, как данные реально ушли?
18. Классический deadlock: родитель делает `join()` перед вычиткой `Queue`. Объясни механику зависания.
19. `Queue.qsize()` на macOS кидает `NotImplementedError`. Почему? Почему `qsize()` в принципе ненадёжен?
20. `multiprocessing.Value` и `Array` — где живёт память? Чем `Value('i', 0)` отличается от `Value('i', 0, lock=False)`?
21. `Manager()` — как он работает? Почему `manager.list()` в 100x медленнее `Array` и почему `m_list[0].append(x)` молча теряет данные?
22. PEP 574, out-of-band buffers, `pickle` protocol 5 — как это ускоряет передачу больших массивов?
23. `multiprocessing.shared_memory` (PEP 554? нет — какой PEP/версия?) — как отдать numpy-массив в 10 воркеров без копии? Кто отвечает за `unlink()`?
24. Ресурс-трекер (`resource_tracker`) — зачем нужен? Откуда берётся warning «leaked shared_memory objects»?
25. Почему `Pool` внутри `Pool` (вложенные пулы) ломается? Что даёт `maxtasksperchild`?
26. `ProcessPoolExecutor` vs `multiprocessing.Pool` — разница в API, в обработке падения воркера, в `BrokenProcessPool`.
27. Что произойдёт с `Pool`, если воркер убит OOM-killer'ом? Как поведут себя `Pool.map` в 3.8 и в `ProcessPoolExecutor`?
28. Логирование из 8 процессов в один файл — почему строки перемешиваются и почему `QueueHandler` — не роскошь?

## ЗАДАЧИ (напиши код)

29. Реализуй параллельный подсчёт частот слов в 200 файлах через `Pool`. Ограничь память: не читай всё в RAM. Дай сложность и вариант с `imap_unordered` + чем он лучше.
30. Напиши worker pool на голых `Process` + `JoinableQueue` с корректным shutdown по sentinel. Обработай Ctrl+C так, чтобы дети не осиротели.
31. Отдай numpy-массив 4GB в 8 воркеров через `shared_memory` без копирования. Гарантируй освобождение при исключении в любом воркере.
32. Реализуй `parallel_map(fn, items, timeout_per_item)`: воркер, зависший дольше timeout, убивается, задача помечается как failed, пул продолжает работу.
33. Напиши процесс-безопасный счётчик на `Value`. Покажи версию с race condition и версию без — объясни, почему `v.value += 1` не атомарен.
34. Реализуй передачу исключения из воркера в родителя с полным traceback (стандартный pickle теряет `__traceback__`). Второй способ?

---

# 20 ВОПРОСОВ НА СЕНЬОРНОСТЬ

**35.** `fork()` копирует таблицу страниц, а не память. Но воркер, только читающий большой `dict`, за минуту раздувает RSS до размера родителя. Кто пишет в read-only страницы и что с этим сделал GC-флаг `gc.freeze()` (3.7)?

**36.** До 3.14 дефолт на Linux — `fork`, на macOS с 3.8 — `spawn`. Что именно сломалось в macOS 10.13+ (`objc`, `+[__NSCFConstantString initialize]`, `CoreFoundation`), что заставило Python сменить дефолт?

**37.** `forkserver` форкается от процесса, замороженного в момент импорта `__main__`. Твой модуль открывает пул соединений к БД на import-time. Что произойдёт и чем это отличается от того же кода под `fork` и под `spawn`?

**38.** POSIX разрешает после `fork()` в многопоточном процессе вызывать только async-signal-safe функции. Python вызывает `malloc`. Почему в 99% случаев это работает и что за 1% выглядит как вечное зависание в `pthread_mutex_lock`?

**39.** `pthread_atfork` handlers в CPython (`PyOS_AfterFork_Child`) — что они переинициализируют? Почему `threading._after_fork()` объявляет все чужие потоки «мёртвыми», и как это ломает `logging`?

**40.** `multiprocessing.Queue.put(obj)` кладёт в `collections.deque` и будит feeder thread. Родитель делает `put()` 1M мелких объектов и сразу `exit()`. Данные дойдут? Что делает `cancel_join_thread()` и почему это footgun?

**41.** Пикл 100MB numpy-массива: protocol 2 — ~0.4s, protocol 5 с `buffer_callback` — ~0.02s. Откуда 20x? Что такое `PickleBuffer` и почему без out-of-band копий именно две, а не одна?

**42.** `SharedMemory` на Linux — это `/dev/shm` (tmpfs), а не anonymous mmap. Что это значит для контейнера с `--shm-size=64m`? Как поведёт себя `shared_memory.SharedMemory(create=True, size=1<<30)` в таком контейнере?

**43.** `resource_tracker` — отдельный процесс, отслеживающий семафоры и сегменты SHM. Почему он переживает падение родителя, но не `kill -9` всей process group? Что остаётся в `/dev/shm` и кто это чистит?

**44.** Воркер `Pool` убит OOM-killer'ом посреди `map()`. В Python 3.7 `Pool.map` висел вечно; в `ProcessPoolExecutor` — `BrokenProcessPool`. Что технически чинил bpo-22393 и почему полное решение потребовало отдельного watchdog-потока?

**45.** `Manager().list()` — это прокси к отдельному серверному процессу поверх `Connection`. Замерь: `mgr_list.append(1)` ~ 30-100 µs, `list.append(1)` ~ 30 ns. Куда ушли 3 порядка? Почему `nested list` в прокси не работает без `.register()`?

**46.** `os.fork()` в приложении, где asyncio event loop уже запущен. Что унаследует ребёнок из `epoll`-дескриптора и почему `loop.run_forever()` в ребёнке будет обрабатывать события родителя? Как это диагностировать?

**47.** `pickle` — известная RCE-поверхность (`__reduce__`). `multiprocessing.connection` использует HMAC-аутентификацию через `authkey`. Как `authkey` попадает в ребёнка при `fork` и при `spawn`, и почему `Listener(address=('0.0.0.0', 5000))` — дыра?

**48.** Go — горутины + channels на одном процессе, Java — потоки на shared heap без GIL, Rust — `Send`/`Sync` на уровне типов. Python выбрал процессы + pickle barrier. Какую конкретную проблему CPython это обходит и что сломает PEP 703 (free-threaded)?

**49.** PEP 684 — per-interpreter GIL (3.12), PEP 734 — `concurrent.interpreters` (3.14). Это конкурент `multiprocessing`? Где subinterpreters выигрывают у процессов, а где всё равно проигрывают?

**50.** Ты обрабатываешь 500M строк CSV (300GB на диске), RAM 64GB, 32 ядра. Спроектируй: сколько воркеров, как шардировать, чем передавать данные (Queue / shared_memory / mmap / файлы), где будет bottleneck и почему не Spark?

**51.** `Pool(processes=64)` на машине с 8 ядрами при CPU-bound задаче даёт замедление относительно `Pool(8)` не на 0%, а на 30-40%. Помимо context switching — что ещё? (cache thrashing, TLB, page faults — объясни вклад каждого).

**52.** `maxtasksperchild=100` — воркер убивается и пересоздаётся. Какую проблему это лечит, кроме memory leak? Почему это ломается под `spawn` при дорогом import-time и как измерить break-even?

**53.** `atexit`-хендлеры и `multiprocessing.util.Finalize`. Ребёнок, созданный `fork`, унаследовал `atexit`-хендлеры родителя. Что произойдёт при `sys.exit()` в ребёнке и почему `os._exit()` часто единственный правильный выход?

**54.** Tail latency: `Pool.map(fn, items, chunksize=None)` вычисляет chunksize как `divmod(len(items), len(pool)*4)`. При 1000 задач и распределении времени с длинным хвостом это даёт 3x худший makespan, чем `chunksize=1`. Объясни механику. Когда наоборот?

---

**Бонус-киллер:** Родитель форкается, ребёнок делает `q.put(huge_obj)` и сразу `os._exit(0)`. Родитель читает `q.get()` — получает объект. Тот же код под `spawn` — родитель висит вечно на `get()`. Объясни разницу, если pickle, Queue и объект идентичны.

Хочешь разбор ответов? Скажи номера.
