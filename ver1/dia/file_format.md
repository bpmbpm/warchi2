# Формат файлов моделей `.warchi`

Каждая модель wArchi2-JS хранится в отдельном файле `*.warchi` в каталоге
`ver1/dia/`. Это требуется пунктом 3 задания https://github.com/bpmbpm/warchi2/issues/3 : «храни каждую модель в
отдельном файле, опиши формат файла».

## Назначение

* Один файл = одна архитектурная модель (узлы + связи + диаграммы).
* Файлы человекочитаемые (UTF-8 JSON) — их можно сравнивать в `git diff`.
* В браузерном клиенте файлы прогружаются на старте через
  `fetch()` (если работаем по `http(s)://`) либо встраиваются в начальные
  данные при работе с `file://` (см. `loadDB()` в `index.html`).

## Расширение и MIME

| Расширение | MIME                       | Кодировка |
|------------|----------------------------|-----------|
| `.warchi`  | `application/json+warchi`  | UTF-8     |

## Структура файла

```jsonc
{
  // Заголовок формата — обязателен для распознавания.
  "format": "warchi-model",
  "formatVersion": "1.0",

  // Идентификация модели (стабильна между сохранениями).
  "model": {
    "id": "model-test",
    "name": "Тестовая модель",
    "description": "Демонстрационная модель ArchiMate ...",
    "version": "1.3.0",
    "notationId": "n-archimate",
    "ownerId": "user1",
    "createdAt": "2026-01-15",
    "updatedAt": "2026-05-06"
  },

  // Узлы модели.
  "nodes": [
    {
      "id": "n1",
      "name": "Портал клиента",
      "nodeTypeId": "nt-app",
      "parentNodeId": null,
      "layer": "Приложение",
      "x": 80, "y": 80, "w": 140, "h": 60
    }
  ],

  // Связи (ориентированные, sourceId → targetId).
  "links": [
    { "id": "l1", "sourceId": "n1", "targetId": "n2", "linkTypeId": "lt-flow", "label": "HTTP" }
  ],

  // Диаграммы (визуальные представления модели).
  "diagrams": [
    {
      "id": "diag-main",
      "name": "Согласование с мейнфреймом",
      "notationId": "n-archimate",
      "description": "Основная диаграмма"
    }
  ]
}
```

## Обязательные поля

* `format` — строго `"warchi-model"`.
* `formatVersion` — строка SemVer.
* `model.id`, `model.name`, `model.notationId`, `model.version`.
* Каждый `node` обязан содержать `id`, `name`, `nodeTypeId`, `x`, `y`.
* Каждый `link` обязан содержать `id`, `sourceId`, `targetId`, `linkTypeId`.

## Правила целостности

1. Все `sourceId`/`targetId` ссылок должны указывать на существующие узлы
   внутри того же файла (`nodes[].id`).
2. `nodeTypeId` и `linkTypeId` ссылаются на типы из
   `defaultData().nodeTypes` / `defaultData().linkTypes` либо описаны в
   соответствующей нотации (`ver1/notation/<id>.json`).
3. Один файл — одна модель. Несколько моделей в одном файле не
   допускаются: это нарушает требование п. 3 задания.
4. `model.id` должен совпадать с именем файла без расширения.

## Список существующих файлов

| Файл                       | Модель                | Нотация       |
|----------------------------|-----------------------|---------------|
| `model-test.warchi`        | Тестовая модель       | ArchiMate 3.1 |
| `model-vi.warchi`          | ВсеИнструменты.ру     | C4 Model      |
| `model-warchi.warchi`      | Архитектура wArchi    | ArchiMate 3.1 |

## Загрузка

```js
fetch('dia/model-test.warchi')
  .then(r => r.json())
  .then(file => importWarchiFile(file));
```

В браузерной версии вызов `fetch()` от `file://` блокируется политикой
CORS — поэтому при открытии `index.html` напрямую модели подгружаются
из встроенного `defaultData()`. Через GitHub Pages
(<https://bpmbpm.github.io/warchi2/ver1/>) `fetch()` работает корректно.
