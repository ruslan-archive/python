---
topic: databases
block: "X. Прикладное"
number: 63
status: todo
---

# Собес Python — DATABASES

## БАЗА

1. PEP 249 (DB-API 2.0) — какие объекты обязан отдавать драйвер и что делает `connect()` кроме открытия сокета?
2. `Connection` vs `Cursor` — кто из них держит транзакцию, кто держит результат выборки?
3. `paramstyle`: `qmark`, `numeric`, `named`, `format`, `pyformat` — почему один и тот же SQL не переносится между `sqlite3` и `psycopg2`?
4. Почему `cursor.execute("SELECT * FROM t WHERE id = %s" % user_id)` — дыра, а `cursor.execute("... = %s", (user_id,))` — нет? Кто именно экранирует?
5. `fetchone()` / `fetchmany(size)` / `fetchall()` — где лежат строки до вызова fetch и сколько их уже в памяти клиента?
6. `sqlite3` по умолчанию открывает транзакцию неявно. Когда именно и на каких statement'ах?
7. Autocommit — что это на уровне протокола БД и что произойдёт с `INSERT`, если не вызвать `commit()` и просто закрыть процесс?
8. `rollback()` при закрытии `Connection` — гарантирован? Что говорит PEP 249 и что делают реальные драйверы?
9. Уровни изоляции: `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE` — какие аномалии каждый убирает?
10. Connection pool — зачем, если открыть соединение = один TCP handshake? Назови настоящую цену открытия соединения в PostgreSQL.
11. SQLAlchemy: `Engine`, `Connection`, `Session` — кто из них потокобезопасен, кто нет?
12. `Session.add()` — когда объект реально уходит в БД? Что такое flush и чем отличается от commit?
13. N+1 — определение. Покажи минимальный код на ORM, который его порождает.

## СРЕДНИЙ УРОВЕНЬ

14. `cursor.executemany()` в `psycopg2` до версии 2.7 работал катастрофически медленно. Почему, и что изменили `execute_values` / `execute_batch`?
15. Server-side cursor (`named cursor`) vs client-side — как выбрать при `SELECT` на 50M строк и почему обычный курсор убьёт процесс?
16. `sqlite3.connect(..., isolation_level=None)` — что это включает и почему это НЕ то же самое, что autocommit в PostgreSQL?
17. `check_same_thread=False` в `sqlite3` — что именно ты отключаешь и какой race получаешь взамен?
18. SQLAlchemy `QueuePool`: `pool_size`, `max_overflow`, `pool_timeout`, `pool_recycle`. Что произойдёт при `pool_size=5, max_overflow=0` и 20 конкурентных запросах?
19. `pool_pre_ping=True` — какую проблему решает и какой оверхед добавляет на каждый checkout?
20. `NullPool` vs `QueuePool` — когда `NullPool` единственно верный выбор (назови два сценария)?
21. Пул + fork (gunicorn preload / celery prefork) — что ломается и почему `Engine.dispose()` обязателен в `post_fork`?
22. `lazy="select"` / `"joined"` / `"subquery"` / `"selectin"` / `"raise"` — чем `selectinload` лучше `joinedload` на коллекции и когда наоборот?
23. `joinedload` на `one-to-many` без `.unique()` в SQLAlchemy 2.0 кидает исключение. Почему картезианское произведение — это проблема ORM, а не SQL?
24. `expire_on_commit=True` (дефолт) — почему после `commit()` доступ к атрибуту объекта внезапно порождает новый `SELECT`?
25. Identity map в `Session` — два `session.get(User, 1)` дают один объект. Что случится, если между ними другой процесс изменил строку?
26. `SELECT ... FOR UPDATE` vs `FOR UPDATE SKIP LOCKED` — как построить очередь задач на таблице и почему `SKIP LOCKED` спасает от конвоя?
27. Оптимистичная блокировка через `version_id_col` — что SQLAlchemy добавляет в `WHERE` и что ловит `StaleDataError`?
28. Deadlock между двумя транзакциями, обновляющими те же две строки в разном порядке — как детерминированно чинить на уровне кода приложения?
29. `asyncio` + БД: почему нельзя отдать один `AsyncSession` двум конкурентным `asyncio.gather` тасках? Что кинет SQLAlchemy?
30. Транзакция открыта → внутри HTTP-запрос к внешнему API на 3 секунды. Что горит в БД всё это время?

## ЗАДАЧИ (напиши код)

31. Напиши контекстный менеджер `transaction(conn)` поверх голого DB-API: commit на выходе, rollback на любом исключении, поддержка вложенности через SAVEPOINT. Что делать с исключением внутри самого rollback?
32. Реализуй итератор `chunked_fetch(cursor, size)`, который отдаёт строки по одной, но тянет из БД пачками по `size` и не держит весь результат в памяти. Оцени память и число round-trip'ов.
33. Дано: `orders` (1M), у каждого `items` (в среднем 5). Нужно отдать JSON `[{order_id, total, item_names}]`. Напиши ORM-версию без N+1 и raw-SQL-версию. Сравни число запросов и время.
34. Напиши retry-декоратор для транзакции: ретраит только на serialization failure / deadlock (`SQLSTATE 40001`, `40P01`), с экспоненциальным бэкоффом и джиттером. Почему нельзя ретраить внутри уже открытой транзакции?
35. Реализуй bulk upsert 100k строк в PostgreSQL из Python. Три варианта: `executemany`, `execute_values`, `COPY` во временную таблицу + `INSERT ... ON CONFLICT`. Порядок по скорости и почему?
36. Напиши свой минимальный пул соединений: `acquire()` / `release()`, лимит N, таймаут ожидания, проверка живости, корректная работа при исключении в пользовательском коде. Где здесь race?

---

# 20 ВОПРОСОВ НА СЕНЬОРНОСТЬ

**37.** PostgreSQL: одно соединение = один backend-процесс, `work_mem` выделяется на операцию, не на соединение. Почему `pool_size=100` на 4-ядерной машине даёт throughput ХУЖЕ, чем `pool_size=10`? Причём тут PgBouncer в режиме `transaction`?

**38.** PgBouncer в `transaction pooling` ломает prepared statements. Почему `psycopg2` выживает, а `asyncpg` падает с `prepared statement "__asyncpg_stmt_1__" already exists`? Как чинят через `statement_cache_size=0`?

**39.** `sqlite3` в CPython до 3.12 держал GIL во время `sqlite3_step()`? Проверь утверждение: что реально делает `Py_BEGIN_ALLOW_THREADS` в `_sqlite3` и почему тяжёлый `SELECT` в SQLite не блокирует другие треды, а `json_serializable` конвертер — блокирует.

**40.** `sqlite3.threadsafety` в 3.11 сменился с `1` на `3` (bpo/gh-issue про `SQLITE_THREADSAFE`). Что это меняет для твоего кода и почему это не значит «теперь можно шарить `Connection` без блокировок»?

**41.** PEP 249 говорит `.rowcount` = -1, если неизвестно. Назови три реальные ситуации, где драйвер вернёт -1 или соврёт, и почему код `if cursor.rowcount == 0: raise NotFound` — бомба.

**42.** MVCC: `UPDATE` в PostgreSQL — это `INSERT` новой версии + пометка старой. Твой воркер обновляет одну строку-счётчик 10k раз в секунду. Что произойдёт с таблицей за час и почему HOT-update не всегда спасает? Твоя архитектура?

**43.** Idle in transaction: соединение открыло транзакцию и уснуло. Почему это блокирует `VACUUM` по ВСЕЙ базе, а не только по затронутым таблицам? Что такое `xmin horizon`?

**44.** SQLAlchemy `Session` — не потокобезопасен, `Engine` — потокобезопасен, `Connection` — нет. Объясни ровно, ЧТО внутри `Session` не защищено (identity map? unit of work? oба?) и почему `scoped_session` — это не решение, а обход.

**45.** Unit of Work в SQLAlchemy сортирует INSERT'ы по топологии зависимостей таблиц. При циклической FK (self-referencing `parent_id`) он вставляет с NULL и делает `UPDATE` вторым проходом. Когда эта эвристика даёт лишний `UPDATE` на каждую строку и как убить через `post_update`?

**46.** `selectinload` генерирует `WHERE id IN (...)` пачками по 500 (`IN`-chunk). Почему именно чанкует, а не льёт все 100k id разом? Что ломается в PostgreSQL при `IN` на 100k параметров (лимит 65535 — откуда число)?

**47.** N+1 существует и в Java Hibernate, и в Rails ActiveRecord, и в Django. Hibernate решает `@BatchSize` и `FetchMode.SUBSELECT`. Почему Python-ORM исторически не могут применить самое элегантное решение — batching на уровне драйвера — и что тут делает отсутствие request-scoped контекста?

**48.** Django `select_related` = JOIN, `prefetch_related` = второй запрос. SQLAlchemy `joinedload` = JOIN, `selectinload` = второй запрос. Дай сценарий, где JOIN проигрывает второму запросу в 10 раз по трафику, и посчитай почему.

**49.** Lazy loading вне сессии → `DetachedInstanceError`. Почему `expire_on_commit=False` — популярный, но опасный фикс? Какую именно гарантию ты теряешь и как это выглядит на проде через полгода?

**50.** `asyncpg` быстрее `psycopg2` в 3-5 раз на простых запросах. Не потому что async. Назови настоящую причину (подсказка: binary protocol vs text). Где преимущество исчезает?

**51.** SQL injection через второй порядок: параметризованный `INSERT` кладёт в БД строку `'; DROP TABLE--`, потом другой код читает её и клеит в SQL. Параметризация не помогла. Где реальная граница доверия и почему «санитизация на входе» — неверный ответ?

**52.** `LIKE '%' || $1 || '%'` с пользовательским вводом — параметризовано, инъекции нет. Какая DoS-атака остаётся? Причём тут ReDoS-подобное поведение и `pg_trgm`?

**53.** Ты пишешь миграцию: `ALTER TABLE users ADD COLUMN x int NOT NULL DEFAULT 0` на 500M строк. В PostgreSQL 11+ это мгновенно, в 10 — переписывает таблицу. Что изменили и какой `ALTER` до сих пор берёт `ACCESS EXCLUSIVE` на всю таблицу?

**54.** Distributed transaction между PostgreSQL и Redis. Two-phase commit (`PREPARE TRANSACTION`) доступен, но в проде его не используют. Почему? Что предлагаешь взамен и какую гарантию сознательно теряешь?

**55.** Trade-off без правильного ответа: 500M строк событий, 16GB RAM, нужно отдавать «последние 100 событий пользователя» за <50ms и агрегат «за месяц» за <2s. ORM или raw SQL? Партиционирование по времени или по user_id? Отдельная read-реплика или materialized view? Защищай выбор.

---

**Бонус-киллер:** Твой код делает `session.query(User).filter_by(id=1).one()`, получает объект, ты меняешь `user.email`, вызываешь `session.commit()`. Всё работает. Теперь ты добавил `@property` с тем же именем поверх колонки — тесты зелёные, но на проде часть UPDATE'ов молча теряется. Что произошло на уровне instrumentation SQLAlchemy и почему именно «часть»?

Хочешь разбор ответов? Скажи номера.
