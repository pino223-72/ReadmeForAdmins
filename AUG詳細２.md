# AgetarouUniqueGuns — 汎用武器システム 完全設定リファレンス

対象：`UniversalWeaponSystem` / `WeaponsSPMode` / `StreakParticleEffect`
これらはすべて `WeaponConfig`（= `weapons/*.yml` の武器名トップレベルキー）を共有して読みます。**1つの武器に対して両システムの設定を同時に書くことも可能**です。

## 目次
1. [全体スケルトン（YAMLイメージ）](#全体スケルトンyamlイメージ)
2. [UniversalWeaponSystem 完全リファレンス](#universalweaponsystem-完全リファレンス)
3. [WeaponsSPMode 完全リファレンス](#weaponsspmode-完全リファレンス)
4. [パーティクル機能（StreakParticleEffect）完全リファレンス](#パーティクル機能streakparticleeffect完全リファレンス)
5. [実装上の癖・ハマりどころまとめ](#実装上の癖ハマりどころまとめ)

---

## 全体スケルトン（YAMLイメージ）

```yaml
MyWeapon:                       # ← CSのWeaponTitleと一致させる
  Original: SomeBaseWeapon      # 任意：名前変更(NC)武器の場合のみ
  Return_Base_Weapon: SomeWeapon # 任意：死亡復元時に別武器を渡したい場合のみ
  Custom_Model_Data_Held: 1001  # 任意：持っている間だけ差し替えるCMD（ルート直下）

  # ── UniversalWeaponSystem ──
  Instant_Reload: { }
  On_Hit_Explosion: { }
  Sneak_Explosive_Shot: { }
  On_Hit_Recovery: { }
  Headshot_Ammo_Recovery: { }
  Life_Steal: { }
  On_Damaged_Effect: { }
  Switch_Lock: { }

  # ── WeaponsSPMode ──
  WhenChangeWeapon: { }
  KillStreak: { }
```

---

## UniversalWeaponSystem 完全リファレンス

### 機能別・対応キー早見表

| セクション | Enable | Chance | Require_Sneak | Required_Effect | Cooldown_Ticks | Custom_Model_Data_CD | Delay_Bar | Message/Sounds |
|---|---|---|---|---|---|---|---|---|
| `Instant_Reload` | ○ | ×（無条件） | × | × | ○ | ○ | ○ | ○ |
| `On_Hit_Explosion` | ○ | △専用キー | ○ | ○ | ○ | ○ | ○ | ○ |
| `Sneak_Explosive_Shot` | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| `On_Hit_Recovery` | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| `Headshot_Ammo_Recovery` | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| `Life_Steal` | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| `On_Damaged_Effect` | ○ | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| `Switch_Lock` | ○ | ×非対応 | ×非対応 | ×非対応 | ×（`Lock_Ticks`で代替） | ×非対応 | ○ | ○ |

`On_Hit_Recovery`/`Headshot_Ammo_Recovery`/`Sneak_Explosive_Shot`/`Life_Steal` は `executeGenericFeature()` という共通ヘルパー経由で発火するため、上表の共通キー（`Enable`〜`Sounds`）は全部同じロジックで判定されます。`Instant_Reload`と`Switch_Lock`だけは専用ロジックなので、一部キーが非対応になっている点に注意してください。

### 共通キーの意味

| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Enable` | bool | `false` | セクションを有効化するトップレベルスイッチ。これがfalse/未指定だと機能ごと無視されます |
| `Chance` | double | `1.0` | 発動確率（`ThreadLocalRandom`で判定）。対応セクションのみ |
| `Require_Sneak` | bool | `false` | trueの場合、スニーク中でないと発動しない |
| `Required_Effect` | string | なし | 保有ポーション効果名（カンマ区切り可）。**⚠後述の癖に注意** |
| `Cooldown_Ticks` | int | `0` | 発動後のクールダウン（Tick）。0以下ならクールダウンなし |
| `Custom_Model_Data_CD` | int | なし | クールダウン中だけ差し替えるカスタムモデルデータ。キーが存在する時のみ動作し、クールダウン終了で元のモデルに自動復元 |
| `Delay_Bar` | section | なし | クールダウン進捗をアクションバーに表示（後述） |
| `Message` | string | なし | 発動時にアクションバーへ1回だけ表示するメッセージ |
| `Sounds` / `Sound` | string | なし | 発動時の効果音。どちらのキー名でも動作（`Sounds`優先）。書式は下記参照 |

**サウンド書式**：`SOUND_NAME-音量-ピッチ-遅延Tick` をカンマ区切りで複数指定可能。音量・ピッチ・遅延は省略可（既定 `1.0` / `1.0` / `0`）。
例：`ENTITY_PLAYER_LEVELUP-1.0-1.2-0,ENTITY_PLAYER_LEVELUP-1.0-1.5-4`

### 各セクション固有キー

**`Instant_Reload`**（スニークで即リロード）
| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Add_Amount` | int | `1` | スニーク1回あたりの弾数回復量 |
| `Reload_On_Sneak` | bool | `false` | trueの場合、リロード中にスニークしていればリロード時間を1Tickに短縮し満タン回復 |

**`On_Hit_Explosion`**（着弾時爆発）
| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Explosion_Chance_Block` | double | `0.0` | ブロック着弾時の爆発確率 |
| `Explosion_Chance_Entity` | double | `0.0` | エンティティ着弾時の爆発確率 |

爆発は `API.getCSUtility().generateExplosion()` を使うため、実際の爆風・威力設定はCS側の `Explode` 設定に従います。多重発火防止の静的ロック(`isExplosionLock`)あり。

**`Sneak_Explosive_Shot`**（スニーク射撃を爆発弾に変換）
| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Extra_Ammo_Cost` | int | `1` | 追加で消費する弾数。所持弾数が足りない場合は発動しない |

**`Life_Steal`**（吸血）
| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Heal_Percent` | double | `0.1` | 与ダメージに対する回復割合（0.1＝10%） |

**`On_Damaged_Effect`**（被弾時効果付与）
| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Self_Potion_Effects` | section | なし | `<効果名>: {Duration, Amplifier, Ambient, Particles, Icon}` の形式で被弾者自身に付与 |
| `Attacker_Potion_Effects` | section | なし | 同形式で攻撃者に付与 |

被弾者側の武器（メインハンド）の`On_Damaged_Effect`設定が参照される点に注意（攻撃者側ではありません）。`Self_Potion_Effects`と`Attacker_Potion_Effects`は最低どちらか1つが実際に何か付与できないと、クールダウン・フィードバックとも発火しません。

**`Switch_Lock`**（持ち替え直後の射撃ロック）
| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Lock_Ticks` | int | `0` | ロック時間（Tick）。0以下ならロックなし |
| `Shoot_Block_Message` | string | なし | ロック中に射撃しようとした際のアクションバー表示（`root`直下から参照される点に注意） |

**`Custom_Model_Data_Held`**（ルート直下・機能横断）
| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Custom_Model_Data_Held` | int | `-1` | 武器を構えている間だけ適用するCMD。手放す／インベントリクリック／ドロップで元のCMDに自動復元 |

**`Delay_Bar`**（各セクション共通のプログレスバー）
| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Action_Bar` | string | なし | `{bar}` を埋め込んでバーを表示するテンプレート文字列 |
| `Symbol` | string | `\|` | バーの1マスに使う文字 |
| `Symbol_Amount` | int | `15` | バーの総マス数 |
| `Left_Color` | string | `&a` | 経過分の色 |
| `Right_Color` | string | `&c` | 未経過分の色 |
| `End_Action_Bar` | string | なし | クールダウン終了時に表示するメッセージ |
| `End_Sound` | string | なし | クールダウン終了時のサウンド |

UniversalWeaponSystemの`Delay_Bar`は`Enable`キー不要（親セクションに`Delay_Bar`が存在するだけで動作開始）です。

---

## WeaponsSPMode 完全リファレンス

### `WhenChangeWeapon`（トリガー式武器変更）

```yaml
WhenChangeWeapon:
  Enable: true
  Shift: TargetWeaponA
  Empty_Ammo: TargetWeaponB
  Off_And_Shift: TargetWeaponC
  Jump_And_Shift: TargetWeaponD
  CoolDown: 40
  Sound: "ITEM_ARMOR_EQUIP_IRON-1.0-1.0-0"
  Message: "&e武器を切り替えた！"
  Title: "&6CHANGE"
  Subtitle: "&7now equipped"
  Shift_Takeover_Ammo: true
  Empty_Ammo_Takeover_Ammo: false
  If_HaveItems:
    Shift:
      Item_type: DIAMOND
      Item_name: "&b特殊弾"
      Values: 1
  Join_Change:
    Enable: true
    Target_Weapon: TargetWeaponE
    Takeover_Ammo: false
  Timed_Change:
    Delay_Ticks: 200
    Target_Weapon: TargetWeaponF
    Takeover_Ammo: false
    Sound: "BLOCK_ANVIL_USE-1.0-1.0-0"
    Message: "&c時間切れで変化した"
    Delay_Bar:
      Enable: true
      Action_Bar: "&7残り変化まで: {bar}"
      Symbol: "|"
      Symbol_Amount: 15
      Left_Color: "&a"
      Right_Color: "&c"
      End_Action_Bar: "&c変化完了！"
      End_Sound: "ENTITY_ENDERMAN_TELEPORT-1.0-1.0-0"
  Kill_Streak:
    "3": TargetWeaponG
    "5": TargetWeaponH
  Return_Cooldown:
    Enable: true
    Ticks: 100
    NotReady_Actionbar: "&cまだ戻せません"
    Action_Bar: "&7戻れるまで: {bar}"
    End_Action_Bar: "&aいつでも戻せます"
    End_Sound: "BLOCK_NOTE_BLOCK_PLING-1.0-1.0-0"
    Symbol: "|"
    Symbol_Amount: 15
    Left_Color: "&a"
    Right_Color: "&7"
```

| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Enable` | bool | `false` | セクション全体の有効化 |
| `Shift` / `Empty_Ammo` / `Off_And_Shift` / `Jump_And_Shift` | string | なし | 各トリガー発生時に変更する先の武器名。未設定のトリガーは無効 |
| `CoolDown` | int | `0` | **全トリガー共通の**クールダウン（Tick）。トリガー種別ごとに個別のクールダウンではない点に注意 |
| `Sound` / `Message` / `Title` / `Subtitle` | string | なし | **全トリガー共通で**発火するフィードバック。どのトリガーで変化しても同じ内容が流れます |
| `<TriggerType>_Takeover_Ammo` | bool | `false` | `Shift_Takeover_Ammo`のようにトリガー名を接頭辞にして個別指定。弾数を変更先武器に引き継ぐか |
| `If_HaveItems.<TriggerType>` | section | なし | `Shift`/`Empty_Ammo`/`Off_And_Shift`/`Jump_And_Shift`にのみ対応（Timed_Change/Join_Change/Kill_Streakには非対応）。`Item_type`（Material名）または`Item_name`（表示名・色コード込みで完全一致）のいずれかを、合計`Values`個以上所持していないと変更が発動しない |
| `Join_Change.Enable/Target_Weapon/Takeover_Ammo` | - | - | サーバー参加20Tick後に、インベントリ・オフハンド内の対象武器を自動変換 |
| `Timed_Change.Delay_Ticks/Target_Weapon/Takeover_Ammo/Sound/Message` | - | - | 武器を持ってから`Delay_Ticks`後に自動変更。武器を手放すとタイマーは自動キャンセル |
| `Timed_Change.Delay_Bar` | section | - | `Enable:true`が必須（UWSと違い明示指定が必要）。以下`Delay_Bar`共通キーは全て同じ形式 |
| `Kill_Streak."<数値>"` | string | なし | そのキル数に到達した瞬間に発動する武器変更。YAML上はキー名なので文字列クォート推奨 |
| `Return_Cooldown.Enable/Ticks` | - | - | 変更先武器から**同じトリガー経路で元に戻す動作だけ**を`Ticks`の間ロック。ロック中に戻そうとすると`NotReady_Actionbar`を表示 |
| `Return_Cooldown.Action_Bar〜Right_Color` | - | - | 変更先武器を構えている間、クールダウン進捗バーを表示（`Right_Color`既定値が`&7`。UWS/`Timed_Change`の`&c`と異なるので注意） |

ルート直下（`WhenChangeWeapon`の外）にある付随キー：

| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Return_Base_Weapon` | string | 変更元武器名 | プレイヤー死亡時、変更後武器を元の武器に復元する際、記録された元武器の代わりに渡す武器名を上書きできる |

### `KillStreak`（キルストリーク蓄積＆演出）

```yaml
KillStreak:
  Enable: true
  Kill_Count: 5
  Ignore_Limit: false
  Accumulate_Streaks_Defeat_Mobs: false
  Streak_Share_Key: SharedGroupA
  Remove_Streak: true
  Remove_Several_Streak: -1
  Streak_Icon:
    Enable: true
    Left: "&4&l▶ "
    Right: "&7&l◀ "
    Bar: "&8[{bar}&8]"
  Streak_Event:
    Enable: true
    Cost_MaxCount:
      Trigger_Action: "shift"
      Notenough_Actionbar: "&c満タンではありません"
      # ... 効果キー群は下記共通表参照
    Cost_Count:
      Trigger_Action: "offhand"
      Cost_Count: 2
      Notenough_Actionbar: "&cキルストリークが足りません"
      Notenough_Saychat: "&cあと{2}キル必要"
      # ... 効果キー群は下記共通表参照
```

| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Enable` | bool | `false` | セクション全体の有効化 |
| `Kill_Count` | int | `5` | ストリーク上限（満タン判定の基準にも使用） |
| `Ignore_Limit` | bool | `false` | trueの場合、上限を超えて無制限に蓄積 |
| `Accumulate_Streaks_Defeat_Mobs` | bool | `false` | trueの場合モブキルも加算対象にする（既定はプレイヤーキルのみ） |
| `Streak_Share_Key` | string | 武器名 | 指定すると、異なる武器名でもこのキーが同じならストリークを共有 |
| `Remove_Streak` | bool | `true` | プレイヤー死亡時にストリークをリセットするか |
| `Remove_Several_Streak` | int | `-1` | `-1`＝全リセット。0以上を指定するとその数だけ減算（全消去しない） |
| `Streak_Icon.Enable` | bool | `false` | アクションバーへのストリーク常時表示を有効化 |
| `Streak_Icon.Left` / `Right` | string | `&4&l▶ ` / `&7&l◀ ` | `Bar`未設定時に、現在値の数だけ`Left`を、残りの数だけ`Right`を並べて表示 |
| `Streak_Icon.Bar` | string | なし | 指定した場合、`Left`/`Right`を並べた文字列を`{bar}`として埋め込むテンプレート形式に切り替わる |

武器のキルストリークは**アイテム個体（PersistentDataContainerの`weapon_instance`）単位**で管理されるため、同名武器を複数所持していても基本的には別々にカウントされます（`Streak_Share_Key`を揃えた場合のみ共有）。

### `Streak_Event`（ストリーク消費イベント）

`Cost_MaxCount`（満タン到達時）と`Cost_Count`（任意消費量到達時）の2セクションがあり、**セクション名と、その中の同名キー`Cost_Count`（消費量）は別物**なので注意してください。

| セクション | 発火条件 | 消費量 |
|---|---|---|
| `Cost_MaxCount` | 現在ストリーク数 ≧ `Kill_Count`（満タン） | 常に`Kill_Count`と同じ数を全消費 |
| `Cost_Count` | 現在ストリーク数 ≧ `Cost_Count`キーの値 | `Cost_Count`キーで指定した数だけ消費（既定`1`） |

両セクション共通の判定・効果キー：

| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Trigger_Action` | string | なし | `"shift"` / `"offhand"` / `"jump"` のいずれか、またはカンマ区切りでAND条件化（例："shift,offhand"）。未指定の場合キル瞬間に即発火（`Cost_Count`は`triggerAction`が`null`のときスキップされる仕様のため、実質`Cost_MaxCount`のみで使うのが自然） |
| `Cost_Count`（`Cost_Count`セクション内のみ） | int | `1` | 消費するストリーク数 |
| `Notenough_Actionbar` | string | なし | ストリーク不足時にアクションバー表示 |
| `Notenough_Saychat` | string | なし | ストリーク不足時にチャット送信 |
| `Potion_Effects.<効果名>` | section | なし | `{Duration, Amplifier, Ambient, Particles, Icon}`。自分自身に付与 |
| `Ally_Potion_Effects.Radius` / `.Effects` | double / section | `8.0` | 半径内の**同チーム**プレイヤーに`Effects`内の効果を付与（形式は`Potion_Effects`と同じ） |
| `Enemy_Potion_Effects.Radius` / `.Effects` | double / section | `8.0` | 半径内の**敵チーム**プレイヤーに付与 |
| `Particle_Effect` | section | なし | パーティクル演出（詳細は次章） |
| `Heal` | double | `0.0` | 自身のHP回復量 |
| `Food` | int | `0` | 満腹度の増減 |
| `Self_Damage` | double | `0.0` | 自傷ダメージ |
| `Change_Weapons` | string | なし | 消費と引き換えに指定武器へ変更 |
| `Takeover_Streak` | bool | `false` | 変更先武器へ、消費後の残りストリーク数を引き継ぐか |
| `Takeover_Ammo` | bool | `false` | 変更先武器へ弾数を引き継ぐか |
| `Cmd` | string | なし | コンソールコマンド実行。`#shooter#`をプレイヤー名に置換 |
| `Actionbar` | string | なし | 発動時アクションバー |
| `Saychat` | string | なし | 発動時チャット送信（`&`カラーコード対応） |
| `Title` / `Subtitle` | string | なし | 発動時タイトル表示（表示時間は固定：フェードイン10/表示70/フェードアウト20Tick） |
| `Sound` | string | なし | 発動時サウンド |

`Trigger_Action`の判定ロジック：単体指定時は完全一致（`jump`は移動イベントで検出したジャンプ状態フラグでも代替可）。複数指定（カンマ区切り）時はAND条件になり、**全ての条件を満たさないと発火しません**。

---

## パーティクル機能（StreakParticleEffect）完全リファレンス

`Streak_Event`の`Cost_MaxCount`/`Cost_Count`内、`Particle_Effect`セクションから呼び出されます。

```yaml
Particle_Effect:
  Enable: true
  Particle: FLAME
  Shape: CIRCLE          # POINT / CIRCLE / SPHERE / HELIX / BURST
  Duration_Ticks: 40
  Period_Ticks: 2
  Follow_Player: true
  Y_Offset: 1.0
  Radius: 1.5
  Points: 24
  Rotate_Degrees_Per_Tick: 6.0
  Height: 2.0
  Turns: 2.0
  Count: 1
  Offset_X: 0.0
  Offset_Y: 0.0
  Offset_Z: 0.0
  Extra: 0.0
  Red: 255
  Green: 0
  Blue: 0
  Size: 1.0
```

| キー | 型 | 既定値 | 説明 |
|---|---|---|---|
| `Enable` | bool | `false` | パーティクル演出全体の有効化 |
| `Particle` | string | `FLAME` | `org.bukkit.Particle`の列挙名。不正な値は演出全体がスキップされる |
| `Shape` | string | `CIRCLE` | `POINT`（1点）/`CIRCLE`（水平円）/`SPHERE`（球面フィボナッチ分布）/`HELIX`（螺旋）/`BURST`（1点噴出、実質POINTと同じ実装）。未対応値もCIRCLE扱い |
| `Duration_Ticks` | int | `40` | 演出継続時間（最低1に補正） |
| `Period_Ticks` | int | `2` | 描画間隔（最低1に補正） |
| `Follow_Player` | bool | `true` | trueなら毎フレーム、プレイヤーの現在地基準で中心座標を再計算。falseなら発動時点の座標に固定 |
| `Y_Offset` | double | `1.0` | 中心座標のY方向オフセット |
| `Radius` | double | `1.5`（`HELIX`のみ既定`1.2`） | `CIRCLE`/`SPHERE`/`HELIX`で使用する半径 |
| `Points` | int | `24`（`CIRCLE`/`HELIX`）／`48`（`SPHERE`、最低8に補正） | 配置する粒子の点数 |
| `Rotate_Degrees_Per_Tick` | double | `0.0`（`CIRCLE`）／`12.0`（`HELIX`） | 経過Tickに応じた回転速度（度／Tick） |
| `Height` | double | `2.0` | `HELIX`専用：螺旋の縦幅 |
| `Turns` | double | `2.0` | `HELIX`専用：巻き数 |
| `Count` | int | `1` | 1点あたりの粒子数（`world.spawnParticle`の第3引数） |
| `Offset_X`/`Offset_Y`/`Offset_Z` | double | `0.0` | 1点あたりの座標ランダムばらつき幅 |
| `Extra` | double | `0.0` | パーティクル種別ごとの追加パラメータ（速度など。種類依存） |
| `Red`/`Green`/`Blue` | int(0-255) | `255`/`255`/`255` | `REDSTONE`（旧名: DUST）パーティクル専用の色指定。範囲外は自動クランプ |
| `Size` | double | `1.0` | `REDSTONE`パーティクル専用のサイズ |

`Particle`が`REDSTONE`の場合のみ`Red`/`Green`/`Blue`/`Size`が使われ、それ以外の粒子種では無視されます。

---

## 実装上の癖・ハマりどころまとめ

以下は設定を書く上で見落としやすい実装上の挙動です。動作確認前に把握しておくと事故を防げます。

1. **`Required_Effect`をカンマ区切りで複数指定しても、実質「先頭1個」しかチェックされません。** ループの1周目で先頭の効果を保有していなければその時点で`return false`になり、2個目以降の効果は評価されない実装になっています（OR条件として複数書いても機能しない点に注意）。
2. **`WhenChangeWeapon`の`Sound`/`Message`/`Title`/`Subtitle`/`CoolDown`はトリガー種別ごとではなく全トリガー共通です。** `Shift`と`Empty_Ammo`で別々の音・メッセージを鳴らしたい場合は非対応なので、演出を変えたいなら武器を分けるなどの工夫が必要です。
3. **バーの`Right_Color`既定値が箇所によって違います。** `UniversalWeaponSystem`の`Delay_Bar`と`WeaponsSPMode`の`Timed_Change.Delay_Bar`は既定`&c`ですが、`Return_Cooldown`のバーだけ既定`&7`です。明示的に指定しておくと安全です。
4. **`If_HaveItems`は`Shift`/`Empty_Ammo`/`Off_And_Shift`/`Jump_And_Shift`にのみ有効**で、`Timed_Change`・`Join_Change`・`Kill_Streak`経由の武器変更には適用されません。
5. **`Cost_MaxCount`セクションには消費量指定キーがありません。** 満タン時イベントは常に`KillStreak.Kill_Count`と同じ数を全消費する仕様です。数量を変えたい場合は`Cost_Count`セクション側で運用してください。
6. **`Custom_Model_Data_Held`はルート直下、`Custom_Model_Data_CD`は各機能セクション内**、と置き場所が異なります。同じ「モデル差し替え」でも役割（持っている間／クールダウン中）が別物なので混同注意です。
7. **`Instant_Reload`と`Switch_Lock`は`Require_Sneak`/`Required_Effect`/`Chance`に非対応**です。他の機能セクションと同じノリで書いても無視されます。

---

📌 本ドキュメントは実装（javaソース）を直接読んで作成しています。挙動が古いバージョンと異なる可能性があるため、実機での動作確認と併用してください。
