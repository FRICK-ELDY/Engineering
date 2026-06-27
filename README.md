# FRICKのエンジニアリング知識

FRICKのエンジニアリングで**できること**と、それに伴う**必要な知識**を整理するためのリポジトリです。

## このリポジトリの目的

- 自分が実際に行うエンジニアリング活動の範囲を言語化し、把握しやすくする
- 領域ごとのスキル・前提知識をドキュメント化し、自己学習や振り返りの指針にする
- メモやリンクの散在を減らし、自分用の参照入口を一つにまとめる

## コンテンツ構成（案）

以下のような章立てで追記・分割していく想定です。必要に応じてディレクトリや別ファイルに切り出してください。

| 領域 | できることの例 | 必要な知識の例 |
|------|------------------|----------------|
| ソフトウェア開発 | 設計、実装、レビュー、リリース | 言語・フレームワーク、Git、テスト |
| インフラ・運用 | デプロイ、監視、インシデント対応 | クラウド、ネットワーク、SRE の考え方 |
| 品質・セキュリティ | テスト戦略、脆弱性対応 | セキュリティ基礎、コンプライアンス |
| データ | パイプライン、分析基盤 | SQL、データモデリング、プライバシー |

※ 上表はプレースホルダです。自分の環境・関心に合わせて書き換えてください。

## 自宅インフラ構成

自宅ネットワーク・サーバー・クラウドの現状構成です。図中の枠線色は導入状況を示します。

| 枠線色 | 意味 |
|--------|------|
| 緑 | 購入・準備済み |
| 赤 | 未購入 |

```mermaid
graph TD
    %% 外部ネットワーク・クラウド・コアルーター
    CF["Cloudflare ネットワーク<br>【WAF / DDoS防御 / SSL暗号化】"]
    WAN(("インターネット<br>@nifty光 10GbE"))
    ONU["NTT 10G ONU<br>(光回線終端装置)"]
    RTX1300["ルーター<br>YAMAHA RTX1300<br>【セカンダリ DHCP / DNS】"]

    %% 【外部クラウド VPS環境】
    subgraph VPS_Zone ["外部クラウド (VPS環境)"]
        VPS_Conoha_1["ConoHa VPS<br>(Ubuntu / 4Core / 4GB)<br>【Misskey管理】"]
        VPS_Conoha_2["ConoHa VPS<br>(Ubuntu / 1Core / 512MB)<br>【Webサイトテスト】"]
        VPS_Sakura["Sakura VPS<br>(Ubuntu / 2Core / 1GB)<br>【Web依頼フォーム】"]
    end

    %% Cloudflareを通じたルーティング・紐づけ
    CF -.-> VPS_Conoha_1
    CF -.-> VPS_Conoha_2
    CF -.-> VPS_Sakura

    CF === WAN
    WAN === ONU
    ONU === RTX1300

    %% 【VLAN 1GbE 1 系統】ルーターから物理隔離
    subgraph VLAN_1GbE_1 ["VLAN 1GbE 1 (ルーター1GbE物理直結)"]
        RPi_Zero["Raspberry Pi Zero 2 W<br>【Web依頼フォーム】<br>※cloudflared 稼働"]
    end
    RTX1300 --- RPi_Zero
    RPi_Zero -.-> CF

    %% 【VLAN 1GbE 2 系統】テストPC環境
    subgraph VLAN_1GbE_2_Zone ["VLAN 1GbE 2 (テストPC環境)"]
        PC_1["PC1<br>(Windows / Ryzen 7 5800H / 32GB)"]
        PC_2["PC2<br>(Mac)"]
        PC_3["PC3<br>(Ubuntu)"]
    end
    RTX1300 --- PC_1
    RTX1300 --- PC_2
    RTX1300 --- PC_3

    %% 【開発用ノートPC 系統】
    subgraph Dev_Laptop_Zone ["開発用ノートPC"]
        PC_ASUS["ASUS TUF Gaming A14<br>(Windows / Ryzen 7 8845HS / 32GB / RTX4060)"]
        PC_MBP["MacBook Pro<br>(M2 Pro / 16GB)"]
    end

    %% 【VLAN 10GbE 系統】10GbEメインインフラエリア
    subgraph VLAN_10GbE_Zone ["VLAN 10GbE (10GbEメインインフラ)"]
        CRS309["10GbE スイッチ<br>MikroTik CRS309-1G-8S+IN"]
        QNAP1["メインNAS<br>QNAP TS-632X-4G"]
        QNAP2["バックアップNAS<br>QNAP TS-632X-4G"]
        MS01["Minisforum MS-01<br>【VMホストサーバー】"]
        BE7200["TP-Link BE7200<br>【APモード】"]
        PC_Dev["開発用PC<br>(Windows / Ryzen 9 7950X / 64GB / RTX4090)"]

        %% VLAN_10GbEの配下にさらにDNSを配置
        subgraph VLAN_2 ["VLAN 2 (ローカルインフラエリア)"]
            RPi_4["Raspberry Pi 4<br>【プライマリ DNS / DHCPサーバー】"]
        end
    end

    %% 【VLAN 3 系統】AIサーバー環境
    subgraph VLAN_3_Zone ["VLAN 3 (AIサーバーエリア)"]
        DGX["NVIDIA DGX Spark x2<br>【AIホストサーバー】"]
    end

    %% VLAN_10GbE 内部およびノートPC類の物理・無線接続
    RTX1300 === CRS309
    CRS309 === QNAP1
    CRS309 -.-> QNAP2
    CRS309 -.-> MS01
    CRS309 -.-> BE7200
    BE7200 -.-> PC_Dev
    BE7200 -.-> PC_ASUS
    BE7200 -.-> PC_MBP
    CRS309 -.-> RPi_4
    CRS309 === DGX

    %% NAS間の直結専用線
    QNAP1 === QNAP2

    %% 電源・制御の管理
    subgraph Power_Management ["電源管理 & 制御レイヤー"]
        direction LR
        Wall["壁面コンセント<br>一般家庭用 100V"] ===> UPS["UPS (無停電電源装置)<br>CyberPower SX750UJP<br><br>【USBシグナル連携機能】<br>メインNASとUSB接続し、停電時に自動シャットダウンを制御"]
        UPS === P1["【バッテリー保護コンセント 接続デバイス】<br>・ルーター：YAMAHA RTX1300<br>・スイッチ：MikroTik CRS309-1G-8S+IN<br>・メインNAS：QNAP TS-632X-4G<br>・バックアップNAS：QNAP TS-632X-4G<br>・VMホスト：Minisforum MS-01"]
    end

    %% ノード（デバイス）自体の外枠カラー指定（緑：準備・購入済 / 赤：未購入）
    %% 購入・準備済み（緑枠）
    style CF stroke:#22c55e,stroke-width:4px
    style WAN stroke:#22c55e,stroke-width:4px
    style ONU stroke:#22c55e,stroke-width:4px
    style RTX1300 stroke:#22c55e,stroke-width:4px
    style PC_1 stroke:#22c55e,stroke-width:4px
    style PC_ASUS stroke:#22c55e,stroke-width:4px
    style PC_MBP stroke:#22c55e,stroke-width:4px
    style BE7200 stroke:#22c55e,stroke-width:4px
    style PC_Dev stroke:#22c55e,stroke-width:4px
    style Wall stroke:#22c55e,stroke-width:4px
    style RPi_4 stroke:#22c55e,stroke-width:4px
    style VPS_Conoha_1 stroke:#22c55e,stroke-width:4px
    style VPS_Conoha_2 stroke:#22c55e,stroke-width:4px
    style VPS_Sakura stroke:#22c55e,stroke-width:4px

    %% 未購入（赤枠）
    style RPi_Zero stroke:#ef4444,stroke-width:4px
    style PC_2 stroke:#ef4444,stroke-width:4px
    style PC_3 stroke:#ef4444,stroke-width:4px
    style UPS stroke:#ef4444,stroke-width:4px
    style P1 stroke:#ef4444,stroke-width:4px
    style CRS309 stroke:#ef4444,stroke-width:4px
    style QNAP1 stroke:#ef4444,stroke-width:4px
    style QNAP2 stroke:#ef4444,stroke-width:4px
    style MS01 stroke:#ef4444,stroke-width:4px
    style DGX stroke:#ef4444,stroke-width:4px
```

## CI/CD パイプライン

ローカル開発から GitHub Actions 経由のデプロイ、およびエンドユーザーへのアクセスまでの流れです。

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

---

*この README は初期テンプレートです。進行に合わせてセクションを増やしたり、別リポジトリへのリンクを足したりしてください。*
