Roslyn-анализаторы - это [статические анализаторы кода](../../../../../Project%20management/Static%20code%20analyze.md), который встраиваются в [Roslyn](./Roslyn.md) и выполняются во время сборки проекта или солюшена.

Есть несколько способов подключения анализаторов к проекту:
- Включение встроенных в дотнет анализаторов. Они появились и идут вместе с дотнетом начиная с .NET 5. Для .NET Framework нужно устанавливать `Microsoft.CodeAnalysis.NetAnalyzers`
- Подключение [Nuget](../Nuget/Nuget.md) пакетов с анализаторами
- Установка расширения на IDE с анализатором

Существует две основные группы стандартных анализаторов:
- Code style (IDExxxx)
- Code quality (CAxxxx)

Помимо стандартных анализаторов есть много Nuget пакетов, которые содержат анализаторы и их можно добавить в проект:
- Microsoft.CodeAnalysis.BannedApiAnalyzers
- Roslynator.Analyzers
- TestableIO.System.IO.Abstractions

## Microsoft.CodeAnalysis.BannedApiAnalyzers
Microsoft.CodeAnalysis.BannedApiAnalyzers - анализатор, который позволяет указать определённые сигнатуры, которые будут запрещены к использованию.