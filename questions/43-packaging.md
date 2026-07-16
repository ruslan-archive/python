---
topic: packaging
block: "VI. Модули, пакеты, окружение"
number: 43
status: todo
---

# Собес Python — PACKAGING

## БАЗА

1. sdist и wheel — в чём разница по содержимому и по тому, что происходит на машине пользователя при `pip install`?

2. Что означают части имени файла `mypkg-1.2.0-cp311-cp311-manylinux_2_17_x86_64.whl`? Каждый сегмент.

3. Что такое `py3-none-any.whl` и когда пакет имеет право на такой тег?

4. Какие три таблицы верхнего уровня бывают в `pyproject.toml` и за что отвечает каждая?

5. Зачем нужна таблица `[build-system]`? Что произойдёт при её отсутствии в 2024+?

6. `requires` vs `build-backend` в `[build-system]` — что задаёт каждое поле?

7. Что такое entry points? Куда они попадают внутри wheel?

8. Разница между `[project.scripts]` и `[project.gui-scripts]`.

9. `dependencies` vs `optional-dependencies` в `[project]` — как пользователь ставит вторые?

10. Что делает `pip install -e .` и чем результат отличается от `pip install .`?

11. Зачем нужен `dynamic` в `[project]`? Приведи типичный случай.

12. Что такое `.dist-info` и какие файлы в нём обязательны?

13. Чем `setup.py install` отличается от `pip install .` и почему первый объявлен deprecated?

## СРЕДНИЙ УРОВЕНЬ

14. PEP 517 и PEP 518 — что именно ввёл каждый и почему их два, а не один?

15. Опиши хуки build backend по PEP 517: `build_wheel`, `build_sdist`, `get_requires_for_build_wheel`, `prepare_metadata_for_build_wheel`. Кто их вызывает и зачем нужен четвёртый?

16. Что такое build isolation? Как её отключить и когда это обосновано?

17. PEP 660 — как editable install реализован без `.pth`-хака setuptools? Что физически лежит в site-packages после `pip install -e .` с современным backend?

18. `.pth`-файл vs import hook (`__editable___*_finder.py`) — какие разные наблюдаемые эффекты у двух стратегий editable?

19. Как читать entry points в рантайме через `importlib.metadata.entry_points()`? Что поменялось в API в 3.10 и 3.12?

20. `[project.entry-points."my.plugin.group"]` — как плагинная система находит чужие пакеты, не импортируя их?

21. src-layout vs flat-layout — какой класс багов ловит src-layout и почему flat-layout его пропускает?

22. Что такое manylinux? Почему нельзя просто собрать wheel на Ubuntu и залить как `linux_x86_64`?

23. `auditwheel` и `delocate` — что они делают с бинарным wheel и зачем?

24. Метаданные `Requires-Dist` в wheel фиксируются на этапе сборки. Какие последствия для пакета с зависимостями, зависящими от платформы?

25. Environment markers (`sys_platform == "win32"`, `python_version < "3.11"`) — на каком этапе вычисляются и кем?

26. Что делает `pip install --no-binary :all:` и какие типичные проблемы всплывают?

27. Version specifiers по PEP 440: чем `~=1.4.2` отличается от `>=1.4.2, <1.5.0`? Что такое post-release и local version?

28. Зачем нужен `MANIFEST.in`, если есть `[tool.setuptools.package-data]`? Что попадает в sdist, а что в wheel?

29. Как нормализуются имена дистрибутивов по PEP 503? Почему `My_Pkg.Name` и `my-pkg-name` — одно и то же?

## ЗАДАЧИ (напиши код)

30. Напиши минимальный `pyproject.toml` для библиотеки `dataflow` с src-layout, зависимостями `httpx>=0.27` и `pydantic>=2`, extra-группой `dev` (pytest, mypy), CLI-командой `dataflow` → `dataflow.cli:main`, версией, читаемой из `dataflow/__init__.py` через `dynamic`.

31. Реализуй plugin registry: функция `load_plugins(group: str) -> dict[str, Callable]` находит все entry points группы, загружает их лениво (импорт только при первом обращении), не падает на битом плагине. Покажи, как объявить такой плагин в чужом `pyproject.toml`.

32. Напиши минимальный PEP 517 build backend (`build_wheel` + `build_sdist`), который собирает пакет из одного `.py`-файла. Wheel — обычный zip: собери `RECORD`, `WHEEL`, `METADATA` вручную.

33. Напиши функцию `parse_wheel_filename(name: str) -> WheelInfo`, разбирающую имя wheel на distribution/version/build tag/python tag/abi tag/platform tag. Учти compressed tag sets (`cp39.cp310-none-any`) и опциональный build tag.

34. Напиши скрипт, который для установленного пакета печатает дерево зависимостей с пометкой, какие из них подтянуты через extras. Только stdlib (`importlib.metadata`).

35. Реализуй сравнение версий по PEP 440 (без `packaging`): epoch, release, pre/post/dev. Отсортируй `["1.0", "1.0.post1", "1.0a1", "1.0.dev1", "1!0.1", "1.0rc1"]`. Сложность? Где твоя реализация разойдётся с `packaging.version`?

---

# 20 ВОПРОСОВ НА СЕНЬОРНОСТЬ

**36.** PEP 517 намеренно не дал backend'у возможности вернуть frontend'у список установленных пакетов, а `get_requires_for_build_wheel` вызывается уже внутри изолированного окружения. Какую атаку/проблему это закрывает и какую цену платят backend'ы вроде `meson-python`?

**37.** `pip install -e .` с PEP 660 и `__editable___finder` — почему после добавления **нового** файла в пакет он импортируется, а после добавления нового **подпакета** может не импортироваться? Что зависит от выбора `--config-settings editable_mode=strict`?

**38.** Wheel — zip. `RECORD` содержит sha256 каждого файла, но сам `RECORD` в себе не хешируется. Что это значит для integrity guarantee и что реально проверяет pip при установке?

**39.** manylinux_2_28 vs manylinux2014 — переход на perennial теги (PEP 600) убрал жёсткий whitelist библиотек. Что сломалось у мейнтейнеров pip/warehouse и почему старый подход не масштабировался?

**40.** Пакет A объявляет `install_requires=["B"]`, B в setup.py импортирует numpy для `get_include()`. Build isolation включена. Опиши точную последовательность отказа и почему `--no-build-isolation` — это лечение симптома.

**41.** `pip install foo==1.2.3` при наличии и sdist, и wheel: опиши полный алгоритм выбора кандидата, включая роль `--prefer-binary`, yanked-релизов (PEP 592) и `Requires-Python`.

**42.** Resolver pip до 20.3 был «first found wins», после — backtracking. Приведи конкретный конфликт зависимостей, который старый резолвер молча устанавливал сломанным, а новый отвергает. Какую цену заплатили по времени резолва?

**43.** У тебя монорепо: 40 пакетов, общий код, взаимные зависимости, CI ставит всё в один venv через `-e`. Опиши, где editable-инсталлы дадут расхождение с прод-сборкой, и как это ловить до релиза.

**44.** `python -m build` создаёт sdist, затем собирает wheel **из sdist**, а не из исходного дерева. Зачем этот лишний шаг и какой класс багов он ловит?

**45.** `setup.py` исполняется как код на машине пользователя при установке sdist. `pyproject.toml` — декларативен. Приведи вектор атаки на CI через sdist и объясни, почему `--only-binary :all:` не полное решение.

**46.** Namespace packages: PEP 420 (implicit) vs `pkgutil`-style vs `pkg_resources`-style. Что произойдёт, если два wheel'а установят один namespace разными стилями? Почему это до сих пор ломается?

**47.** Entry points читаются сканированием `.dist-info/entry_points.txt` всех дистрибутивов на `sys.path`. При 300 установленных пакетах старт CLI занимает 400 мс. Где именно уходит время и какие три способа это срезать ты знаешь?

**48.** ABI tag `cp311` vs `abi3`. Что даёт Limited API (PEP 384) при упаковке, какие ограничения на C-код и почему numpy/pydantic-core не собираются как abi3?

**49.** Requirements-файл с хешами (`--require-hashes`) + wheel, который pip пересобирает из sdist. Почему hash-pinning на sdist не даёт reproducible install, а на wheel — почти даёт? Что мешает «почти» стать «полностью»?

**50.** В Java есть shading, в Go — vendoring с версией в import path, в Rust — semver-совместимые дубликаты крейтов в одном бинаре. Python не может держать две версии одного пакета в процессе. Что конкретно в модели импорта это запрещает и что сломалось бы, если разрешить?

**51.** `Requires-Dist` и `Requires-Python` вшиваются в wheel при сборке, но `Requires-Python` warehouse ещё отдаёт в JSON API индекса. Зачем дублировать и что происходит, если они разойдутся?

**52.** PEP 621 стандартизировал `[project]`, но `[tool.poetry]` до 2.0 жил параллельно со своим синтаксисом версий (`^1.2`). Почему caret-синтаксис не попал в PEP 440 и какой аргумент был у обеих сторон?

**53.** У тебя пакет с C-расширением, нужно поддержать CPython 3.9–3.13 × {linux x86_64, aarch64, macOS arm64/x86_64, win amd64} = 25 wheel'ов. Опиши архитектуру CI, где вставишь abi3, и как решишь, что аудировать через auditwheel.

**54.** `pip install .` в проекте с `setuptools_scm` и без git-истории (shallow clone в CI) — что произойдёт с версией и почему это failure mode целого класса dynamic-version плагинов? Как проектировать, чтобы sdist оставался самодостаточным?

**55.** UV/pixi ставят пакеты жёсткими ссылками из глобального кэша вместо копирования. Какие предположения packaging-модели это ломает (namespace-пакеты, `.pth`, editable, права на запись в site-packages) и почему pip так не делает?

---

**Бонус-киллер:** `pip install -e .` ставит пакет, `pip freeze` показывает его как `-e git+https://...` или как `mypkg==1.0`, `importlib.metadata.version("mypkg")` возвращает версию на момент установки, а код в рабочем дереве уже другой версии. Пакет одновременно установлен и не установлен: `__file__` указывает в src, метаданные — в site-packages, а `importlib.metadata.files()` врёт про оба. Какая из этих «истин» правильная и что это говорит о том, является ли editable install вообще инсталляцией?

Хочешь разбор ответов? Скажи номера.
