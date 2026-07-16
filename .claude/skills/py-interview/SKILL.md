---
name: py-interview
description: Generate a Python interview question set on any topic — basics → mid-level → coding tasks → 20 senior-level killer questions. Use when the user asks for interview questions, prep, or a deep-dive quiz on a Python topic (strings, asyncio, GIL, descriptors, memory, typing, imports, metaclasses, etc.).
---

# Python Interview Generator

Turn a Python topic into a graded interview question set. Topic comes from args (e.g. `/py-interview asyncio`). No topic in args → ask for one, nothing else.

## Output structure (strict)

```
# Собес Python — <ТЕМА>

## БАЗА
<10-14 вопросов>

## СРЕДНИЙ УРОВЕНЬ
<12-17 вопросов>

## ЗАДАЧИ (напиши код)
<5-7 задач>

---

# 20 ВОПРОСОВ НА СЕНЬОРНОСТЬ
<20 вопросов, нумерация сквозная, каждый bold-номером>

---

**Бонус-киллер:** <один вопрос-парадокс>

Хочешь разбор ответов? Скажи номера.
```

Нумерация сквозная через все секции.

## Rules per section

**БАЗА** — что знает мидл. Определения, разница между X и Y, сложность операций, частые ошибки.

**СРЕДНИЙ** — CPython impl detail, PEP-ы, edge cases, stdlib за пределами очевидного, perf-компромиссы.

**ЗАДАЧИ** — код руками. Алгоритмы + edge cases. Спрашивай сложность и второй способ.

**СЕНЬОРНОСТЬ** — вопрос senior-уровня удовлетворяет хотя бы одному:
- CPython internals (структуры C, аллокатор, байткод, оптимизации ядра)
- Конкретный номер PEP + что он поменял и какую цену заплатили
- Паттерн, который на первый взгляд работает, но ломается при условии X
- Числа/бенчмарк в теле вопроса → объясни разницу
- Security-угол (timing, injection, randomization, DoS)
- Архитектурный сценарий с реальным constraint (500M строк, RAM 16GB — твоя архитектура?)
- Кросс-языковое сравнение (в Java/Go/Rust — так, в Python — иначе, почему)
- История: почему убрали / зачем добавили / что было в py2
- Trade-off без правильного ответа → проверяет вкус

## Quality bar

- Конкретика вместо общего. FORBIDDEN: «расскажи про GIL». REQUIRED: «GIL отпускается каждые 5 мс (`sys.setswitchinterval`). Почему не по байткодам, как было до 3.2? Что сломал convoy effect?»
- Каждый senior-вопрос содержит зацепку: имя функции C, номер PEP, число, флаг, имя алгоритма.
- Вопрос ≤ 3 предложения. Многосоставные вопросы через `?` подряд — ОК.
- Нет вопросов, на которые гуглится ответ первой ссылкой.
- Нет ответов в тексте вопроса. Подсказка в скобках — только если вопрос иначе нерешаем.
- Не пиши ответы. Пользователь просит номера → разбирай.

## Topic → senior angle cheatsheet

Не список тем — список углов атаки. Для любой темы прогони по ним:

| Угол | Что искать |
|---|---|
| Память | размер объекта, sizeof, аллокатор, freelist, фрагментация |
| Байткод | `dis`, что оптимизирует компилятор, peephole, constant folding |
| Конкурентность | thread safety, GIL, atomic vs non-atomic, race |
| Числа | бенчмарк, где O() врёт, где константа решает |
| Ломается | что перестаёт работать при подклассе / втором ref / большом N |
| Security | timing, DoS, randomization, injection, десериализация |
| История | что было в py2 / до версии X, почему поменяли |
| Масштаб | 500M элементов, 16GB RAM — архитектура? |
| Соседи | Java/Go/Rust делают иначе — почему |

## Language

Вопросы на русском. Технические термины, имена функций, код — как есть, без перевода.
