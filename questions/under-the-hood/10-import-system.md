---
topic: import-system
block: "Под капотом"
number: 10
status: todo
---

# Собес Python — Система импорта

## БАЗА

1. Что делает `import foo` пошагово: где ищется модуль, куда кладётся результат, какое имя биндится в текущем namespace?
2. Зачем нужен `sys.modules` и что произойдёт, если удалить оттуда запись и повторить `import`?
3. Разница между `import foo`, `from foo import bar` и `import foo.bar` — что именно оказывается в локальных именах?
4. Что такое пакет vs модуль? Что делает файл `__init__.py`?
5. Разница между `__import__` и `importlib.import_module` — что вернёт каждый для `import a.b.c`?
6. Что такое абсолютный и относительный импорт (`from . import x`)? Когда относительный работает, когда падает?
7. Почему запуск `python -m package.module` и `python package/module.py` дают разное поведение относительных импортов?
8. Что за директория `__pycache__` и файлы `.pyc`? Когда Python их перегенерирует?
9. Что такое циклический импорт? Приведи пример, когда он падает с `ImportError`, и когда проходит.
10. Что содержат `__name__`, `__file__`, `__package__`, `__path__` у модуля/пакета?
11. Разница между `importlib.reload(mod)` и повторным `import mod` — что реально перевыполняется?
12. Что такое namespace package (PEP 420) и чем отличается от обычного пакета?

## СРЕДНИЙ УРОВЕНЬ

13. Опиши цепочку `sys.meta_path` → finder → `spec` → loader → module. Кто вызывает `exec_module`?
14. Что такое `ModuleSpec`? Какие поля (`name`, `loader`, `origin`, `submodule_search_locations`) на что влияют?
15. Разница между meta path finder (`sys.meta_path`) и path entry finder (`sys.path_hooks` + `sys.path_importer_cache`)?
16. Как `PathFinder` использует `sys.path`, `sys.path_hooks` и `sys.path_importer_cache`? Что кэшируется?
17. PEP 328: что убрали относительно py2 и почему `from __future__ import absolute_import` был нужен?
18. PEP 366: зачем `__package__`, как он чинит относительные импорты при запуске через `-m`?
19. Как работает module-level `__getattr__` (PEP 562)? Для чего его добавили в 3.7?
20. Как сделать lazy import через `importlib.util.LazyLoader`? Когда реально выполнится код модуля?
21. PEP 3147: почему `.pyc` переехали в `__pycache__` с тегом версии (`foo.cpython-311.pyc`)?
22. PEP 552: чем deterministic/hash-based `.pyc` отличается от timestamp-based? Что лежит в заголовке?
23. При циклическом импорте модуль в `sys.modules` уже есть, но частично инициализирован. Что видно из другого модуля в этот момент?
24. Как `-X importtime` помогает найти дорогой импорт? Что показывают колонки `self` и `cumulative`?
25. `importlib.metadata` (PEP 566): как получить версию и entry points установленного пакета без импорта самого пакета?
26. Что делает `sys.path[0]` при разных способах запуска (`script.py`, `-m`, `-c`, REPL)? Почему это security-угол?
27. Как namespace package (PEP 420) собирает `__path__` из нескольких директорий на разных `sys.path`? Что за `_NamespacePath`?
28. Чем `find_spec` отличается от старого `find_module`/`load_module` (deprecated protocol)?

## ЗАДАЧИ (напиши код)

29. Напиши meta path finder + loader, который перехватывает `import greeting` и отдаёт модуль с функцией `hello()` из строки-исходника. Второй способ — через `importlib.util.spec_from_loader`.
30. Реализуй lazy-import обёртку: `lazy("numpy")` возвращает прокси, код `numpy` выполняется только при первом обращении к атрибуту. Сравни с `importlib.util.LazyLoader`.
31. Дан пакет с циклическим импортом `a`↔`b`, падающий на `AttributeError`. Почини двумя способами: перенос импорта внутрь функции и реструктуризация. Объясни разницу.
32. Напиши функцию `import_from_path(name, filepath)`, грузящую модуль из произвольного `.py` не через `sys.path`. Используй `spec_from_file_location` + `module_from_spec` + `exec_module`.
33. Реализуй плагин-загрузчик: просканируй директорию `plugins/`, импортируй каждый `.py`, собери все классы-наследники `BasePlugin`. Второй способ — через entry points (`importlib.metadata`).
34. Напиши module-level `__getattr__`, который на доступ к удалённому имени выдаёт `DeprecationWarning` и возвращает новый объект. Добавь `__dir__`.

---

# 20 ВОПРОСОВ НА СЕНЬОРНОСТЬ

**35.** `importlib._bootstrap` и `_bootstrap_external` — почему они написаны так, что могут работать без импорта? Как frozen-модули (`python -X frozen_modules`) ускоряют старт интерпретатора и что заморожено?

**36.** `sys.meta_path` по умолчанию содержит `BuiltinImporter`, `FrozenImporter`, `PathFinder`. В каком порядке они опрашиваются и что сломается, если ты вставишь свой finder в конец, а не в начало?

**37.** PEP 420 убрал требование `__init__.py`. Какую цену заплатили: почему namespace package медленнее резолвится и как это бьёт по `import` в проекте с длинным `sys.path`?

**38.** Циклический импорт: `from a import thing` внутри `b` падает, а `import a; a.thing` — нет. Объясни через порядок «bind имя в `sys.modules` → выполнить тело» и момент, когда `thing` ещё не определён.

**39.** `-X importtime` показывает 200 ms на `import requests`. Как по колонкам `self`/`cumulative` найти виновника, и почему `self` дорогого модуля может быть маленьким при большом `cumulative`?

**40.** Import hijacking: пустой `sys.path[0]` (текущая директория) при запуске `python script.py` позволяет подложить `os.py` рядом. Как именно это эксплуатируется и что изменил PEP 432 / safe path (`-P`, `PYTHONSAFEPATH`) в 3.11?

**41.** `ModuleSpec.submodule_search_locations` — как через него сделать пакет, `__path__` которого вычисляется динамически? Приведи сценарий (виртуальный пакет-агрегатор нескольких неймспейсов).

**42.** Архитектура: плагин-система на 500 плагинов, часть тяжёлых (torch). Спроектируй загрузку через `sys.meta_path` + `LazyLoader` так, чтобы старт был O(1) по числу плагинов. Где узкие места?

**43.** PEP 552: hash-based `.pyc` с флагом `check_source=0` (unchecked). Как это ускоряет старт в контейнере и какой security trade-off (кто гарантирует, что `.pyc` соответствует исходнику)?

**44.** `importlib.reload` не обновляет уже связанные имена (`from mod import f` держит старую `f`). Почему технически невозможно сделать «настоящий» reload и как это решают hot-reload фреймворки (Django autoreload перезапускает процесс)?

**45.** Node.js `require` кэширует по абсолютному пути файла, Python `sys.modules` — по dotted-имени. К каким расхождениям это ведёт: один физический файл, импортированный как `pkg.mod` и как `mod`, — сколько объектов модуля в памяти?

**46.** py2 разрешал implicit relative import (`import string` внутри пакета брал локальный `string.py`). PEP 328 это убил. Какой класс багов исчез и какой новый появился (shadowing stdlib через `sys.path[0]`)?

**47.** `module.__getattr__` (PEP 562) вызывается только при промахе обычного lookup. Почему это дороже прямого атрибута и когда `__getattr__`-based lazy submodule ломает статические анализаторы/IDE?

**48.** `pickle` сериализует функции/классы по `module.__qualname__` и при `unpickle` делает `import`. Как это превращается в RCE-вектор и почему переименование модуля ломает старые пиклы?

**49.** Java грузит классы лениво по classpath через ClassLoader-иерархию с parent delegation. Сопоставь с Python `sys.meta_path`: где у Python аналог parent delegation и почему у Python нет проблемы «один класс — два ClassLoader → `ClassCastException`»?

**50.** `sys.path_importer_cache` кэширует path entry finders по строке пути. Что произойдёт, если во время работы программы создать пакет в директории, которая уже была в кэше как «не пакет»? Как инвалидировать (`importlib.invalidate_caches`)?

**51.** Спор: eager-импорты наверху файла vs lazy-импорты внутри функций. Приведи по два аргумента за каждый (старт CLI-тула vs предсказуемость `ImportError`, циклы, тестируемость). Где нет правильного ответа?

**52.** `python -c "import antigravity"` и подобные side-effect-при-импорте. Почему выполнение произвольного кода на этапе `import` — фундаментальное свойство модели Python, и как это ограничивают в песочницах (audit hooks, PEP 578 `sys.addaudithook`)?

**53.** `__pycache__` не пишется при read-only FS или `PYTHONDONTWRITEBYTECODE`. Насколько это замедляет старт и как `compileall` + предзагенеренные `.pyc` в Docker-образе решают проблему? Цена по размеру образа?

**54.** Editable install (`pip install -e`) в современном виде (PEP 660) кладёт `.pth` или finder-хук вместо копии. Как `.pth`-файлы выполняют произвольный код при старте интерпретатора и почему это одновременно фича и дыра?

---

**Бонус-киллер:** Модуль `m` в процессе выполнения своего тела делает `del sys.modules['m']`, затем `import m` (сам себя). Сколько раз выполнится тело, сколько объектов модуля создастся, и какой из них увидит внешний код, сделавший `import m` первым?

Хочешь разбор ответов? Скажи номера.
