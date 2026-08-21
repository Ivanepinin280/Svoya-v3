# Своя кухня — Telegram Mini App

Доставка еды по Нячангу. Фронтенд — один HTML-файл (Tailwind нет, чистый CSS + vanilla JS + Telegram WebApp SDK). Бэкенд — Google Apps Script поверх Google Sheets.

## 1. Google Таблица

Создайте новую таблицу и добавьте 4 листа с точными названиями и заголовками:

**Menu**
```
id | name | description | price | image_url | stock | category | is_day_dish | addon_group
```
- `is_day_dish` — `TRUE`/`FALSE`, показывается во вкладке «Блюда дня»
- `addon_group` — название группы допов из листа Addons, или пусто

**Addons**
```
group | option_name | price_extra
```
Строки с одинаковым `group` образуют один выбор (например, `garnish`: «Рис» / «Картофель фри»).

**Orders** (создайте только заголовки — заполняется автоматически)
```
order_id | timestamp | telegram_user_id | telegram_username | phone | address | lat | lng | items_json | subtotal | delivery_fee | total | status | comment
```

**Config**
```
key | value
```
Заполните строки:
```
restaurant_lat     | 12.2388   (широта кухни)
restaurant_lng      | 109.1967  (долгота кухни)
bot_token            | токен вашего Telegram-бота от @BotFather
admin_chat_id        | chat_id администратора (куда падают уведомления о заказах)
hours_weekday        | ПН-ПТ: 11:00 – 20:00
hours_weekend        | СБ-ВС: выходные
```

## 2. Apps Script (бэкенд)

1. В таблице: **Расширения → Apps Script**.
2. Вставьте содержимое `Code.gs`.
3. В начале файла замените `PASTE_YOUR_SPREADSHEET_ID_HERE` на ID вашей таблицы (из URL таблицы).
4. **Развернуть → Новое развертывание → Веб-приложение**:
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Скопируйте URL веб-приложения (заканчивается на `/exec`).

## 3. Фронтенд

1. В `index.html` замените `PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL_HERE` на URL из шага 2.
2. Залейте файл в репозиторий на GitHub.
3. Включите **GitHub Pages** (Settings → Pages → ветка `main`, папка `/`) — либо задеплойте на **Vercel** (просто импортируйте репозиторий, статический сайт, доп. настроек не нужно).
4. Скопируйте итоговый URL (например `https://username.github.io/repo/index.html`).

## 4. Подключение к Telegram-боту

1. В @BotFather: `/newapp` (или `/setmenubutton` для существующего бота) → укажите URL из шага 3.
2. Чтобы бот мог присылать клиенту сообщение о статусе заказа, пользователь должен хотя бы раз нажать «Start» у бота — иначе Telegram не позволит боту писать первым.

## Логика приложения

- **Остатки:** количество на складе (`stock`) читается при загрузке меню, повторно проверяется прямо перед отправкой заказа (`checkStock`), а на бэкенде — ещё раз при записи заказа, с блокировкой (`LockService`), чтобы два одновременных заказа не увели товар в минус.
- **Доставка:** расстояние считается по формуле гаверсинуса от координат кухни (`Config.restaurant_lat/lng`) до точки, присланной через `Telegram.WebApp.LocationManager` (с запасным вариантом через `navigator.geolocation`). Тарифы — в начале `<script>` в `index.html`, константа `DELIVERY_TIERS`.
- **Избранное и корзина** хранятся в `Telegram.CloudStorage` (с откатом на `localStorage` при открытии вне Telegram — например, для тестирования в браузере).
- **История заказов** подтягивается с бэкенда по `telegram_user_id`; повторный заказ блокируется, если товара не осталось.

## Что стоит донастроить под себя

- Реальные фото блюд (`image_url` — прямые ссылки на изображения).
- Тарифная сетка доставки под фактическую географию Нячанга.
- Тексты подтверждений и сообщений администратору в `Code.gs` (`notifyAdmin_`).
