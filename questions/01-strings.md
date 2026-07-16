---
topic: strings
block: "I. Типы данных и модель объекта"
number: 1
status: todo
---

# Собес Python — strings

## БАЗА

1. Строка в Python — mutable или immutable? Что даёт immutability?
2. `str` vs `bytes` vs `bytearray`. Когда что?
3. Что такое Unicode code point? Чем отличается от grapheme?
4. `encode()` / `decode()` — что делают, какие ошибки бросают?
5. `'abc' * 3`, `'a' + 'b'` — сложность?
6. Разница `''.join(list)` vs конкатенация в цикле. Почему?
7. f-string vs `%` vs `.format()` vs Template. Что быстрее, что безопаснее?
8. `str.split()` без аргумента vs `split(' ')` — разница?
9. `strip('abc')` — что реально делает? Частая ошибка?
10. `find()` vs `index()`?
11. `is` vs `==` для строк. Почему `'a' is 'a'` → True, а иногда нет?
12. Raw strings — зачем? `r'\n'` = что?
13. Слайсы: `s[::-1]`, `s[::2]`, `s[-1]`. Сложность?
14. `in` для строк — какой алгоритм под капотом?

## СРЕДНИЙ УРОВЕНЬ

15. String interning — что интернируется автоматически? `sys.intern()` — зачем?
16. Compile-time constant folding: `'a' + 'b'` в коде → сколько объектов?
17. PEP 393 — Flexible String Representation. Что за latin-1/UCS-2/UCS-4?
18. `len('😀')` = ? Почему? А в Python 2?
19. `sys.getsizeof('')` vs `sys.getsizeof('a')` vs `sys.getsizeof('😀')`?
20. Нормализация Unicode: NFC/NFD/NFKC/NFKD. Когда критично?
21. `'ß'.upper()` = ? `len` изменился?
22. Casefold vs lower. Зачем casefold?
23. `str.translate()` + `maketrans()` — когда быстрее replace?
24. Regex: жадность, lookahead/lookbehind, `re.compile` кэш.
25. Catastrophic backtracking — что, пример, как избежать?
26. `re.match` vs `re.search` vs `re.fullmatch`.
27. `%`-formatting и format spec mini-language: `f'{x:>10.2f}'`, `f'{x!r}'`, `f'{x=}'`.
28. `__str__` vs `__repr__` vs `__format__`.
29. `bytes.decode('utf-8', errors=...)` — все стратегии errors?
30. `surrogateescape` — зачем существует?
31. Почему нельзя использовать `==` для сравнения паролей/токенов? `hmac.compare_digest`.

## ЗАДАЧИ (напиши код)

32. Реверс строки с учётом grapheme clusters (emoji с модификаторами).
33. Проверить анаграмму. Два способа, сложность.
34. Найти самый длинный палиндром-подстроку. Manacher?
35. Сжатие строки RLE. In-place на bytearray.
36. Валидный парсер вложенных скобок.
37. `str.count('aa')` в `'aaaa'` = ? Почему не 3?
38. Написать `split` вручную. Учесть все edge cases.

---

# 20 ВОПРОСОВ НА СЕНЬОРНОСТЬ

**39.** `'a' * 10**9` — что произойдёт? Как посчитать память заранее? Почему CPython не аллоцирует лениво?

**40.** Почему `s += x` в цикле иногда работает за O(n), а не O(n²)? Что за оптимизация `unicode_concatenate` в CPython, и почему она ломается при наличии второй ссылки на строку?

**41.** `sys.getsizeof('a')` = 50, `sys.getsizeof('ab')` = 51, `sys.getsizeof('日')` = 76. Объясни разницу в overhead. Что такое compact ascii vs compact unicode vs legacy string?

**42.** Строка кэширует свой hash. Где хранится? Что происходит при hash randomization (PYTHONHASHSEED)? Почему это security-фича, а не perf?

**43.** `'\N{GREEK SMALL LETTER ALPHA}'` — когда резолвится: runtime или compile-time? Как это влияет на размер .pyc?

**44.** Дано: файл 50GB, надо найти все строки с паттерном. Почему `open().read()` — плохо, `readlines()` — плохо, а итерация по файлу — норм? Что делает io.TextIOWrapper с буфером и декодером при `seek()`?

**45.** Turkish I problem: `'I'.lower()` в турецкой локали. Почему Python игнорирует локаль в `str.lower()`, а `str.casefold()` тоже? Где это ломает реальные системы?

**46.** Unicode confusables / homograph attack. Как задетектить в Python? Что даёт `unicodedata`, а чего не хватает?

**47.** `'́'` (combining acute). `'é' == 'é'` может быть False. Как ты это защитишь на входе в систему? На каком слое — API gateway, ORM, DB collation?

**48.** Почему `str.replace()` возвращает новую строку, а не мутирует? Обоснуй с точки зрения дизайна: hash caching, dict keys, thread safety, sharing.

**49.** `bytes` vs `memoryview` для парсинга протокола. Когда slicing bytes убивает perf, и как memoryview спасает? Zero-copy — где ломается?

**50.** UTF-8 vs UTF-16 vs UTF-32 в контексте индексации. Почему PEP 393 выбрал не UTF-8 внутри? Какую цену платим?

**51.** `re` vs `regex` библиотека. Что даёт `regex`: atomic groups, possessive quantifiers, `\X`, fuzzy matching. Когда стоит переезда?

**52.** Реализуй строковый поиск: KMP, Boyer-Moore, Rabin-Karp. Что использует CPython в `str.find()`? (Crochemore-Perrin / two-way + Bloom filter). Почему именно это?

**53.** Утечка памяти через слайс строки — существует ли в Python? (В Java до 7 — да, через shared char[]). Почему CPython не шарит буферы?

**54.** ROT13, base64, hex — почему `'abc'.encode('rot13')` не работает в py3? Что за `codecs.encode` и почему str↔str кодеки убрали из `str.encode`?

**55.** `logging` + f-strings: почему `logger.info(f'{expensive()}')` — антипаттерн, а `logger.info('%s', expensive())` — нет? Что с lazy evaluation?

**56.** Timing attack на сравнение строк — покажи как реально утечёт длина секрета даже при `compare_digest`. Как защититься полностью?

**57.** Дана задача: нормализовать 500M строк-адресов, дедуплицировать. RAM 16GB. Твоя архитектура? (interning? trie? bloom filter? external sort? sqlite?)

**58.** `str` подкласс: почему `class MyStr(str)` ломается при `s[:]`, `s.upper()`, `s + x`? Как правильно наследоваться? Почему в stdlib предпочитают композицию?

**59.** Как работает `str.__contains__` для substring длины 1 vs длины 1000? Оптимизации в CPython для коротких паттернов (memchr).

**60.** `text.split('\n')` vs `text.splitlines()` — назови 5 различий. Какие символы splitlines считает переводом строки, а split — нет? (`\v`, `\f`, `\x1c`, ` `, ``)

---

**Бонус-киллер:** Объясни, почему `'-'.join(['a'] * 10**7)` быстрее, чем цикл `+=`, но `''.join(gen)` медленнее, чем `''.join(list)`. Что делает `join` с генератором?

Хочешь разбор ответов? Скажи номера.
