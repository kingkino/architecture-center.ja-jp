---
title: マイクロサービスのデータに関する考慮事項
description: マイクロサービスのデータに関する考慮事項。
author: MikeWasson
ms.date: 02/25/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: dd620cdcb6cb3a06d4fae4fb34eeb086ec17b4c5
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241523"
---
# <a name="data-considerations-for-microservices"></a><span data-ttu-id="f75ea-103">マイクロサービスのデータに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="f75ea-103">Data considerations for microservices</span></span>

<span data-ttu-id="f75ea-104">この記事では、マイクロサービス アーキテクチャでデータを管理するための考慮事項を説明します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-104">This article describes considerations for managing data in a microservices architecture.</span></span> <span data-ttu-id="f75ea-105">それぞれのマイクロサービスが自らのデータを管理するため、データの整合性とデータの一貫性が重要な課題です。</span><span class="sxs-lookup"><span data-stu-id="f75ea-105">Because every microservice manages its own data, data integrity and data consistency are critical challenges.</span></span>

<span data-ttu-id="f75ea-106">マイクロサービスの基本原則は、各サービスがそれぞれ自身のデータを管理することです。</span><span class="sxs-lookup"><span data-stu-id="f75ea-106">A basic principle of microservices is that each service manages its own data.</span></span> <span data-ttu-id="f75ea-107">2 つのサービスはデータ ストアを共有しません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-107">Two services should not share a data store.</span></span> <span data-ttu-id="f75ea-108">代わりに、各サービスは自身のプライベート データ ストアに責任を持ち、他のサービスがそのデータ ストアに直接アクセスすることはできません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-108">Instead, each service is responsible for its own private data store, which other services cannot access directly.</span></span>

<span data-ttu-id="f75ea-109">このルールの理由は、サービス同士の意図しない結合を回避するためです。サービスが基礎となるデータ スキーマを共有する場合はサービスの結合が発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-109">The reason for this rule is to avoid unintentional coupling between services, which can result if services share the same underlying data schemas.</span></span> <span data-ttu-id="f75ea-110">データ スキーマに対する変更がある場合、そのデータベースに依存するすべてのサービス間でその変更を調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-110">If there is a change to the data schema, the change must be coordinated across every service that relies on that database.</span></span> <span data-ttu-id="f75ea-111">各サービスのデータ ストアを分離させることにより、変更の範囲をg限定し、完全に独立したデプロイの機敏性を維持できます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-111">By isolating each service's data store, we can limit the scope of change, and preserve the agility of truly independent deployments.</span></span> <span data-ttu-id="f75ea-112">もう 1 つの理由は、各マイクロサービスが独自のデータ モデル、クエリ、読み取り/書き込みパターンを持つことができるようにするためです。</span><span class="sxs-lookup"><span data-stu-id="f75ea-112">Another reason is that each microservice may have its own data models, queries, or read/write patterns.</span></span> <span data-ttu-id="f75ea-113">共有データ ストアを使用すると、各チームによる特定のサービス用のデータ ストレージの最適化が制限されます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-113">Using a shared data store limits each team's ability to optimize data storage for their particular service.</span></span>

![CQRS に対する間違ったアプローチの図](../../guide/architecture-styles/images/cqrs-microservices-wrong.png)

<span data-ttu-id="f75ea-115">このアプローチは、[多言語パーシステンス](https://martinfowler.com/bliki/PolyglotPersistence.html) (単一アプリケーションで複数のデータ ストレージ テクノロジを使用すること) に自然につながります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-115">This approach naturally leads to [polyglot persistence](https://martinfowler.com/bliki/PolyglotPersistence.html) &mdash; the use of multiple data storage technologies within a single application.</span></span> <span data-ttu-id="f75ea-116">あるサービスでは、ドキュメント データベースのスキーマオンリード機能が必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-116">One service might require the schema-on-read capabilities of a document database.</span></span> <span data-ttu-id="f75ea-117">別のサービスでは、RDBMS で提供される参照整合性が必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-117">Another might need the referential integrity provided by an RDBMS.</span></span> <span data-ttu-id="f75ea-118">各チームはそれぞれのサービスに最適な選択を自由に行うことができます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-118">Each team is free to make the best choice for their service.</span></span> <span data-ttu-id="f75ea-119">多言語パーシステンスの原則について詳しくは、「[ジョブに最適なデータ ストアの使用](../../guide/design-principles/use-the-best-data-store.md)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="f75ea-119">For more about the general principle of polyglot persistence, see [Use the best data store for the job](../../guide/design-principles/use-the-best-data-store.md).</span></span>

> [!NOTE]
> <span data-ttu-id="f75ea-120">複数のサービスが同じ物理データベース サーバーを共有することは問題ありません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-120">It's fine for services to share the same physical database server.</span></span> <span data-ttu-id="f75ea-121">問題が発生するのは、サービスが同じスキーマを共有するとき、すなわちデータベース テーブルの同じセットに対して読み取りや書き込みを行うときです。</span><span class="sxs-lookup"><span data-stu-id="f75ea-121">The problem occurs when services share the same schema, or read and write to the same set of database tables.</span></span>

## <a name="challenges"></a><span data-ttu-id="f75ea-122">課題</span><span class="sxs-lookup"><span data-stu-id="f75ea-122">Challenges</span></span>

<span data-ttu-id="f75ea-123">データの管理に対するこのような分散アプローチからいくつかの課題が生じています。</span><span class="sxs-lookup"><span data-stu-id="f75ea-123">Some challenges arise from this distributed approach to managing data.</span></span> <span data-ttu-id="f75ea-124">まず、複数の場所に同一アイテムのデータが出現し、データ ストア間で冗長性が存在する場合があります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-124">First, there may be redundancy across the data stores, with the same item of data appearing in multiple places.</span></span> <span data-ttu-id="f75ea-125">たとえば、データがトランザクションの一部として格納され、さらに分析、レポート、またはアーカイブのために別の場所に格納される場合があります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-125">For example, data might be stored as part of a transaction, then stored elsewhere for analytics, reporting, or archiving.</span></span> <span data-ttu-id="f75ea-126">重複したデータつまりパーティション分割されたデータが、データの整合性や一貫性の問題を引き起こす可能性があります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-126">Duplicated or partitioned data can lead to issues of data integrity and consistency.</span></span> <span data-ttu-id="f75ea-127">データ リレーションシップが複数のサービスにまたがる場合、従来のデータ管理方法を使用してそのリレーションシップを適用することはできません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-127">When data relationships span multiple services, you can't use traditional data management techniques to enforce the relationships.</span></span>

<span data-ttu-id="f75ea-128">従来のデータ モデルでは "1 箇所に 1 つのファクト" というルールが使用されます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-128">Traditional data modeling uses the rule of "one fact in one place."</span></span> <span data-ttu-id="f75ea-129">どのエンティティもスキーマ内で 1 回だけ出現します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-129">Every entity appears exactly once in the schema.</span></span> <span data-ttu-id="f75ea-130">他のエンティティがそのエンティティに対する参照を保持することはできますが、複製することはできません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-130">Other entities may hold references to it but not duplicate it.</span></span> <span data-ttu-id="f75ea-131">従来のアプローチの明らかな利点は、更新が 1 箇所で行われるためデータ一貫性に関する問題を回避できることです。</span><span class="sxs-lookup"><span data-stu-id="f75ea-131">The obvious advantage to the traditional approach is that updates are made in a single place, which avoids problems with data consistency.</span></span> <span data-ttu-id="f75ea-132">マイクロサービス アーキテクチャでは、サービス間で更新を伝播する方法や、データが複数の場所に存在するために一貫性が十分でない場合に最終的な一貫性を管理する方法を考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-132">In a microservices architecture, you have to consider how updates are propagated across services, and how to manage eventual consistency when data appears in multiple places without strong consistency.</span></span>

## <a name="approaches-to-managing-data"></a><span data-ttu-id="f75ea-133">データ管理のアプローチ</span><span class="sxs-lookup"><span data-stu-id="f75ea-133">Approaches to managing data</span></span>

<span data-ttu-id="f75ea-134">すべてのケースで正解になる 1 つのアプローチはありませんが、マイクロサービス アーキテクチャでデータを管理するための一般的なガイドラインをいくつか説明します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-134">There is no single approach that's correct in all cases, but here are some general guidelines for managing data in a microservices architecture.</span></span>

- <span data-ttu-id="f75ea-135">可能であれば、最終的な整合性を優先します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-135">Embrace eventual consistency where possible.</span></span> <span data-ttu-id="f75ea-136">システム内で、強力な一貫性すなわち ACID トランザクションを必要とする場所と、最終的な一貫性が許容される場所を理解します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-136">Understand the places in the system where you need strong consistency or ACID transactions, and the places where eventual consistency is acceptable.</span></span>

- <span data-ttu-id="f75ea-137">強力な一貫性が必要な場合には、1 つのサービスが所定のエンティティの真のソースを表すことができます。これは API を介して公開されます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-137">When you need strong consistency guarantees, one service may represent the source of truth for a given entity, which is exposed through an API.</span></span> <span data-ttu-id="f75ea-138">その他のサービスは、データまたはデータのサブセットのコピーを独自に保持する可能性があります。これらは、最終的にマスター データと一貫性を持つようになりますが、真のソースとは見なされません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-138">Other services might hold their own copy of the data, or a subset of the data, that is eventually consistent with the master data but not considered the source of truth.</span></span> <span data-ttu-id="f75ea-139">たとえば、顧客発注サービスと推奨サービスを含む eコマース システムを想像してください。</span><span class="sxs-lookup"><span data-stu-id="f75ea-139">For example, imagine an e-commerce system with a customer order service and a recommendation service.</span></span> <span data-ttu-id="f75ea-140">推奨サービスは発注サービスのイベントをリッスンしますが、顧客が払い戻しを要求した場合に、完全なトランザクション履歴を持っているのは推奨サービスではなく発注サービスです。</span><span class="sxs-lookup"><span data-stu-id="f75ea-140">The recommendation service might listen to events from the order service, but if a customer requests a refund, it is the order service, not the recommendation service, that has the complete transaction history.</span></span>

- <span data-ttu-id="f75ea-141">トランザクションの場合は、[Scheduler Agent Supervisor](../../patterns/scheduler-agent-supervisor.md) や[補正トランザクション](../../patterns/compensating-transaction.md)などのパターンを使用して、複数のサービス間でデータ一貫性を保ちます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-141">For transactions, use patterns such as [Scheduler Agent Supervisor](../../patterns/scheduler-agent-supervisor.md) and [Compensating Transaction](../../patterns/compensating-transaction.md) to keep data consistent across several services.</span></span>  <span data-ttu-id="f75ea-142">複数のサービス間での部分的な失敗を回避するため、場合によっては、複数のサービスにまたがる作業単位の状態を取得する追加データを格納する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-142">You may need to store an additional piece of data that captures the state of a unit of work that spans multiple services, to avoid partial failure among multiple services.</span></span> <span data-ttu-id="f75ea-143">たとえば、複数ステップのトランザクションが進行している間、永続的なキューに作業項目を保持します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-143">For example, keep a work item on a durable queue while a multi-step transaction is in progress.</span></span>

- <span data-ttu-id="f75ea-144">サービスに必要なデータだけを格納します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-144">Store only the data that a service needs.</span></span> <span data-ttu-id="f75ea-145">サービスでは、ドメイン エンティティに関する情報のサブセットのみが必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-145">A service might only need a subset of information about a domain entity.</span></span> <span data-ttu-id="f75ea-146">たとえば、配送のコンテキスト境界では、特定の配送にどの顧客が関連付けられているかを知る必要があります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-146">For example, in the Shipping bounded context, we need to know which customer is associated to a particular delivery.</span></span> <span data-ttu-id="f75ea-147">ただし、顧客の請求先住所は必要がありません。これはアカウントのコンテキスト境界によって管理されます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-147">But we don't need the customer's billing address &mdash; that's managed by the Accounts bounded context.</span></span> <span data-ttu-id="f75ea-148">ドメインについて注意深く考えることと DDD アプローチを使用することが、ここでは役立ちます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-148">Thinking carefully about the domain, and using a DDD approach, can help here.</span></span>

- <span data-ttu-id="f75ea-149">サービスが明確であり、ゆるやかに結合かれているかどうかを考慮します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-149">Consider whether your services are coherent and loosely coupled.</span></span> <span data-ttu-id="f75ea-150">2 つのサービスが互いに情報の交換を継続し、API の使用頻度が高い場合は、2 つのサービスをマージするか、それらのサービスの機能をリファクタリングして、サービスの境界を設定し直す必要があるかもしれません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-150">If two services are continually exchanging information with each other, resulting in chatty APIs, you may need to redraw your service boundaries, by merging two services or refactoring their functionality.</span></span>

- <span data-ttu-id="f75ea-151">[イベント ドリブン アーキテクチャのスタイル](../../guide/architecture-styles/event-driven.md)を使用します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-151">Use an [event driven architecture style](../../guide/architecture-styles/event-driven.md).</span></span> <span data-ttu-id="f75ea-152">このアーキテクチャ スタイルでは、サービスがイベントを発行するのは、サービスのパブリック モデルまたはエンティティに変更がある場合です。</span><span class="sxs-lookup"><span data-stu-id="f75ea-152">In this architecture style, a service publishes an event when there are changes to its public models or entities.</span></span> <span data-ttu-id="f75ea-153">関心を持つサービスがそのようなイベントをサブスクライブできます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-153">Interested services can subscribe to these events.</span></span> <span data-ttu-id="f75ea-154">たとえば、別のサービスがイベントを使用して、データの具体化されたビュー (クエリに適している) を構築できます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-154">For example, another service could use the events to construct a materialized view of the data that is more suitable for querying.</span></span>

- <span data-ttu-id="f75ea-155">イベントを所有するサービスはスキーマを公開する必要があります。これはイベントのシリアル化やシリアル化解除の自動化に使用でき、パブリッシャーとサブスクライバーの密接な結合が回避されます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-155">A service that owns events should publish a schema that can be used to automate serializing and deserializing the events, to avoid tight coupling between publishers and subscribers.</span></span> <span data-ttu-id="f75ea-156">JSON スキーマ、または [Microsoft Bond](https://github.com/Microsoft/bond)、Protobuf、Avro などのフレームワークを検討してください。</span><span class="sxs-lookup"><span data-stu-id="f75ea-156">Consider JSON schema or a framework like [Microsoft Bond](https://github.com/Microsoft/bond), Protobuf, or Avro.</span></span>

- <span data-ttu-id="f75ea-157">大きな規模で見ると、イベントはシステムのボトルネックになることもあります。集計やバッチ処理を使用して、全体的な負荷を下げることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="f75ea-157">At high scale, events can become a bottleneck on the system, so consider using aggregation or batching to reduce the total load.</span></span>

## <a name="example-choosing-data-stores-for-the-drone-delivery-application"></a><span data-ttu-id="f75ea-158">例:ドローン配送アプリケーションのデータ ストアを選択する</span><span class="sxs-lookup"><span data-stu-id="f75ea-158">Example: Choosing data stores for the Drone Delivery application</span></span>

<span data-ttu-id="f75ea-159">このシリーズのこれまでの記事では、継続的な例としてドローン配送サービスについて説明しています。</span><span class="sxs-lookup"><span data-stu-id="f75ea-159">The previous articles in this series discuss a drone delivery service as a running example.</span></span> <span data-ttu-id="f75ea-160">シナリオおよび対応する参照実装の詳細については、[こちら](./index.md)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="f75ea-160">You can read more about the scenario and the corresponding reference implementation [here](./index.md).</span></span>

<span data-ttu-id="f75ea-161">要約すると、このアプリケーションでは、ドローンによる配送をスケジュールするための複数のマイクロサービスを定義します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-161">To recap, this application defines several microservices for scheduling deliveries by drone.</span></span> <span data-ttu-id="f75ea-162">ユーザーが新しい配送をスケジュール設定すると、クライアント要求には、配送 (集荷場所や引き渡し場所) とパッケージ (サイズや重量) に関する情報が含まれます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-162">When a user schedules a new delivery, the client request includes information about the delivery, such as pickup and dropoff locations, and about the package, such as size and weight.</span></span> <span data-ttu-id="f75ea-163">この情報によって作業単位が定義されます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-163">This information defines a unit of work.</span></span>

<span data-ttu-id="f75ea-164">さまざまなバックエンド サービスは、要求の情報の異なる部分に関心を持ち、読み取りと書き込みのプロファイルも異なっています。</span><span class="sxs-lookup"><span data-stu-id="f75ea-164">The various backend services care about different portions of the information in the request, and also have different read and write profiles.</span></span>

![データに関する考慮事項の図](../images/data-considerations.png)

### <a name="delivery-service"></a><span data-ttu-id="f75ea-166">配送サービス</span><span class="sxs-lookup"><span data-stu-id="f75ea-166">Delivery service</span></span>

<span data-ttu-id="f75ea-167">配送サービスは、現在スケジュールされている配送または進行中の配送すべてについての情報を格納します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-167">The Delivery service stores information about every delivery that is currently scheduled or in progress.</span></span> <span data-ttu-id="f75ea-168">ドローンからのイベントをリッスンし、進行中の配送の状態を追跡します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-168">It listens for events from the drones, and tracks the status of deliveries that are in progress.</span></span> <span data-ttu-id="f75ea-169">配送状態の更新を含むドメイン イベントの送信も行います。</span><span class="sxs-lookup"><span data-stu-id="f75ea-169">It also sends domain events with delivery status updates.</span></span>

<span data-ttu-id="f75ea-170">ユーザーはパッケージを待っている間に配送の状態を頻繁に確認すると考えられます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-170">It's expected that users will frequently check the status of a delivery while they are waiting for their package.</span></span> <span data-ttu-id="f75ea-171">そのため、配送サービスでは、長期的な格納よりもスループット (読み取りおよび書き込み) を重視したデータ ストアが必要です。</span><span class="sxs-lookup"><span data-stu-id="f75ea-171">Therefore, the Delivery service requires a data store that emphasizes throughput (read and write) over long-term storage.</span></span> <span data-ttu-id="f75ea-172">また、配送サービスは複雑なクエリや分析は実行しません。所定の配送に関する最新状態をフェッチするだけです。</span><span class="sxs-lookup"><span data-stu-id="f75ea-172">Also, the Delivery service does not perform any complex queries or analysis, it simply fetches the latest status for a given delivery.</span></span> <span data-ttu-id="f75ea-173">配信サービス チームは、読み取りと書き込みの性能の高さから Azure Redis Cache を選択しました。</span><span class="sxs-lookup"><span data-stu-id="f75ea-173">The Delivery service team chose Azure Redis Cache for its high read-write performance.</span></span> <span data-ttu-id="f75ea-174">Redis に格納される情報の存続期間は比較的短めです。</span><span class="sxs-lookup"><span data-stu-id="f75ea-174">The information stored in Redis is relatively short-lived.</span></span> <span data-ttu-id="f75ea-175">配信が完了すると、配送履歴サービスがレコードのシステムになります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-175">Once a delivery is complete, the Delivery History service is the system of record.</span></span>

### <a name="delivery-history-service"></a><span data-ttu-id="f75ea-176">配送履歴サービス</span><span class="sxs-lookup"><span data-stu-id="f75ea-176">Delivery History service</span></span>

<span data-ttu-id="f75ea-177">配送履歴サービスは、配送サービスからの配送状態イベントをリッスンします。</span><span class="sxs-lookup"><span data-stu-id="f75ea-177">The Delivery History service listens for delivery status events from the Delivery service.</span></span> <span data-ttu-id="f75ea-178">そのデータを長期的なストレージに格納します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-178">It stores this data in long-term storage.</span></span> <span data-ttu-id="f75ea-179">この履歴データには 2 つの異なるユースケースがあり、データ記憶域の要件も異なります。</span><span class="sxs-lookup"><span data-stu-id="f75ea-179">There are two different use-cases for this historical data, which have different data storage requirements.</span></span>

<span data-ttu-id="f75ea-180">最初のシナリオは、業務の最適化またはサービス品質の向上を目的とした、データ分析のためのデータの集計です。</span><span class="sxs-lookup"><span data-stu-id="f75ea-180">The first scenario is aggregating the data for the purpose of data analytics, in order to optimize the business or improve the quality of the service.</span></span> <span data-ttu-id="f75ea-181">配送履歴サービスは実際のデータ分析を実行しないことに注意してください。</span><span class="sxs-lookup"><span data-stu-id="f75ea-181">Note that the Delivery History service doesn't perform the actual analysis of the data.</span></span> <span data-ttu-id="f75ea-182">インジェストと格納のみを担当します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-182">It's only responsible for the ingestion and storage.</span></span> <span data-ttu-id="f75ea-183">このシナリオでは、ストレージは大容量のデータ セット用ではなくデータ分析用に最適化する必要があります。スキーマオンリード アプローチを使用して、さまざまなデータ ソースを格納するようにします。</span><span class="sxs-lookup"><span data-stu-id="f75ea-183">For this scenario, the storage must be optimized for data analysis over a large set of data, using a schema-on-read approach to accommodate a variety of data sources.</span></span> <span data-ttu-id="f75ea-184">[Azure Data Lake Store](/azure/data-lake-store/) はこのシナリオに最適です。</span><span class="sxs-lookup"><span data-stu-id="f75ea-184">[Azure Data Lake Store](/azure/data-lake-store/) is a good fit for this scenario.</span></span> <span data-ttu-id="f75ea-185">Data Lake Store は、Hadoop 分散ファイル システム (HDFS) と互換性のある Apache Hadoop ファイル システムであり、データ分析シナリオに適したパフォーマンスにチューニングされています。</span><span class="sxs-lookup"><span data-stu-id="f75ea-185">Data Lake Store is an Apache Hadoop file system compatible with Hadoop Distributed File System (HDFS), and is tuned for performance for data analytics scenarios.</span></span>

<span data-ttu-id="f75ea-186">もう 1 つのシナリオは、配送の完了後にユーザーが配送履歴を調べられるようにすることです。</span><span class="sxs-lookup"><span data-stu-id="f75ea-186">The other scenario is enabling users to look up the history of a delivery after the delivery is completed.</span></span> <span data-ttu-id="f75ea-187">Azure Data Lake はこのシナリオに特に適しているわけではありません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-187">Azure Data Lake is not particularly optimized for this scenario.</span></span> <span data-ttu-id="f75ea-188">最適なパフォーマンスを得るには、Data Lake において日付でパーティション分割されたフォルダーに時系列データを格納することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="f75ea-188">For optimal performance, Microsoft recommends storing time-series data in Data Lake in folders partitioned by date.</span></span> <span data-ttu-id="f75ea-189">(「[Azure Data Lake Store のパフォーマンス チューニング](/azure/data-lake-store/data-lake-store-performance-tuning-guidance)」をご覧ください)。</span><span class="sxs-lookup"><span data-stu-id="f75ea-189">(See [Tuning Azure Data Lake Store for performance](/azure/data-lake-store/data-lake-store-performance-tuning-guidance)).</span></span> <span data-ttu-id="f75ea-190">ただし、この構造は個々のレコードを ID で検索するには向いていません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-190">However, that structure is not optimal for looking up individual records by ID.</span></span> <span data-ttu-id="f75ea-191">タイムスタンプもわかっているのではない限り、ID による検索ではコレクション全体のスキャンが必要です。</span><span class="sxs-lookup"><span data-stu-id="f75ea-191">Unless you also know the timestamp, a lookup by ID requires scanning the entire collection.</span></span> <span data-ttu-id="f75ea-192">そのため、配送履歴サービスは迅速な検索のために履歴データのサブセットを Cosmos DB にも格納します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-192">Therefore, the Delivery History service also stores a subset of the historical data in Cosmos DB for quicker lookup.</span></span> <span data-ttu-id="f75ea-193">レコードを Cosmos DB に無期限に保存する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-193">The records don't need to stay in Cosmos DB indefinitely.</span></span> <span data-ttu-id="f75ea-194">たとえば、1 か月以上前の古い配送レコードはアーカイブできます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-194">Older deliveries can be archived &mdash; say, after a month.</span></span> <span data-ttu-id="f75ea-195">これはバッチ プロセスを不定期に実行して処理できます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-195">This could be done by running an occasional batch process.</span></span>

### <a name="package-service"></a><span data-ttu-id="f75ea-196">パッケージ サービス</span><span class="sxs-lookup"><span data-stu-id="f75ea-196">Package service</span></span>

<span data-ttu-id="f75ea-197">パッケージ サービスは、すべてのパッケージに関する情報を格納します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-197">The Package service stores information about all of the packages.</span></span> <span data-ttu-id="f75ea-198">パッケージのストレージの要件は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="f75ea-198">The storage requirements for the Package are:</span></span>

- <span data-ttu-id="f75ea-199">長期ストレージ。</span><span class="sxs-lookup"><span data-stu-id="f75ea-199">Long-term storage.</span></span>
- <span data-ttu-id="f75ea-200">多数のパッケージを処理できるように、高い書き込みスループットが必要です。</span><span class="sxs-lookup"><span data-stu-id="f75ea-200">Able to handle a high volume of packages, requiring high write throughput.</span></span>
- <span data-ttu-id="f75ea-201">パッケージ ID による単純なクエリに対応します。</span><span class="sxs-lookup"><span data-stu-id="f75ea-201">Support simple queries by package ID.</span></span> <span data-ttu-id="f75ea-202">複雑な結合はなく、参照整合性も必要ありません。</span><span class="sxs-lookup"><span data-stu-id="f75ea-202">No complex joins or requirements for referential integrity.</span></span>

<span data-ttu-id="f75ea-203">パッケージのデータはリレーショナルではないため、ドキュメント指向データベースが適切です。Cosmos DB はシャード コレクションを使用して非常に高いスループットを実現できます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-203">Because the package data is not relational, a document oriented database is appropriate, and Cosmos DB can achieve very high throughput by using sharded collections.</span></span> <span data-ttu-id="f75ea-204">パッケージ サービスに取り組むチームは MEAN スタック (MongoDB、Express.js、AngularJS、Node.js) に詳しいため、Cosmos DB 対応の [MongoDB API](/azure/cosmos-db/mongodb-introduction) を選択しました。</span><span class="sxs-lookup"><span data-stu-id="f75ea-204">The team that works on the Package service is familiar with the MEAN stack (MongoDB, Express.js, AngularJS, and Node.js), so they select the [MongoDB API](/azure/cosmos-db/mongodb-introduction) for Cosmos DB.</span></span> <span data-ttu-id="f75ea-205">これにより、MongoDB での経験を活用しながら、管理型 Azure サービスである Cosmos DB のメリットを得ることができます。</span><span class="sxs-lookup"><span data-stu-id="f75ea-205">That lets them leverage their existing experience with MongoDB, while getting the benefits of Cosmos DB, which is a managed Azure service.</span></span>