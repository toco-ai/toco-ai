<div align="center">

[English](./README.md) · **日本語** · [简体中文](./README.zh-CN.md)

<img height="100" alt="TocoAI" src="assets/logo.png" />

<h1>TocoAI</h1>

<h3>DSL-Spec ベースのサーバーサイド Harness Engineering</h3>

[![][docs-shield]][docs-link]
[![][doc-license-shield]][doc-license-link]
[![][stack-shield]][stack-link]
[![][engine-shield]][engine-link]

<br/>

[![][github-stars-shield]][github-stars-link]
[![][github-issues-shield]][github-issues-link]
[![][github-contributors-shield]][github-contributors-link]


</div>

---

TocoAI はサーバーサイド開発のための **Harness Engineering** ソリューションです。Spec はコードを一度生成するためのテキストではなく、**システムの構造制御レイヤー**であるべきです。

- **DSL-Spec** によって大規模言語モデルの生成を制約し、コードと継続的に一貫性を保ちます。

- **モデリングエンジン**によって構造的コードを確定的にレンダリングし、コードベースを常に安定させドリフトを防ぎます。

長期的なイテレーションが必要なサーバーサイドプロジェクトにおいて、AI を常に人の設計意図のもとで動かします。

<video src="https://github.com/user-attachments/assets/9c3dab56-f927-42d6-83c0-77a25ff65ecc" controls width="100%"></video>

> [!TIP]
> 動くシステムを作りたいなら Cursor。長く維持できるシステムを作りたいなら TocoAI。

<details>
<summary><kbd>目次</kbd></summary>

- [🚀 クイックスタート](#-クイックスタート)
- [🏗️ Harness Engineering の考え方](#️-harness-engineering-の考え方)
- [⚙️ コアコンポーネント](#️-コアコンポーネント)
  - [📐 DSL-Spec](#-dsl-spec)
  - [🔧 モデリングエンジン](#-モデリングエンジン)
  - [🧭 Human-in-the-Loop](#-human-in-loop)
- [⚖️ 他ツールとの比較](#️-他ツールとの比較)
- [🎬 デモケース](#-デモケース)
- [📋 実際の事例](#-実際の事例)
- [🗺️ 適用シナリオと限界の考察](#️-適用シナリオと限界の考察)
- [🤝 コミュニティと参加](#-コミュニティと参加)

</details>

---

## 🚀 クイックスタート

- インストールと設定
  - [IntelliJ Plugin →][intellij-link]
  - [VS Code Plugin →][vscode-link]
- [DSL-Spec 文法リファレンス →][dsl-docs-link]

---

## 🏗️ Harness Engineering の考え方

サーバーサイドシステムは長期間の保守が必要であり、3D プリントではなく建物を建てることに近いです。私たちは**建築的思考**でシステムを構築します：

- 精密な設計図が必要 — そのため DDD と CQRS に準拠した **DSL-Spec** を定義しました
- 設計図ができたら、一貫性のある保守しやすいコード構造が必要 — そのため**モデリングエンジン**を開発し、構造的コードを確定的にレンダリングします。これは複雑なプロジェクトの約 **80%** を占めます
- 最後に、構造が堅固でインターフェースが整ったビルの中で、AI に内装作業（DSL-Spec では表現しにくい if/else のビジネスロジック）を担当させます

<div align="center">

<img src="assets/tocoai-arch.png" alt="TocoAI アーキテクチャ" width="100%"/>

</div>

---

## ⚙️ コアコンポーネント

### 📐 DSL-Spec

> *"形式的記号の使用は、あらゆるナンセンスを排除するための極めて効果的なツールである。自然言語の『自然さ』とは、荒唐無稽さが一見明らかでないことを容易に言える能力に過ぎない。"*
>
> — Edsger Dijkstra, EWD667, 1978

要件とアーキテクチャ設計は、構造化された DSL-Spec として統一的に表現され、システム全体の **Single Source of Truth** として機能します。DSL-Spec は人間が読め、機械が解析できます。設定ファイルではありません — **実行可能なアーキテクチャの意図**です。

DSL-Spec はサーバーサイドシステムの完全な設計階層をカバーします：

| レイヤー | 要素 |
|---------|------|
| **ドメインモデル** | Entity / Relation / Enum / EO (Value Object) |
| **集約** | BO / ドメインイベント |
| **データ転送** | DTO / VO |
| **クエリプラン** | ReadPlan |
| **書き込みプラン** | WritePlan (Based on BO) |
| **フロープラン** | FuncFlow（タスクオーケストレーション）/ メッセージ管理 |
| **サービスインターフェース** | API / RPC / Service |

<br/>

1 つの DTO の DSL-Spec 定義から自動生成されるファイル：

<table>
<tr>
<th width="22%">曖昧な要件</th>
<th width="50%">DSL-Spec</th>
<th width="28%">自動生成ファイル</th>
</tr>
<tr>
<td>

*"客室タイプの情報（対応する客室リストを含む）— さらに利用可能な客室数も含む。"*

| 曖昧な点 | DSL での決定 |
|---|---|
| ルートエンティティはどれ？ | `fromEntity: room_type` |
| 客室との関連は？ | `room.room_type_id` FK → **逆方向注入** |
| `available_count` の出所は？ | エンティティにないフィールド → `customField` |

</td>
<td>

```json
{
  "dto": {
    "name": "room_type_with_rooms_dto",
    "fromEntity": "room_type",
    "reverseExpandList": [{
      "foreignKeyInOtherEntity": "room_type_id",
      "dtoFieldName": "room_list",
      "dto": { "name": "room_base_dto", "fromEntity": "room" }
    }],
    "customFieldList": [{
      "name": "available_count",
      "type": "Integer"
    }]
  }
}
```

</td>
<td>

```
RoomTypeWithRoomsDto.java                (~60 Lines)
RoomTypeWithRoomsDtoManager.java         (~25 Lines)
RoomTypeWithRoomsDtoManagerImpl.java
RoomTypeWithRoomsDtoConverter.java       (~80 Lines)
RoomTypeWithRoomsDtoService.java         (~70 Lines)
RoomTypeWithRoomsDtoDataAssembler.java
RoomTypeWithRoomsDtoBaseDataAssembler.java
```

</td>
</tr>
</table>

[DSL-Spec 完全文法ドキュメントを見る →][dsl-docs-link]

<br/>

#### DSL-Spec：開発システムの Ontology

従来の開発では、テスト・CI/CD・変更管理はそれぞれ異なるデータソースに依存していました — テストは開発者のコード理解に依存し、CI はコンパイルとテスト実行しか検証できず、変更管理は手動の影響範囲追跡に頼っています。これらのプロセスはシステムを真に「理解」しているのではなく、コードという事実が生まれた後に受動的に追随しているだけです。DSL-Spec の核心的価値は、開発パイプライン全体の唯一の Ontology（本体）になることにあります：システムの構造・ルール・関係が Spec に一度定義され、すべてのダウンストリームプロセスが同じ真実から派生し、それぞれが独自の現実の近似を維持する必要がなくなります。

1. **テスト**：Spec に明示的に宣言されたビジネスルール（フィールド制約・列挙型制限など）が直接テストアサーションのソースになり、コードからの逆推論に依存しません。各 Function Flow ノードは Context オブジェクトを通じて明確な入出力契約を定義し、フルコールチェーンをモックせずにノードごとの独立したユニットテストが書けます。エンジンが生成する 80% の構造的コードの正確性はエンジン自体が保証するため、テストリソースを本当にビジネスリスクのある 20% に集中できます。

2. **CI/CD**：パイプラインに Spec コンプライアンスチェックを組み込めます — コードベースの生成コードが現在の Spec と完全一致するかを検証し、エンジンをバイパスした手動編集があれば CI が即座にブロック、メカニズムレベルで技術的負債を排除します。モデルフィールドや関連関係の変更時、エンジンが同期的に DDL マイグレーションスクリプトを出力し、CI がスキーマ変更とコードデプロイを順序どおりに強制バインド実行して、データベースとコードが同期しないウィンドウを排除します。API シグネチャの変更はエンジンが自動的に diff し、CI が breaking change かどうかを判定してインターフェースバージョンアップグレードフローをトリガーできます。

3. **変更管理**：1 つの集約フィールドを変更すると、エンジンが影響を受けるすべての Service・DTO・API・データベーステーブルを自動計算してリストアップ — 影響範囲が Spec から直接推導され、開発者の経験に依存しません。変更レビューはコード diff ではなく Spec diff を見ることになり、非技術系のステークホルダーも直接参加できます。各 Spec コミットが構造化された変更記録を自動生成し、「誰が何を変更し、どのモジュールに影響したか」を完全に追跡でき、コンプライアンス監査の証跡として直接使用できます。

<br/>

|  | 自然言語 Spec | プログラム的 IaC | **DSL-Spec** |
|--|:------------:|:--------------:|:------------:|
| **明確な意味、可読性** | 誰でも読めるが、曖昧さはエラーにならず本番障害として現れる | 正確だが、意図を理解するには実行フローの理解が必要 | ✅ 自然言語のように読めて、コードのように正確 — 曖昧さはコンパイルエラー |
| **詳細が明確（What + How）** | 曖昧な What のみ — How は AI の即興に任せる | How のみ — アーキテクチャの意図が実行ロジックに埋もれる | ✅ What（データの意図）と How（構造的制約）が両方明示的に表現される |
| **検証可能、保守しやすい** | 機械検証不可；フィールド変更の影響範囲を手動で追跡する必要がある | 実行可能だが独立した意図レイヤーがない；変更の影響は分析ツールが必要 | ✅ 機械解析・検証可能；DSL-Spec を変更するとエンジンが影響する全構造を自動カスケード同期 |
| **常にコードと一致** | 起動時は正確、3 ヶ月でドリフト、6 ヶ月で歴史文書になる | コードが実装だが、アーキテクチャの意図はイテレーションで失われる | ✅ Spec とコードの関係は常に `=`、`≈` ではない |
| **開発全パイプラインの Ontology 価値** | Ontology として機能できない — テスト・CI・変更管理がそれぞれ独自のシステム近似を維持 | 実行レイヤーの Ontology として機能できるが、意図レイヤーが欠如しており、下流プロセスが設計意図を検証しにくい | ✅ 統一 Ontology — テストアサーション・DDL 変更・影響範囲分析すべてが同じ真実から派生 |

<br/>

### 🔧 モデリングエンジン

エンジンはすべての構造的コードの生成をカバーします：

- 階層スケルトン（`persist` / `manager` / `service` / `entrance`）
- インターフェース契約とデータモデル
- CQRS コマンド/クエリ分離
- クロスレイヤーコンバーター（`DtoConverter` / `VoConverter`）
- データアセンブラー（`DataAssembler`）
- 集約書き込みチェーン（`BoService`）

**開発者は残りの 20% のビジネスロジックだけに集中すればよい** — レビューコストが桁違いに下がります。

> [!NOTE]
> モデリングエンジンの本質は、ソフトウェアエンジニアリングの品質管理を**「設計時」に左シフト**することです — DSL-Spec で定義されたコードフレームワークを確定的にレンダリング生成します。構造的コードはエンジンが生成し、LLM がイテレーションごとに生成するのではありません。これにより AI のランダムなエラーとアーキテクチャドリフトを根本的に排除します。何度イテレーションしても、何人が関わっても、基盤構造は常に設計と一致し、時間の経過とともに腐敗しません。

> [!IMPORTANT]
> エンジンは **2026 年下半期**にオープンソース化される予定です。

<br/>

### 🧭 Human-in-the-Loop

AI は全知ではありません。システムに長期的な影響を与える決定 — ドメイン境界、データ構造定義、インターフェース記述、読み書きプラン（トランザクション制約、操作パフォーマンス）— については、AI 支援の修正と手動制御を組み合わせた標準的なアプローチを提供します。

**AI を常にアシスタントに留め、意思決定者にさせない。**

---

## ⚖️ 他ツールとの比較

Cursor と Claude Code は優れた汎用コーディングアシスタントです — 実際、TocoAI 内部でもビジネスロジックの実装に使用しています。私たちは異なるレベルの問題を解決します。

> [!NOTE]
> TocoAI の設計対象：**リレーショナルデータベース駆動のサーバーサイドシステムで、長期的なイテレーションと多人数の協業が必要なもの**。プロトタイプ、スクリプトツール、フロントエンドプロジェクトを作っているなら Cursor で十分です。

|  | Cursor / Claude Code | TocoAI |
|--|:--------------------:|:------:|
| **役割** | 汎用対話式コーディングアシスタント | サーバーサイドエンジニアリング Harness、特定シナリオの構造化ソリューション |
| **コードの出所** | LLM がプロンプトに基づいて生成 | DSL-Spec → エンジンによる確定的生成（80%）+ LLM によるビジネスロジック |
| **アーキテクチャ一貫性** | プロンプトとレビューで維持 | DSL-Spec + エンジンで保証、人間依存なし |
| **最適フェーズ** | 迅速なプロトタイピング、断片的なコーディング | 長期保守が必要な複雑なビジネスシステム |
| **チーム規模** | 個人または小規模チームに最適 | チームが大きく、イテレーションが長いほど優位性が増す |
| **学習コスト** | ほぼなし | DSL-Spec とモデリングアプローチの理解が必要 |

---

## 🎬 デモケース

### BnB 民泊予約プラットフォーム

物件管理、ショッピングカート、注文決済、在庫検証、会員ポイントをカバーする完全な民泊予約プラットフォーム — TocoAI の要件から納品までのエンドツーエンドのワークフローをデモンストレーションします。

**ビジネスカバレッジ：** 客室在庫 &nbsp;·&nbsp; カート &nbsp;·&nbsp; 注文 / サブ注文 &nbsp;·&nbsp; 支払い &nbsp;·&nbsp; ポイント控除 &nbsp;·&nbsp; 消費明細

[プロセス例を見る →][bnb-demo-link] · [プロジェクトコード →][bnb-code-link]

---

## 📋 実際の事例

### 大規模病院 HIS システム

**背景：** 次世代全院管理システム、120+ コアモジュール、200+ 業務フロー、マルチチーム並列開発。

**課題：** 医療業務はゼロエラー許容；多人数の時間をまたいだ協業でロジックギャップと技術的負債が生じやすい。

**結果：** アーキテクチャ標準が一貫して適用され、生成コードの精度が大幅に向上；プロジェクト全体の AI コード採用率は **97%** 近く；引き継ぎ時に新メンバーがプロジェクト全体を迅速に把握できる。

[事例の詳細を見る →][case-his-link]

<br/>

### 金融証拠金決済システムのリファクタリング

**背景：** レガシーな証拠金決済システムのリファクタリング。元の技術スタックは SQLServer 2008 + 大量のストアドプロシージャ + 複数のサードパーティミドルウェア。

**課題：** 業務フローが分散；データとビジネスの関係が不明確；パフォーマンスボトルネックが顕著。

**結果：** DSL-Spec で再モデリングし、ビジネス関係を明示化；エンドツーエンドのフローが追跡可能；システムは TocoAI フレームワークでのイテレーションを継続できる。

---

## 🗺️ 適用シナリオと限界の考察

すべての DSL は **domain-specific** です。特定のドメイン内でのみ有効であり、すべてをカバーできる汎用 DSL は存在しません — 無理に汎用化すると、別のプログラミング言語を作ることになります。TocoAI の DSL-Spec はリレーショナルデータベース駆動のサーバーサイドシステムのみを記述します。これは意図的な境界であり、能力の制限ではありません。

単一ドメイン内でも、DSL-Spec はプログラミングコードを置き換えるものではありません。私たちの境界は**構造化記述に適した情報を DSL-Spec 化する**ことであり、残りのビジネスロジックはプログラミング言語と AI に委ねます。

**レガシープロジェクトの互換性：** レガシープロジェクトに新しいモジュールを開発することはできますが、既存のレガシーコードをすべて引き継ぐことはスコープ外です。正しいアプローチは、新しい要件を新しいモジュールで処理し、古いロジックを徐々に DSL-Spec の管轄範囲に移行することです — **リファクタリングは AI 時代の日常業務**です。

Harness Engineering のアプローチ自体は汎用的です。組み込みシステム、フロントエンドコンポーネント、インフラ設定など、他のチームが各自のドメインで拡張することを期待しています。

---

## 🤝 コミュニティと参加

- [tocoai.jp][docs-link] で完全なドキュメントを参照
- [Discord コミュニティ][discord-link]に参加して質問、機能の議論、実践の共有
- [GitHub Issues][github-issues-link] でバグ報告や機能提案を提出
- モデリングエンジンは **2026 年下半期**にオープンソース化予定 — Star でウォッチ

完璧な技術はなく、適したシナリオがあるだけです。AI が開発を変えてまだ 1 年ですが、ソフトウェア開発自体には 70〜80 年の歴史があります。

[コントリビューションガイド →](CONTRIBUTING.md)

---

<div align="center">

Copyright © 2025 TocoAI. ドキュメントは [CC BY 4.0][doc-license-link] ライセンスの下で公開されています。

</div>

<!-- LINK GROUP -->
[docs-shield]: https://img.shields.io/badge/Docs-tocoai.jp-brightgreen?style=flat-square&color=73DC8C&labelColor=black
[docs-link]: https://tocoai.jp/docs

[license-shield]: https://img.shields.io/badge/License-Apache_2.0-blue?style=flat-square&color=4B78E6&labelColor=black
[license-link]: LICENSE

[doc-license-shield]: https://img.shields.io/badge/ドキュメント-CC%20BY%204.0-lightgrey?style=flat-square&color=a78bfa&labelColor=black
[doc-license-link]: https://creativecommons.org/licenses/by/4.0/

[stack-shield]: https://img.shields.io/badge/Stack-Java_|_Spring_Boot-orange?style=flat-square&color=ffcb47&labelColor=black
[stack-link]: https://tocoai.cn

[engine-shield]: https://img.shields.io/badge/Engine_OSS-H2_2026-pink?style=flat-square&color=FA9BFA&labelColor=black
[engine-link]: https://tocoai.cn/docs/engine

[github-stars-shield]: https://img.shields.io/github/stars/toco-ai/toco-ai?style=flat-square&color=ffcb47&labelColor=black&logo=github
[github-stars-link]: https://github.com/toco-ai/toco-ai/stargazers

[github-issues-shield]: https://img.shields.io/github/issues/toco-ai/toco-ai?style=flat-square&color=ff80eb&labelColor=black&logo=github
[github-issues-link]: https://github.com/toco-ai/toco-ai/issues

[github-contributors-shield]: https://img.shields.io/github/contributors/toco-ai/toco-ai?style=flat-square&color=c4f042&labelColor=black&logo=github
[github-contributors-link]: https://github.com/toco-ai/toco-ai/graphs/contributors

[intellij-link]: https://tocoai.jp/docs/installation
[vscode-link]: https://tocoai.jp/docs/installation-vscode
[dsl-docs-link]: ./assets/dsl.md
[engine-docs-link]: https://tocoai.cn/docs/engine
[bnb-demo-link]: https://tocoai.jp/docs/your-first-toco-project
[bnb-code-link]: https://github.com/toco-ai/homestay
[case-his-link]: https://tocoai.cn/cases/his
[case-finance-link]: https://tocoai.cn/cases/finance
[discord-link]: https://discord.gg/NubsdbF3MK
