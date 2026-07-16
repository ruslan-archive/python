# Python Interview Roadmap

Прогон через `/py-interview <тема>`. Отмечай `[x]` по мере прохождения.

## I. Типы данных и модель объекта
- [x] `strings` — immutability, PEP 393, interning, unicode, normalization
- [ ] `numbers` — int (arbitrary precision, small int cache), float (IEEE 754), Decimal, Fraction, complex
- [ ] `bytes-bytearray-memoryview` — buffer protocol, zero-copy
- [ ] `list` — over-allocation, growth factor, resize, list vs array
- [ ] `tuple` — freelist, namedtuple, зачем immutable
- [ ] `dict` — compact dict, open addressing, perturb, order guarantee, resize
- [ ] `set-frozenset` — отличия от dict, операции над множествами
- [ ] `collections` — deque, Counter, defaultdict, OrderedDict, ChainMap
- [ ] `object-model` — PyObject, refcount, `id`, `is` vs `==`, `__slots__`
- [ ] `mutability-copy` — shallow/deep copy, `__copy__`, mutable default arg
- [ ] `comparison-ordering` — `__eq__`/`__hash__` контракт, total_ordering, NaN

## II. Функции и исполнение
- [ ] `functions` — first-class, closure, cell, LEGB, late binding
- [ ] `arguments` — positional-only `/`, keyword-only `*`, `*args/**kwargs`
- [ ] `decorators` — с аргументами, functools.wraps, стек, класс-декоратор
- [ ] `generators` — yield, send/throw/close, PEP 342/380
- [ ] `iterators` — протокол, StopIteration, itertools, ленивость
- [ ] `comprehensions` — scope, walrus, генератор vs list comp
- [ ] `functools` — lru_cache/cache, partial, reduce, singledispatch, cached_property
- [ ] `context-managers` — `__enter__/__exit__`, contextlib, ExitStack

## III. ООП и протоколы
- [ ] `classes-inheritance` — MRO, C3, super(), diamond, `__init_subclass__`
- [ ] `descriptors` — data vs non-data, как работают property/method/slots
- [ ] `magic-methods` — dunder-набор, `__getattr__` vs `__getattribute__`
- [ ] `metaclasses` — type, `__new__` vs `__init__`, `__prepare__`, когда НЕ нужны
- [ ] `properties-slots` — property, slots, конфликты с наследованием
- [ ] `abc-protocols` — ABC, abstractmethod, `__subclasshook__`, Protocol
- [ ] `dataclasses` — field, frozen, `__post_init__`, slots=True, vs attrs/pydantic
- [ ] `enum` — Enum/IntEnum/StrEnum/Flag, auto, 3.11+ изменения

## IV. Конкурентность
- [ ] `gil` — switch interval, convoy effect, free-threaded build PEP 703
- [ ] `threading` — Lock/RLock/Semaphore/Event/Condition, deadlock, atomic ops
- [ ] `multiprocessing` — fork vs spawn vs forkserver, pickle-барьер, shared memory
- [ ] `asyncio-basics` — event loop, coroutine, Task, await, run
- [ ] `asyncio-advanced` — gather vs TaskGroup, cancellation, shielding, timeout
- [ ] `concurrent-futures` — Executor, ThreadPool vs ProcessPool, map vs submit
- [ ] `async-patterns` — async ctx manager, async gen, backpressure, sync↔async мосты

## V. Внутренности CPython
- [ ] `memory-management` — pymalloc, arena/pool/block, refcount, GC поколения
- [ ] `garbage-collection` — generational GC, `gc`, `__del__`-ловушки, weakref
- [ ] `bytecode` — `dis`, VM, стековая машина, peephole, specializing interpreter
- [ ] `interpreter-internals` — frame, code object, cell/free vars, `sys._getframe`
- [ ] `c-api-extensions` — C API, Cython, ctypes, cffi, GIL в расширениях
- [ ] `performance-optimization` — профилирование, векторизация, PyPy/Numba
- [ ] `python-versions` — что дала каждая версия 3.8→3.14, deprecations

## VI. Модули, пакеты, окружение
- [ ] `imports` — sys.modules, finders/loaders, circular imports, `__init__`
- [ ] `packaging` — pyproject.toml, wheel vs sdist, entry points, editable install
- [ ] `dependency-management` — pip/poetry/uv, lock-файлы, resolver
- [ ] `virtual-environments` — venv, изоляция, PYTHONPATH, site-packages
- [ ] `namespace-packages` — PEP 420, plugins, `__path__`

## VII. Типизация
- [ ] `typing-basics` — Optional/Union, generics, TypeVar, runtime vs static
- [ ] `typing-advanced` — ParamSpec, Concatenate, overload, Self, TypedDict, variance
- [ ] `mypy-static-analysis` — strict, gradual typing, Any-дыры, stubs
- [ ] `runtime-validation` — pydantic v2, `get_type_hints`, PEP 563/649

## VIII. Ошибки, отладка, качество
- [ ] `exceptions` — иерархия, chaining, ExceptionGroup, EAFP
- [ ] `logging` — hierarchy, handlers, filters, structured logging, perf
- [ ] `debugging` — pdb, breakpoint(), post-mortem, tracing, faulthandler
- [ ] `testing` — pytest, fixtures, parametrize, mock/patch, hypothesis
- [ ] `code-quality` — ruff, black, pre-commit, complexity metrics

## IX. Stdlib и I/O
- [ ] `io-files` — buffering, text vs binary, encoding, seek/tell, pathlib
- [ ] `datetime` — naive vs aware, tz, zoneinfo, DST-ловушки
- [ ] `json-serialization` — json, pickle (дыры), msgpack, protobuf
- [ ] `regex` — re vs regex, backtracking, compile-кэш, флаги
- [ ] `itertools-functools-operator` — рецепты, ленивые пайплайны
- [ ] `subprocess-os` — run vs Popen, shell=True, pipes, deadlock, signals
- [ ] `stdlib-hidden-gems` — bisect, heapq, array, struct, shelve, difflib, secrets

## X. Прикладное
- [ ] `databases` — DB-API, pooling, transactions, SQLAlchemy, N+1
- [ ] `web-frameworks` — WSGI vs ASGI, Django/Flask/FastAPI
- [ ] `http-clients` — requests vs httpx vs aiohttp, retry, timeouts
- [ ] `security` — pickle RCE, injection, secrets vs random, timing attacks
- [ ] `data-structures-algorithms` — реализация на Python, реальная сложность
- [ ] `design-patterns-python` — почему GoF в Python другие, DI, композиция

---

**Порядок:** I → II → III → V → IV → VII → остальное.

**Минимум для senior, если мало времени:** dict, object-model, descriptors, magic-methods, generators, decorators, gil, asyncio-advanced, memory-management, garbage-collection, imports, typing-advanced.
