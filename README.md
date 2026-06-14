# Bot for Moderators — Docker fixed fork

VK-бот для управления модераторским составом.

Этот форк исправляет базовый запуск проекта через Docker и добавляет нормальную инструкцию по установке.

## Что исправлено

## Что исправлено

* Добавлен отсутствующий `package.json`, без которого проект не собирался через Docker.
* Добавлены npm-скрипты для сборки и запуска проекта.
* Исправлен `docker-compose.yml`: контейнер бота теперь получает переменные окружения из `.env`.
* Добавлен `.env.example` с примером конфигурации.
* Добавлена инструкция по запуску через Docker Compose.
* Добавлена инструкция по добавлению первого администратора в MySQL.

## Требования

На сервере должны быть установлены:

* Docker;
* Docker Compose v2.

Проверка:

```bash
docker --version
docker compose version
```

## Установка

Клонируйте репозиторий:

```bash
git clone https://github.com/YOUR_USERNAME/bot-for-moderators.git
cd bot-for-moderators
```

Создайте `.env`:

```bash
cp .env.example .env
nano .env
```

Пример `.env`:

```env
BOT_TOKEN=your_vk_group_token
MYSQL_HOST=mysql
MYSQL_USER=usr
MYSQL_DATABASE=bot
MYSQL_PORT=3306
MYSQL_PASSWORD=root
```

`BOT_TOKEN` — это токен сообщества VK, а не токен личной страницы.

## Запуск

```bash
docker compose up -d --build
```

Проверить контейнеры:

```bash
docker ps
```

Логи бота:

```bash
docker logs -f bot
```

Логи MySQL:

```bash
docker logs -f my_mysql
```

## Остановка

```bash
docker compose down
```

Не используйте без необходимости:

```bash
docker compose down -v
```

Флаг `-v` удаляет Docker volume. Вместе с ним можно удалить базу MySQL.

## Добавление первого администратора

После первого запуска бот автоматически создаст таблицы в MySQL.

Зайдите в MySQL:

```bash
docker exec -it my_mysql mysql -uroot -p123 bot
```

Добавьте пользователя в `admins`:

```sql
INSERT INTO admins (userId, nick, appointment)
VALUES ('YOUR_NUMERIC_VK_ID', 'Your_Nickname', NOW());
```

Добавьте пользователя в `moderators`:

```sql
INSERT INTO moderators (
  userId,
  moderatorType,
  rang,
  nickname,
  firstAppointment,
  lastAppointment,
  hasPc,
  points,
  discord,
  forum,
  age,
  preds,
  vigs,
  aban
) VALUES (
  'YOUR_NUMERIC_VK_ID',
  'basic',
  'chief',
  'Your_Nickname',
  NOW(),
  NOW(),
  1,
  0,
  'none',
  'none',
  18,
  0,
  0,
  0
);
```

Проверка:

```sql
SELECT * FROM admins WHERE userId = 'YOUR_NUMERIC_VK_ID';
SELECT userId, nickname, rang FROM moderators WHERE userId = 'YOUR_NUMERIC_VK_ID';
```

Выход:

```sql
exit;
```

Перезапустите бота:

```bash
docker restart bot
```

## Частые ошибки

### `Access denied: token required`

Бот не получил `BOT_TOKEN`.

Проверьте, что в `.env` указан токен:

```env
BOT_TOKEN=your_vk_group_token
```

И что в `docker-compose.yml` у сервиса `app` есть:

```yaml
env_file:
  - .env
```

### `Я тебя не знаю`

Пользователь не добавлен в базу.

Нужно добавить его в две таблицы:

* `admins`;
* `moderators`.

Для максимального ранга используйте:

```sql
rang='chief'
```

### `ECONNREFUSED mysql:3306`

Обычно это значит, что бот стартовал раньше MySQL.
Подождите несколько секунд и проверьте логи:

```bash
docker logs -f bot
docker logs -f my_mysql
```

Если контейнер бота упал, перезапустите:

```bash
docker restart bot
```

## Безопасность

Файл `.env` нельзя заливать в GitHub, потому что внутри находится VK-токен.

Проверьте перед коммитом:

```bash
git status
```

Если `.env` попал в git:

```bash
git rm --cached .env
```

## Лицензия

Оригинальный проект распространяется по лицензии Creative Commons Attribution-NonCommercial 4.0 International.

При публикации форка сохраняйте указание на оригинального автора и лицензию.

