# Подключение клиентов

Нужны работающий прокси, разрешённая провайдером авторизация и **клиентский ключ прокси**. Google запрещает сторонний доступ к Antigravity, включая OAuth через прокси: [подробности и риск блокировки](login.md#google-antigravity). Адрес в примерах — `https://cpa.example.com`; для локальной установки используй адрес с вкладки **API Access**. Не вставляй ключи в чат и не сохраняй их в репозитории.

## Pi

Установи [Pi](https://pi.dev/) и расширение согласно [документации пакета](https://pi.dev/packages/pi-cliproxyapi-provider). Для Pi нужен Node.js 22.19+:

```bash
npm install -g @earendil-works/pi-coding-agent
pi install npm:pi-cliproxyapi-provider
```

Перезапусти Pi. Выполни `/cliproxyapi config`, укажи `https://cpa.example.com/v1`, включи авторизацию Bearer и сохрани. Затем `/login cpa`: введи клиентский ключ прокси. Через `/model` выбери доступную модель провайдера `cpa`.

## Claude Code

Установи [Claude Code](https://code.claude.com/docs/en/setup), ознакомься с [правилами использования](login.md) и [документацией gateway](https://code.claude.com/docs/en/gateways). В среде запуска установи `ANTHROPIC_BASE_URL` в `https://cpa.example.com` **без `/v1`**, а `ANTHROPIC_AUTH_TOKEN` — в значение клиентского ключа прокси. Выполни команды из раздела [безопасного ввода](#безопасность-переменных), запусти `claude` из того же терминала и выбери совместимую модель. При нескольких способах авторизации проверь [приоритет учётных данных](https://code.claude.com/docs/en/authentication). Настройка gateway сама по себе не гарантирует возможность использовать подписку через него.

## Codex

Установи [Codex](https://developers.openai.com/codex/quickstart/) и ознакомься с [настройкой провайдеров](https://developers.openai.com/codex/config-advanced). В среде запуска задай `CPA_API_KEY` как значение клиентского ключа прокси способом из [раздела ниже](#безопасность-переменных). Сохрани резервную копию `~/.codex/config.toml`. В этом файле установи общую настройку до первой секции и отдельную секцию провайдера. Если ключ или секция уже существуют, измени их вместо добавления дубликатов:

```toml
model_provider = "cpa"

[model_providers.cpa]
name = "CLIProxyAPI"
base_url = "https://cpa.example.com/v1"
env_key = "CPA_API_KEY"
wire_api = "responses"
```

`env_key` — имя переменной, не её значение. На Windows файл находится в `%USERPROFILE%\.codex\config.toml`. Запусти Codex из того же терминала, где задан ключ, и выбери доступную модель. Откат: восстанови прежнее значение `model_provider` и другие изменённые настройки.

## Безопасность переменных

Для macOS и Linux открой отдельный терминал и запусти `bash`. Введи ключ без отображения и без записи значения в историю; затем назначь его нужному клиенту:

```bash
read -r -s -p 'Ключ прокси: ' CPA_API_KEY; echo
export CPA_API_KEY
# Для Claude Code также:
export ANTHROPIC_AUTH_TOKEN="$CPA_API_KEY"
export ANTHROPIC_BASE_URL='https://cpa.example.com'
```

В PowerShell используй скрытый ввод, затем задай переменные процесса:

```powershell
$key = Read-Host 'Ключ прокси' -AsSecureString
$credential = [System.Management.Automation.PSCredential]::new('proxy', $key)
$env:CPA_API_KEY = $credential.GetNetworkCredential().Password
$env:ANTHROPIC_AUTH_TOKEN = $env:CPA_API_KEY
$env:ANTHROPIC_BASE_URL = 'https://cpa.example.com'
```

Запусти `codex` или `claude` из того же терминала: другие уже запущенные процессы эти переменные не получат. Не печатай переменные и не передавай ключ в командной строке. После работы закрой терминал; в текущем процессе bash можно выполнить `unset CPA_API_KEY ANTHROPIC_AUTH_TOKEN ANTHROPIC_BASE_URL`, а в PowerShell — `Remove-Item Env:CPA_API_KEY, Env:ANTHROPIC_AUTH_TOKEN, Env:ANTHROPIC_BASE_URL; Remove-Variable key, credential`. Секреты находятся в памяти процессов, пока те работают. Эти шаги относятся к CLI, не к GUI-приложениям.

Далее: [проверка](check.md).
