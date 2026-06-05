# Thinkee

**Thinkee** は、日々の考え事を記録・整理し、過去の自分の思考と向き合うための思考支援アプリです。

---

## 設計書一覧

### [00. 技術選定・設計思想](design/00_Tech_Stack.md)

採用技術スタック（Flutter / Supabase / Riverpod）の選定理由、クリーンアーキテクチャに基づくレイヤー設計の方針、ディレクトリ構成を記載しています。

### [01. 要件定義書](design/01_Requirements.md)

アプリのコンセプト・ターゲット、投稿タブ・振り返りタブ・アカウントタブの機能要件、非機能要件、主要ユースケースを記載しています。

### [02. 基本設計書](design/02_Basic_Design.md)

画面一覧・画面遷移図（Mermaid）、各画面の UI 要素定義、UI/UX ポリシー、データストア方針（Supabase / ローカル）を記載しています。

### [03. 詳細設計書](design/03_Detail_Design.md)

Supabase の DB スキーマ（SQL）、Dart Entity 定義、Riverpod Provider 設計、RLS ポリシー、CSV メール送信の処理フローを記載しています。

### [04. テスト設計書](design/04_Test_Design.md)

クリーンアーキテクチャの各層に対するテスト戦略（単体テスト・Widget テスト・統合テスト）、Riverpod のモック化方針を記載しています。
