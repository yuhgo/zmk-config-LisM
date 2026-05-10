# ZMK Keymap 基礎

公式: <https://zmk.dev/docs/keymaps>

## ファイル先頭の include

`config/lism.keymap` は devicetree 形式。先頭で必要なヘッダを include する:

```c
#define ZMK_POINTING_DEFAULT_MOVE_VAL 400  // ポインティング設定（任意）
#define ZMK_POINTING_DEFAULT_SCRL_VAL 200

#include <dt-bindings/zmk/mouse.h>     // &mkp などのマウスキー
#include <behaviors.dtsi>              // &kp, &mo, &lt, &mt などの基本 behavior
#include <dt-bindings/zmk/bt.h>        // BT_SEL, BT_CLR
#include <dt-bindings/zmk/keys.h>      // A-Z, F1-F12, LCTRL など
#include <dt-bindings/zmk/pointing.h>  // SCRL_UP, SCRL_DOWN
```

include の追加が必要なケース:

- マクロでメディアキー → `<dt-bindings/zmk/keys.h>` で十分
- マウスポインタ動作 → `<dt-bindings/zmk/pointing.h>` 追加
- RGB 制御 → `<dt-bindings/zmk/rgb.h>` 追加
- 外部キー (Outputs) → `<dt-bindings/zmk/outputs.h>` 追加

## ルートノード

すべての設定は `/ { ... };` の中に書く。

```dts
/ {
    keymap {
        compatible = "zmk,keymap";
        // レイヤー定義...
    };
    combos {
        compatible = "zmk,combos";
        // コンボ定義...
    };
    macros {
        // マクロ定義...
    };
    behaviors {
        // カスタム behavior 定義...
    };
};
```

## レイヤー定義

`keymap` ノード（`compatible = "zmk,keymap"`）の中にレイヤーを並べる:

```dts
keymap {
    compatible = "zmk,keymap";

    default_layer {
        display-name = "Default Layer";  // ZMK Studio で表示される名前
        bindings = < /* 42 個のキー behavior */ >;
        sensor-bindings = <&encoder_msc SCRL_DOWN SCRL_UP>;  // エンコーダー（任意）
    };

    mark_layer {
        display-name = "Mark Layer";
        bindings = < ... >;
    };
    // 以下、追加レイヤー...
};
```

### レイヤー番号

レイヤー番号は **定義順で 0 から自動採番** される。
LisM では先頭で `#define` を使って意味的に名前を付けている:

```c
#define DEFAULT_LAYER 0
#define MARK_LAYER 1
#define ARROW_LAYER 2
// ...
```

bindings 中で `&mo MARK_LAYER` のように使えるが、現状の lism.keymap では直接数字 (`&mo 1`) を使っている。

### display-name

ZMK Studio で表示される人間向けの名前。設定推奨。

## bindings の書き方

`bindings` プロパティは behavior の配列。`<` `>` で囲む:

```dts
bindings = <&kp A &kp B &kp C ... >;
```

LisM では可読性のため改行を入れて 4 行で記述する:

```dts
bindings = <
&kp Q             &kp W         &kp E             &kp R        &kp T                                       &kp Y        &kp U   &kp I      &kp O      &kp P
&mt LCTRL A       &kp S         &kp D             &kp F        &kp G                                       &kp H        &kp J   &kp K      &lt 2 L    &kp RETURN
&mt LEFT_SHIFT Z  &kp X         &kp C             &kp V        &kp B                                       &kp N        &kp M   &mkp LCLK  &mkp RCLK  &mt LEFT_SHIFT SLASH
&kp LCTRL         &kp LEFT_ALT  &mt LEFT_WIN ESC  &lt 1 LANG2  &lt 2 SPACE  &lt 3 DELETE    &kp BACKSPACE  &lt 6 LANG1  &trans  &mac_p     &master_p  &kp TAB
>;
```

## 主要な behavior

### キープレス系

| behavior | 用途 | 例 |
|---------|------|---|
| `&kp KEY` | 通常のキープレス | `&kp A`, `&kp LCTRL` |
| `&trans` | 透過（下のレイヤーへ委譲） | `&trans` |
| `&none` | 無反応 | `&none` |
| `&kp LS(KEY)` | Shift + KEY | `&kp LS(TAB)` |
| `&kp LC(KEY)` | Ctrl + KEY | `&kp LC(C)` |
| `&kp LG(KEY)` | GUI/Win/Cmd + KEY | `&kp LG(SPACE)` |
| `&kp LA(KEY)` | Alt + KEY | `&kp LA(F4)` |

修飾は組み合わせ可能: `&kp LS(LC(LG(SPACE)))` = Cmd+Ctrl+Shift+Space

### レイヤー系

| behavior | 用途 |
|---------|------|
| `&mo N` | 押下中だけレイヤー N をアクティブ |
| `&lt N KEY` | hold でレイヤー N、tap で KEY（layer-tap） |
| `&to N` | レイヤー N に切り替え（永続） |
| `&tog N` | レイヤー N の ON/OFF を切り替え |
| `&sl N` | sticky layer（次の 1 キーの間だけアクティブ） |

### 修飾タップ系

| behavior | 用途 |
|---------|------|
| `&mt MOD KEY` | hold で MOD、tap で KEY（mod-tap） |
| `&sk MOD` | sticky modifier（次の 1 キーに修飾を適用） |

例: `&mt LCTRL A` → 押し続けると Ctrl、軽くタップで A

### マウス系（pointing 必要）

| behavior | 用途 |
|---------|------|
| `&mkp BUTTON` | マウスクリック (`LCLK`, `RCLK`, `MCLK`) |
| `&mmv MOVE` | マウス移動 |
| `&msc SCROLL` | マウススクロール |

### Bluetooth

| behavior | 用途 |
|---------|------|
| `&bt BT_SEL N` | プロファイル N を選択 (0-4) |
| `&bt BT_CLR` | 現在のプロファイルをクリア |
| `&bt BT_CLR_ALL` | 全プロファイルをクリア |

### システム

| behavior | 用途 |
|---------|------|
| `&studio_unlock` | ZMK Studio のロック解除 |
| `&reset` | ソフトリセット |
| `&bootloader` | ブートローダーへ |

## レイヤー間の優先順位

> 重要: レイヤーの優先順位は **定義順ではなく、レイヤー番号の大きさ** で決まる。
> 番号が大きい = 優先される。

`&mo 1` で複数レイヤーがアクティブな場合、最も大きい番号のレイヤーが先に評価される。

## 透過 (`&trans`) の挙動

`&trans` を置くと、その位置は **より低い番号の有効なレイヤー** から動作を引き継ぐ。
特定のキーだけ default 動作を残したい場合に使う。

## センサー (エンコーダー) の bindings

`sensor-bindings` プロパティを使い、各レイヤーごとにエンコーダー動作を変えられる:

```dts
default_layer {
    bindings = < ... >;
    sensor-bindings = <&encoder_msc SCRL_DOWN SCRL_UP>;
};
```

`encoder_msc` などのカスタム behavior は `behaviors` ノードで定義する（lism.keymap 既存）。

## 参考: 公式ドキュメント

- Keymaps: <https://zmk.dev/docs/keymaps>
- Behaviors 一覧: <https://zmk.dev/docs/keymaps/behaviors>
- Modifiers: <https://zmk.dev/docs/keymaps/modifiers>
