---
title: EF Core 工具参考（.NET CLI）-EF Core
author: bricelam
ms.author: bricelam
ms.date: 07/11/2019
uid: core/miscellaneous/cli/dotnet
ms.openlocfilehash: 29434c26a503fabb16b43ee8f0c36136a0b5b745
ms.sourcegitcommit: 2355447d89496a8ca6bcbfc0a68a14a0bf7f0327
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2019
ms.locfileid: "72811970"
---
# <a name="entity-framework-core-tools-reference---net-cli"></a>Entity Framework Core 工具参考-.NET CLI

用于 Entity Framework Core 的命令行接口（CLI）工具执行设计时开发任务。 例如，他们基于现有数据库创建[迁移](/aspnet/core/data/ef-mvc/migrations?view=aspnetcore-2.0)、应用迁移和生成模型的代码。 命令是跨平台[dotnet](/dotnet/core/tools)命令的扩展，它是[.NET Core SDK](https://www.microsoft.com/net/core)的一部分。 这些工具适用于 .NET Core 项目。

如果使用的是 Visual Studio，我们建议改用[包管理器控制台工具](powershell.md)：

* 它们自动使用在**包管理器控制台**中选择的当前项目，而无需手动切换目录。
* 在命令完成后，它们会自动打开由命令生成的文件。

## <a name="installing-the-tools"></a>安装工具

安装过程取决于项目类型和版本：

* EF Core 1。x
* ASP.NET Core 版本2.1 及更高版本
* EF Core 2。x
* EF Core 1。x

### <a name="ef-core-3x"></a>EF Core 1。x

* `dotnet ef` 必须安装为全局或本地工具。 大多数开发人员会使用以下命令将 `dotnet ef` 安装为全局工具：

  ``` console
  dotnet tool install --global dotnet-ef
  ```

  你还可以使用 `dotnet ef` 作为本地工具。 若要将其用作本地工具，请使用[工具清单文件](https://github.com/dotnet/cli/issues/10288)还原项目的依赖项，将该项目声明为工具依赖项。

* 安装[.NET Core SDK 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0)）。 即使使用最新版本的 Visual Studio，也必须安装 SDK。

* 安装最新的 `Microsoft.EntityFrameworkCore.Design` 包。

  ``` Console
  dotnet add package Microsoft.EntityFrameworkCore.Design
  ```

### <a name="aspnet-core-21"></a>ASP.NET Core 2.1 +

* 安装当前[.NET Core SDK](https://www.microsoft.com/net/download/core)。 即使有 Visual Studio 2017 的最新版本，也必须安装 SDK。

  这是 ASP.NET Core 2.1 + 所需的全部，因为 `Microsoft.EntityFrameworkCore.Design` 包包含在[AspNetCore 元包](/aspnet/core/fundamentals/metapackage-app)中。

### <a name="ef-core-2x-not-aspnet-core"></a>EF Core 2.x （不 ASP.NET Core）

`dotnet ef` 命令包含在 .NET Core SDK 中，但要启用这些命令，必须安装 `Microsoft.EntityFrameworkCore.Design` 包。

* 安装当前[.NET Core SDK](https://www.microsoft.com/net/download/core)。 即使使用最新版本的 Visual Studio，也必须安装 SDK。

* 安装最新的稳定 `Microsoft.EntityFrameworkCore.Design` 包。

  ``` Console
  dotnet add package Microsoft.EntityFrameworkCore.Design
  ```

### <a name="ef-core-1x"></a>EF Core 1。x

* 安装 .NET Core SDK 版本2.1.200。 更高版本与用于 EF Core 1.0 和1.1 的 CLI 工具不兼容。

* 通过修改应用程序的[global.asax](/dotnet/core/tools/global-json)文件，将该应用程序配置为使用 2.1.200 SDK 版本。 此文件通常包含在解决方案目录中（项目上面的一个）。

* 编辑项目文件，并将 `Microsoft.EntityFrameworkCore.Tools.DotNet` 作为 `DotNetCliToolReference` 项添加。 指定最新的1.x 版本，例如：1.1.6。 请参阅本部分末尾的项目文件示例。

* 安装最新版本的 `Microsoft.EntityFrameworkCore.Design` 包，例如：

  ```console
  dotnet add package Microsoft.EntityFrameworkCore.Design -v 1.1.6
  ```

  添加这两个包引用后，项目文件将如下所示：

  ``` xml
  <Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
      <OutputType>Exe</OutputType>
      <TargetFramework>netcoreapp1.1</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
      <PackageReference Include="Microsoft.EntityFrameworkCore.Design"
                        Version="1.1.6"
                         PrivateAssets="All" />
    </ItemGroup>
    <ItemGroup>
       <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet"
                              Version="1.1.6" />
    </ItemGroup>
  </Project>
  ```

  不向引用此项目的项目公开带有 `PrivateAssets="All"` 的包引用。 对于通常只在开发过程中使用的包，此限制特别有用。

### <a name="verify-installation"></a>验证安装

运行以下命令以验证是否正确安装了 EF Core CLI 工具：

  ``` Console
  dotnet restore
  dotnet ef
  ```

命令的输出标识所使用的工具的版本：

```console

                     _/\__
               ---==/    \\
         ___  ___   |.    \|\
        | __|| __|  |  )   \\\
        | _| | _|   \_/ |  //|\\
        |___||_|       /   \\\/\\

Entity Framework Core .NET Command-line Tools 2.1.3-rtm-32065

<Usage documentation follows, not shown.>
```

## <a name="using-the-tools"></a>使用工具

在使用这些工具之前，您可能需要创建一个启动项目或设置环境。

### <a name="target-project-and-startup-project"></a>目标项目和启动项目

命令引用*项目*和*启动项目*。

* 该*项目*也称为*目标项目*，因为它是命令在其中添加或删除文件的位置。 默认情况下，当前目录中的项目是目标项目。 您可以使用<nobr>`--project`</nobr>选项指定其他项目作为目标项目。

* *启动项目*是工具生成和运行的项目。 这些工具必须在设计时执行应用程序代码，以获取有关项目的信息，例如数据库连接字符串和模型的配置。 默认情况下，当前目录中的项目是启动项目。 您可以使用 " <nobr>`--startup-project`</nobr> " 选项指定一个不同的项目作为启动项目。

启动项目和目标项目通常是同一个项目。 它们是不同的项目的典型方案是：

* EF Core 的上下文和实体类位于 .NET Core 类库中。
* .NET Core 控制台应用程序或 web 应用程序引用类库。

还可以[将迁移代码放在与 EF Core 上下文分离](xref:core/managing-schemas/migrations/projects)的类库中。

### <a name="other-target-frameworks"></a>其他目标框架

CLI 工具适用于 .NET Core 项目和 .NET Framework 项目。 .NET Standard 类库中具有 EF Core 模型的应用可能没有 .NET Core 或 .NET Framework 项目。 例如，这适用于 Xamarin 和通用 Windows 平台应用。 在这种情况下，你可以创建一个仅供其使用的 .NET Core 控制台应用项目，作为工具的启动项目。 项目可以是不包含实际代码 &mdash; 的虚拟项目，只需为工具提供目标。

为什么需要虚拟项目？ 如前文所述，这些工具必须在设计时执行应用程序代码。 为此，需要使用 .NET Core 运行时。 当 EF Core 模型位于面向 .NET Core 或 .NET Framework 的项目中时，EF Core 工具会借用项目中的运行时。 如果 EF Core 模型在 .NET Standard 类库中，则无法执行此操作。 .NET Standard 不是实际的 .NET 实现;它是 .NET 实现所必须支持的一组 Api 的规范。 因此 .NET Standard 不足以执行应用程序代码 EF Core 工具。 你创建的用于启动项目的虚拟项目提供了一个具体的目标平台，工具可在其中加载 .NET Standard 类库。

### <a name="aspnet-core-environment"></a>ASP.NET Core 环境

若要为 ASP.NET Core 项目指定环境，请在运行命令之前设置**ASPNETCORE_ENVIRONMENT**环境变量。

## <a name="common-options"></a>常用选项

|                   | 选项                            | 描述                                                                                                                                                                                                                                                   |
|:------------------|:----------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                   | `--json`                          | 显示 JSON 输出。                                                                                                                                                                                                                                             |
| <nobr>`-c`</nobr> | `--context <DBCONTEXT>`           | 要使用的 `DbContext` 类。 仅命名空间或完全限定类名。  如果省略此选项，EF Core 将查找上下文类。 如果有多个上下文类，则需要此选项。                                            |
| `-p`              | `--project <PROJECT>`             | 目标项目的项目文件夹的相对路径。  默认值为当前文件夹。                                                                                                                                                              |
| `-s`              | `--startup-project <PROJECT>`     | 启动项目的项目文件夹的相对路径。 默认值为当前文件夹。                                                                                                                                                              |
|                   | `--framework <FRAMEWORK>`         | [目标框架](/dotnet/standard/frameworks)的[目标框架名字对象](/dotnet/standard/frameworks#supported-target-framework-versions)。  当项目文件指定多个目标框架，并想要选择其中一个时，请使用。 |
|                   | `--configuration <CONFIGURATION>` | 生成配置，例如： `Debug` 或 `Release`。                                                                                                                                                                                                   |
|                   | `--runtime <IDENTIFIER>`          | 要为其还原包的目标运行时的标识符。 有关运行时标识符 (RID) 的列表，请参阅 [RID 目录](/dotnet/core/rid-catalog)。                                                                                                      |
| `-h`              | `--help`                          | 显示帮助信息。                                                                                                                                                                                                                                        |
| `-v`              | `--verbose`                       | 显示详细输出。                                                                                                                                                                                                                                          |
|                   | `--no-color`                      | 不要为输出着色。                                                                                                                                                                                                                                        |
|                   | `--prefix-output`                 | 用 level 作为输出前缀。                                                                                                                                                                                                                                     |

## <a name="dotnet-ef-database-drop"></a>dotnet ef 数据库删除

删除数据库。

选项:

|                   | 选项                   | 描述                                              |
|:------------------|:-------------------------|:---------------------------------------------------------|
| <nobr>`-f`</nobr> | <nobr>`--force`</nobr>   | 不要确认。                                           |
|                   | <nobr>`--dry-run`</nobr> | 显示要删除的数据库，但不删除它。 |

## <a name="dotnet-ef-database-update"></a>dotnet ef 数据库更新

将数据库更新到上次迁移或指定迁移。

参数：

| 参数      | 描述                                                                                                                                                                                                                                                     |
|:--------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `<MIGRATION>` | 目标迁移。 可以按名称或 ID 识别迁移。 数字0是一种特殊情况，表示在*第一次迁移之前*，并导致还原所有迁移。 如果未指定迁移，则该命令默认为上一次迁移。 |

下面的示例将数据库更新为指定的迁移。 第一个使用迁移名称，第二个使用迁移 ID：

```console
dotnet ef database update InitialCreate
dotnet ef database update 20180904195021_InitialCreate
```

## <a name="dotnet-ef-dbcontext-info"></a>dotnet ef dbcontext 信息

获取有关 `DbContext` 类型的信息。

## <a name="dotnet-ef-dbcontext-list"></a>dotnet ef dbcontext 列表

列出可用 `DbContext` 类型。

## <a name="dotnet-ef-dbcontext-scaffold"></a>dotnet ef dbcontext 基架

为数据库的 `DbContext` 和实体类型生成代码。 为了使此命令生成实体类型，数据库表必须具有主键。

参数：

| 参数       | 描述                                                                                                                                                                                                             |
|:---------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `<CONNECTION>` | 数据库的连接字符串。 对于 ASP.NET Core 2.x 项目，值可以是*名称 =\<连接字符串 > 的名称*。 在这种情况下，该名称来自为项目设置的配置源。 |
| `<PROVIDER>`   | 要使用的提供程序。 通常，这是 NuGet 包的名称，例如： `Microsoft.EntityFrameworkCore.SqlServer`。                                                                                           |

选项:

|                 | 选项                                   | 描述                                                                                                                                                                    |
|:----------------|:-----------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>-d.ddd...e</nobr> | `--data-annotations`                     | 使用属性配置模型（如果可能）。 如果省略此选项，则只使用 Fluent API。                                                                |
| `-c`            | `--context <NAME>`                       | 要生成的 `DbContext` 类的名称。                                                                                                                                 |
|                 | `--context-dir <PATH>`                   | 要在其中放置 `DbContext` 类文件的目录。 路径相对于项目目录。 命名空间是从文件夹名称派生的。                                 |
| `-f`            | `--force`                                | 覆盖现有文件。                                                                                                                                                      |
| `-o`            | `--output-dir <PATH>`                    | 要在其中放置实体类文件的目录。 路径相对于项目目录。                                                                                       |
|                 | <nobr>`--schema <SCHEMA_NAME>...`</nobr> | 要为其生成实体类型的表的架构。 若要指定多个架构，请对每个架构重复 `--schema`。 如果省略此选项，则包括所有架构。          |
| `-t`            | `--table <TABLE_NAME>`...                | 要为其生成实体类型的表。 若要指定多个表，请对每个表重复 `-t` 或 `--table`。 如果省略此选项，则包括所有表。                |
|                 | `--use-database-names`                   | 使用表和列的名称与数据库中显示的名称完全相同。 如果省略此选项，则更改数据库名称以更严格地C#符合名称样式约定。 |

下面的示例基架所有架构和表，并将新文件放在*模型*文件夹中。

```console
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models
```

以下示例仅基架选定的表，并在具有指定名称的单独文件夹中创建上下文：

```console
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models -t Blog -t Post --context-dir Context -c BlogContext
```

## <a name="dotnet-ef-migrations-add"></a>dotnet ef 迁移添加

添加新的迁移。

参数：

| 参数 | 描述                |
|:---------|:---------------------------|
| `<NAME>` | 迁移的名称。 |

选项:

|                   | 选项                             | 描述                                                                                                      |
|:------------------|:-----------------------------------|:-----------------------------------------------------------------------------------------------------------------|
| <nobr>`-o`</nobr> | <nobr>`--output-dir <PATH>`</nobr> | 要使用的目录（和子命名空间）。 路径相对于项目目录。 默认值为 "迁移"。 |

## <a name="dotnet-ef-migrations-list"></a>dotnet ef 迁移列表

列出可用迁移。

## <a name="dotnet-ef-migrations-remove"></a>dotnet ef 迁移删除

删除上一次迁移（回滚针对迁移进行的代码更改）。

选项:

|                   | 选项    | 描述                                                                     |
|:------------------|:----------|:--------------------------------------------------------------------------------|
| <nobr>`-f`</nobr> | `--force` | 恢复迁移（回滚应用于数据库的更改）。 |

## <a name="dotnet-ef-migrations-script"></a>dotnet ef 迁移脚本

从迁移生成 SQL 脚本。

参数：

| 参数 | 描述                                                                                                                                                   |
|:---------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `<FROM>` | 开始迁移。 可以按名称或 ID 识别迁移。 数字0是一个特殊情况，表示在*第一次迁移之前*。 默认值为0。 |
| `<TO>`   | 结束迁移。 默认为上次迁移。                                                                                                         |

选项:

|                   | 选项            | 描述                                                        |
|:------------------|:------------------|:-------------------------------------------------------------------|
| <nobr>`-o`</nobr> | `--output <FILE>` | 要写入脚本的文件。                                   |
| `-i`              | `--idempotent`    | 生成可用于任何迁移的数据库的脚本。 |

以下示例创建用于 InitialCreate 迁移的脚本：

```console
dotnet ef migrations script 0 InitialCreate
```

以下示例在 InitialCreate 迁移之后为所有迁移创建一个脚本。

```console
dotnet ef migrations script 20180904195021_InitialCreate
```

## <a name="additional-resources"></a>其他资源

* [迁移](xref:core/managing-schemas/migrations/index)
* [反向工程](xref:core/managing-schemas/scaffolding)
