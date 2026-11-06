# VulnerabilityApi

REST API на ASP.NET Core (.NET 8) для управления уязвимостями и запуска фоновых сканирований репозиториев.

## Что сделано

- Добавление уязвимости (название + критичность), дата добавления фиксируется автоматически, возвращается ID.
- Получение уязвимости по ID.
- Запуск асинхронного сканирования репозитория по URL, возвращается ID задачи.
- Статус задачи: `Queued`, `Running`, `Completed`, `Failed`, `Cancelled`, время последней смены статуса, результат для завершённых.
- Ограничение числа параллельных сканирований, лишние задачи ждут в очереди со статусом `Queued`.
- Отмена сканирования в очереди и уже запущенного (запущенное прерывается через `CancellationToken`).
- Логирование всех HTTP-запросов (метод, путь, код ответа, длительность) в консоль.
- Обработка ошибок: 400 при невалидном запросе, 404 при неизвестном ID, 500 с `ProblemDetails` при необработанном исключении.
- Состояние хранится только в памяти, после перезапуска данных нет.
- Сканирование эмулируется: задержка `SimulatedScanDurationSeconds`, затем случайный набор найденных уязвимостей.

## Запуск

Требуется .NET 8 SDK.

```bash
dotnet run --project src/VulnerabilityApi
```

Сервис слушает `http://localhost:5000`.

## Тесты

```bash
dotnet test
```

## Настройки

`src/VulnerabilityApi/appsettings.json`, секция `Scanning`:

| Параметр | По умолчанию | Описание |
|---|---|---|
| `MaxParallelScans` | 2 | Максимум одновременно выполняемых сканирований (>= 1) |
| `SimulatedScanDurationSeconds` | 10 | Длительность эмулируемого сканирования |

Переопределение при запуске:

```bash
dotnet run --project src/VulnerabilityApi -- --Scanning:MaxParallelScans=1
```

## Эндпоинты

| Метод | Путь | Тело | Ответ |
|---|---|---|---|
| POST | `/api/vulnerabilities` | `{"name":"XSS","severity":"High"}` | 201 `{"id"}` |
| GET | `/api/vulnerabilities/{id}` | | 200 `{"id","name","severity","createdAt"}` |
| POST | `/api/scans` | `{"repositoryUrl":"https://gitlab.com/group/project.git"}` | 202 `{"id"}` |
| GET | `/api/scans/{id}` | | 200 `{"id","status","statusChangedAt","result"}` |
| POST | `/api/scans/{id}/cancel` | | 200 `{"id","success","time"}` |

`severity`: `Low`, `Medium`, `High`, `Critical`.

## Структура

- `src/VulnerabilityApi/Controllers` — контроллеры
- `src/VulnerabilityApi/Services` — хранилище уязвимостей, менеджер сканирований, эмулятор сканера
- `src/VulnerabilityApi/Middleware` — логирование запросов и обработка ошибок
- `src/VulnerabilityApi/Models`, `Validation` — модели и валидация
- `tests/VulnerabilityApi.Tests` — unit-тесты (xUnit)
