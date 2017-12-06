---
title: Priority Queue
description: "サービスに送信される要求に優先順位を設定し、優先順位の高い要求から順番に受信および処理されるようにします。"
keywords: "設計パターン"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- messaging
- performance-scalability
ms.openlocfilehash: ecfbb38304bb95587e9ca15523ad9594898d9b32
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="priority-queue-pattern"></a>Priority Queue パターン

[!INCLUDE [header](../_includes/header.md)]

サービスに送信される要求に優先順位を設定し、優先順位の高い要求から順番に受信および処理されるようにします。 このパターンは、個々のクライアントにそれぞれ異なるサービス レベルを保証するアプリケーションにおいて有用です。

## <a name="context-and-problem"></a>コンテキストと問題

アプリケーションは、特定のタスクを他のサービスに委任して、バック グラウンド処理の実行や他のアプリケーションまたはサービスとの統合などを行うことができます。 クラウドでは通常、タスクをバック グラウンド処理に委任するためにメッセージ キューを使用しています。 多くの場合に、サービスが要求を受信する順番は重要ではありません。 しかし、場合によっては、特定の要求を優先する必要があります。 そのような要求は、アプリケーションによって先に送信された優先順位の低い要求よりも早く処理する必要があります。

## <a name="solution"></a>解決策

キューは一般に先入れ先出し (FIFO) 構造になっているため、コンシューマーは通常、キューにポストされた順にメッセージを受信することになります。 ただし、一部のメッセージ キューでは、優先順位メッセージングをサポートします。 メッセージをポストするアプリケーションは、優先順位を割り当てることができます。これにより、キュー内のメッセージは自動的に並べ替えられ、優先順位の高い順に受信されるようになります。 図に優先順位メッセージングを使用したキューを示します。

![図 1 - メッセージの優先順位付けをサポートするキュー メカニズムの使用](./_images/priority-queue-pattern.png)

> ほとんどのメッセージ キューの実装では複数のコンシューマーをサポートし ([競合コンシューマー パターン](https://msdn.microsoft.com/library/dn568101.aspx) に従って)、要求に応じてコンシューマー プロセスの数をスケールアップまたはスケールダウンします。

優先順位に基づくメッセージ キューをサポートしないシステムでは、代替ソリューションとして、優先順位ごとに個別のキューを維持します。 メッセージを適切なキューに送信するのはアプリケーションの役割です。 各キューはそれぞれ異なるコンシューマー プールを備えることができます。 優先順位の高いキューほどコンシューマー プールのサイズは大きく、より高速のハードウェアで実行されます。 次の図に、優先順位ごとに別々のメッセージ キューを使用する方法を示します。

![図 2 - 優先順位ごとに別々のメッセージ キューを使用](./_images/priority-queue-separate.png)


この方法の変形型として、コンシューマー プールを 1 つだけ備え、まず優先順位の高いキューにメッセージがないか確認し、その後で優先順位の低いキューからメッセージの取得を開始するという方法があります。 単一のコンシューマー プールを使用して処理するソリューション (さまざまな優先順位のメッセージをサポートする単一キューを使用する場合、または複数のキューを使用しそれぞれが単一の優先順位を持つメッセージを処理する場合) と、複数のキューを使用しキューごとに別々のプールを備えるソリューションとの間にはセマンティックの差異があります。

単一プール アプローチでは、常に優先度の高い順にメッセージが受信され処理されます。 理論上、優先順位がとても低いメッセージは、繰り返し置き換えられ、いつまでたっても処理されない可能性があります。 複数プール アプローチでは、優先順位の低いメッセージも、優先順位の高いメッセージほど早くはないにしても (プールのサイズの相対関係と、利用可能なリソースに依存する)、必ず処理されます。

優先順位キュー メカニズムを使用すると、次の利点が得られます。

- 特定の顧客グループに異なるレベルのサービスを提供するなど、アプリケーションで、可用性またはパフォーマンスの優先順位付けを必要とするビジネス要件を満たすことができます。

- 運用コストを最小限に抑えるのに役立ちます。 単一キュー アプローチでは、必要に応じてコンシューマーの数を減らすことができます。 優先順位の高いメッセージはやはり最初に処理されます (遅くなる可能性はありますが)。しかし、優先順位の低いメッセージは待ち時間がさらに長くなる場合があります。 キューごとに別々のコンシューマー プールを使用する複数メッセージ キュー アプローチを実装している場合は、優先順位の低いキュー用のコンシューマー プールを削減することも、優先順位がとても低いキューでメッセージを待機しているすべてのコンシューマーを停止してそれらのキューに対する処理を中断することもできます。

- 複数メッセージ キュー アプローチでは、処理要件に基づいてメッセージを分割することにより、アプリケーションのパフォーマンスとスケーラビリティを最大限に高めることができます。 たとえば、重要なタスクは、すぐに実行する受信側によって処理されるように優先順位を設定できます。一方、重要度の低いバック グラウンド タスクは、稼働率の低い期間に実行するようにスケジュールされた受信側で処理することができます。

## <a name="issues-and-considerations"></a>問題と注意事項

このパターンの実装方法を決めるときには、以下の点に注意してください。

ソリューションのコンテキストで、優先順位を定義します。 たとえば、高い優先順位の場合は、メッセージを 10 秒以内に処理する必要があるものとします。 優先度の高い項目を処理するための要件、および、これらの条件を満たすために割り当てる必要があるその他のリソースを明らかにします。

優先順位の高い項目から順番に処理していく必要があるかどうかを決定します。 単一のコンシューマー プールでメッセージを処理する場合、メッセージの処理中に、より優先順位の高いメッセージが有効になったら、優先順位の低い方のメッセージを処理しているタスクを置き換えて中断するメカニズムを備える必要があります。

複数キュー アプローチにおいて、キューごとに専用のコンシューマー プールを使用するのでなく、すべてのキューで待機するコンシューマー プロセスの単一プールを使用する場合、コンシューマーは優先順位の低いキューからのメッセージよりも優先順位の高いキューからのメッセージを必ず先に処理するアルゴリズムを適用する必要があります。

優先順位の高いキューおよび優先順位の低いキューで処理速度を監視し、これらのキューにあるメッセージが期待どおりのレートで処理されるようにします。

優先順位の低いメッセージも処理されるように保証する必要がある場合は、複数のコンシューマー プールを使用して複数メッセージ キュー アプローチを実装する必要があります。 あるいは、メッセージの優先順位設定をサポートするキューでは、キューに置かれたメッセージの優先順位をその年齢に応じて動的に上げることができます。 ただし、このアプローチは、この機能を提供するメッセージ キューによって異なります。

メッセージの優先順位ごとに別々のキューを使用する方法は、明確に定義された優先順位の数が少ないシステムで最適に機能します。

メッセージの優先順位は、システムによって論理的に決定できます。 たとえば、メッセージの優先順位の高い低いを明示的に指定しているのでなく、"料金を払っている顧客" または "料金を払っていない顧客" として指定している場合が考えられます。 ビジネス モデルによっては、システムは、"料金を払っていない顧客" よりも "料金を払っている顧客" からのメッセージの処理に多くのリソースを割り当てる可能性があります。

キューにメッセージが置かれているかどうかの確認に関連して財務および処理費用が発生する場合があります (商用システムの中にはメッセージのポストまたは検索が行われるたびに、またキューに対してメッセージの照会が行われるたびに少額の料金を請求するものがあります)。 複数のキューをチェックする場合は、コストが増えます。

プールで処理するキューの長さに基づいてコンシューマー プールのサイズを動的に調整することができます。 詳細については、「[自動スケール ガイダンス](https://msdn.microsoft.com/library/dn589774.aspx)」を参照してください。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

このパターンは、次のシナリオで役立ちます。

- システムでは、優先順位が異なる複数のタスクを処理する必要があります。

- 優先順位の異なるさまざまなユーザーまたはテナントに対応する必要があります。

## <a name="example"></a>例

Microsoft Azure では、並べ替えによるメッセージの自動的な優先順位付けをネイティブでサポートするキュー メカニズムは提供していません。 しかしながら、Microsoft Azure では Azure Service Bus トピックおよびサブスクリプションを提供しています。これにより、メッセージ フィルター機能を備えたキュー メカニズムと共に、大部分の優先順位キュー実装での使用に適した柔軟性の高い各種機能がサポートされています。

Azure ソリューションでは、アプリケーションがキューと同じ方法でメッセージをポストできる Service Bus トピックを実装できます。 メッセージには、アプリケーションで定義されたカスタム プロパティの形式でメタデータを含めることができます。 Service Bus サブスクリプションはトピックに関連付けることができます。このようなサブスクリプションは、そのプロパティに基づいてメッセージをフィルター処理できます。 アプリケーションがトピックにメッセージを送信すると、そのメッセージは、コンシューマーがメッセージを読み取り可能な適切なサブスクリプションに送信されます。 コンシューマー プロセスでは、メッセージ キュー (サブスクリプションは論理キュー) と同じセマンティクスを使用してサブスクリプションからメッセージを取得できます。 次の図に、Azure Service Bus トピックとサブスクリプションを使用した優先順位キューの実装を示します。

![図 3 - Azure Service Bus トピックとサブスクリプションを使用した優先順位キューの実装](./_images/priority-queue-service-bus.png)


上図で、アプリケーションは複数のメッセージを作成し、各メッセージ内に `High` または `Low` のいずれかの値を持つ `Priority` と呼ばれるカスタム プロパティを割り当てています。 アプリケーションは、これらのメッセージをトピックにポストします。 トピックには 2 つの関連するサブスクリプションがあります。両方とも、`Priority` プロパティを確認してメッセージをフィルター処理します。 1 つのサブスクリプションは、`Priority` プロパティが `High` に設定されたメッセージを受け入れ、もう 1 つのサブスクリプションは `Priority` プロパティが `Low` に設定されたメッセージを受け入れます。 コンシューマー プールは、各サブスクリプションからメッセージを読み取ります。 優先順位の高いサブスクリプションには、より大きなプールが用意されます。これらのコンシューマーの場合は、優先順位の低いプールのコンシューマーに比べて、より多くのリソースが使用可能なより強力なコンピューターで実行されます。

この例では、優先順位の高いメッセージと優先順位の低いメッセージの指定に関して特筆すべき点はありません。 それらは各メッセージ内のプロパティに従って指定された単なるラベルであり、特定のサブスクリプションにメッセージを送信するために使用されます。 追加の優先順位が必要な場合は、これらのプロパティを処理するためのコンシューマー プロセスのプールを比較的簡単に作成できます。

[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue) で入手可能な PriorityQueue ソリューションには、このアプローチの実装が含まれています。 このソリューションには、`PriorityQueue.High` および `PriorityQueue.Low` という名前の 2 つの worker ロール プロジェクトが含まれています。 これらの worker ロールは、`OnStart` メソッド内の指定されたサブスクリプションに接続するための機能を含む `PriorityWorkerRole` クラスを継承します。

`PriorityQueue.High` worker ロールと `PriorityQueue.Low` worker ロールは、それぞれの構成設定によって定義された別々のサブスクリプションに接続します。 管理者は、実行する各ロールの数をそれぞれ異なる値に構成できます。 通常、`PriorityQueue.Low` worker ロールのインスタンスよりも `PriorityQueue.High` worker ロールのインスタンスの方が多くなります。

`PriorityWorkerRole` クラス内の `Run` メソッドは、仮想的な `ProcessMessage` メソッド (`PriorityWorkerRole` クラスでも定義されている) がキューで受信した各メッセージに対して実行されるように配置されます。 次のコードは、`Run` メソッドと `ProcessMessage` メソッドを示します。 PriorityQueue.Shared プロジェクトで定義された `QueueManager` クラスは、Azure Service Bus キューを使用するためのヘルパー メソッドを提供します。

```csharp
public class PriorityWorkerRole : RoleEntryPoint
{
  private QueueManager queueManager;
  ...

  public override void Run()
  {
    // Start listening for messages on the subscription.
    var subscriptionName = CloudConfigurationManager.GetSetting("SubscriptionName");
    this.queueManager.ReceiveMessages(subscriptionName, this.ProcessMessage);
    ...;
  }
  ...

  protected virtual async Task ProcessMessage(BrokeredMessage message)
  {
    // Simulating processing.
    await Task.Delay(TimeSpan.FromSeconds(2));
  }
}
```
`PriorityQueue.High` worker ロールと `PriorityQueue.Low` worker ロールは両方とも、`ProcessMessage` メソッドの既定の機能をオーバーライドします。 次のコードは、`PriorityQueue.High` worker ロール `ProcessMessage` メソッドを示しています。

```csharp
protected override async Task ProcessMessage(BrokeredMessage message)
{
  // Simulate message processing for High priority messages.
  await base.ProcessMessage(message);
  Trace.TraceInformation("High priority message processed by " +
    RoleEnvironment.CurrentRoleInstance.Id + " MessageId: " + message.MessageId);
}
```

`PriorityQueue.High` worker ロールと `PriorityQueue.Low` worker ロールで使用されるサブスクリプションに関連付けられたトピックにアプリケーションがメッセージをポストする場合、アプリケーションは、次のコード例に示すように、`Priority` カスタム プロパティを使用して優先順位を指定します。 このコード (PriorityQueue.Sender プロジェクト内の `WorkerRole` クラスに実装されている) は、`QueueManager` クラスの `SendBatchAsync` ヘルパー メソッドを使用して、メッセージをバッチでトピックにポストします。

```csharp
// Send a low priority batch.
var lowMessages = new List<BrokeredMessage>();

for (int i = 0; i < 10; i++)
{
  var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
  message.Properties["Priority"] = Priority.Low;
  lowMessages.Add(message);
}

this.queueManager.SendBatchAsync(lowMessages).Wait();
...

// Send a high priority batch.
var highMessages = new List<BrokeredMessage>();

for (int i = 0; i < 10; i++)
{
  var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
  message.Properties["Priority"] = Priority.High;
  highMessages.Add(message);
}

this.queueManager.SendBatchAsync(highMessages).Wait();
```

## <a name="related-patterns-and-guidance"></a>関連のあるパターンとガイダンス

このパターンを実装する場合は、次のパターンとガイダンスも関連している可能性があります。

- このパターンを示すサンプルは [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue) から入手できます。

- [非同期メッセージングの基本](https://msdn.microsoft.com/library/dn589781.aspx)。 要求を処理するコンシューマー サービスは、要求をポストしたアプリケーションのインスタンスに応答を送信することが必要な場合があります。 要求/応答メッセージングを実装する場合に使用する方法に関する情報を提供します。

- [競合コンシューマー パターン](competing-consumers.md)。 キューのスループットを高めるには、同一のキューで待機する複数のコンシューマーを使用し、タスクを並列に処理します。 これらのコンシューマーはメッセージに対して競合し、各メッセージを処理できるのは 1 つのコンシューマーだけです。 このアプローチを実装することでもたらされる利点と発生するトレードオフの詳細について説明します。

- [スロットル パターン](throttling.md)。 キューを使用してスロットルを実装することができます。 優先順位メッセージングを使用すると、重要なアプリケーション (または重要な顧客によって実行されているアプリケーション) からの要求に対する優先順位を重要度が低いアプリケーションからの要求に対する優先順位より高くするすることができます。

- [自動スケール ガイダンス](https://msdn.microsoft.com/library/dn589774.aspx)。 キューの長さに応じてキューを処理するコンシューマー プロセスのプールのサイズは、スケールを変更できる場合があります。 この方法は、特に優先度の高いメッセージを処理するプールにおいて、パフォーマンスを向上するのに役立ちます。

- Abhishek Lal のブログにある「[Service Bus を使用したEnterprise Integration パターン](http://abhishekrlal.com/2013/01/11/enterprise-integration-patterns-with-service-bus-part-2/)」。
