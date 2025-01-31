---
title: 数据库提供程序 - EF Core
author: rowanmiller
ms.date: 02/23/2018
ms.assetid: 14fffb6c-a687-4881-a094-af4a1359a296
uid: core/providers/index
ms.openlocfilehash: 0995c2cb403cf4f0db1462f1e3446a13563a1064
ms.sourcegitcommit: 18ab4c349473d94b15b4ca977df12147db07b77f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2019
ms.locfileid: "73654819"
---
# <a name="database-providers"></a>数据库提供程序

Entity Framework Core 可通过名为数据库提供程序的插件库访问许多不同的数据库。

## <a name="current-providers"></a>当前提供程序

> [!IMPORTANT]  
> EF Core 提供程序由多种源生成。 并非所有提供程序均作为 [Entity Framework Core 项目](https://github.com/aspnet/EntityFrameworkCore)的组成部分进行维护。 考虑使用提供程序时，请务必评估质量、授权、支持等因素，确保其满足要求。 同时也请务必查看每个提供程序的文档，详细了解版本兼容性信息。

| NuGet 程序包                                                                                                        | 支持的数据库引擎 | 维护商/供应商                                                           | 备注/要求 | 有用的链接                                                                                                                                                                                       |
|:---------------------------------------------------------------------------------------------------------------------|:---------------------------|:------------------------------------------------------------------------------|:---------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer)    | SQL Server 2012 以上版本    | [EF Core 项目](https://github.com/aspnet/EntityFrameworkCore/) (Microsoft) |                      | [docs](xref:core/providers/sql-server/index)                                                                                                                                                       |
| [Microsoft.EntityFrameworkCore.Sqlite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite)          | SQLite 3.7 及以上版本         | [EF Core 项目](https://github.com/aspnet/EntityFrameworkCore/) (Microsoft) |                      | [docs](xref:core/providers/sqlite/index)                                                                                                                                                           |
| [Microsoft.EntityFrameworkCore.InMemory](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory)      | EF Core 内存中数据库 | [EF Core 项目](https://github.com/aspnet/EntityFrameworkCore/) (Microsoft) | 仅用于测试     | [docs](xref:core/providers/in-memory/index)                                                                                                                                                        |
| [Microsoft.EntityFrameworkCore.Cosmos](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Cosmos)          | Azure Cosmos DB SQL API    | [EF Core 项目](https://github.com/aspnet/EntityFrameworkCore/) (Microsoft) |                      | [docs](xref:core/providers/cosmos/index)                                                                                                                                                           |
| [Npgsql.EntityFrameworkCore.PostgreSQL](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL)        | postgresql                 | [Npgsql 开发团队](https://github.com/npgsql)                          |                      | [docs](https://www.npgsql.org/efcore/index.html)                                                                                                                                                   |
| [Pomelo.EntityFrameworkCore.MySql](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql)                  | MySQL、MariaDB             | [Pomelo Foundation 项目](https://github.com/PomeloFoundation)              |                      | [自述文件](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql/blob/master/README.md)                                                                                               |
| [Pomelo.EntityFrameworkCore.MyCat](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MyCat)                  | MyCAT 服务器               | [Pomelo Foundation 项目](https://github.com/PomeloFoundation)              | 仅预发行版      | [自述文件](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MyCat/blob/master/README.md)                                                                                               |
| [EntityFrameworkCore.SqlServerCompact40](https://www.nuget.org/packages/EntityFrameworkCore.SqlServerCompact40)      | SQL Server Compact 4.0     | [Erik Ejlskov Jensen](https://github.com/ErikEJ/)                             | .NET Framework       | [wiki](https://github.com/ErikEJ/EntityFramework.SqlServerCompact/wiki/Using-EF-Core-with-SQL-Server-Compact-in-Traditional-.NET-Applications)                                                     |
| [EntityFrameworkCore.SqlServerCompact35](https://www.nuget.org/packages/EntityFrameworkCore.SqlServerCompact35)      | SQL Server Compact 3.5     | [Erik Ejlskov Jensen](https://github.com/ErikEJ/)                             | .NET Framework       | [wiki](https://github.com/ErikEJ/EntityFramework.SqlServerCompact/wiki/Using-EF-Core-with-SQL-Server-Compact-in-Traditional-.NET-Applications)                                                     |
| [FirebirdSql.EntityFrameworkCore.Firebird](https://www.nuget.org/packages/FirebirdSql.EntityFrameworkCore.Firebird/) | Firebird 2.5 和 3.x       | [Jiří Činčura](https://github.com/cincuranet)                                 |                      | [docs](https://github.com/cincuranet/FirebirdSql.Data.FirebirdClient/blob/master/Provider/docs/entity-framework-core.md)                                                                           |
| [EntityFrameworkCore.FirebirdSql](https://www.nuget.org/packages/EntityFrameworkCore.FirebirdSql/)                   | Firebird 2.5 和 3.x       | [Rafael Almeida](https://github.com/ralmsdeveloper)                           |                      | [wiki](https://github.com/ralmsdeveloper/EntityFrameworkCore.FirebirdSQL/wiki)                                                                                                                     |
| [MySql.Data.EntityFrameworkCore](https://www.nuget.org/packages/MySql.Data.EntityFrameworkCore)                      | MySQL                      | [MySQL 项目](https://dev.mysql.com) (Oracle)                               |                      | [docs](https://dev.mysql.com/doc/connector-net/en/connector-net-entityframework-core.html)                                                                                                         |
| [Oracle.EntityFrameworkCore](https://www.nuget.org/packages/Oracle.EntityFrameworkCore/)                             | Oracle DB 11.2 及更高版本     | [Oracle](https://www.oracle.com/technetwork/topics/dotnet/)                   | 预发行           | [网站](https://www.oracle.com/technetwork/topics/dotnet/)                                                                                                                                       |
| [IBM.EntityFrameworkCore](https://www.nuget.org/packages/IBM.EntityFrameworkCore)                                    | Db2、Informix              | [IBM](https://ibm.com)                                                        | Windows 版本      | [博客](https://www.ibm.com/developerworks/community/blogs/96960515-2ea1-4391-8170-b0515d08e4da/entry/Creating_Entity_Data_Model_using_IBM_Data_Server_providers_for_Entity_Framework_Core?lang=en) |
| [IBM.EntityFrameworkCore-lnx](https://www.nuget.org/packages/IBM.EntityFrameworkCore-lnx)                            | Db2、Informix              | [IBM](https://ibm.com)                                                        | Linux 版本        | [博客](https://www.ibm.com/developerworks/community/blogs/96960515-2ea1-4391-8170-b0515d08e4da/entry/Creating_Entity_Data_Model_using_IBM_Data_Server_providers_for_Entity_Framework_Core?lang=en) |
| [IBM.EntityFrameworkCore-osx](https://www.nuget.org/packages/IBM.EntityFrameworkCore-osx)                            | Db2、Informix              | [IBM](https://ibm.com)                                                        | macOS 版本        | [博客](https://www.ibm.com/developerworks/community/blogs/96960515-2ea1-4391-8170-b0515d08e4da/entry/Creating_Entity_Data_Model_using_IBM_Data_Server_providers_for_Entity_Framework_Core?lang=en) |
| [EntityFrameworkCore.Jet](https://www.nuget.org/packages/EntityFrameworkCore.Jet/)                                   | Microsoft Access 文件     | [Bubi](https://github.com/bubibubi)                                           | .NET Framework       | [自述文件](https://github.com/bubibubi/EntityFrameworkCore.Jet/blob/master/docs/README.md)                                                                                                           |
| [EntityFrameworkCore.OpenEdge](https://www.nuget.org/packages/EntityFrameworkCore.OpenEdge/)                         | Progress OpenEdge          | [Alex Wiese](https://github.com/alexwiese)                                    |                      | [自述文件](https://github.com/alexwiese/EntityFrameworkCore.OpenEdge/blob/master/README.md)                                                                                                          |
| [Devart.Data.Oracle.EFCore](https://www.nuget.org/packages/Devart.Data.Oracle.EFCore/)                               | Oracle DB 9.2.0.4 及更高版本  | [DevArt](https://www.devart.com/)                                             | 已付                 | [docs](https://www.devart.com/dotconnect/oracle/docs/)                                                                                                                                             |
| [Devart.Data.PostgreSql.EFCore](https://www.nuget.org/packages/Devart.Data.PostgreSql.EFCore/)                       | PostgreSQL 8.0 及以上版本     | [DevArt](https://www.devart.com/)                                             | 已付                 | [docs](https://www.devart.com/dotconnect/postgresql/docs/)                                                                                                                                         |
| [Devart.Data.SQLite.EFCore](https://www.nuget.org/packages/Devart.Data.SQLite.EFCore/)                               | SQLite 3 及以上版本           | [DevArt](https://www.devart.com/)                                             | 已付                 | [docs](https://www.devart.com/dotconnect/sqlite/docs/)                                                                                                                                             |
| [Devart.Data.MySql.EFCore](https://www.nuget.org/packages/Devart.Data.MySql.EFCore/)                                 | MySQL 5 及以上版本            | [DevArt](https://www.devart.com/)                                             | 已付                 | [docs](https://www.devart.com/dotconnect/mysql/docs/)                                                                                                                                              |
| [FileContextCore](https://www.nuget.org/packages/FileContextCore/)                                 | 在文件中存储数据            | [Morris Janatzek](https://github.com/morrisjdev)                                             | 用于开发                 | [自述文件](https://github.com/morrisjdev/FileContextCore/blob/master/README.md)                                                                                                                                              |

## <a name="adding-a-database-provider-to-your-application"></a>向应用程序添加数据库提供程序

EF Core 的大多数数据库提供程序都是作为 NuGet 包分发的，可按如下所示安装：

## <a name="net-core-clitabdotnet-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

``` console
dotnet add package provider_package_name
```

## <a name="visual-studiotabvs"></a>[Visual Studio](#tab/vs)

``` powershell
install-package provider_package_name
```

***

安装后，需采用 `OnConfiguring` 方法或 `AddDbContext` 方法（如果使用的是依赖关系注入容器）在 `DbContext` 中配置提供程序。
例如，以下行使用传递的连接字符串配置 SQL Server 提供程序：

``` csharp
optionsBuilder.UseSqlServer(
    "Server=(localdb)\mssqllocaldb;Database=MyDatabase;Trusted_Connection=True;");
```  

数据库提供程序可扩展 EF Core，启用特定数据库特有的功能。
一些概念为大多数据库共有，它们包含于 EF Core 主要组件中。
此类概念包括 LINQ 中的表达查询、事务以及对象从数据库加载后的跟踪更改。
另一些概念特定于具体的提供程序。
例如，通过 SQL Server 提供程序可[配置内存优化表](xref:core/providers/sql-server/memory-optimized-tables)（SQL Server 的一种特定功能）。
其他一些概念特定于某一类提供程序。
例如，用于关系数据库的 EF Core 提供程序构建于公共 `Microsoft.EntityFrameworkCore.Relational` 库上，该库提供的 API 可用于配置表和列映射、外键约束等。提供程序通常作为 NuGet 包分发。

> [!IMPORTANT]  
> 发布 EF Core 的新补丁版本时，其中通常包含 `Microsoft.EntityFrameworkCore.Relational` 包的更新。
> 添加关系数据库提供程序时，该包将成为应用程序的传递依赖项。
> 但许多提供程序是独立于 EF Core 发布的，因此可能不会更新为依赖该包的较新补丁版本。
> 为确保能修复所有 bug，建议将 `Microsoft.EntityFrameworkCore.Relational` 补丁版本添加为应用程序的直接依赖项。
