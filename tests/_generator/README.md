# Генератор набора

Весь набор — корневой `README.md`, `SCENARIOS.md`, `scenarios.json`, `_registry.jsonl`, `init/README.md`, `main/README.md` и все `readme.md` тестов — собирается этими скриптами. Тест правится здесь, затем набор пересобирается; правки, внесённые прямо в `readme.md`, при пересборке теряются.

Сборка (Python 3.9+, без внешних зависимостей) — в пустую папку, потому что реестр `_registry.jsonl` дописывается:

```
export EVAL_OUT=/путь/к/пустой/папке
for m in init_nodes init_e2e main_orch main_local main_e2e system readmes; do python3 $m.py; done
```

`readmes.py` запускается последним: он читает реестр, который пишут остальные, и считает состав набора.

| Файл | Что в нём |
|---|---|
| `common.py` | общие блоки: коды сценариев, стенды, записи о серверах, документ контекста, каталог ручек и ошибки-enum, ручки и чек-лист инспектора, форма, функция `test()` и реестр |
| `main_fix.py` | фикстуры рабочего запроса: сводки и ошибки ручек, ответы агентов |
| `init_nodes.py`, `init_e2e.py` | тесты добавления сервера: инспектор и куратор (node-isolate), e2e |
| `main_orch.py`, `main_local.py`, `main_e2e.py` | тесты рабочего запроса: оркестратор и локальный агент (node-isolate), e2e |
| `system.py` | системные тесты вне eval-сета |
| `readmes.py` | корневой README, README групп, `SCENARIOS.md`, `scenarios.json` |
