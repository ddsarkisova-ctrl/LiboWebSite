# Libo.Science website — handover

Короткая шпаргалка по инфраструктуре сайта `liboscience.ru`.

## Страницы

- **Главная**: https://liboscience.ru — общий лендинг про Libo.Science
- **Тестовый лендинг (антидепрессанты)**: https://liboscience.ru/pills.html

## Где что лежит

| Что | Где |
|-----|-----|
| Исходники сайта | этот GitHub репозиторий |
| Сервер | Timeweb Cloud, IP `72.56.39.162`, root доступ через панель Timeweb |
| Веб-сервер | nginx, конфиг: `/etc/nginx/sites-available/liboscience.ru` |
| Сайт на сервере | `/var/www/liboscience` — это git-клон этого репо |
| SSL | Let's Encrypt, автопродление через certbot (ничего делать не надо) |
| Домен | `liboscience.ru` — DNS управляется в Timeweb Cloud, A-записи указывают на `72.56.39.162` |
| Google Analytics | `G-Q80E9XPZG9` (вставлен в `index.html` и `pills.html`) |
| Email-форма заявок | отправляет POST на Google Apps Script (URL в `pills.html`, поиск по `SCRIPT_URL`) — данные собираются в Google Sheet |

## Как обновить сайт

1. Изменить файлы локально в этой папке
2. `git add . && git commit -m "..." && git push`
3. GitHub Actions автоматически залогинится по SSH на сервер и обновит файлы (~15 секунд после push)
4. Проверить https://liboscience.ru

Workflow: `.github/workflows/deploy.yml`

## Автодеплой: как устроен

- На сервере (`/root/.ssh/`) лежит SSH-ключ `github_deploy` (публичная часть в `authorized_keys`)
- В GitHub repo Settings → Secrets есть secret `SSH_PRIVATE_KEY` с приватной частью этого ключа
- При push в `main` GitHub Actions подключается по SSH и делает `git pull` + `systemctl reload nginx`

**Если автодеплой сломался** (например ключ отозван):
1. SSH на сервер: `ssh root@72.56.39.162`
2. Сгенерировать новый ключ: `ssh-keygen -t ed25519 -f ~/.ssh/github_deploy -N "" -C "github-actions"`
3. Добавить публичный в authorized_keys: `cat ~/.ssh/github_deploy.pub >> ~/.ssh/authorized_keys`
4. Вывести приватный: `cat ~/.ssh/github_deploy`
5. В GitHub: Settings → Secrets → Actions → обновить `SSH_PRIVATE_KEY`

## Ручной деплой (если автодеплой не работает)

```
ssh root@72.56.39.162
cd /var/www/liboscience
git pull
systemctl reload nginx
```

## Структура файлов

- `index.html` — главная страница (общий лендинг)
- `pills.html` — тестовый лендинг под антидепрессанты
- `order.html`, `technology.html` — служебные подстраницы
- `*.mp4`, `*.png` — медиа
- `.github/workflows/deploy.yml` — конфиг автодеплоя

## Известные особенности

- `photo1.png` (15 МБ) и `photo2.png` (12 МБ) — тяжёлые, стоит сжать через TinyPNG для скорости
- `hero.mp4` оптимизирован для iOS (H.264 Baseline, без аудио, ~1.3 МБ)
- На iPhone с Low Power Mode видео не автоплеит (это особенность Apple, не сайта)
