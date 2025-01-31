---
title: Azure Cosmos DB Provider - EF Core
description: 数据库提供程序的文档，该提供程序允许将 Entity Framework Core 与 Azure Cosmos DB SQL API 一起使用
author: AndriySvyryd
ms.author: ansvyryd
ms.date: 11/05/2019
uid: core/providers/cosmos/index
ms.openlocfilehash: 6cac695288d9ba84968b7fab6361f55e9b51be67
ms.sourcegitcommit: 18ab4c349473d94b15b4ca977df12147db07b77f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2019
ms.locfileid: "73656094"
---
# <a name="ef-core-azure-cosmos-db-provider"></a>EF Core Azure Cosmos DB Provider

>[!NOTE]
> 此提供程序是 EF Core 3.0 新增内容。

此数据库提供程序允许将 Entity Framework Core 与 Azure Cosmos DB 一起使用。 该提供程序作为 [Entity Framework Core 项目](https://github.com/aspnet/EntityFrameworkCore)的组成部分进行维护。

在阅读本部分之前，强烈建议先熟悉 [Azure Cosmos DB 文档](/azure/cosmos-db/introduction)。

>[!NOTE]
> 此提供程序仅适用于 Azure Cosmos DB 的 SQL API。

## <a name="install"></a>安装

安装 [Microsoft.EntityFrameworkCore.Cosmos NuGet 包](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Cosmos/)。

## <a name="net-core-clitabdotnet-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

``` console
dotnet add package Microsoft.EntityFrameworkCore.Cosmos
```

## <a name="visual-studiotabvs"></a>[Visual Studio](#tab/vs)

``` powershell
Install-Package Microsoft.EntityFrameworkCore.Cosmos
```

***

## <a name="get-started"></a>入门

> [!TIP]  
> 可在 [GitHub 示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Cosmos)中查看此文章的示例。

与其他提供程序一样，第一步是调用 [UseCosmos](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosDbContextOptionsExtensions.UseCosmos)：

[!code-csharp[Configuration](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=Configuration)]

> [!WARNING]
> 为了简单起见，此处对终结点和密钥进行了硬编码，但在生产应用中，应[安全地存储](/aspnet/core/security/app-secrets#secret-manager)这些终结点和密钥

在此示例中，`Order` 是一个简单实体，其中包含对[从属类型](../../modeling/owned-entities.md) `StreetAddress` 的引用。

[!code-csharp[Order](../../../../samples/core/Cosmos/ModelBuilding/Order.cs?name=Order)]

[!code-csharp[StreetAddress](../../../../samples/core/Cosmos/ModelBuilding/StreetAddress.cs?name=StreetAddress)]

保存和查询数据遵循常规 EF 模式：

[!code-csharp[HelloCosmos](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?name=HelloCosmos)]

> [!IMPORTANT]
> 要创建所需的容器并插入[种子数据](../../modeling/data-seeding.md)（如果存在于模型中），则需要调用 [EnsureCreatedAsync](/dotnet/api/Microsoft.EntityFrameworkCore.Storage.IDatabaseCreator.EnsureCreatedAsync)。 但是只应在部署期间调用 `EnsureCreatedAsync`，而不应在正常操作中调用，否则可能会导致性能问题。

## <a name="cosmos-specific-model-customization"></a>特定于 Cosmos 的模型自定义

默认情况下，所有实体类型都映射到同一个容器，该容器以派生的上下文命名（在本例中为 `"OrderContext"`）。 要更改默认容器名称，请使用 [HasDefaultContainer](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosModelBuilderExtensions.HasDefaultContainer)：

[!code-csharp[DefaultContainer](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=DefaultContainer)]

要将实体类型映射到其他容器，请使用 [ToContainer](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosEntityTypeBuilderExtensions.ToContainer)：

[!code-csharp[Container](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=Container)]

为了标识给定项表示的实体类型，EF Core 添加鉴别器值（即使没有派生实体类型）。 [可以更改](../../modeling/inheritance.md)鉴别器的名称和值。

如果其他实体类型永远不会存储在同一个容器中，则可以通过调用 [HasNoDiscriminator](/dotnet/api/Microsoft.EntityFrameworkCore.Metadata.Builders.EntityTypeBuilder.HasNoDiscriminator) 删除鉴别器：

[!code-csharp[NoDiscriminator](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=NoDiscriminator)]

### <a name="partition-keys"></a>分区键

默认情况下，EF Core 将创建分区键设置为 `"__partitionKey"` 的容器，而不会在插入项时为其提供任何值。 但若要充分利用 Azure Cosmos 的性能功能，[应仔细选择应使用的分区键](/azure/cosmos-db/partition-data)。 可以通过调用 [HasPartitionKey](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosEntityTypeBuilderExtensions.HasPartitionKey)来配置它：

[!code-csharp[PartitionKey](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=PartitionKey)]

>[!NOTE]
>只要分区键属性[转换为字符串](xref:core/modeling/value-conversions)，则它可以为任意类型。

配置分区键属性后，应始终具有非 null 值。 发出查询时，可以添加条件将其设置为单分区。

[!code-csharp[PartitionKey](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?name=PartitionKey)]

## <a name="embedded-entities"></a>嵌入的实体

对于 Cosmos，从属实体嵌入到所有者所在的项中。 要更改属性名称，请使用 [ToJsonProperty](/dotnet/api/Microsoft.EntityFrameworkCore.CosmosEntityTypeBuilderExtensions.ToJsonProperty)：

[!code-csharp[PropertyNames](../../../../samples/core/Cosmos/ModelBuilding/OrderContext.cs?name=PropertyNames)]

对于此配置，以上示例中的顺序存储如下：

``` json
{
    "Id": 1,
    "PartitionKey": "1",
    "TrackingNumber": null,
    "id": "1",
    "Address": {
        "ShipsToCity": "London",
        "ShipsToStreet": "221 B Baker St"
    },
    "_rid": "6QEKAM+BOOABAAAAAAAAAA==",
    "_self": "dbs/6QEKAA==/colls/6QEKAM+BOOA=/docs/6QEKAM+BOOABAAAAAAAAAA==/",
    "_etag": "\"00000000-0000-0000-683c-692e763901d5\"",
    "_attachments": "attachments/",
    "_ts": 1568163674
}
```

还嵌入了从属实体的集合。 对于下一个示例，我们将使用具有 `StreetAddress` 集合的 `Distributor` 类：

[!code-csharp[Distributor](../../../../samples/core/Cosmos/ModelBuilding/Distributor.cs?name=Distributor)]

从属实体不需要提供要存储的显式键值：

[!code-csharp[OwnedCollection](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?name=OwnedCollection)]

它们将以这种方式持久保存：

``` json
{
    "Id": 1,
    "Discriminator": "Distributor",
    "id": "Distributor|1",
    "ShippingCenters": [
        {
            "City": "Phoenix",
            "Street": "500 S 48th Street"
        },
        {
            "City": "Anaheim",
            "Street": "5650 Dolly Ave"
        }
    ],
    "_rid": "6QEKANzISj0BAAAAAAAAAA==",
    "_self": "dbs/6QEKAA==/colls/6QEKANzISj0=/docs/6QEKANzISj0BAAAAAAAAAA==/",
    "_etag": "\"00000000-0000-0000-683c-7b2b439701d5\"",
    "_attachments": "attachments/",
    "_ts": 1568163705
}
```

在内部而言，EF Core 始终需要对所有被跟踪实体提供唯一键值。 默认情况下，为从属类型集合创建的主键包含指向所有者的外键属性和与 JSON 数组中的索引对应的 `int` 属性。 要检索这些值，可使用以下条目 API：

[!code-csharp[ImpliedProperties](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?name=ImpliedProperties)]

> [!TIP]
> 必要时，可更改从属实体类型的默认主键，但应显式提供键值。

## <a name="working-with-disconnected-entities"></a>使用断开连接的实体

每个项都需要具有一个对于给定分区键唯一的 `id` 值。 默认情况下 EF Core 通过使用 '|' 作为分隔符串联鉴别器和主键值来生成值。 仅当实体进入 `Added` 状态时才生成键值。 如果[附加实体](../../saving/disconnected-entities.md)在 .NET 类型上没有用于存储值的 `id` 属性，则这可能会导致问题。

要解决此限制，可以手动创建并设置 `id` 值，或者先将实体标记为已添加，然后将其更改为所需状态：

[!code-csharp[Attach](../../../../samples/core/Cosmos/ModelBuilding/Sample.cs?highlight=4&name=Attach)]

生成的 JSON 如下：

``` json
{
    "Id": 1,
    "Discriminator": "Distributor",
    "id": "Distributor|1",
    "ShippingCenters": [
        {
            "City": "Phoenix",
            "Street": "500 S 48th Street"
        }
    ],
    "_rid": "JBwtAN8oNYEBAAAAAAAAAA==",
    "_self": "dbs/JBwtAA==/colls/JBwtAN8oNYE=/docs/JBwtAN8oNYEBAAAAAAAAAA==/",
    "_etag": "\"00000000-0000-0000-9377-d7a1ae7c01d5\"",
    "_attachments": "attachments/",
    "_ts": 1572917100
}
```
