---
title: Code First 迁移在团队环境中-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 4c2d9a95-de6f-4e97-9738-c1f8043eff69
ms.openlocfilehash: b3c4c35d636caf4ddd251dd78e026587abc57d42
ms.sourcegitcommit: 708b18520321c587b2046ad2ea9fa7c48aeebfe5
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2019
ms.locfileid: "72182611"
---
# <a name="code-first-migrations-in-team-environments"></a>团队环境中的 Code First 迁移
> [!NOTE]
> 本文假设你知道如何在基本方案中使用 Code First 迁移。 如果没有，则需要先阅读[Code First 迁移](~/ef6/modeling/code-first/migrations/index.md)，然后再继续。

## <a name="grab-a-coffee-you-need-to-read-this-whole-article"></a>抓住咖啡，你需要阅读这篇文章

团队环境中的问题主要涉及两个开发人员在其本地基本代码中生成迁移时的合并。 虽然解决这些问题的步骤非常简单，但需要您对迁移的工作方式有深刻的了解。 请不要直接跳到结尾–请花时间阅读整篇文章，以确保成功。

## <a name="some-general-guidelines"></a>一些一般准则

在深入探讨如何管理由多个开发人员生成的合并迁移之前，下面是一些用于设置成功的一般准则。

### <a name="each-team-member-should-have-a-local-development-database"></a>每个团队成员都应具有一个本地开发数据库

迁移使用 **\_ @ no__t-2MigrationsHistory**表来存储已应用到数据库的迁移。 如果你有多个开发人员在尝试将同一数据库定向到目标时生成不同的迁移（因而共享了一个 **\_ @ no__t-2MigrationsHistory**表），则迁移将会感到困惑。

当然，如果您的团队成员不生成迁移，则不会有任何问题与中央开发数据库共享。

### <a name="avoid-automatic-migrations"></a>避免自动迁移

最后一行是，自动迁移最初在团队环境中看起来很好，但实际上它们不起作用。 如果希望了解原因，请继续阅读–否则，可以跳到下一部分。

自动迁移使你能够更新数据库架构以匹配当前模型，而无需生成代码文件（基于代码的迁移）。 如果你只使用了这些迁移，并且从未生成任何基于代码的迁移，则自动迁移将在团队环境中正常工作。 问题在于自动迁移是有限的，并且不处理多个操作–属性/列重命名、将数据移动到另一个表等。若要处理这些情况，您最终会生成基于代码的迁移（和编辑基架代码），这些迁移在自动迁移处理的更改之间混合使用。 这使得在两个开发人员签入迁移时，无法合并更改。

## <a name="screencasts"></a>屏幕广播

如果你想要观看 screencast 而不是阅读本文，以下两个视频会涵盖与本文相同的内容。

### <a name="video-one-migrations---under-the-hood"></a>视频一："迁移-在后台"

[此 screencast](https://channel9.msdn.com/blogs/ef/migrations-under-the-hood)介绍了迁移如何跟踪并使用有关模型的信息来检测模型更改。

### <a name="video-two-migrations---team-environments"></a>视频2："迁移-团队环境"

根据前一视频中的概念构建，[此 screencast](https://channel9.msdn.com/blogs/ef/migrations-team-environments)涵盖了团队环境中出现的问题以及如何解决这些问题。

## <a name="understanding-how-migrations-works"></a>了解迁移的工作方式

在团队环境中成功使用迁移的关键是要了解迁移如何跟踪并使用有关模型的信息来检测模型更改。

### <a name="the-first-migration"></a>第一次迁移

将首次迁移添加到项目时，会在程序包管理器控制台中首先运行类似于**添加迁移**的操作。 此命令执行的高级步骤如下图所示。

![首次迁移](~/ef6/media/firstmigration.png)

当前模型是从您的代码（1）计算得到的。 然后，模型将计算所需的数据库对象（2）-由于这是第一次迁移模型，因此模型不同只是使用空模型进行比较。 所需的更改会传递到代码生成器，以生成所需的迁移代码（3），然后将其添加到你的 Visual Studio 解决方案（4）。

除了存储在主代码文件中的实际迁移代码外，迁移还会生成一些附加的代码隐藏文件。 这些文件是迁移使用的元数据，而不是你应该编辑的内容。 其中一项文件是资源文件（.resx），其中包含在生成迁移时模型的快照。 下一步将介绍如何使用此方法。

此时，你可能会运行 "**更新数据库**" 以将更改应用到数据库，然后再开始实现应用程序的其他区域。

### <a name="subsequent-migrations"></a>后续迁移

稍后返回并对模型进行一些更改-在本示例中，我们会将**Url**属性添加到**博客**。 然后，将发出一个命令（如**添加迁移 AddUrl** ）基架迁移，以应用相应的数据库更改。 此命令执行的高级步骤如下图所示。

![第二次迁移](~/ef6/media/secondmigration.png)

与上一次一样，当前模型是从代码（1）计算而来的。 但是，这一次存在现有的迁移，因此从最新迁移（2）中检索上一个模型。 这两个模型将 diffed，以查找所需的数据库更改（3），然后该进程将像以前一样完成。

此过程用于添加到项目中的任何其他迁移。

### <a name="why-bother-with-the-model-snapshot"></a>为什么要干扰模型快照？

您可能想知道 EF 为什么要麻烦模型快照–为什么不只是查看数据库。 如果是这样，请继续阅读。 如果你不感兴趣，则可以跳过此部分。

EF 保留模型快照的原因有很多：

-   它允许您的数据库与 EF 模型的偏差。 可以直接在数据库中进行这些更改，也可以更改迁移中的基架代码以进行更改。 下面是其中几个示例：
    -   您希望向一个或多个表中添加插入的和更新的到列，但不希望在 EF 模型中包含这些列。 如果迁移过程中查看了数据库，则每次基架迁移时，它都会不断尝试删除这些列。 使用模型快照，EF 只会检测到对模型的合法更改。
    -   要更改用于更新的存储过程的正文，以包含一些日志记录。 如果迁移从数据库中查看此存储过程，它将继续尝试并将其重置回 EF 需要的定义。 通过使用模型快照，在 EF 模型中更改过程的形状时，EF 只会基架代码来更改存储过程。
    -   这些相同的原则适用于添加额外的索引，包括数据库中的额外表、将 EF 映射到表中的数据库视图等。
-   EF 模型只包含数据库的形状。 整个模型允许迁移查看有关模型中的属性和类的信息，以及它们如何映射到这些列和表。 此信息允许在基架的代码中更智能地迁移。 例如，如果您更改属性映射到迁移的列的名称，则可以通过查看它是相同的属性来检测重命名，如果只有数据库架构，则无法执行此操作。 

## <a name="what-causes-issues-in-team-environments"></a>导致团队环境中出现问题的原因

当你是处理应用程序的单个开发人员时，上一部分介绍的工作流非常有用。 如果您是更改模型的唯一人员，它在团队环境中也能正常工作。 在这种情况下，你可以进行模型更改，生成迁移并将它们提交到源控件。 其他开发人员可以同步您的更改并运行**更新数据库**，以应用架构更改。

当你有多个开发人员更改 EF 模型并同时提交到源代码管理中时，就会出现问题。 EF 缺乏哪一种方法是将本地迁移与其他开发人员自上次同步后已提交到源代码管理的迁移合并在一起。

## <a name="an-example-of-a-merge-conflict"></a>合并冲突的示例

首先，让我们看一看此类合并冲突的具体示例。 我们会继续学习前面所述的示例。 作为起点，假设先前部分中的更改已由原始开发人员签入。 我们将在两个开发人员更改代码库时进行跟踪。

我们将通过多个更改跟踪 EF 模型和迁移。 对于起始点，这两个开发人员已同步到源代码管理存储库，如下图所示。

![起点](~/ef6/media/startingpoint.png)

开发人员 \#1 和开发人员 @no__t 现在会在其本地代码库中对 EF 模型进行一些更改。 开发人员 \#1 将**分级**属性添加到**博客**–并生成**AddRating**迁移，以将更改应用到数据库。 开发 \#2 将**Readers 器**属性添加到**博客**–并生成相应的**AddReaders**迁移。 这两个开发人员都运行**更新数据库**，以将更改应用到本地数据库，然后继续开发应用程序。

> [!NOTE]
> 迁移以时间戳为前缀，因此我们的图形表示 AddReaders @no__t 从开发人员 @no__t 迁移到开发人员-11 之后的迁移。 开发人员 @no__t 为第01次还是 @no__t 12 生成迁移首先对团队中工作的问题没有任何区别，或将其合并的过程与下一节中介绍的过程不相同。

![本地更改](~/ef6/media/localchanges.png)

这是一个幸运的日子，开发人员 \#1，因为他们需要首先提交更改。 由于其他人在同步其存储库后未签入，因此他们只需提交其更改而无需执行任何合并。

![提交](~/ef6/media/submit.png)

现在，开发人员 \#2 提交。 它们不太幸运。 由于其他人已在同步后提交了更改，因此他们将需要下拉更改并进行合并。 源代码管理系统可能会自动将更改合并到代码级别，因为它们非常简单。 开发人员在同步后 @no__t 的本地存储库的状态如下图所示。 

![请求](~/ef6/media/pull.png)

在此阶段，开发人员 @no__t 可以运行**更新数据库**，它将检测新的**AddRating**迁移（尚未应用于开发人员 @no__t 32 的数据库）并应用该迁移。 现在，"**评级**" 列将添加到 "**博客**" 表中，并且数据库与模型同步。

但有几个问题：

1.  尽管**更新-数据库**将应用**AddRating**迁移，但它还会引发警告：*无法更新数据库以匹配当前模型，因为存在挂起的更改，并且已禁用自动迁移 。*
    问题在于，上一次迁移（**AddReader**）中存储的模型快照缺少**博客**上的 "**分级**" 属性（因为它不是生成迁移时模型的一部分）。 Code First 检测到上一次迁移中的模型与当前模型不匹配，并引发警告。
2.  运行应用程序会导致 InvalidOperationException，指出 @no__t "" Bloggingcontext "" 上下文在创建数据库后发生了更改。请考虑使用 Code First 迁移更新数据库 ... "*
    同样，问题在于上一次迁移中存储的模型快照与当前模型不匹配。
3.  最后，我们希望运行**添加迁移**现在会生成一个空迁移（因为没有要应用于数据库的更改）。 但是，因为迁移会将当前模型与上一次迁移（缺少**分级**属性）的模型进行比较，所以它将真正基架另一次**AddColumn**调用以添加到**分级**列中。 当然，此迁移将在**更新数据库**过程中失败，因为 "**分级**" 列已经存在。

## <a name="resolving-the-merge-conflict"></a>解决合并冲突

好消息是，如果你对迁移的工作方式有所了解，就不难手动处理合并了。 如果您跳过此部分， 抱歉，您需要返回本文的剩余部分！

有两个选项，最简单的方法是生成一个将正确的当前模型作为快照的空白迁移。 第二个选项是更新上一次迁移中的快照，使其具有正确的模型快照。 第二个选项更难，不能用于每个方案中，但它也更清晰，因为它不涉及添加额外的迁移。

### <a name="option-1-add-a-blank-merge-migration"></a>选项 1：添加空白的 "合并" 迁移

在此选项中，我们将仅生成一个空白迁移，目的是确保最新迁移中存储了正确的模型快照。

无论上次迁移的用户是谁，都可以使用此选项。 在本示例中，我们已遵循开发 \#2 正在处理合并，并且它们会生成上次迁移。 但是，如果开发人员 \#1 生成了上一次迁移，则可以使用这些相同的步骤。 如果涉及多个迁移，则这些步骤也适用–我们只需查看两个步骤，使其保持简单。

以下过程可用于此方法，从你认识到需要从源控件同步的更改开始。

1.  确保已将本地代码库中的任何挂起的模型更改写入迁移。 此步骤可确保在生成空白迁移时不会遗漏任何合法更改。
2.  与源代码管理同步。
3.  运行 "**更新-数据库**" 以应用其他开发人员已签入的任何新迁移。
    **_注意：_** *如果你没有从更新-数据库命令收到任何警告，则没有来自其他开发人员的新迁移，无需执行任何进一步的合并。*
4.  运行**添加迁移 &lt;pick @ no__t-2a @ no__t-3name @ no__t-4 – IgnoreChanges** （例如，**添加迁移合并– IgnoreChanges**）。 这会生成包含所有元数据（包括当前模型的快照）的迁移，但在将当前模型与上一次迁移中的快照进行比较时，会忽略它检测到的任何更改（这意味着你会获得一个空的**向上**和**向下**方法）。
5.  继续开发或提交到源代码管理（当然在运行单元测试后）。

下面是开发人员在使用此方法之后 \#2 的本地代码库的状态。

![合并迁移](~/ef6/media/mergemigration.png)

### <a name="option-2-update-the-model-snapshot-in-the-last-migration"></a>选项 2：在上一次迁移中更新模型快照

此选项与选项1非常相似，但会删除额外的空白迁移，因为让我们面对自己的解决方案，需要额外的代码文件。

**此方法仅在以下情况下可行：仅在本地代码库中存在最新的迁移，并且尚未提交到源控件（例如，如果执行合并的用户生成了最后一个迁移）** 。 编辑其他开发人员可能已应用于其开发数据库（甚至更糟）的迁移的元数据可能会导致意外的副作用。 在此过程中，我们将在本地数据库中回滚上一次迁移，并使用更新的元数据重新应用它。

虽然最后的迁移需要位于本地代码库中，但在此过程中，迁移的数量或顺序并没有限制。 可以从多个不同的开发人员进行多个迁移，相同的步骤同样适用–我们一直在寻找两个，使其保持简单。

以下过程可用于此方法，从你认识到需要从源控件同步的更改开始。

1.  确保已将本地代码库中的任何挂起的模型更改写入迁移。 此步骤可确保在生成空白迁移时不会遗漏任何合法更改。
2.  与源代码管理同步。
3.  运行 "**更新-数据库**" 以应用其他开发人员已签入的任何新迁移。
    **_注意：_** *如果你没有从更新-数据库命令收到任何警告，则没有来自其他开发人员的新迁移，无需执行任何进一步的合并。*
4.  运行**update-database – TargetMigration &lt;second @ no__t-2last @ no__t-3migration @ no__t-4** （在本示例中，我们将是**更新-数据库-TargetMigration AddRating**）。 这会将数据库恢复到第二次迁移的状态，这实际上是从数据库中 "取消应用" 最后一次迁移。
    **_纪录_** 需要 @no__t 0This 步骤才能安全地编辑迁移的元数据，因为元数据也存储在数据库的 \_ @ no__t-2MigrationsHistoryTable 中。这就是仅当最后一次迁移仅在本地代码库中时才应使用此选项的原因。如果其他数据库应用了上次迁移，则还必须将其回滚并重新应用上次迁移以更新元数据。 * 
5.  运行**add-迁移 &lt;full @ no__t-2name @ no__t-3including @ no__t-4timestamp @ no__t-5of @ no__t-6last @ no__t-7migration @ 201311062215252-no__ @-@no__t** **10AddReaders**）。
    **_纪录_** *需要包含时间戳，以便迁移知道要编辑现有迁移，而不是新的基架。*
    这将更新上一次迁移的元数据以匹配当前模型。 当命令完成时，你将收到以下警告，但这正是你所希望的。 "@no__t 0Only，用于迁移" 201311062215252 @ no__t-1AddReaders "的设计器代码已重新基架。若要重新基架整个迁移，请使用-Force 参数。 "*
6.  运行**Update-Database** ，以使用更新的元数据重新应用最新的迁移。
7.  继续开发或提交到源代码管理（当然在运行单元测试后）。

下面是开发人员在使用此方法之后 \#2 的本地代码库的状态。

![更新的元数据](~/ef6/media/updatedmetadata.png)

## <a name="summary"></a>总结

在团队环境中使用 Code First 迁移时，需要一些挑战。 不过，基本了解迁移的工作方式，以及解决合并冲突的一些简单方法，可以轻松地克服这些难题。

基本问题是最新迁移中存储的元数据不正确。 这会导致 Code First 错误检测到当前模型和数据库架构不匹配，也不会在下一次迁移中基架错误代码。 通过使用正确的模型生成空白迁移，或在最新的迁移中更新元数据，可以解决这种情况。
