---
title: Solution metaproject
---
[MSBuild](./MSBuild.md) во время сборки оперирует [проектами](./Dotnet%20project.md). При сборке [solution'а](./Dotnet%20solution.md) создаётся искусственный проект `SolutionName.csproj.metaproj`, на котором и вызывается собка.