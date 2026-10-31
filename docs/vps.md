# CLIProxyAPI на VPS

## Подготовка

Нужны VPS с Debian 12+ или Ubuntu 22.04+, SSH-доступ под root или с sudo без пароля, домен с управлением DNS и локальный компьютер с Node.js 22.17+ для sitectl, Git, SSH, `ssh-copy-id` и `rsync`. На macOS может потребоваться свежий `rsync`: `brew install rsync`. На Windows команды выполняй в WSL. Проверь `node --version`, `git --version`, `ssh -V` и `rsync --version`. Проверь, что страна доступа и расположение сервера соответствуют условиям сервисов. Пример ниже использует имя сервера `vps` и домен `cpa.example.com`.

Установка и команды зависят от версий [sitectl](https://github.com/shura-v/sitectl), [sitectl-extensions](https://github.com/shura-v/sitectl-extensions) и [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI). Перед выполнением изучи их документацию и действия установщиков.

## Установка

1. Установи `sitectl` и зарегистрируй сервер:

   ```bash
   npm install -g sitectl
   sitectl init
   sitectl
   ```

   В меню **Manage servers** → **Add server** добавь VPS как `vps`. При необходимости создай ключ `ssh-keygen -t ed25519` и установи его на сервер через `sitectl ssh-copy-id`. Проверь `sitectl ssh vps 'echo ok'`.
2. Установи Docker и команды прокси:

   ```bash
   sitectl run docker/install-docker vps
   sitectl install https://github.com/shura-v/sitectl-extensions.git
   sitectl run cliproxyapi/install vps
   ```

   При установке набора выбери **CLIProxyAPI**. Убедись, что прокси слушает только `127.0.0.1:8317` на VPS.
3. Создай DNS A-запись `cpa` на IP VPS и проверь `nslookup cpa.example.com`. Для обычной HTTP-проверки Caddy разреши входящие TCP-порты 80 и 443 в firewall сервера и у хостинга.
4. Установи и настрой Caddy через **Manage caddy** в `sitectl`, добавь сайт и настрой обратный прокси:

   ```caddyfile
   cpa.example.com {
       @management path /management.html /v0/management /v0/management/* /v8/management /v8/management/*
       respond @management 403
       reverse_proxy 127.0.0.1:8317
   }
   ```

   Выбери **Copy conf files to server** и сервер `vps`. Проверь HTTPS. Этот пример закрывает перечисленные management-маршруты; проверь актуальные пути в документации CLIProxyAPI для установленной версии: [v0](https://help.router-for.me/management/api) и [v8](https://help.router-for.me/management/apiv8). Перед использованием извне проверь, что `https://cpa.example.com/management.html` и management API возвращают 403, а клиентские API доступны с ключом. Прямой порт 8317 не открывай наружу. Панель используй через SSH-туннель, как описано в [инструкции входа](login.md).

## Ключи и хранение

- **Ключ управления** — вход в панель; получить можно командой `sitectl run cliproxyapi/show-management-key vps`. Не показывай вывод другим.
- **Клиентский ключ прокси** — для Pi и других клиентов; создаётся или просматривается в панели конфигурации.
- **OAuth-токены** — учётные данные подписок на сервере. Это не клиентские ключи.

Храни ключи отдельно от репозитория; при утечке отзови и замени. Удаление прокси может удалить токены и потребовать повторного входа: сначала изучи команду удаления и сделай необходимую резервную копию.

Далее: [вход в подписки](login.md).
