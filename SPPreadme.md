# SimplePointPlugin

<div align="center">

![Version](https://img.shields.io/badge/version-2.0.0-blue?style=for-the-badge)
![Minecraft](https://img.shields.io/badge/Minecraft-1.16.5+-green?style=for-the-badge)
![License](https://img.shields.io/badge/license-GPL--3.0-orange?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-8+-red?style=for-the-badge)

**シンプルだけど使いやすい、複数ポイント対応の Spigot ポイント管理プラグイン。**

</div>

---

## 📋 概要

SimplePointPlugin は、Spigot サーバー向けのポイント管理プラグインです。複数のポイント種別を同時に運用でき、GUI 操作だけで報酬ショップを構築できます。チーム機能やランキング表示、PlaceholderAPI 連携など、サーバー運営に必要な機能を幅広くカバーしています。

**主な特徴:**

- 🎯 **複数ポイント管理** — ポイント種別を無制限に作成・管理
- 🛒 **GUI 報酬ショップ** — アイテムを置くだけで報酬設定が完了
- 👥 **チーム機能** — グループ・チームを作成し、ポイントを共有・競争
- 📊 **ランキング** — 個人・チーム別ランキングをリアルタイム表示
- 🔗 **PlaceholderAPI 連携** — スコアボードやチャットへのデータ表示
- 📝 **ログ記録** — ポイント変動・購入履歴を日次ローテーションで自動記録

---

## ⚙️ 動作環境

| 項目 | 要件 |
|------|------|
| サーバーソフトウェア | Spigot / Paper |
| Minecraft バージョン | 1.16.5 以上 |
| Java | 8 以上 |
| 依存プラグイン | [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) |

---

## 🚀 インストール

1. [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) をサーバーの `plugins/` フォルダに配置する
2. `SimplePointPlugin.jar` を `plugins/` フォルダに配置する
3. サーバーを起動（または `/reload confirm`）する
4. `plugins/SimplePointPlugin/` フォルダが自動生成されます

---

## 📁 ディレクトリ構造

```
plugins/SimplePointPlugin/
├── settings.yml          # メッセージ設定
├── point_names.yml       # ポイントIDと表示名の対応
├── points/               # 各ポイントのプレイヤーデータ（UUID.yml）
│   └── <pointId>.yml
├── rewards/              # 各ポイントの報酬ショップデータ
│   └── <pointId>.yml
├── teams/
│   ├── reward/           # グループ設定（倍率・連携ポイント等）
│   │   └── <groupId>.yml
│   ├── team/             # チーム基本情報
│   │   └── <groupId>/<teamId>.yml
│   └── member/           # チームメンバー・貢献スコア
│       └── <groupId>/<teamId>.yml
└── logs/                 # ポイント変動・購入ログ
    ├── yyyy-MM-dd.log
    └── yyyy-MM-dd.log.gz # 自動圧縮された過去ログ
```

---

## 🎮 コマンド一覧

### 管理者コマンド `/spp`

**権限:** `spp.admin`

#### ポイント管理

| コマンド | 説明 |
|----------|------|
| `/spp create <ID> <表示名>` | 新しいポイント種別を作成。表示名にカラーコード・日本語・スペース使用可 |
| `/spp add <ID> <プレイヤー> <数>` | ポイントを加算（累計獲得ポイントにも加算） |
| `/spp remove <ID> <プレイヤー> <数>` | ポイントを減算 |
| `/spp set <ID> <プレイヤー> <数>` | ポイントを直接設定（累計獲得には反映されない） |

#### 情報・ランキング

| コマンド | 説明 |
|----------|------|
| `/spp score <ID> <プレイヤー>` | 指定プレイヤーの現在ポイントと累計ポイントを表示 |
| `/spp ranking <ID>` | 上位7名のランキングを全体チャットに表示 |
| `/spp userrank <ID> <順位>` | 指定した順位のプレイヤー名を検索 |
| `/spp adminranking <ID> <順位>` | 指定順位までの一覧を表示 |

#### 報酬・機能管理

| コマンド | 説明 |
|----------|------|
| `/spp rewardgui <ID>` | 報酬編集 GUI を開く |
| `/spp setreq <ID> <スロット> <数>` | 指定スロットの報酬に累計ポイント解放条件を設定 |
| `/spp toggleranking <ID>` | ランキング表示を有効/無効切り替え |
| `/spp togglereward <ID>` | 報酬ショップを有効/無効切り替え |
| `/spp togglefunction <ID>` | ランキング・報酬ショップを一括で有効/無効切り替え |

#### プラグイン管理

| コマンド | 説明 |
|----------|------|
| `/spp reload` | config・報酬・ポイントデータのキャッシュをリロード |
| `/spp help` | ヘルプを表示 |

---

### プレイヤーコマンド `/spt`

| コマンド | 説明 |
|----------|------|
| `/spt myp <ID>` | 自分の現在ポイントと累計獲得ポイントを確認 |
| `/spt reward <ID>` | 報酬ショップ GUI を開く |
| `/spt ranking <ID>` | 上位7名のランキングと自分の現在順位を表示 |

---

### チーム管理コマンド `/sppt`

**権限:** `sppt.admin`

| コマンド | 説明 |
|----------|------|
| `/sppt create <グループID> <表示名>` | グループを作成 |
| `/sppt teamcreate <グループID> <チームID> <表示名>` | グループ内にチームを作成 |
| `/sppt join <グループID> <チームID> <プレイヤー>` | プレイヤーをチームに追加 |
| `/sppt leave <グループID> <プレイヤー>` | プレイヤーをチームから脱退させる |
| その他 | チームポイント管理・設定変更など |

---

### ショートカットコマンド

| コマンド | 同等の操作 |
|----------|-----------|
| `/myp <ID>` | `/spt myp <ID>` |
| `/ranking <ID>` | `/spt ranking <ID>` |
| `/teaminfo <グループID>` | チーム情報を表示 |

---

## 🛒 報酬ショップの設定

### GUI での基本設定

`/spp rewardgui <ポイントID>` で管理 GUI を開き、報酬として設定したいアイテムを置くだけで設定完了です。

| GUI ボタン | 機能 |
|-----------|------|
| コンパレーター | 在庫モード切り替え（有限 / 無限） |
| 絵画 | 装飾モード切り替え（購入不可の飾りアイテムとして表示） |

在庫はサーバー全体共有・プレイヤー個人の2種類から選択できます。

### YML による詳細設定

`rewards/<ポイントID>.yml` を直接編集することで、以下の追加設定が可能です。

**購入時のコマンド実行:**

```yaml
# %player% = プレイヤー名, %point% = ポイントID
commands:
  - "broadcast %point% から %player% が豪華賞品をゲットしました！"
  - "give %player% diamond 1"
```

**カスタム説明文の追加:**

```yaml
# "|" の左側がラベル（任意）、右側が説明文
customlore:
  - "test|&6こんにちは"
  - "&eあなたの名前: &f%player%|&a所持ポイント: &f%point% pt"
```

**累計ポイント解放条件の設定:**

```bash
/spp setreq <ポイントID> <スロット番号> <必要累計ポイント>
```

---

## 👥 チーム機能

### 仕組み

プレイヤーをグループ内のチームに所属させることで、個人のポイント獲得がチームスコアに自動で反映されます。倍率設定により、チームメンバーの貢献度を増幅させることもできます。

### 基本的な流れ

```bash
# 1. グループを作成
/sppt create zettai &b絶対王者杯

# 2. チームを作成
/sppt teamcreate zettai red &cレッドチーム
/sppt teamcreate zettai blue &9ブルーチーム

# 3. プレイヤーをチームへ追加
/sppt join zettai red PlayerName
```

### チーム設定 (teams/reward/<グループID>.yml)

```yaml
display_name: "&b絶対王者杯"
linked_point: "event"     # 同期するポイントID
multiplier: 1.5           # ポイント倍率
multiplier_expiry: 0      # 倍率の有効期限（Unixタイム, 0=無制限）
enabled: true
battle:
  active: false           # VS モードの有効化
  team1: "red"
  team2: "blue"
  color_self: "&b"
  color_enemy: "&e"
```

---

## 🔗 PlaceholderAPI 連携

プラグインは `%simplepoint_...%` 形式のプレースホルダーを提供します。

| プレースホルダー | 説明 |
|-----------------|------|
| `%simplepoint_point_<ID>%` | プレイヤーの現在ポイント |
| `%simplepoint_total_point_<ID>%` | プレイヤーの累計獲得ポイント |
| `%simplepoint_total_score_<グループID>%` | プレイヤーが所属するチームの合計スコア |
| `%simplepoint_team_name_<グループID>%` | プレイヤーが所属するチームの表示名 |
| `%simplepoint_group_name_<グループID>%` | グループの表示名 |
| `%simplepoint_vs_bar_<グループID>%` | VS バー（対戦スコアの視覚的表示） |

**使用例 (TAB プラグインのスコアボード):**

```yaml
- "&e所持: &f%simplepoint_point_event%pt"
- "&7累計: &f%simplepoint_total_point_event%pt"
- "&b所属: &f%simplepoint_team_name_zettai%"
```

---

## 📝 ログ

すべてのポイント変動と報酬購入は `plugins/SimplePointPlugin/logs/` に自動記録されます。

**ログフォーマット:**

```
[2026-01-15 12:34:56] PlayerName, event, 100, ADD
[2026-01-15 12:35:10] [PURCHASE] PlayerName, event, 豪華な報酬, 500 pt
```

前日以前のログは起動時に自動的に `.gz` 形式に圧縮されます。手動での管理は不要です。

---

## 💬 settings.yml

```yaml
messages:
  prefix: "§e[SimplePoint] "
  no-points: "§cポイントが足りません！"
  purchase-success: "§a購入完了！残りポイント: {point}pt"
  once-only: "§cこの報酬は一度しか受け取れません。"
  out-of-stock: "§c在庫切れです！"
  delete-success: "§c報酬を削除しました。"
  team-created: "§bチーム「{team}」を結成しました！"
  team-donated: "§aチームに {amount}pt 貢献しました！"
```

---

## 🛠️ ビルド方法

Maven を使用してビルドします。

```bash
git clone https://github.com/azisaba/SimplePointPlugin.git
cd SimplePointPlugin
mvn clean package
```

ビルド成果物は `target/SimplePointPlugin-2.0.0.jar` に生成されます。

---

## 📜 ライセンス

このプラグインは [GPL-3.0](LICENSE) ライセンスの下で公開されています。

---

## 👤 作者

**pino223** — [azisaba](https://github.com/azisaba)
