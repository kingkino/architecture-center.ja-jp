---
title: マイクロサービスの設計パターン
description: 堅牢なマイクロサービス アーキテクチャを実装する設計パターン。
author: MikeWasson
ms.date: 02/25/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 8cdbf95c753770910fa4a94384c9809db9fda746
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243573"
---
# <a name="design-patterns-for-microservices"></a>マイクロサービスの設計パターン

マイクロサービスの目標は、個別にデプロイできる小さい自律的なサービスにアプリケーションを分解して、アプリケーションのリリース速度を上げることです。 マイクロサービス アーキテクチャには、いくつかの課題もあります。 ここに示す設計パターンは、それらの課題の緩和に役立ちます。

![マイクロサービスの設計パターン](../images/microservices-patterns.png)

[**アンバサダー**](../../patterns/ambassador.md)は、監視、ログ記録、ルーティング、セキュリティ (TLS など) といった一般的なクライアント接続のタスクを言語に関係ない方法でオフロードするのに役立ちます。 アンバサダー サービスは多くの場合、サイドカー (下記参照) としてデプロイされます。

[**破損対策レイヤー** ](../../patterns/anti-corruption-layer.md)は、新しいアプリケーションの設計がレガシ システムへの依存によって確実に制限されないようにするため、新しいアプリケーションとレガシ アプリケーションの間にファサードを実装します。

[**フロントエンド用バックエンド**](../../patterns/backends-for-frontends.md)は、デスクトップやモバイルなど、クライアントのさまざまな種類に応じて独立したバックエンド サービスを作成します。 そうすれば、さまざまな種類のクライアントの競合する要件を単一のバックエンド サービスで処理する必要がありません。 このパターンを使用すると、クライアント固有の問題を切り離すことによって、各マイクロサービスのシンプルさを維持できます。

[**バルクヘッド**](../../patterns/bulkhead.md)は、接続プール、メモリ、CPU などの重要なリソースをワークロードまたはサービスごとに独立させます。 バルクヘッドを使用すると、単一のワークロード (またはサービス) がすべてのリソースを消費して他のワークロードのリソースが枯渇することがなくなります。 このパターンでは、1 つのサービスによって発生した障害の連鎖を防ぐことによって、システムの回復性が向上します。

[**ゲートウェイ集約**](../../patterns/gateway-aggregation.md)では、複数の個々のマイクロサービスへの要求を単一の要求に集約し、コンシューマーとサービスの間のトラフィックを削減します。

[**ゲートウェイ オフロード**](../../patterns/gateway-offloading.md)では、SSL 証明書の使用などの共有サービス機能を各マイクロサービスから API ゲートウェイにオフロードすることができます。

[**ゲートウェイ ルーティング**](../../patterns/gateway-routing.md)では、単一のエンドポイントを使用して要求を複数のマイクロサービスにルーティングします。これにより、コンシューマーは多数の個別エンドポイントを管理する必要がありません。

[**サイドカー**](../../patterns/sidecar.md)では、アプリケーションのヘルパー コンポーネントを別のコンテナーまたはプロセスとしてデプロイし、分離性とカプセル化を実現します。

[**ストラングラー**](../../patterns/strangler.md)は、機能の特定の部分を新しいサービスに徐々に置き換えることで、アプリケーションの段階的なリファクタリングをサポートします。

Azure アーキテクチャ センターでクラウド設計パターンの完全なカタログを見るには、[クラウド設計パターン](../../patterns/index.md)に関するページを参照してください。
