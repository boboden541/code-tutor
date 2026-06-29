# design_spec.md (Файл №4) — эталонная структура

> Пример показывает обязательные разделы и формат. Реальный файл покрывает ВСЕ экраны проекта. Экраны сокращены до 3 для иллюстрации; в реальном файле — каждый экран матрицы по `screen_spec_schema.md`.

---

# Дизайн-спецификация: <Домен>

## 1. Суть и эстетика
- **Предмет / аудитория / задача:** <напр.> маркетплейс подарков; покупатели и продавцы-флористы/кондитеры; задача — выбрать и заказать подарок с гарантией «фото до доставки».
- **Визуальный язык (выведен из сути):** тёплая праздничная палитра, дружелюбная типографика, акцент доверия вокруг фотоконтроля. (Источник дисциплины: навык `frontend-design`.)
- **Способ стилизации:** дизайн-токены (CSS-переменные) + CSS Modules, без новых зависимостей.
- **Прототип:** Claude Design проект «<Domain> UI» (утверждён) | либо Artifact <ссылка> | либо `PROTOTYPE: spec-only (<причина>)`.

## 2. Дизайн-система
### Токены
```
--color-bg:#fff7f2; --color-surface:#fff; --color-text:#2b2024; --color-muted:#8a7a72;
--color-accent:#ff6b6b; --color-accent-ink:#fff; --color-border:#f0e0d8;
--color-success:#2e9e6b; --color-warning:#d9a441; --color-danger:#d64550;
--font-sans:system-ui,sans-serif; --font-display:Georgia,serif;
--fs-1:2rem; --fs-2:1.5rem; --fs-3:1.125rem; --fs-body:1rem; --fs-sm:.875rem;
--space-1:4px;--space-2:8px;--space-3:12px;--space-4:16px;--space-6:24px;--space-8:32px;
--radius:14px;--radius-sm:8px;--shadow:0 2px 8px #0001;--shadow-lg:0 8px 24px #0002;
```
### Компоненты
Button(primary/secondary/danger), Input, Select, Textarea, FormField(label+error), Card, StatusBadge(по статусу), Price, Spinner, EmptyState, ErrorState, Layout, Nav(роле-зависимая), ProtectedRoute.
### Тон
Тёплые человеческие тексты; ошибки: 401→«Войдите снова», 403→«Недостаточно прав», 409→«Действие недоступно в текущем статусе».
### Статусы → бейдж
created→muted, paid→accent, accepted→warning, photo_review→accent, assembled→success, delivered→success, cancelled→danger.

## 3. Матрица покрытия «экран × эндпоинт»
| Экран | Чтения | Действия | Эндпоинты покрыты |
|-------|--------|----------|-------------------|
| catalog | GET /products | — | /products |
| buyer_order_detail | GET /orders/{id} | pay, cancel, approve-photo, reject-photo | …/pay …/cancel …/approve-photo …/reject-photo |
| seller_order_detail | GET /orders/{id} | accept, photo, deliver | …/accept …/photo …/deliver |
| … | … | … | … |
> Проверка: каждая операция контракта стоит ≥1 раз (нет `endpoint-unconsumed`).

## 4. Экраны (каждый по schema)

### screen: catalog
- name: Каталог; purpose: выбрать подарок.
- route: /; role_gating: guest/buyer
- api_calls: GET /products (фильтр city, category)
- input_forms: фильтр — city (text, optional), category (select: flowers|cake|craft, optional)
- data_displayed: список ProductRead → ProductCard (title, Price(price ₽), category, photo_url); shop title
- states: loading→грид Spinner; empty→EmptyState «Ничего не найдено»; error→ErrorState+Повторить
- navigation: вход — корень; исход — клик карточки → /products/:id; Nav → вход/регистрация
- components_used: Layout, Nav, Card, Price, Select, Input, EmptyState, ErrorState, Spinner

### screen: buyer_order_detail
- (см. пример в `resources/screen_spec_schema.md` — полностью заполненный)

### screen: seller_order_detail
- name: Заказ (продавец); purpose: принять, загрузить фото, отметить доставленным.
- route: /seller/orders/:id; role_gating: seller (owner: shop)
- api_calls: GET /orders/{id}; POST …/accept (paid); POST …/photo {ready_photo_url} (accepted); POST …/deliver (assembled)
- input_forms: фото — ready_photo_url (text/URL, required) — это URL, не файл
- data_displayed: product, buyer phone, delivery_date, status(StatusBadge), total(Price)
- states: loading/error как обычно; mutation pending→кнопка disabled+Spinner; 409→«Действие недоступно в текущем статусе»
- navigation: вход из incoming_orders; «Назад» → /seller/orders; после deliver — статус delivered
- components_used: Layout, Card, StatusBadge, FormField, Input, Button, Spinner, ErrorState

## 5. Примечания
- Версии/идиомы стилизации — из `<frontend_pack>/styling.md`.
- Имена `screen_id` → имена шагов книги (snake_case латиницей).
