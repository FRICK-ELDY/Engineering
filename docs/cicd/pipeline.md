# CI/CD パイプライン

ローカル開発から GitHub Actions 経由のデプロイ、およびエンドユーザーへのアクセスまでの流れです。

## フロー図

```mermaid
graph LR
    subgraph Local ["① ローカル開発環境"]
        DevPC["開発端末"]
    end

    subgraph GitHub_Env ["② GitHub (バージョン管理 & CI/CD)"]
        Repo["GitHub リポジトリ"]
        Actions["GitHub Actions<br>(CI/CD ランナー)"]
        Releases["GitHub Releases<br>(バイナリ配布)"]

        Repo -- "Pushを検知" --> Actions
        Actions -- "ビルド・タグ付け" --> Releases
    end

    subgraph Production ["③ 本番環境 (外部クラウド)"]
        VPS["VPS<br>(デプロイ先)"]
    end

    subgraph Public ["④ エンドユーザーアクセス"]
        CF["Cloudflare"]
        User(("ブラウザ"))
        User2(("ユーザー<br>(実行ファイル取得)"))
    end

    DevPC -- "git push" --> Repo
    Actions -- "自動デプロイ" --> VPS

    %% リリースからの配布ルート
    Releases -- "実行ファイルDL" --> User2

    User -- "HTTPS アクセス" --> CF
    CF -- "プロキシ通信" --> VPS
```

## ステージ概要

| ステージ | 内容 |
|----------|------|
| ① ローカル開発 | 開発端末での実装・コミット |
| ② GitHub | Push 検知 → CI ビルド → Releases へのバイナリ配布 |
| ③ 本番環境 | VPS への自動デプロイ |
| ④ エンドユーザー | Cloudflare 経由の HTTPS アクセス、または Releases からの実行ファイル取得 |

## 技術メモ

<!-- ワークフロー定義、デプロイ手順、環境変数、シークレット管理など詳細はここに追記 -->
