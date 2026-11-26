# TimeCapsule Events

Тема SNS: `timecapsule-dev-events` (Standard). Подписка SQS `timecapsule-dev-mail` использует Raw message delivery.

## capsule.sealed

После сохранения капсулы PHP публикует событие:

```json
{"version":1,"type":"capsule.sealed","event_id":"capsule.sealed:42","capsule_id":42}
```

## capsule.opened

Opener выполняет атомарный UPDATE ... RETURNING и возвращает список открытых капсул. Lambda Destination (On success) доставляет invocation record в SNS. Значимая часть записи:

```json
{"responsePayload":{"type":"capsule.opened","opened_count":1,"events":[{"version":1,"type":"capsule.opened","event_id":"capsule.opened:42","capsule_id":42}]}}
```

| Поле | Тип | Назначение |
|---|---|---|
| version | integer | Версия контракта |
| type | string | Тип события |
| event_id | string | Детерминированный ключ идемпотентности |
| capsule_id | integer | Ссылка на запись PostgreSQL |
| opened_count | integer | Количество открытых капсул |
| events | array | Список событий открытия |

Mailer загружает получателя и данные капсулы из RDS, имитирует отправку письма и записывает уведомление. UNIQUE INDEX по event_id и ON CONFLICT DO NOTHING исключают повторную вставку. Пустой массив не создаёт уведомлений; подписка SQS не фильтруется. Email-фильтр по MessageBody:

```json
{"responsePayload":{"type":["capsule.opened"],"opened_count":[{"numeric":[">",0]}]}}
```

В событии отсутствуют текст капсулы, вложения, адрес получателя и секреты. При сбое между коммитом БД и публикацией событие может потеряться; это ограничение текущей реализации.
