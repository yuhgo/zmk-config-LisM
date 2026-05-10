---
name: zmk-keymap-editor
description: ZMK keymap・combos・macros・hold-tap・*.conf・build.yaml・west.yml を安全に編集するスキル。レイヤー追加、キー差し替え、コンボ追加、マクロ作成、ホールドタップ behavior 定義、Kconfig 変更、ビルドターゲット追加に使用。「キーマップ編集」「コンボ追加」「マクロ作って」「レイヤー追加」「ZMK 設定変更」「behavior 追加」などで起動。
---

# ZMK Keymap Editor

LisM 専用の ZMK 設定編集スキル。devicetree 構文を直接編集する層と、ZMK CLI（リポジトリ管理）層を組み合わせて使う。

## いつ使うか

以下のような依頼で起動する:

- 「default レイヤーの A を B に変えて」
- 「新しいレイヤーを追加して」
- 「キー位置 11 と 12 で TAB のコンボを追加して」
- 「絵文字入力のマクロを作って」
- 「LCTRL を hold したら shift、tap したら A の behavior を作って」
- 「スリープ時間を 10 分に変更して」
- 「ZMK Studio 対応のビルドターゲットを追加して」
- 「新しい ZMK モジュールを追加して」

## ファイル構造

| ファイル | 編集対象 |
|---------|---------|
| `config/lism.keymap` | レイヤー、bindings、combos、macros、behaviors |
| `config/lism_left.conf` / `config/lism_right.conf` | Kconfig（スリープ、Bluetooth、RGB） |
| `config/lism.json` | 物理レイアウト（ZMK Studio 用、通常編集不要） |
| `config/west.yml` | 依存モジュール |
| `build.yaml` | ビルドターゲット定義 |

## 二層アーキテクチャ

このスキルは2つの層で構成されている:

### 1. devicetree 編集層

`config/lism.keymap` の中身（keymap/combos/macros/behaviors）を Edit/Write で直接書き換える。
ZMK CLI には keymap を構造化編集する API は無く、`zmk code` はエディタで開くだけ。

### 2. ZMK CLI ラッパー層

リポジトリ操作（モジュール追加、設定管理、更新）は ZMK CLI を使う:

- `zmk module add <url>` — モジュール追加
- `zmk update` — ZMK・モジュール更新
- `zmk keyboard list` — サポートキーボード一覧

詳細は [references/zmk-cli.md](references/zmk-cli.md)。

## リファレンス（深掘り用）

| ファイル | 内容 |
|---------|------|
| [references/lism-layout.md](references/lism-layout.md) | LisM 物理配置・key-position マップ |
| [references/keymap-basics.md](references/keymap-basics.md) | keymap 基本構文・レイヤー定義 |
| [references/keycodes.md](references/keycodes.md) | キーコード一覧 |
| [references/combos.md](references/combos.md) | コンボの定義方法 |
| [references/macros.md](references/macros.md) | マクロの定義方法 |
| [references/hold-tap.md](references/hold-tap.md) | hold-tap behavior の flavor と書き方 |
| [references/kconfig.md](references/kconfig.md) | `*.conf` 主要オプション |
| [references/build-yaml.md](references/build-yaml.md) | build.yaml スキーマ |
| [references/west-yml.md](references/west-yml.md) | west.yml モジュール管理 |
| [references/zmk-cli.md](references/zmk-cli.md) | ZMK CLI コマンド一覧 |

## 共通の編集フロー

1. **影響範囲を特定**: 何のレイヤー・どの key-position を変更するかユーザーに確認
2. **該当ファイルを Read**: 編集前の状態を必ず読み取る
3. **devicetree 構文の整合性を保ったまま Edit**: 括弧 `{}`, セミコロン `;`, アングルブラケット `<>` を厳守
4. **`git diff` で変更を提示**: 変更箇所を明示してユーザー確認
5. **オプション: ビルド検証**: `--verify` または明示要求があれば `build-firmware` skill を呼び出す

## キー差し替えの基本

例: default_layer のキー位置 0（左上の Q）を A に変更する場合:

1. `config/lism.keymap` の `default_layer` ブロックを Read
2. bindings の最初のトークン `&kp Q` を `&kp A` に差し替え
3. `git diff config/lism.keymap` を提示

```diff
         default_layer {
             bindings = <
-&kp Q             &kp W         ...
+&kp A             &kp W         ...
```

## レイヤー追加

新規レイヤーを追加する場合の手順:

1. ファイル先頭の `#define` で `<NEW>_LAYER N` を定義（既存レイヤー番号と衝突しないこと）
2. `keymap` ノード内に `<name>_layer { ... };` を追加
3. `display-name`, `bindings`（4×11 のキーバインド）を記述
4. `combos` の `layers = <...>` プロパティに新レイヤー番号を追加（必要なら）
5. 既存レイヤーから `&mo N` / `&lt N KEY` / `&to N` で遷移可能にする

詳細は [references/keymap-basics.md](references/keymap-basics.md) を参照。

## コンボ追加

例: キー位置 0 と 1 で ESC を発火するコンボ:

```dts
combos {
    compatible = "zmk,combos";
    /* 既存 ... */

    esc_combo {
        bindings = <&kp ESC>;
        key-positions = <0 1>;
        timeout-ms = <50>;
        layers = <0>;          // ← 子で明示すると親 combos の layers 設定を上書きする
    };
};
```

> 注: LisM の `combos` ノードは親側で `layers = <0 1 2 3 4 5 6 7 8 9 10 11>;` と全レイヤー指定されている。
> 「default レイヤーだけ」など特定レイヤーに限定したい場合は **子 combo 側で必ず `layers` を上書き** する必要がある（省略では親設定が継承されてしまう）。

命名規則: 既存 LisM combo は `tab` `shift_tab` `homerow-click` のように **snake_case とハイフンが混在** している。新規追加時は **snake_case 推奨**（devicetree のラベル慣習に合わせる）。

key-position 番号は [references/lism-layout.md](references/lism-layout.md) で確認。
詳細は [references/combos.md](references/combos.md)。

## マクロ追加

例: "claude" と入力するマクロ:

```dts
macros {
    type_claude: type_claude {
        compatible = "zmk,behavior-macro";
        #binding-cells = <0>;
        bindings = <&kp C &kp L &kp A &kp U &kp D &kp E>;
        label = "TYPE_CLAUDE";
    };
};
```

bindings 内で `&type_claude` として呼び出せる。
詳細は [references/macros.md](references/macros.md)。

## カスタム hold-tap behavior 追加

例: hold で LSHIFT、tap で SPACE の behavior:

```dts
behaviors {
    space_shift: space_shift {
        compatible = "zmk,behavior-hold-tap";
        #binding-cells = <2>;
        flavor = "balanced";       // hold-preferred / balanced / tap-preferred / tap-unless-interrupted
        tapping-term-ms = <200>;
        quick-tap-ms = <150>;
        bindings = <&kp>, <&kp>;
    };
};
```

bindings 内で `&space_shift LSHIFT SPACE` として呼び出せる。

### flavor 選択ガイド

用途別の推奨:

| 用途 | 推奨 flavor | 理由 |
|------|-----------|------|
| 親指キー（mod-tap, layer-tap） | `balanced` | 既存 LisM の `&mt` グローバル設定 |
| ホームロウ修飾 (HRM) | `tap-preferred` | タイピング誤爆を最小化 |
| 即時反応が欲しい修飾キー | `hold-preferred` | hold 確定が早い、誤爆リスクあり |

### ホームロウ修飾 (HRM) のテンプレ

ユーザーが「ホームロウ修飾」「HRM」を依頼した場合、誤爆対策として **以下 3 点をデフォルトで盛り込む**:

1. `flavor = "tap-preferred"`
2. `quick-tap-ms = <175>` 連打時の修飾誤爆防止
3. `hold-trigger-key-positions = <反対側のキー位置一覧>;` および `hold-trigger-on-release;`

左手用 HRM の `hold-trigger-key-positions` には **右半分のキー位置のみ** を列挙する（左手キー間の hold は許可しない）:

```dts
hrm_l: home_row_mod_left {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    quick-tap-ms = <175>;
    require-prior-idle-ms = <150>;
    bindings = <&kp>, <&kp>;
    hold-trigger-key-positions = <5 6 7 8 9 15 16 17 18 19 25 26 27 28 29 36 37 38 39 40 41>;
    hold-trigger-on-release;
};
```

右手用 HRM (`hrm_r`) は逆に **左半分のキー位置** を指定する。
HRM 依頼があった場合、左右両方のセットを提案するか、ユーザーに片手だけかを確認する。

詳細は [references/hold-tap.md](references/hold-tap.md)。

## *.conf 編集

`config/lism_left.conf` と `config/lism_right.conf` は **同期が必要**。
左右で異なる設定が必要な場合のみ個別編集する。

例: スリープ時間を 1 時間に変更:

```diff
-CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000
+CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=3600000
```

詳細は [references/kconfig.md](references/kconfig.md)。

## build.yaml 編集

例: 新しい artifact を追加:

```yaml
- board: nice_nano_v2
  shield: lism_right
  snippet: studio-rpc-usb-uart
  cmake-args: -DCONFIG_ZMK_STUDIO=y
  artifact-name: lism_right_studio_custom
```

詳細は [references/build-yaml.md](references/build-yaml.md)。

## ZMK CLI を使う場面

| 操作 | コマンド |
|------|---------|
| 新規モジュール追加 | `zmk module add <github-url>` |
| ZMK 本体・モジュール更新 | `zmk update` |
| キーボード一覧 | `zmk keyboard list` |
| 設定確認 | `zmk config` |

ZMK CLI が未インストールの場合は `uv tool install zmk` を案内する。
詳細は [references/zmk-cli.md](references/zmk-cli.md)。

## 編集後の必須検証

すべての編集後に以下を実施する:

1. **構文チェック**: 括弧の対応、セミコロンの欠落を目視確認
2. **`git diff`**: 変更全量をユーザーに提示
3. **依存関係の整合**: レイヤー番号変更時は `&mo` / `&lt` / `&to` の参照も更新する

### ロールバック・取り消し

ユーザーが「やっぱりやめて」「元に戻して」と言った場合、または評価・テスト編集を取り消す場合:

- ✅ **Edit ツールで逆向きに置換する**（追加した内容を削除、変更した内容を元の値に戻す）
- ❌ **`git checkout <file>` は使わない**

`git checkout` はファイル全体を HEAD の状態に戻すため、**他の作業中差分（ユーザーが別途編集中の内容）まで巻き戻す危険がある**。
特に `config/lism.keymap` のように複数の変更が同時進行しがちなファイルでは、必ず Edit で局所的に元に戻す。

例: 追加した combo を取り消す:

```
old_string: """
    new_combo {
        bindings = <&kp ESC>;
        key-positions = <33 34>;
    };
"""
new_string: """
"""
```

例: キー差し替えを取り消す:

```
old_string: "&kp A             &kp W"
new_string: "&kp Q             &kp W"
```

### オプション: ビルド検証

ユーザーが `--verify` フラグを付けた場合、または明示的に「ビルドして」と依頼した場合のみ、
`build-firmware` skill を呼び出してファームウェアをビルドする。

通常の編集では Dev Container 起動の負荷を避けるため、ビルドはユーザーが必要に応じて手動で実行する。

## 関連スキル

- `build-firmware` — ビルド実行（このスキルでオプション呼び出し可）

## ZMK 公式ドキュメント

- Keymaps: <https://zmk.dev/docs/keymaps>
- Combos: <https://zmk.dev/docs/keymaps/combos>
- Macros: <https://zmk.dev/docs/keymaps/behaviors/macros>
- Hold-Tap: <https://zmk.dev/docs/keymaps/behaviors/hold-tap>
- ZMK CLI: <https://zmk.dev/docs/zmk-cli>
- Build/Flash: <https://zmk.dev/docs/development/build-flash>
