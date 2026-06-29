# FRICKのエンジニアリング知識

FRICKのエンジニアリングで**できること**を整理するためのリポジトリです。技術詳細や前提知識は `docs/` 配下の各ファイルに分けて管理します。

## このリポジトリの目的

- 自分が実際に行うエンジニアリング活動の範囲を言語化し、把握しやすくする
- 領域ごとのスキル・前提知識をドキュメント化し、自己学習や振り返りの指針にする
- メモやリンクの散在を減らし、自分用の参照入口を一つにまとめる

## できること

| 領域 | 概要 |
|------|------|
| [ソフトウェア開発](docs/knowledge/software-development.md) | 設計、実装、レビュー、リリース |
| [ITインフラ](docs/knowledge/it-infrastructure.md) | デプロイ、監視、インシデント対応 |
| [品質・セキュリティ](docs/knowledge/quality-security.md) | テスト戦略、脆弱性対応 |
| [データ](docs/knowledge/data.md) | パイプライン、分析基盤 |

## 環境・構成

| 項目 | 概要 | 詳細 |
|------|------|------|
| 自宅ネットワーク | 10GbE ネットワーク、NAS、VMホスト、AIサーバーなど | [構成図・技術メモ](docs/home-network/home-network.md) |
| CI/CD | GitHub Actions によるビルド・デプロイ・配布 | [パイプライン詳細](docs/cicd/pipeline.md) |
| 外部クラウド | ConoHa / Sakura VPS 上の公開サービス | [自宅ネットワーク構成](docs/home-network/home-network.md#ゾーン概要) |

## ドキュメント構成

```
docs/
├── home-network/   # 自宅ネットワーク構成の技術詳細
├── cicd/             # CI/CD パイプラインの技術詳細
└── knowledge/        # 領域別のスキル・前提知識
```

詳細な知識・技術メモは [docs/knowledge/](docs/knowledge/README.md) から辿ってください。
