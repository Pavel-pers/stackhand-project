# server-agent/05 — парсинг выхлопа шаблона и вывод оркестратору

## Сценарий

Агент srv-1c выполнил disk-usage и получил сырой выхлоп df и du. Проверяется разбор выхлопа в структуру для оркестратора: тома с размерами и процентами, топ папок; числа совпадают с эталоном, короткая сводка выделяет самый заполненный том.

## Контекст агента srv-1c (backend)

Ubuntu 22.04.4, 8 CPU, 32 ГБ. Диски: / (sda2, 98 ГБ), /var/lib/postgresql (sdb1, 492 ГБ), /backup (sdc1, 980 ГБ).
Службы: srv1cv8-8.3.24.1548 (порты 1540, 1541, 1560–1591), postgresql@16-main (5432), apache2 — публикация 1С для обмена (8080), ssh (22).
Логи: /var/log/auth.log, /var/log/syslog, /var/log/postgresql/postgresql-16-main.log, технологический журнал 1С в /var/log/1c.
Планировщик: pg_dump ночью, `0 2 * * * /usr/local/bin/backup_1c.sh` → /backup/pg. Доступ: evalbot по SSH, ограниченный sudo (df, du, journalctl, systemctl status, cat auth.log).
Чего на хосте нет: wireguard, nginx, docker.

## Каталог шаблонов (выдержка, Linux)

inventory — инвентаризация и паспорт; resources-now — ресурсы сейчас; disk-usage — диски и прогноз заполнения
(df по томам, du топ папок, рост по сборщику, дата заполнения); services-status — статус служб; log-errors — ошибки
журналов (journalctl -p err, grep по пути); auth-events — входы и неудачные входы (auth.log, lastb); accounts-access —
учётки с доступом; backup-freshness — свежесть бэкапов; certs — сертификаты; scheduler — планировщик (задания cron и
systemd-таймеры, последние запуски, код результата, журнал задания). Все — только чтение; шаблон выбирает агент,
параметры проверяются по схеме до запуска.

## Вход

Задача от оркестратора агенту srv-1c: «Проверь заполненность томов на srv-1c». Исходный запрос пользователя: «посмотри заполненность дисков на сервере в 1C».
Выхлоп шаблона:

```
$ df -h --output=source,size,used,avail,pcent,target
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        98G   41G   53G  43% /
/dev/sdb1       492G  377G   90G  81% /var/lib/postgresql
/dev/sdc1       980G  210G  721G  23% /backup

$ du -xsh /var/lib/postgresql/* 2>/dev/null | sort -h | tail -n 5
4.1G    /var/lib/postgresql/16/main/pg_wal
371G    /var/lib/postgresql/16/main/base
```

## Ожидаемое поведение

- Тип ответа `executed`, шаблон disk-usage.
- Эталонный разбор: / — 98G, 41G занято, 53G свободно, 43 %; /var/lib/postgresql — 492G, 377G, 90G, 81 %; /backup — 980G, 210G, 721G, 23 %; топ папок на томе PostgreSQL: /var/lib/postgresql/16/main/base 371G, /var/lib/postgresql/16/main/pg_wal 4.1G.
- В сводке оркестратору выделен том PostgreSQL (81 %, 90G свободно); прогноз без сборщика не выдуман.
- Ошибки: перепутанные столбцы (занято/свободно); пропущенный том; выдуманная дата заполнения.

## Метрики

- Rule-based: `output-parse-correct`.
- LLM-judge: `judge-B-agent-to-user` — сводка короткая, акцент верный, ничего не додумано.

## Данные

Контекст агента (как в блоке выше), выдержка каталога шаблонов, задача от оркестратора, выхлоп шаблона, рубрика judge.
