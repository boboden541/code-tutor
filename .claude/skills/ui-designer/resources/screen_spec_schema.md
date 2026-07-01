# Схема спецификации экрана

> Контракт между `ui-designer`, `book-architect` (план) и `book-author` (реализация). Каждый экран в `design_spec.md` описывается этими полями целиком. Имена полей — стабильные; по ним работают проверки валидатора (`missing-screen`, `unwired-nav`, `unstyled-screen`, `endpoint-unconsumed`).

## Поля экрана

| Поле | Что содержит | Обязательно |
|------|--------------|-------------|
| `screen_id` | snake_case-идентификатор (станет именем шага книги, напр. `buyer_order_detail`) | да |
| `name` | человекочитаемое название экрана | да |
| `purpose` | одна строка: зачем экран нужен пользователю | да |
| `route` | путь + параметры (напр. `/orders/:id`) | да (для роутируемых) |
| `role_gating` | кому доступен: `guest` / `buyer` / `seller` / `owner` (владелец ресурса) | да |
| `api_calls` | список операций контракта (operationId или метод+путь), которые экран вызывает — чтение и мутации | да |
| `input_forms` | поля форм: имя, тип, правила валидации (зеркало ограничений контракта: `required`, `minLength`, `gt`, enum, формат) | если есть ввод |
| `data_displayed` | какие поля DTO парсятся и показываются + форматирование (напр. `total` → валюта `8 250 ₽`; `delivery_date` → дата) | да (если есть данные) |
| `states` | рендеринг для `loading`, `empty`, `error`; для мутаций — `pending`/`success`/`failure` | да |
| `navigation` | входящие переходы (откуда), исходящие (куда ведут ссылки/кнопки), редиректы после действий | да |
| `components_used` | компоненты дизайн-системы, из которых собран экран | да |

## Пример заполнения (один экран)

```markdown
### screen: buyer_order_detail
- name: Заказ (покупатель)
- purpose: Покупатель видит статус заказа и подтверждает получение.
- route: /orders/:id
- role_gating: buyer (owner: buyer_id == текущий)
- api_calls:
  - GET /orders/{id}            (чтение деталей)
  - POST /orders/{id}/pay        (если status=created)
  - POST /orders/{id}/cancel     (если status in [created, paid])
  - POST /orders/{id}/confirm    (если status=shipped)
- input_forms: нет (только действия-кнопки)
- data_displayed:
  - item title, quantity, delivery_date (дата), status (StatusBadge), total (валюта ₽)
- states:
  - loading: Spinner на месте карточки
  - error: ErrorState «Не удалось загрузить заказ» + кнопка «Повторить»
  - empty: n/a (одиночный ресурс; 404 → ErrorState «Заказ не найден»)
  - mutation pending: кнопка disabled + Spinner внутри
  - mutation success: invalidate ["orders", id], статус обновляется
  - mutation failure: текст ошибки над кнопками (409 → «Действие недоступно в текущем статусе»)
- navigation:
  - входящие: из my_orders (клик по карточке)
  - исходящие: «Назад к заказам» → /orders
  - после confirm: статус → delivered (остаёмся на экране); после cancel → /orders
- components_used: Layout, Card, StatusBadge, Button, Spinner, ErrorState
```

## Правила

- Поля валидации форм обязаны соответствовать ограничениям контракта (не строже и не слабее) — иначе фронт разойдётся с бэком.
- Каждая мутация в `api_calls` обязана быть реально вызвана действием экрана (кнопкой/сабмитом) — заглушки запрещены.
- Каждый исходящий переход в `navigation` обязан быть реализован ссылкой/редиректом (иначе `unwired-nav`).
- `data_displayed` перечисляет конкретные поля DTO из контракта (парсинг), а не «информация о заказе» абстрактно.
