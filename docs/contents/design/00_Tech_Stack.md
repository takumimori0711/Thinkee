# 00. 技術選定・設計思想

## 1. 採用技術スタック

| 分類 | 技術 |
|------|------|
| フロントエンド | Flutter (Dart) / iOS・Android クロスプラットフォーム |
| 状態管理 | Riverpod |
| BaaS | Supabase |
| データベース | PostgreSQL（Supabase 内蔵） |
| 認証 | Supabase Auth（Google OAuth） |
| メール送信 | Resend |
| ダイアグラム | Mermaid（mkdocs-material ネイティブ） |

---

## 2. 採用理由

### Flutter
- iOS / Android を単一コードベースでカバーし、開発コストを削減できる
- Material Design 3 の豊富な UI コンポーネントにより、X 風カードフィードを迅速に実装できる
- Hot Reload による高速な開発サイクル

### Supabase
- PostgreSQL ベースの強力なクエリ機能（全文検索・ランダム取得など）が利用できる
- Row Level Security（RLS）により、ユーザーごとのデータ分離をデータベース層で保証できる
- Supabase Auth で認証基盤を迅速に構築できる
- Edge Functions で CSV 生成・メール送信などのサーバーサイド処理を実装できる

### Riverpod
- Flutter 向けの型安全な状態管理ライブラリ
- `AsyncValue` によるローディング・エラー状態の統一的な扱い
- Provider オーバーライドによるテスト時のモック注入が容易

---

## 3. アーキテクチャ方針

クリーンアーキテクチャ（Clean Architecture）をベースとしたレイヤードアーキテクチャを採用する。

### 各層の責務

| 層 | 主要コンポーネント | 責務 |
|----|------|------|
| Presentation | Widget, Riverpod Provider（ViewModel 相当） | UI 描画・ユーザーインタラクション・状態保持 |
| Domain | Entity, Repository Interface | ビジネスロジックとドメインモデルの定義（外部依存なし） |
| Application | UseCase | ユースケース単位の処理フロー調整（必要に応じて） |
| Infrastructure | Repository 実装, Supabase クライアント | 外部サービスとのデータ通信 |

### 依存関係のルール

```
Presentation ──→ Domain ←── Infrastructure
```

- Presentation・Infrastructure は Domain に依存する
- Domain は他の層に依存しない（依存性逆転の原則）
- Infrastructure は Domain の Repository インターフェースを実装する

---

## 4. ディレクトリ構成

```
lib/
├── core/                          # 共通処理・横断的関心事
│   ├── constants/                 # 定数（色・フォントサイズ・文字列）
│   ├── error/                     # 共通エラー定義
│   ├── router/                    # go_router による画面遷移定義
│   └── utils/                     # 汎用ユーティリティ
│
└── features/                      # 機能単位のモジュール
    ├── auth/                      # 認証機能
    │   ├── presentation/          # ログイン画面・サインアップ画面・Provider
    │   ├── domain/                # User Entity, AuthRepository Interface
    │   └── infrastructure/        # Supabase Auth 連携
    │
    ├── post/                      # 投稿機能
    │   ├── presentation/          # 投稿一覧・作成・編集画面・Provider
    │   ├── domain/                # Post Entity, PostRepository Interface
    │   ├── application/           # CreatePostUseCase, UpdatePostUseCase, DeletePostUseCase
    │   └── infrastructure/        # Supabase posts テーブル連携
    │
    ├── reflection/                # 出会うタブ機能
    │   ├── presentation/          # 振り返り画面・Provider
    │   ├── domain/                # ReflectionRepository Interface
    │   └── infrastructure/        # ランダム投稿取得・更新日時管理
    │
    └── account/                   # アカウント機能
        ├── presentation/          # アカウント画面・Provider
        ├── domain/                # AccountRepository Interface
        └── infrastructure/        # ユーザー情報取得・CSV 出力連携
```

---

## 5. 主要パッケージ（案）

| パッケージ | 用途 |
|-----------|------|
| `flutter_riverpod` | 状態管理 |
| `riverpod_annotation` / `riverpod_generator` | Riverpod コード生成 |
| `freezed` | Immutable データクラス生成 |
| `json_serializable` | JSON シリアライズ |
| `supabase_flutter` | Supabase クライアント |
| `google_sign_in` | Google OAuth サインイン |
| `go_router` | 画面遷移 |
| `shared_preferences` | ローカルデータ保存（振り返り更新日時等） |
| `build_runner` | コード生成実行 |

---

## 6. 開発環境

本プロジェクトでは、**VSCode** を中心とした開発環境を構築する。`fvm` によるSDKバージョン管理と、`Docker` によるBaaSの隔離を組み合わせることで、ホスト環境のクリーンさと開発の利便性を両立させる。

### 6.1. ローカル開発環境の構成方針

* **フロントエンド（Flutter）:**
    * **SDK管理:** `fvm` (Flutter Version Management) を採用する。プロジェクト固有のSDKバージョンを `.fvm/flutter_sdk` に固定し、ホストマシンのグローバル環境との競合を避ける。
    * **実行環境:** OSのネイティブビルドツール（Xcode / Android SDK）を利用するため、ビルドおよび実行（Simulator/Emulator）はホストマシン上で行う。これにより高速な「Hot Reload」を最大限活用する。
* **バックエンド（Supabase Local Dev）:**
    * **隔離環境:** `Supabase CLI` と **Docker** を活用し、PostgreSQL、Auth、Storage、Edge Functionsなどの機能をコンテナとしてローカルに構築する。ホスト環境を汚さず、商用環境と同一の挙動を確認可能とする。

### 6.2. 必須ソフトウェア・SDK

1.  **fvm** (Flutter SDKのバージョン管理)
2.  **Docker Desktop** (Supabaseおよび関連ツールの実行)
3.  **Supabase CLI** (ローカル環境の制御、マイグレーション管理)
4.  **Xcode** (iOSシミュレーター・ビルド用 ※Mac必須)
5.  **Android Studio / SDK** (Androidエミュレーター・ビルド用)

### 6.3. VSCode 拡張機能（推奨）

`.vscode/extensions.json` に定義し、プロジェクト開始時に一括導入を推奨する。

| 拡張機能名 | 用途・選定理由 |
| :--- | :--- |
| **Flutter / Dart** | 必須。静的解析、デバッグ、Hot Reloadの提供。 |
| **Error Lens** | エラー内容をコード行に直接表示し、開発スピードを向上。 |
| **PostgreSQL** | Docker上のSupabase DBへVSCodeから直接クエリを実行。 |
| **Better Comments** | TODOや重要事項を色分けし、多層アーキテクチャ内のタスクを可視化。 |
| **Thunder Client** | Edge Functionsや外部APIの疎通確認をエディタ内で完結。 |

### 6.4. 開発ワークフロー（イメージ）

1.  **環境起動:** `supabase start` でDockerコンテナ群を起動する。
2.  **SDK設定:** `fvm install` でプロジェクト指定のSDKを取得し、VSCodeのSDKパスを `.fvm/flutter_sdk` に向ける。
3.  **アプリ実行:** `fvm flutter run` でホスト側のシミュレーターを起動。
4.  **高速開発:** UIの微調整は「Hot Reload」、状態の再構築を伴う場合は「Hot Restart」を使い分ける。
5.  **DB更新:** テーブル変更はローカルのマイグレーションファイルを生成して適用する。

---

## 7. コーディング規約

### 7.1. 命名規則

| 対象 | 規則 | 例 |
| ------ | ------ | ---- |
| クラス名 / 型名 | UpperCamelCase | `PostRepository`, `UserEntity` |
| 変数名 / 関数名 | lowerCamelCase | `fetchPosts()`, `isLoading` |
| 定数 | lowerCamelCase | `maxPostLength` |
| ファイル名 | snake_case | `post_repository.dart` |
| プライベートメンバー | アンダースコアプレフィックス | `_posts`, `_fetchData()` |

### 7.2. 型定義

- 関数のパラメータ・戻り値には明示的な型注釈を必ず付ける
- `dynamic` の使用を原則禁止とする
- `var` は型が文脈から明らかな場合のみ使用する
- Nullable 型（`?`）は必要最小限に留め、早期リターンパターンを活用する
- データクラスには `freezed` を使い、イミュータブルに設計する

```dart
// Good
Future<List<Post>> fetchPosts(String userId) async { ... }

// Bad
fetchPosts(userId) async { ... }
```

### 7.3. Widget 設計

- 1 Widget = 1 ファイルを原則とする
- `StatelessWidget` を優先し、状態が必要な場合は Riverpod で管理する
- `build()` メソッド内のネストが 3 段を超える場合は、private メソッドまたは別 Widget クラスに分割する
- `const` コンストラクタを積極的に活用してリビルドを抑制する

### 7.4. 非同期処理

- `async/await` に統一し、`.then()` チェーンは使用しない
- Riverpod の `AsyncValue` を活用し、loading / error / data を統一的に扱う
- `Future` の結果を捨てることを禁止する（`unawaited_futures` lint を有効化）

### 7.5. その他のベストプラクティス

- `print()` の使用禁止。ログには `debugPrint()` または logger パッケージを使用する
- ハードコードされた文字列・マジックナンバーは `core/constants/` に集約する
- `analysis_options.yaml` で `flutter_lints` を適用し、静的解析を必ず通す
- `build_runner` によるコード生成（`freezed`, `riverpod_generator`）はコミット前に実行する
