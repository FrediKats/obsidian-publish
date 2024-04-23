Obsidian (https://obsidian.md/) - это приложение для создания заметок и/или персональной базы знаний.

## Obsidian vault
Obsidian vault - это директория, которая содержит заметки и директорию `.obsidian` с конфигурациями.
Среди файлов в `.obsidian` есть файл workspace, который сохраняет временную информацию. При сохранении в [git](../git/git.md) стоит добавить его в [gitignore](../git/gitignore.md).
Источник: [How Obsidian stores data - Obsidian Help](https://help.obsidian.md/Files+and+folders/How+Obsidian+stores+data)

## Plugins
Obsidian расширяется за счёт плагинов. Есть ряд встроенных плагинов:
- [Obsidian Templates](./Obsidian%20Templates.md)

## Publish
Obsidian предоставляет услуги развёртывания базы знаний как сайта. Более подробно тут: [Obsidian Publish](https://obsidian.md/publish)

Алгоритм:
- Внести изменения
- Использовать плагин Publish, выбрать файлы, которые нужно запаблишить
- Проверить результат на https://publish.obsidian.md/site-name, где site-name - название сайта, которое было указано при конфигурации Publish'а.

Плюсы:
- Максимально упрощённый способ развёртывания - в UI появляется кнопка publish. Первый раз нужно конфигурировать, а далее просто выбирать какие файлы запаблишить.
- Поддержка стандартных плагинов остаётся на стороне Obsidian. С коробки работают backlinks, Graph

Минусы:
- Требует платной подписки
- Поддерживается только [CloudFlare](../../Services/CloudFlare.md) домены. Подключить [GoDaddy](../../Services/GoDaddy.md) без дополнительных усилий
- Нет готовой интеграции с [Github Actions](../../Services/Github/Github%20Actions.md) для автоматического деплоя

Альтернативы:
- [GitHub - ObsidianPublisher/obsidian-github-publisher: Github Publisher helps you to publish your notes on a preconfigured GitHub repository from your Obsidian Vault, for free, and more!](https://github.com/ObsidianPublisher/obsidian-github-publisher)