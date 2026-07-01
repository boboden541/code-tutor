# design_spec.md (Файл №4) — эталонная структура

> Пример показывает обязательные разделы и формат. Реальный файл покрывает ВСЕ экраны проекта. Экраны сокращены до 3 для иллюстрации; в реальном файле — каждый экран матрицы по `screen_spec_schema.md`.

---

# Дизайн-спецификация: <Домен>

## 1. Суть и эстетика
- **Предмет / аудитория / задача:** <напр.> маркетплейс товаров; покупатели и продавцы-магазины; задача — выбрать и заказать товар с прозрачным статусом доставки.
- **Визуальный язык (выведен из сути):** тёплая праздничная палитра, **сильная шрифтовая пара**, продающие блоки (рейтинги/отзывы/бейджи), акцент доверия вокруг прозрачности заказа. (Источник дисциплины: навык `frontend-design` + `resources/art_direction.md`.)
- **Способ стилизации:** дизайн-токены (CSS-переменные) + CSS Modules, без новых зависимостей.
- **Прототипы:** три направления в Claude Design проекте «<Domain> UI» | либо Artifact (3 ссылки/переключатель) | либо `PROTOTYPE: spec-only (<причина>)`.

## 1a. Визуальные направления (3) и выбранное
> Три **системно разных** направления (≥3 осей различия — типографика/композиция/эмоция, не только цвет), по `art_direction.md`. Пользователь выбрал одно — его дизайн-система ниже.

| # | Ярлык | Позиционирование (на кого / эмоция) | Шрифтовая пара | Композиция | Файл-прототип |
|---|-------|--------------------------------------|----------------|------------|----------------|
| 1 | Тёплый ремесленный | для ценящих ручную работу; уют и доверие | Fraunces + Plus Jakarta Sans | асимметричный editorial | `<domain>_dir1_warm.html` |
| 2 | Современный продуктовый | занятым; быстро и уверенно | Space Grotesk + Inter | bento-сетка | `<domain>_dir2_modern.html` |
| 3 | Смелый журнальный | ценящим вкус; «вау» | Archivo Expanded + Manrope | split-screen + overlap | `<domain>_dir3_editorial.html` |

**Выбрано: №1 «Тёплый ремесленный»** (решение пользователя). Дизайн-система ниже описывает именно его.

## 2. Дизайн-система
### Токены
```
--color-bg:#fff7f2; --color-surface:#fff; --color-text:#2b2024; --color-muted:#8a7a72;
--color-accent:#ff6b6b; --color-accent-ink:#fff; --color-border:#f0e0d8;
--color-success:#2e9e6b; --color-warning:#d9a441; --color-danger:#d64550;
/* шрифтовая пара (Google Fonts): <link href="…Fraunces…Plus+Jakarta+Sans…"> */
--font-display:"Fraunces",Georgia,serif; --font-text:"Plus Jakarta Sans",system-ui,sans-serif;
--fs-1:2.6rem; --fs-2:1.85rem; --fs-3:1.3rem; --fs-body:1rem; --fs-sm:.875rem;  /* модульная шкала ×1.3 */
--space-1:4px;--space-2:8px;--space-3:12px;--space-4:16px;--space-6:24px;--space-8:32px;
--radius:14px;--radius-sm:8px;--shadow:0 8px 24px -10px rgba(255,107,107,.25);--shadow-lg:0 18px 50px -16px rgba(43,32,36,.28);
```
### Компоненты (с состояниями)
Button(primary/secondary/danger; default/**hover**/active/**focus-visible**/disabled/pending; `cursor:pointer`), Input, Select, Textarea, FormField(label+error+focus/invalid), Card, StatusBadge(по статусу), Price, **RatingStars**, **ReviewCard**, **TrustBadge**, Spinner, EmptyState, ErrorState, Layout, Nav(роле-зависимая), ProtectedRoute.
### Тон
Тёплые человеческие тексты; ошибки: 401→«Войдите снова», 403→«Недостаточно прав», 409→«Действие недоступно в текущем статусе».
### Статусы → бейдж
created→muted, paid→accent, accepted→warning, shipped→accent, delivered→success, cancelled→danger.

## 3. Матрица покрытия «экран × эндпоинт»
| Экран | Чтения | Действия | Эндпоинты покрыты |
|-------|--------|----------|-------------------|
| catalog | GET /products | — | /products |
| buyer_order_detail | GET /orders/{id} | pay, cancel, confirm | …/pay …/cancel …/confirm |
| seller_order_detail | GET /orders/{id} | accept, ship | …/accept …/ship |
| … | … | … | … |
> Проверка: каждая операция контракта стоит ≥1 раз (нет `endpoint-unconsumed`).

## 4. Экраны (каждый по schema)

### screen: catalog
- name: Каталог; purpose: выбрать товар.
- route: /; role_gating: guest/buyer
- api_calls: GET /products (фильтр city, category)
- input_forms: фильтр — city (text, optional), category (select — категории каталога, optional)
- data_displayed: список ProductRead → ProductCard (title, Price(price ₽), category, photo_url); shop title
- states: loading→грид Spinner; empty→EmptyState «Ничего не найдено»; error→ErrorState+Повторить
- navigation: вход — корень; исход — клик карточки → /products/:id; Nav → вход/регистрация
- components_used: Layout, Nav, Card, Price, Select, Input, EmptyState, ErrorState, Spinner

### screen: buyer_order_detail
- (см. пример в `resources/screen_spec_schema.md` — полностью заполненный)

### screen: seller_order_detail
- name: Заказ (продавец); purpose: принять заказ и отметить отправленным.
- route: /seller/orders/:id; role_gating: seller (owner: shop)
- api_calls: GET /orders/{id}; POST …/accept (paid); POST …/ship {tracking_number} (accepted)
- input_forms: отправка — tracking_number (text, required, minLength 6) — трек-номер посылки
- data_displayed: product, buyer phone, delivery_date, status(StatusBadge), total(Price)
- states: loading/error как обычно; mutation pending→кнопка disabled+Spinner; 409→«Действие недоступно в текущем статусе»
- navigation: вход из incoming_orders; «Назад» → /seller/orders; после ship — статус shipped
- components_used: Layout, Card, StatusBadge, FormField, Input, Button, Spinner, ErrorState

## 5. Примечания
- Версии/идиомы стилизации — из `<frontend_pack>/styling.md`.
- Имена `screen_id` → имена шагов книги (snake_case латиницей).
