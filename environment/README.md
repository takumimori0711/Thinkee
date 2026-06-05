# 環境構築・デプロイ手順

## 構成概要

MkDocs + Material テーマを使ったドキュメント環境。PlantUMLダイアグラムはKrokiサービス経由でレンダリングされる。

```
mkdocs.yml                  # MkDocs設定（KROKI_URL環境変数でKroki接続先を切り替え可）
docs/                       # ドキュメントファイル（.md）
  index.md                  # トップページ
  diagrams/                 # 図を含むページ
  stylesheets/extra.css     # テーマカスタマイズ
environment/
  docker/mkdocs/            # MkDocs Dockerイメージ
  docker/kroki/             # Kroki Dockerイメージ
  local/compose.yml         # ローカル開発用 docker compose
.github/workflows/
  deploy-pages.yml          # GitHub Pages自動デプロイ（mainブランチへのpush時）
```

### KrokiのURL設定

- ローカルDocker: `http://thinkee-kroki:8000`（`mkdocs.yml`デフォルト）
- CI（GitHub Actions）: `KROKI_URL=http://localhost:8000` で上書き

---

## ローカルDoc環境

### 起動

```bash
cd environment/local
docker compose up --build
docker compose up -d # バックグラウンド実行
```

ブラウザで http://localhost:8000 を開く。ファイルを編集すると自動でリロードされる。

### 停止

```bash
docker compose down
```

---

## GitHub Pages へのデプロイ

### 初回設定

1. GitHubリポジトリの **Settings > Pages** を開く
2. **Source** を `Deploy from a branch` に設定
3. **Branch** を `gh-pages` / `/ (root)` に設定して保存

### 自動デプロイ

`main` ブランチに以下のいずれかの変更をpushすると、GitHub Actionsが自動でビルド・デプロイする。

- `docs/**`
- `mkdocs.yml`
- `.github/workflows/deploy-pages.yml`

Actions の実行状況は GitHub リポジトリの **Actions** タブで確認できる。

### 手動デプロイ

GitHub リポジトリの **Actions > Build and Deploy MkDocs > Run workflow** から手動実行できる。

### ドキュメントのリンク

Pagesのリンクは[こちら](https://takumimori0711.github.io/Thinkee/)
