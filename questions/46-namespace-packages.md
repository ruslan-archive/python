---
topic: namespace-packages
block: "VI. Модули, пакеты, окружение"
number: 46
status: todo
---

# Собес Python — NAMESPACE PACKAGES (PEP 420, plugins, `__path__`)

## БАЗА

1. Что такое namespace package и чем он отличается от regular package?
2. Какой файл обязателен для regular package и что происходит с пакетом, если этот файл удалить в Python 3.3+?
3. Что вернёт `type(mypkg.__path__)` для namespace package и что — для regular?
4. Чему равен `__file__` у namespace package?
5. Namespace package лежит в двух разных директориях `sys.path`. Сколько раз он попадёт в `sys.modules`?
6. Портион (portion) — что это в терминах PEP 420?
7. Почему namespace package нельзя собрать в один `.pyc`?
8. Может ли namespace package содержать внутри себя regular package и наоборот?
9. Что произойдёт, если в одном портионе namespace-пакета появится `__init__.py`?
10. Как импорт-система понимает, что директория без `__init__.py` — namespace package, а не мусор?
11. Что покажет `mypkg.__loader__` и `mypkg.__spec__.origin` для namespace package?
12. Зачем namespace packages нужны для плагинов — какую конкретную проблему решают?

## СРЕДНИЙ УРОВЕНЬ

13. Опиши алгоритм PEP 420: что делает `FileFinder` для каждой записи `sys.path`, когда встречает директорию с нужным именем, но без `__init__.py`?
14. Почему namespace package создаётся только после того, как **весь** `sys.path` просканирован до конца, а regular package — сразу?
15. `_NamespacePath` — почему это не `list`, а свой класс? Что он пересчитывает и по какому триггеру?
16. Ты добавил новый портион в `sys.path` уже после импорта namespace-пакета. Подхватится ли он при `import mypkg.newsub` без `importlib.invalidate_caches()`?
17. В чём разница между PEP 420 namespace packages и старыми `pkgutil.extend_path` / `pkg_resources.declare_namespace`?
18. Почему `pkg_resources.declare_namespace` и PEP 420 в одном дистрибутиве — источник багов? Что ломается первым?
19. Что такое implicit namespace package в контексте `setuptools.find_packages()` и почему появился `find_namespace_packages()`?
20. `sys.path_importer_cache` и `FileFinder._path_mtime` — как кеш директорий влияет на обнаружение новых портионов?
21. Namespace package внутри zip-архива или wheel'а — работает? Что говорит `zipimport`?
22. Как namespace packages ломают `python -m` и `runpy`?
23. Почему mypy/Pyright требуют флаг (`--namespace-packages`, `namespacePackages`) вместо автодетекта?
24. Что вернёт `importlib.resources.files("ns_pkg")` для namespace package, разбросанного по трём портионам?
25. `pkgutil.iter_modules(mypkg.__path__)` для namespace package с портионами в разных директориях — вернёт всё или только первый?
26. Почему editable install (`pip install -e`) исторически конфликтует с namespace packages? Что изменил PEP 660?
27. Namespace package с именем, совпадающим с именем директории в CWD — какой сценарий shadowing тут возможен и почему он тише обычного?

## ЗАДАЧИ (напиши код)

28. Собери на диске (через `tmp_path`) namespace package `acme`, разбросанный по двум корням: `root_a/acme/one.py` и `root_b/acme/two.py`. Импортируй оба подмодуля в одном процессе. Покажи `acme.__path__` до и после второго импорта.
29. Напиши функцию `discover_plugins(ns: str) -> dict[str, ModuleType]`, которая находит и импортирует все подмодули namespace-пакета через `pkgutil.iter_modules` + `importlib.import_module`. Обработай портион, который падает на импорте.
30. Реализуй `is_namespace(name: str) -> bool` без импорта самого модуля — только через `importlib.util.find_spec`. По какому полю `ModuleSpec` определяешь?
31. Напиши тест на pytest, который доказывает: новый портион, добавленный в `sys.path` после первого импорта, не виден до `importlib.invalidate_caches()`. Тест должен падать без вызова и проходить с ним.
32. Сделай кастомный `MetaPathFinder`, который отдаёт виртуальный namespace package `virt`, чей `__path__` собирается из списка директорий, приходящего из env var. Каким должен быть `spec.loader` и `spec.submodule_search_locations`?
33. Дан namespace `app.plugins` с 200 портионами. Напиши дискавери, который импортирует только те подмодули, чьё имя матчит префикс, и меряет время. Сравни с `entry_points()` из `importlib.metadata` — что дешевле и почему?

---

# 20 ВОПРОСОВ НА СЕНЬОРНОСТЬ

**34.** PEP 420 принят в 3.3, но `pkgutil.extend_path` жив до сих пор. Назови конкретный сценарий, где `extend_path` делает то, чего PEP 420 не может.

**35.** Почему PEP 402 (предшественник, "simplified packages") был отклонён в пользу PEP 420? Какую именно семантику `sys.path` он ломал?

**36.** `_bootstrap_external.PathFinder._get_spec()` собирает `namespace_path` в цикле по `sys.path`. Почему найденный regular package или module **прерывает** цикл, а найденная namespace-директория — нет? Какую цену за это платит импорт несуществующего модуля?

**37.** `import nonexistent_pkg` в проекте с 40 записями в `sys.path` — сколько `stat()`/`listdir()` в худшем случае и почему namespace packages сделали "промах импорта" дороже, чем он был в 3.2?

**38.** `_NamespacePath` пересчитывает содержимое через `_recalculate()` при каждом `__iter__`/`__len__`, сравнивая `sys.path` через `_epoch`/`path_importer_cache`. Напиши сценарий, где это даёт O(N) обход `sys.path` в горячем цикле импорта.

**39.** Ты мутируешь `mypkg.__path__.append("/extra")` для namespace package. Что произойдёт при следующем `_recalculate()` и почему это отличается от того же трюка на regular package?

**40.** Два дистрибутива, `acme-core` и `acme-plugins`, оба ставят `acme/` без `__init__.py`. Один разработчик добавил `acme/__init__.py` "чтобы линтер молчал". Опиши точную цепочку до продового ImportError и почему она не воспроизводится локально.

**41.** Security: namespace package permanently "открыт" для дописывания портионов любым, кто может положить директорию в любую запись `sys.path`. Сравни риск с regular package. Что делает `sys.path[0] == ''` в этом сценарии?

**42.** В Java есть sealed packages и class loader isolation, в Go — module path. Почему Python выбрал "склеить всё, что нашлось" и какой класс атак/багов это открывает, которого нет в Java?

**43.** `importlib.metadata.entry_points()` vs namespace-package discovery для плагинов. Дай два конкретных технических аргумента за entry points и один сценарий, где namespace-дискавери всё же выигрывает.

**44.** Почему `importlib.resources.files()` на namespace package исторически кидал ошибку, а с 3.12 отдаёт `MultiplexedPath`? Что нельзя сделать с `MultiplexedPath`, что можно с обычным `Traversable`?

**45.** `__init__.py`-based плагин-пакет vs namespace: при 200 плагинах и cold start в Lambda замерь, что именно доминирует — `stat()` по портионам или байткод-компиляция `__init__`. Как ты это померяешь без профайлера?

**46.** Namespace package + `zipapp`/PyInstaller/`--onefile`. Почему сборщики стабильно теряют портионы и какой минимальный хук чинит это в PyInstaller?

**47.** PEP 660 (editable installs) и `__editable___*_finder.py`: как современный setuptools подсовывает namespace-портион, не трогая `sys.path`? Что это ломает в inspection-инструментах?

**48.** `mypkg.__spec__.submodule_search_locations` у namespace package — это `_NamespacePath`. Ты сериализуешь spec (pickle, передача в subprocess). Что сломается и почему `__spec__` намеренно не picklable?

**49.** `python -m acme.plugins.cli` при namespace `acme.plugins` в двух портионах, где обе директории содержат `cli.py`. Кто выиграет и почему это не тот же порядок, что у `pkgutil.iter_modules`?

**50.** Опиши архитектуру плагин-системы: 500 плагинов, cold start ≤ 200 мс, плагины ставятся независимыми wheel'ами, нужен lazy import. Namespace packages в этой схеме — да или нет, и что вместо `iter_modules`?

**51.** Monorepo, 30 сервисов, общий namespace `company.*`. Что конкретно ломается в pytest (`rootdir`, `conftest`, `--import-mode=prepend` vs `importlib`) и почему `importlib` mode обязателен?

**52.** Namespace package `acme` присутствует и в site-packages, и в CWD (частичный чекаут). Импорт "работает", тесты зелёные, прод падает. Какой механизм PEP 420 сделал эту ошибку тихой и как её ловить в CI одной проверкой?

**53.** Trade-off без правильного ответа: ты пишешь фреймворк с публичным extension point. Namespace package (`framework_plugins.*`), entry points, или явный registry-декоратор + config? Защити выбор через версионирование, uninstall и debuggability.

---

**Бонус-киллер:** `len(mypkg.__path__)` вернул 2, следом `list(mypkg.__path__)` — вернул 3, при этом `sys.path` не менялся ни одной строкой Python-кода между вызовами. Объясни, как это возможно, и почему сам факт, что `__path__` — не список, делает любой код вида `if "/x" in mypkg.__path__` потенциально недетерминированным.

Хочешь разбор ответов? Скажи номера.
