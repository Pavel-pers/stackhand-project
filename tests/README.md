# eval-test — golden-набор

Структура: `init/` (инициализация), `node-isolate/` (узлы по отдельности: `orchestrator/`, `server-agent/`), `e2e/`.
Каждый тест — листовая директория с `readme.md`: сценарий по шагам, вход, контекст узла, ожидаемое поведение, метрики
с рубрикой judge и данные, которые нужны для реализации.

## Типы ответа (enum в response модели — по ним считаются rule-based доли)

- оркестратор: `answered` | `not-in-scope` (отказ: нет данных или вне возможностей) | `clarify` (уточняющий вопрос) | `delegated`
- серверный агент: `executed` | `no-template` | `wrong-addressee` | `template-error`

## Метрики

LLM-judge по рубрике теста: `judge-A-context-answer` (ответы по контексту оркестратора), `judge-B-agent-to-user`
(перенос ответа агента пользователю), `judge-C-document` (документы инициализации), `judge-D-task-formulation`
(постановка задачи агенту или инспектору).

Rule-based: `tag-accuracy` (обязательные теги есть, запрещённых нет), `orchestrator-refusal-correct`,
`agent-refusal-correct`, `false-refusal`, `agent-choice-correct` (кому делегировано), `template-choice-correct`,
`output-parse-correct` (числа совпадают с эталонным разбором), `journal-template-match` (e2e: шаблон из журнала).

## Стенды

- A — один Windows Server «всё в одном»: AD, файлы, 1С (файловая база и MSSQL Express), RDS, бэкап на NAS (init).
- B — гипервизор Proxmox и две ВМ: 1С с MS SQL и терминальный сервер (init).
- C — сайт на nginx, 1С с PostgreSQL, микросервисы бухгалтерии в Docker (init).
- C + VPN-шлюз gw-01 с WireGuard — стенд node-isolate и e2e; все хосты Ubuntu 22.04. e2e требует живой стенд,
  выбранный шаблон читается из журнала; в init инспекция на живом стенде — реальная, в описании теста — её эталон.

## Прогон

Одна модификация (шаблоны, промпты, модель) — один прогон, ~5 повторов при temperature > 0.
Rule-based — доля успешных по классу, judge — средний балл. Регрессия по любому классу — откат модификации.
