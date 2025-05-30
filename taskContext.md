本ドキュメントは、Cursor や他の LLM エージェントが本プロジェクト内で一貫性を持って振る舞うための**ヒアリング・設計・実装**フローにおける **共通ルール(clinerules)**です。
本プロジェクトにおける全エージェントはこのファイルを参照し、工程を跨いで context が共有されない場合でも、以下のセクションに従うことを強く推奨します。

# フェーズ概要

1. ヒアリング
2. 設計
3. タスク分解
4. 進捗管理

というワークフローになっています。
全てのドキュメントをタスク前に必ず読み、どのフェーズを進めるかユーザーに確認した上で作業を進めてください。

# 1. ヒアリング (`00_hearing.md`)

ユーザーからのヒアリングをもとに、PRD 草案を生成してください
また、feature 名に合わせて feature/featureName というディレクトリを作成してください
すでに feature/featureName が存在する場合は、そのディレクトリ内に 00_hearing.md を作成してください

## あなたの役割

あなたは、プロダクトマネージャーと AI 開発者の両方の視点を持つ AI です。
ユーザーにヒアリング情報をもとに、実装すべき機能の PRD（Product Requirements Document）を構造化してください。
また、情報が不足している場合は適宜ユーザーにヒアリングをしてください。

## 出力形式

以下の形式で `00_hearing.md` を生成してください：

========================

# 機能名

{わかりやすい名前（例: 駐車場予約機能）}

# 概要（TL;DR）

{この機能が何をするかを 2〜3 行でまとめる}

# ビジネス背景・目的

{なぜこの機能が必要なのか？誰の課題か？それによりどんな価値をもたらすか？}

# ユーザー視点のユースケース

- ユーザーは、◯◯ をしたい
- 管理者は、◯◯ を簡単に処理したい
- 外部サービスは、◯◯ をトリガーにしたい

# 機能要件（Functional Requirements）

- エンドポイント：`POST /api/reservations`
- バリデーション条件
- モデル構成（エンティティ単位で）
- ユースケースごとの処理フロー（if applicable）

# 非機能要件（Non-functional Requirements）

- スケーラビリティ
- 可用性
- セキュリティ要件（例：JWT 認証、Rate limit）
- SLA またはレスポンスタイム要件（例：1 秒以内）

# タスク分類（Task Type）

- [ ] API 開発
- [ ] DB 設計・マイグレーション
- [ ] 外部サービス連携
- [ ] テスト
- [ ] フロントエンド対応（あれば）

# 対応優先度・マイルストーン

- 優先度: 高 / 中 / 低
- リリース希望日: YYYY/MM/DD

# オープンな課題 / 未確定要素

- ◯◯ サービスとの API 仕様が未定
- 通知手段は後続フェーズで確定

## 入力情報（ヒアリングログなど）

{ここにプロダクトオーナーや開発者の発言、メモ、スレッド履歴などをそのまま貼り付けてください}

=============================

# 2. 設計 (`01_plan.md`)

`00_hearing.md` の内容をもとに ADR + コード設計 (dir 設計) を作成してください

## あなたの役割

あなたは優れたシステムアーキテクトです。
ADR や DIR 設計を記述してください

## 出力形式

以下の形式で `01_plan.md` という内容で出力してください

====================================

## Title: {機能名}の設計方針と実装計画

例: 駐車場予約機能の設計方針と実装計画

## Status: Proposed | Accepted | Deprecated

## Date: YYYY-MM-DD

## Author: {作成者名または AI}

---

## Context（背景・前提）

この機能は {ビジネス的背景（なぜ必要か）} に基づいて提案された。

既存の {システム/エンティティ/モジュール} との関係性、及び運用上の課題に対応することが目的である。

- 利用者が ◯◯ を行いたい
- システムとして △△ を保証する必要がある
- 外部サービスとの連携が前提条件にある

---

## Impact（影響範囲）

この機能は実際のコードベースのどの部分に影響を及ぼすかを記述してください

## Decision（設計上の意思決定）

以下のような技術/設計上の意思決定を採用する：

- **API 設計**: REST / GraphQL / RPC のうち、{選定理由} により REST を選択
- **アーキテクチャ**: {例: Clean Architecture を採用し、UseCase 層でロジックを分離}
- **永続化戦略**: {例: PostgreSQL + Prisma、予約範囲重複を SQL で排他制御}
- **エラーハンドリング**: HTTP ステータスコードに応じて分類
- **認証方式**: Firebase Auth による JWT トークン検証を middleware で実施

---

## Alternatives considered（検討されたが採用されなかった案）

| 案                               | 理由                                          |
| -------------------------------- | --------------------------------------------- |
| GraphQL API                      | 構造が複雑になりすぎるため、REST で十分と判断 |
| MongoDB（NoSQL）                 | 時系列・整合性制約が強く、RDB を優先          |
| 全ロジックを Controller 内に集約 | 保守性・テスト性が低下するため分離型へ        |

---

## Consequences（この決定による影響）

- Prisma のトランザクションを活用するため、DB の制約レベルを事前に設計する必要がある
- 各ロジックの粒度が小さくなるため、タスク分担がしやすい反面、学習コストが若干増す
- 今後の機能追加時に変更の影響が少ない構造になる

## QA (Quality Assurance)

この機能を実装後にどのような Unit test, Integration test, E2E test を実装するかを記述してください

---

## Implementation Plan（実装計画）

### ディレクトリ構成（予定）

```
src/
├── routes/
│   └── reservations.ts        # APIルーティング
├── services/
│   └── reservation_service.ts # ビジネスロジック
├── validators/
│   └── reservation.schema.ts  # 入力スキーマ（例: zod）
├── db/
│   └── models.prisma          # モデル定義
└── notifications/
    └── notify.stub.ts         # 通知処理（スタブ）

```

- `00_hearing.md`を読み込んだ上で、以下 2 つのファイルを出力する。
- 設計は「人間と AI の共同作業」であり、後工程の AI が再実行できる再現性を重視する。

=======================================

# 3. タスクのブレイクダウン (`02_task.md`)

`01_plan.md` の内容をもとに PR に相当するような task のブレイクダウンをしてください

## あなたの役割

あなたは優れたプロジェクトマネージャーでありエンジニアリングマネージャーです

ADR で与えられた仕様を最も効率的に実装するためにタスクのブレイクダウンを行ってください

## 出力形式

`02_tasks.md` という内容で出力してください

# 4. 進捗の記述 (`03_progress.md`)

実装状況に合わせて 03_progress.md を更新してください

## あなたの役割

あなたは優れたプロジェクトマネージャーです

実装したないように合わせて 03_progress.md を作成 / 更新するようにしてください

## 出力形式

`03_progress.md` という内容で出力してください
