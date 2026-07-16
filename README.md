# Python Interview Prep

68 тем для подготовки к собесу на Python. Каждая тема = отдельный файл вопросов в `questions/`: база → средний уровень → задачи на код → 20 вопросов на сеньорность → бонус-киллер. Ответов в файлах нет.

Вопросы генерирует скилл `.claude/skills/py-interview/` — `/py-interview <тема>` соберёт набор по любой теме, в т.ч. новой.

Отмечай `[x]` по мере прохождения.

## I. Типы данных и модель объекта
- [x] [`strings`](questions/01-strings.md) — immutability, PEP 393, interning, unicode, normalization
- [ ] [`numbers`](questions/02-numbers.md) — int (arbitrary precision, small int cache), float (IEEE 754), Decimal, Fraction, complex
- [ ] [`bytes-bytearray-memoryview`](questions/03-bytes-bytearray-memoryview.md) — buffer protocol, zero-copy
- [ ] [`list`](questions/04-list.md) — over-allocation, growth factor, resize, list vs array
- [ ] [`tuple`](questions/05-tuple.md) — freelist, namedtuple, зачем immutable
- [ ] [`dict`](questions/06-dict.md) — compact dict, open addressing, perturb, order guarantee, resize
- [ ] [`set-frozenset`](questions/07-set-frozenset.md) — отличия от dict, операции над множествами
- [ ] [`collections`](questions/08-collections.md) — deque, Counter, defaultdict, OrderedDict, ChainMap
- [ ] [`object-model`](questions/09-object-model.md) — PyObject, refcount, `id`, `is` vs `==`, `__slots__`
- [ ] [`mutability-copy`](questions/10-mutability-copy.md) — shallow/deep copy, `__copy__`, mutable default arg
- [ ] [`comparison-ordering`](questions/11-comparison-ordering.md) — `__eq__`/`__hash__` контракт, total_ordering, NaN

## II. Функции и исполнение
- [ ] [`functions`](questions/12-functions.md) — first-class, closure, cell, LEGB, late binding
- [ ] [`arguments`](questions/13-arguments.md) — positional-only `/`, keyword-only `*`, `*args/**kwargs`
- [ ] [`decorators`](questions/14-decorators.md) — с аргументами, functools.wraps, стек, класс-декоратор
- [ ] [`generators`](questions/15-generators.md) — yield, send/throw/close, PEP 342/380
- [ ] [`iterators`](questions/16-iterators.md) — протокол, StopIteration, itertools, ленивость
- [ ] [`comprehensions`](questions/17-comprehensions.md) — scope, walrus, генератор vs list comp
- [ ] [`functools`](questions/18-functools.md) — lru_cache/cache, partial, reduce, singledispatch, cached_property
- [ ] [`context-managers`](questions/19-context-managers.md) — `__enter__/__exit__`, contextlib, ExitStack

## III. ООП и протоколы
- [ ] [`classes-inheritance`](questions/20-classes-inheritance.md) — MRO, C3, super(), diamond, `__init_subclass__`
- [ ] [`descriptors`](questions/21-descriptors.md) — data vs non-data, как работают property/method/slots
- [ ] [`magic-methods`](questions/22-magic-methods.md) — dunder-набор, `__getattr__` vs `__getattribute__`
- [ ] [`metaclasses`](questions/23-metaclasses.md) — type, `__new__` vs `__init__`, `__prepare__`, когда НЕ нужны
- [ ] [`properties-slots`](questions/24-properties-slots.md) — property, slots, конфликты с наследованием
- [ ] [`abc-protocols`](questions/25-abc-protocols.md) — ABC, abstractmethod, `__subclasshook__`, Protocol
- [ ] [`dataclasses`](questions/26-dataclasses.md) — field, frozen, `__post_init__`, slots=True, vs attrs/pydantic
- [ ] [`enum`](questions/27-enum.md) — Enum/IntEnum/StrEnum/Flag, auto, 3.11+ изменения

## IV. Конкурентность
- [ ] [`gil`](questions/28-gil.md) — switch interval, convoy effect, free-threaded build PEP 703
- [ ] [`threading`](questions/29-threading.md) — Lock/RLock/Semaphore/Event/Condition, deadlock, atomic ops
- [ ] [`multiprocessing`](questions/30-multiprocessing.md) — fork vs spawn vs forkserver, pickle-барьер, shared memory
- [ ] [`asyncio-basics`](questions/31-asyncio-basics.md) — event loop, coroutine, Task, await, run
- [ ] [`asyncio-advanced`](questions/32-asyncio-advanced.md) — gather vs TaskGroup, cancellation, shielding, timeout
- [ ] [`concurrent-futures`](questions/33-concurrent-futures.md) — Executor, ThreadPool vs ProcessPool, map vs submit
- [ ] [`async-patterns`](questions/34-async-patterns.md) — async ctx manager, async gen, backpressure, sync↔async мосты

## V. Внутренности CPython
- [ ] [`memory-management`](questions/35-memory-management.md) — pymalloc, arena/pool/block, refcount, GC поколения
- [ ] [`garbage-collection`](questions/36-garbage-collection.md) — generational GC, `gc`, `__del__`-ловушки, weakref
- [ ] [`bytecode`](questions/37-bytecode.md) — `dis`, VM, стековая машина, peephole, specializing interpreter
- [ ] [`interpreter-internals`](questions/38-interpreter-internals.md) — frame, code object, cell/free vars, `sys._getframe`
- [ ] [`c-api-extensions`](questions/39-c-api-extensions.md) — C API, Cython, ctypes, cffi, GIL в расширениях
- [ ] [`performance-optimization`](questions/40-performance-optimization.md) — профилирование, векторизация, PyPy/Numba
- [ ] [`python-versions`](questions/41-python-versions.md) — что дала каждая версия 3.8→3.14, deprecations

## VI. Модули, пакеты, окружение
- [ ] [`imports`](questions/42-imports.md) — sys.modules, finders/loaders, circular imports, `__init__`
- [ ] [`packaging`](questions/43-packaging.md) — pyproject.toml, wheel vs sdist, entry points, editable install
- [ ] [`dependency-management`](questions/44-dependency-management.md) — pip/poetry/uv, lock-файлы, resolver
- [ ] [`virtual-environments`](questions/45-virtual-environments.md) — venv, изоляция, PYTHONPATH, site-packages
- [ ] [`namespace-packages`](questions/46-namespace-packages.md) — PEP 420, plugins, `__path__`

## VII. Типизация
- [ ] [`typing-basics`](questions/47-typing-basics.md) — Optional/Union, generics, TypeVar, runtime vs static
- [ ] [`typing-advanced`](questions/48-typing-advanced.md) — ParamSpec, Concatenate, overload, Self, TypedDict, variance
- [ ] [`mypy-static-analysis`](questions/49-mypy-static-analysis.md) — strict, gradual typing, Any-дыры, stubs
- [ ] [`runtime-validation`](questions/50-runtime-validation.md) — pydantic v2, `get_type_hints`, PEP 563/649

## VIII. Ошибки, отладка, качество
- [ ] [`exceptions`](questions/51-exceptions.md) — иерархия, chaining, ExceptionGroup, EAFP
- [ ] [`logging`](questions/52-logging.md) — hierarchy, handlers, filters, structured logging, perf
- [ ] [`debugging`](questions/53-debugging.md) — pdb, breakpoint(), post-mortem, tracing, faulthandler
- [ ] [`testing`](questions/54-testing.md) — pytest, fixtures, parametrize, mock/patch, hypothesis
- [ ] [`code-quality`](questions/55-code-quality.md) — ruff, black, pre-commit, complexity metrics

## IX. Stdlib и I/O
- [ ] [`io-files`](questions/56-io-files.md) — buffering, text vs binary, encoding, seek/tell, pathlib
- [ ] [`datetime`](questions/57-datetime.md) — naive vs aware, tz, zoneinfo, DST-ловушки
- [ ] [`json-serialization`](questions/58-json-serialization.md) — json, pickle (дыры), msgpack, protobuf
- [ ] [`regex`](questions/59-regex.md) — re vs regex, backtracking, compile-кэш, флаги
- [ ] [`itertools-functools-operator`](questions/60-itertools-functools-operator.md) — рецепты, ленивые пайплайны
- [ ] [`subprocess-os`](questions/61-subprocess-os.md) — run vs Popen, shell=True, pipes, deadlock, signals
- [ ] [`stdlib-hidden-gems`](questions/62-stdlib-hidden-gems.md) — bisect, heapq, array, struct, shelve, difflib, secrets

## X. Прикладное
- [ ] [`databases`](questions/63-databases.md) — DB-API, pooling, transactions, SQLAlchemy, N+1
- [ ] [`web-frameworks`](questions/64-web-frameworks.md) — WSGI vs ASGI, Django/Flask/FastAPI
- [ ] [`http-clients`](questions/65-http-clients.md) — requests vs httpx vs aiohttp, retry, timeouts
- [ ] [`security`](questions/66-security.md) — pickle RCE, injection, secrets vs random, timing attacks
- [ ] [`data-structures-algorithms`](questions/67-data-structures-algorithms.md) — реализация на Python, реальная сложность
- [ ] [`design-patterns-python`](questions/68-design-patterns-python.md) — почему GoF в Python другие, DI, композиция

---

**Порядок:** I → II → III → V → IV → VII → остальное.

**Минимум для senior, если мало времени:** dict, object-model, descriptors, magic-methods, generators, decorators, gil, asyncio-advanced, memory-management, garbage-collection, imports, typing-advanced.
