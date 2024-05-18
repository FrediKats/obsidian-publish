---
title: NuGet deploy workflow evolution
---
NuGet - отличный пакетный менеджер, который решает огромное количество задач. Иногда даже те задачи, под которые он изначально не разрабатывался. Но тут перед разработчиком появляется задача - "докатить" свой код до пакетного менеджера. Сделать "хоть как-то" достаточно легко и из-за этого появляется множество способов достижения цели. Конфигурация сборки и упаковки проходит жизненный путь вместе с проектом и разработчиками, улучшается, эволюционирует. И чем больше разных способов решения изначально задачи, тем сложнее актуализировать и удерживать в консистентном состоянии процесс, когда количество пакетов переваливает за 30.

## Этап 0. Ручная загрузка
Загрузка пакета начинается с его сборки. В .NET для этого есть [CLI](../../Knowledge%20base/Developing/Dotnet/Build%20process/Dotnet%20CLI.md), который позволяет легко собрать из проекта пакет:
```
dotnet build -c Release MyProject.csproj
dotnet pack MyProject.csproj
```

На выходе получится [.nupkg файл](../../Knowledge%20base/Developing/Dotnet/Nuget/%D0%A4%D0%B0%D0%B9%D0%BB%20.nupkg.md), который и распространяется пакетным менеджером. Важным элементом архитектуры пакетных менеджеров являются централизованные репозитории, где эти пакеты хранятся. В случае NuGet основным таким является nuget.org. Репозитории предоставляют набор HTTP Endpoint'ов, через которые можно загрузить свой пакет. В случае nuget.org на самом сайте даже есть Web UI куда можно dran'n'drop'ом закинуть файл.
После того как файл будет загружен, nuget проанализирует его, проверит, проиндексирует и через несколько минут он станет доступным для скачивания.

## Этап 1. Интеграция в CI/CD
Проблема этапа 0 отражена в названии. Это ручная работа, которая может быть ок при 1-3-5 пакетах. Но перекладывать руками пакеты после сборки каждой новой версии, когда их 10-30-50 уже становится больно. На помощь приходит [CI](../../Knowledge%20base/DevOps/Continuous%20integration.md)/[CD](../../Knowledge%20base/DevOps/Continuous%20Delivery.md). Обычно репозитории существуют в какой-то [системе управления проектами](../../Knowledge%20base/Project%20management/Project%20management%20system.md), которая поддерживает CI/CD и даёт возможность настроить процесс сборки и развёртывания пакетов. Например, у GitHub репозитории можно завести файл `.github/workflows/ci-cd.yaml`, который будет парсится GitHub'ом при работе с репозиторием и выполнятся.
Пример такой конфигурации:
```yaml
name: Build and test

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ '*' ]

env:
  dotnet-version: 8.0.x

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build -c Release --no-restore --verbosity normal

    - name: Test
      run: dotnet test -c Release --no-build
```

К нему достаточно добавить шаг с выполнением `dotnet pack` и загрузить в nuget.org:
```yaml
- name: Pack
  run: dotnet pack --no-build

- name: Publish to Nuget
  run: dotnet nuget push ${{ env.release-directory }}/*.nupkg --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json --skip-duplicate
```

## Этап 2. Версионирование пакета
В тексте про [Solution configuration](./2024-05-12-Solution-configuration-as-a-package.md) было описано как можно распространять настройки для пакетов с помощью пакета. Но версия - это значение, которое уникальное для каждого проекта или группы пакетов в solution'е. Это значит, что задавать нужно руками в Directory.Build.props. А также обновлять руками перед публикацией.
Первая проблема данного подхода заметка даже в CI шаге:
```yaml
dotnet nuget push ${{ env.release-directory }}/*.nupkg --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json --skip-duplicate
```

Рассмотрим сценарий:
1. Создаётся коммит с изменениями, вешается версия 1.2.3;
2. Запускается CI, собирается пакет, публикуется;
3. Создаётся коммит с новыми изменениями, но не увеличивается версия;
4. Запускается CI, собирается пакет, начинается публикация;
5. Публикация падает с тем, что пакет с версией 1.2.3 уже существует.

Данная проблема решается флагом `--skip-duplicate`, который говорит nuget.org'у, что это окей, если пакет с такой версией существует. В такой ситуации ничего не публикуется.

Вторая проблема - это необходимость фиксировать в коде версии до того, как изменения попадут в репозиторий. Во-первых, разработчики постоянно забывают увеличивать версию и приходится отдельным PR докидывать увеличение версии. Во-вторых, может быть такой процесс разработки, когда версию увеличивают не после каждого изменения, а только когда было принято решение, что пора. В такой ситуации также необходимо будет создавать отдельный PR и явно увеличивать версию.

Третья проблема - помимо версий в `.props` файле было бы круто ещё и повесить git-теги, чтобы по истории комитов легко можно было бы найти нужную версию. Это значит, что нужно каждую версию указывать дважды. 

У этих проблем есть множество разных решений, но одно из них - это версионирование построенное на метаданных git'а. Есть несколько инструментов, которые поддерживают такое поведение: [adamralph/minver](https://github.com/adamralph/minver), [dotnet/Nerdbank.GitVersioning](https://github.com/dotnet/Nerdbank.GitVersioning), [GitTools/GitVersion](https://github.com/GitTools/GitVersion). Они работают немного разным способом. Был выбран minver как самый простой из них. Алгоритм работы такой:
- Удаляется указание версии пакета из исходного кода - пакет теперь собирается с версий 1.0.0
- В проект, который пакуется, добавляет использование MinVer.
- Запускается сборка. Во время сборки MinVer вытаскивает информацию о git репозитории, где он находится, ищет последний тег и вставляет его как версию.
	- Если тег висит на текущем комите, то он используется as is
	- Если относительно последнего коммита с тегом появились новые теги без коммитов, то добавляется постфикс `-alpha.{количество коммитов}`
	- Если тегов никаких не было, то используется `0.0.0-aplha.{количество комитов}`

Это означает, что теперь теги git'а контролируют версию, а значит проблема №3 закрыта. Более того, выставление тегов отвязано от создания коммитов, а значит их можно навесить уже после создания. И проблема №2 также будет решена.

## Этап 3. Символы
Если отладка кода кажется чем-то сложным, то у вас простой проект. На больших проектах отладка - это всегда сущий ад. И пакеты внесли в это свой вклад. По мере распространения пакетов всё чаще появлялись ситуации, когда нужно было отладить не свой код, а код пакета. Со своим кодом всё просто - он собран в debug'е, есть символы от него. А вот пакеты собираются в Release и с ними уже символов нет.

Но на самом деле решение уже есть, но не все пакеты его поддерживают. Это решение - это `.snupkg`. Идея данного подхода в том, что символы можно распространять точно также, как и пакеты используя пакетный менеджер. Более подробно описано в документации Microsoft: https://learn.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg.

Для разработчика это превращается в два действия. Первое - это добавление свойств в MSBuild для генерации `.snupkg`:
```xml
<PropertyGroup>
  <IncludeSymbols>true</IncludeSymbols>
  <SymbolPackageFormat>snupkg</SymbolPackageFormat>
</PropertyGroup>
```

Второе - это добавление публикации сгенерированного файла вместе с пакетом:
```yaml
- name: Publish to Nuget
  run: dotnet nuget push ${{ env.release-directory }}/*.nupkg --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json --skip-duplicate

- name: Publish to Nuget symbols
  run: dotnet nuget push ${{ env.release-directory }}/*.snupkg --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json --skip-duplicate
```

## Этап 4. Синхронизация action'ов между репозиториями
И вот наступает момент, когда нужно внести изменения в `ci-cd.yaml`. А репозиториев уже 30. А после того, как руками 30 репозиториев будут изменены вдруг окажется, что помимо публикации пакетов хотелось бы ещё для всех нюгетов генерировать test coverage. И нужно ещё раз пройтись по 30 репозиториям.
Ситуация напоминает проблему с конфигурацией solution'ов и решается таким же способом - избавлением от дублирования. И нет, никто не будет добавлять `ci-cd.yaml` файл в пакет. Решение будет рассмотрено на примере GitHub, но в Azure DevOps эта задача решается аналогично (даже проще, местами). Решение строится на Resusable workflows - https://docs.github.com/en/actions/using-workflows/reusing-workflows#overview. GitHub Actions позволяет вызывать из workflow другой workflow. Делается это в два шага. Шаг первый - выделение "образцового" workflow. Создаётся в отдельном репозитории workflow, который описывает универсальный CI/CD для всех репозиториев с нюгетами. С большой вероятностью, если это схожие пакеты, которые пишет одна команда, то он будет идентичный CI/CD и его можно шаблонизировать:
```yaml
on:
  workflow_call:
    inputs:
      runs-on:
        type: string
        required: false
        default: 'ubuntu-latest'
      dotnet-version:
        type: string
        required: false
        default: '8.0'

    secrets:
      NUGET_API_KEY:
        required: true

env:
  DOTNET_SKIP_FIRST_TIME_EXPERIENCE: true

jobs:
  build:
    runs-on: ${{ inputs.runs-on }}

    permissions:
      actions: write
      contents: write

    steps:
    - name: Checkout main repository
      uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Install dotnet
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ inputs.dotnet-version }}

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build -c Release --no-restore --verbosity normal

    - name: Test
      run: dotnet test -c Release --no-build
```

Второй шаг - замена во всех репозиториях логики из `ci-cd.yaml` на вызов данного workflow:
```yaml
name: CI/CD

on:
  push:
  pull_request:

jobs:
  build:
    uses: Kysect/.github/.github/workflows/ci-cd.yaml@master
    with:
      dotnet-version: '8.0'
    secrets:
      NUGET_API_KEY: ${{ secrets.NUGET_API_KEY }}
```

Таким образом решается проблема дублирования и появляется централизованный workflow, который работает для всех репозиториев. Разумеется, в какой-то момент универсальность даст сбой, появится шаг, который нужен только для одного репозитория. Примером такого шага может стать дополнительная зависимость на компонент, который нужно установить:
```yaml
on:
  workflow_call:
    inputs:
# skip lines
      install-dotnet-aspire:
        type: boolean
        required: false
        default: false

jobs:
  build:
    runs-on: ${{ inputs.runs-on }}

    permissions:
      actions: write
      contents: write

    steps:
# skip lines

	- if: ${{ inputs.install-dotnet-aspire == true }}
      name: Install .NET Aspire workload
      run: dotnet workload install aspire

# skip lines
```

Но тут уже нужно балансировать между сложностью использовать общий шаблон и затратами на поддержку множества конфигураций.

## Этап 5. dotnet-releaser
После этапа 4 уже достигнута точка, на котором можно было бы остановиться. Но иногда хочется, чтобы для проблемы существовал инструмент, который с коробки решает проблемы пользователя даже лучше, чем сам пользователь мог бы представить. И самое главное, что такие инструменты существуют. Один из них - это [xoofx/dotnet-releaser](https://github.com/xoofx/dotnet-releaser). `dotnet-releaser` - это CLI приложение, которые выполняет набор шагов необходимых для публикации NuGet пакета:
- Сборка solution'а
- Выполнение тестов (и генерация test coverage)
- Создание пакета
- Публикация пакета
- (!) Создание GitHub release

Это значит, что вместо всего нашего `ci-cd.yaml` можно написать:
```yaml
# skip lines

jobs:
  build:
# skip lines
    steps:
# skip lines
    - name: Install dotnet
      uses: actions/setup-dotnet@v4
      with:

    - name: Run dotnet-releaser
      shell: bash
      run: |
        dotnet tool install -g dotnet-releaser
        dotnet-releaser run --nuget-token "${{secrets.NUGET_API_KEY}}"  --github-token "${{secrets.GITHUB_TOKEN}}" Sources/dotnet-releaser.toml
```

И весь функционал библиотеки `dotnet-releaser` появится в CI/CD репозитория. О всех возможностях можно почитать в user guide. Но из интересного (помимо замены всех шагов, которые и так был раньше указаны) есть генерация GitHub release. `dotnet-releaser` отслеживает изменения версии и при добавлении тега запускает формирование дельты относительно прошлой версии. dotnet-releaser парсит коммиты и Pull requrest'ы, которые были сделаны между двумя тегами и по ним генерирует GitHub release: https://github.com/xoofx/markdig/releases/tag/0.37.0.

## Summary
Был рассмотрен поэтапный процесс эволюции подхода к публикации NuGet пакета. На входе был ручной процесс, который требовал много усилий, усложнялся с каждым новым репозиторием. На выходе получили переиспользуемый workflow, который легко включается в новый репозиторий и _задаёт минимальный уровень качества_ процесса сборки.