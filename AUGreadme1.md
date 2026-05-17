# AgetarouUniqueGuns

**Minecraft Java Edition 1.16.5 向けユニーク武器プラグイン**

CrackShot / CrackShotPlus をベースに、武器ごとの特殊なSPモード（武器変更・キルストリーク・モデル変更・爆発制御など）を追加するプラグインです。

---

## 目次

- [動作環境](#動作環境)
- [依存プラグイン](#依存プラグイン)
- [インストール](#インストール)
- [コマンド](#コマンド)
- [設定ファイル構成](#設定ファイル構成)
- [UniversalWeaponSystem 機能リファレンス](#universalweaponsystem-機能リファレンス)
  - [Instant_Reload](#instant_reload)
  - [On_Hit_Explosion](#on_hit_explosion)
  - [Sneak_Explosive_Shot](#sneak_explosive_shot)
  - [On_Hit_Recovery](#on_hit_recovery)
  - [Headshot_Ammo_Recovery](#headshot_ammo_recovery)
  - [Life_Steal](#life_steal)
  - [Switch_Lock](#switch_lock)
  - [Custom_Model_Data_Held](#custom_model_data_held)
  - [Delay_Bar](#delay_bar)
- [WeaponsSPMode 機能リファレンス](#weaponsspmode-機能リファレンス)
  - [WhenChangeWeapon](#whenchangeweapon)
  - [KillStreak](#killstreak)
- [設定例](#設定例)
- [カラーコード](#カラーコード)
- [サウンド記法](#サウンド記法)
- [名前変更武器の作り方](#名前変更武器の作り方)

---

## 動作環境

| 項目         | バージョン          |
| ------------ | ------------------- |
| Minecraft    | Java Edition 1.16.5 |
| Java         | 8 以上              |
| ビルドツール | Maven               |

---

## 依存プラグイン

| プラグイン                                                       | 用途                         |
| ---------------------------------------------------------------- | ---------------------------- |
| [CrackShot](https://www.spigotmc.org/resources/crackshot.14215/) | 武器システム基盤             |
| [CrackShotPlus](https://github.com/Mamorune/CrackShotPlus)       | 弾薬取得API・WeaponHeldEvent |
| [LGW-Core](https://github.com/AzisabaNetwork/LGW-Core)           | PlayerKillEvent              |
| [ACF (aikar/commands)](https://github.com/aikar/commands)        | コマンド管理                 |

---

## インストール

1. 上記の依存プラグインをすべて導入する
2. ビルド済みの `AgetarouUniqueGuns-*.jar` を `plugins/` フォルダに配置する
3. サーバーを起動すると `plugins/AgetarouUniqueGuns/weapons/` フォルダが生成される
4. `weapons/` 以下に武器設定 YAML ファイルを配置する
5. `/aug reload` で設定を反映する

---

## コマンド

コマンドエイリアス：`/agetarouuniqueguns` または `/aug`

| サブコマンド                     | 権限                             | 説明                                  |
| -------------------------------- | -------------------------------- | ------------------------------------- |
| `/aug help`                      | `agetarouuniqueguns.help`        | ヘルプを表示                          |
| `/aug reload`                    | `agetarouuniqueguns.reload`      | `weapons/` フォルダの設定を再読み込み |
| `/aug ncmanual`                  | `agetarouuniqueguns.ncmanual`    | 名前変更武器の作成手順を表示          |
| `/aug setvariable <key> <value>` | `agetarouuniqueguns.setvariable` | 手持ち武器の固有変数を変更            |

`setvariable` で現在対応している武器と変数：

| 武器            | key         | 説明                   |
| --------------- | ----------- | ---------------------- |
| `HurtfulSpine`  | `killcount` | キルカウントを直接設定 |
| `HurtlessSpine` | `killcount` | キルカウントを直接設定 |

---

## 設定ファイル構成

```
plugins/
└── AgetarouUniqueGuns/
    └── weapons/
        ├── MCX.yml        # 武器ごとに1ファイル or まとめて1ファイルどちらでもOK
        ├── MCX2.yml
        └── special/       # サブディレクトリも再帰的に読み込まれる
            └── MCX3.yml
```

各 YAML ファイルのトップレベルキーが武器IDになります。

```yaml
MCX: # ← CrackShot の武器ID
  KillStreak: ...
  WhenChangeWeapon: ...
```

---

## UniversalWeaponSystem 機能リファレンス

すべての武器で共通して使える汎用機能です。

### 共通オプション（各機能セクションで使用可能）

| キー                   | 型      | 説明                                                           |
| ---------------------- | ------- | -------------------------------------------------------------- |
| `Enable`               | boolean | 機能の有効/無効                                                |
| `Chance`               | double  | 発動確率（0.0〜1.0）                                           |
| `Cooldown_Ticks`       | int     | クールダウン（tick）                                           |
| `Require_Sneak`        | boolean | シフト中のみ発動                                               |
| `Required_Effect`      | string  | 必要なポーション効果名（カンマ区切りで複数可）                 |
| `Message`              | string  | アクションバーメッセージ                                       |
| `Sound` / `Sounds`     | string  | 効果音（[サウンド記法](#サウンド記法)参照）                    |
| `Custom_Model_Data_CD` | int     | クールダウン中に適用するカスタムモデルデータ（終了時自動復元） |
| `Delay_Bar`            | section | プログレスバー表示（[Delay_Bar](#delay_bar)参照）              |

---

### Instant_Reload

**シフトキーを押すことで弾薬を即座に補充する機能です。**

```yaml
武器名:
  Instant_Reload:
    Enable: true
    Add_Amount: 1 # 1回のシフトで補充する弾数
    Reload_On_Sneak: false # true にすると通常リロード中にシフトで即時フル補充
    Cooldown_Ticks: 20
    Message: "&a補充！"
    Sound: "ITEM_ARMOR_EQUIP_LEATHER-1.0-1.5"
    Custom_Model_Data_CD: 1002 # クールダウン中のモデルID（省略可）
    Delay_Bar:
      Action_Bar: "&7リロード中... {bar}"
      End_Action_Bar: "&a準備完了！"
      Symbol: "|"
      Symbol_Amount: 10
      Left_Color: "&e"
      Right_Color: "&7"
```

---

### On_Hit_Explosion

**弾丸がエンティティまたはブロックに命中したときに爆発を発生させる機能です。**  
爆発パワーなどは CrackShot 側の設定に従います。二重爆発防止ロックが内蔵されています。

```yaml
武器名:
  On_Hit_Explosion:
    Enable: true
    Explosion_Chance_Entity: 0.5 # エンティティ命中時の爆発確率（0.0〜1.0）
    Explosion_Chance_Block: 0.3 # ブロック命中時の爆発確率
    Cooldown_Ticks: 10
```

---

### Sneak_Explosive_Shot

**シフトしながら射撃すると追加弾薬を消費して爆発弾を発射する機能です。**

```yaml
武器名:
  Sneak_Explosive_Shot:
    Enable: true
    Extra_Ammo_Cost: 2 # 通常消費に加えて追加で消費する弾数
    Require_Sneak: true
    Cooldown_Ticks: 0
    Sound: "ENTITY_GENERIC_EXPLODE-0.5-1.2"
```

---

### On_Hit_Recovery

**エンティティに命中するたびに弾薬を1回復する機能です。**

```yaml
武器名:
  On_Hit_Recovery:
    Enable: true
    Chance: 1.0
    Cooldown_Ticks: 0
    Message: "&a命中回復！"
```

---

### Headshot_Ammo_Recovery

**ヘッドショット命中時に弾薬を1回復する機能です。**

```yaml
武器名:
  Headshot_Ammo_Recovery:
    Enable: true
    Chance: 1.0
    Message: "&6ヘッドショット！弾薬回復"
```

---

### Life_Steal

**エンティティへのダメージに応じてHPを回復する機能です。**

```yaml
武器名:
  Life_Steal:
    Enable: true
    Heal_Percent: 0.1 # ダメージの何%回復するか（0.1 = 10%）
    Chance: 1.0
    Cooldown_Ticks: 0
```

---

### Switch_Lock

**武器を持ち替えたあと一定時間、射撃をロックする機能です（抜刀モーション表現などに使用）。**

```yaml
武器名:
  Switch_Lock:
    Enable: true
    Lock_Ticks: 10 # ロック時間（tick）
    Shoot_Block_Message: "&c構え中..." # ロック中に射撃しようとしたときのメッセージ
    Sound: "ITEM_ARMOR_EQUIP_GENERIC-1.0-1.0"
    Delay_Bar:
      Action_Bar: "&7構え中... {bar}"
      End_Action_Bar: "&a射撃可能！"
      End_Sound: "ENTITY_PLAYER_LEVELUP-1.0-1.0"
      Symbol: "|"
      Symbol_Amount: 10
      Left_Color: "&e"
      Right_Color: "&7"
```

---

### Custom_Model_Data_Held

**武器を手に持っている間だけカスタムモデルデータを変更する機能です。持ち替え・ドロップ・インベントリ操作時に自動で元に戻ります。**

```yaml
武器名:
  Custom_Model_Data_Held: 1001 # 手持ち中に適用するモデルID
```

---

### Delay_Bar

各機能の `Delay_Bar` セクションで使える共通のプログレスバー設定です。

| キー             | 型     | 説明                                     |
| ---------------- | ------ | ---------------------------------------- |
| `Action_Bar`     | string | 表示文字列（`{bar}` がバーに置換される） |
| `End_Action_Bar` | string | 完了時に表示する文字列                   |
| `End_Sound`      | string | 完了時の効果音                           |
| `Symbol`         | string | バーのシンボル文字                       |
| `Symbol_Amount`  | int    | シンボルの総数                           |
| `Left_Color`     | string | 経過分の色                               |
| `Right_Color`    | string | 残り分の色                               |

---

## WeaponsSPMode 機能リファレンス

### WhenChangeWeapon

**特定のアクションをトリガーに武器を別の武器へ切り替える機能です。**

```yaml
武器名:
  WhenChangeWeapon:
    Enable: true

    # トリガー別の変更先武器
    Shift: "武器名" # シフトキー
    Empty_Ammo: "武器名" # 弾切れ時
    Offhand: "武器名" # Fキー単体
    Off_And_Shift: "武器名" # シフト + Fキー
    Jump_And_Shift: "武器名" # シフト + ジャンプ
    Jump_And_Off: "武器名" # ジャンプ + Fキー

    Sound: "SOUND名-VOLUME-PITCH"
    Message: "&a武器変更！"
    Title: "&6タイトル"
    Subtitle: "&eサブタイトル"
    CoolDown: 20 # クールダウン（tick）

    # 条件：特定アイテムを所持しているときのみ発動
    If_HaveItems:
      Shift: # トリガー名ごとに設定
        Item_type: "DIAMOND" # マテリアル名（大文字）
        Item_name: "特別なダイヤ" # 表示名（省略可）
        Values: 1 # 必要個数

    # キルストリーク到達時に自動変更
    Kill_Streak:
      3: "武器名" # 3キルで変更
      5: "武器名" # 5キルで変更

    # 時間経過で自動的に武器を変更（付与した瞬間からカウント開始）
    Timed_Change:
      Delay_Ticks: 200
      Target_Weapon: "変更先武器名"
      Sound: "SOUND名-VOLUME-PITCH"
      Message: "&a変換完了！"
      Delay_Bar:
        Enable: true
        Action_Bar: "&7変換中... {bar}"
        End_Action_Bar: "&a変換完了！"
        End_Sound: "ENTITY_PLAYER_LEVELUP-1.0-1.0"
        Symbol: "|"
        Symbol_Amount: 15
        Left_Color: "&a"
        Right_Color: "&c"
```

---

### KillStreak

**武器ごとのキルストリークを管理し、ストリーク消費でイベントを発動する機能です。**

```yaml
武器名:
  KillStreak:
    Enable: true
    Kill_Count: 5
    Streak_Share_Key: "グループ名" # 同じキーの武器でストリークを共有
    Ignore_Limit: false # true で上限超えカウントを許可
    Remove_Streak: true # 死亡時にリセットするか
    Remove_Several_Streak: -1 # 死亡時の削除数（-1 で全削除）
    Remove_Server_Move: true # サーバー移動時に全削除するか
    Accumulate_Streaks_Defeat_Mobs: true # モブキルでも貯まるか

    # アクションバーへのカウンター表示（持ち替えなしで常時表示）
    Streak_Icon:
      Enable: true
      Left: "&4&l▶ " # 貯まった分のシンボル（カラーコード含む）
      Right: "&7&l◀ " # 未貯蔵分のシンボル
      Bar: "&7魂集め... {bar}" # 省略時は Left/Right を並べるだけ

    Streak_Event:
      Enable: true

      # 通常消費（指定数を消費して発動）
      Cost_Count:
        Cost_Count: 3
        Change_Weapons: "変更先武器名"
        Change_Weapons_WithAction: "shift" # shift / offhand / jump（カンマ区切り複数可）
        Takeover_Streak: false # 変更先にストリークを引き継ぐか
        Cmd: "say #shooter# が発動！" # コンソールコマンド（#shooter# = プレイヤー名）
        Actionbar: "&e発動！"
        Saychat: "&e自分だけに見えるメッセージ"
        Title: "&6タイトル"
        Subtitle: "&eサブタイトル"
        Sound: "SOUND名-VOLUME-PITCH"
        Notenough_Actionbar: "&cストリーク不足！" # 不足時（2秒表示後カウンターに戻る）
        Notenough_Saychat: "&cストリーク不足！"

      # 満タン消費（Kill_Count 到達時のみ発動）
      # Cost_Count と異なるアクションなら両立する
      # 同じアクションの場合は Cost_MaxCount が優先される
      Cost_MaxCount:
        Change_Weapons: "変更先武器名"
        Change_Weapons_WithAction: "offhand"
        Saychat: "&a発動！"
        Sound: "SOUND名-VOLUME-PITCH"
        Notenough_Actionbar: "&c満タンじゃないと使えない！"
```

#### Change_Weapons_WithAction の選択肢

| 値        | 発動条件                            |
| --------- | ----------------------------------- |
| `shift`   | シフトキーを押したとき              |
| `offhand` | Fキー（オフハンドキー）を押したとき |
| `jump`    | ジャンプしたとき                    |

#### Streak_Share_Key について

同じキーを持つ武器同士はストリークを共有します。異なるグループには干渉しません。

```yaml
MCX:
  KillStreak:
    Streak_Share_Key: "MCX_group" # MCX と MCX3 でストリーク共有

MCX3:
  KillStreak:
    Streak_Share_Key: "MCX_group"

M4A1:
  KillStreak:
    Streak_Share_Key: "M4_group" # 別グループ。MCX_group に干渉しない
```

---

## 設定例

MCX → MCX2（Fキー・満タン時） / MCX3（シフト・3カウント消費）に変化し、MCX2は10秒後に自動でMCXに戻る構成です。

```yaml
MCX:
  KillStreak:
    Enable: true
    Kill_Count: 5
    Streak_Share_Key: "MCX_group"
    Streak_Icon:
      Enable: true
      Left: "&4&l▶ "
      Right: "&7&l◀ "
      Bar: "&7魂集め... {bar}"
    Ignore_Limit: false
    Remove_Streak: true
    Remove_Several_Streak: -1
    Remove_Server_Move: true
    Accumulate_Streaks_Defeat_Mobs: true
    Streak_Event:
      Enable: true
      Cost_Count:
        Cost_Count: 3
        Change_Weapons: "MCX3"
        Change_Weapons_WithAction: "shift"
        Saychat: "&e生きるために!"
        Title: "&6変身！"
        Subtitle: "&eMCX3 装備"
        Sound: "ENTITY_ARROW_HIT_PLAYER-0.8-1.8"
        Notenough_Saychat: "&eストリーク不足！"
      Cost_MaxCount:
        Change_Weapons: "MCX2"
        Change_Weapons_WithAction: "offhand"
        Saychat: "&aFireFly 起動..."
        Sound: "ENTITY_ENDERMAN_TELEPORT-1.0-1.5"
        Notenough_Actionbar: "&c不足"

MCX2:
  WhenChangeWeapon:
    Enable: true
    Timed_Change:
      Delay_Ticks: 200
      Target_Weapon: "MCX"
      Sound: "ENTITY_IRON_GOLEM_DEATH-0.6-0.8"
      Message: "&a武器変更！"
      Delay_Bar:
        Enable: true
        Action_Bar: "&7武器変更中... {bar}"
        End_Action_Bar: "&a武器変更完了！"
        End_Sound: "ENTITY_PLAYER_LEVELUP-1.0-1.2"
        Symbol: "|"
        Symbol_Amount: 15
        Left_Color: "&a"
        Right_Color: "&c"

MCX3:
  KillStreak:
    Enable: true
    Kill_Count: 5
    Streak_Share_Key: "MCX_group"
    Streak_Icon:
      Enable: true
      Left: "&4&l▶ "
      Right: "&7&l◀ "
  WhenChangeWeapon:
    Enable: true
    Shift: "MCX"
    Sound: "ENTITY_ARROW_HIT_PLAYER-0.8-0.6"
```

---

## カラーコード

通常の `&` カラーコードに加え、1.16以降の16進数カラーコードに対応しています。

```
&x&R&R&G&G&B&B
```

| 色       | HEX       | 記法             |
| -------- | --------- | ---------------- |
| オレンジ | `#FF6600` | `&x&f&f&6&6&0&0` |
| 水色     | `#00CCFF` | `&x&0&0&c&c&f&f` |
| ゴールド | `#FFD700` | `&x&f&f&d&7&0&0` |

```yaml
Left: "&x&f&f&6&6&0&0&l▶ "
```

> **注意：** 16進数カラーコードは `WeaponsSPMode` の `Streak_Icon` / `Saychat` / `Title` / `Subtitle` など文字列系の設定で有効です。`UniversalWeaponSystem` 側は通常の `&` コードのみ対応しています。

---

## サウンド記法

```
SOUND名-VOLUME-PITCH
SOUND名-VOLUME-PITCH-DELAY_TICK
```

カンマ区切りで複数同時指定が可能です。

```yaml
Sound: "ENTITY_ARROW_HIT_PLAYER-0.8-1.8,ENTITY_ENDERMAN_TELEPORT-1.0-1.5"
```

| フィールド | 説明                          | デフォルト |
| ---------- | ----------------------------- | ---------- |
| SOUND名    | Bukkit Sound 列挙名（大文字） | 必須       |
| VOLUME     | 音量                          | `1.0`      |
| PITCH      | 音程（0.5〜2.0）              | `1.0`      |
| DELAY_TICK | 遅延tick数（省略可）          | `0`        |

---

## 名前変更武器の作り方

`/aug ncmanual` でも確認できます。

1. CrackShot 側でネームド武器（名前変更武器）を作成する
2. `weapons/` フォルダにある変更元の武器クラスをコピーする
3. コピーしたクラスの武器IDを CrackShot の `weaponID` に合わせて変更する
4. `plugins/AUG/weapons/AUG_NAME_CHANGE.yml` に登録する
5. `AgetarouUniqueGuns.java` の `onEnable` にリスナーを登録する
6. `/aug reload` で反映する

---

_このプラグインは [AzisabaNetwork](https://github.com/AzisabaNetwork) 向けに開発されました。_
