# ZMK Macros

公式: <https://zmk.dev/docs/keymaps/behaviors/macros>

複数の behavior を順番に実行する仕組み。文字列入力、レイヤー切替+修飾、複合ショートカットなどに使う。

## 基本構造

`macros` ノードの中にマクロを定義する:

```dts
/ {
    macros {
        my_macro: my_macro {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp H &kp E &kp L &kp L &kp O>;
            label = "MY_MACRO";
        };
    };
};
```

呼び出しは `&my_macro`。

## 必須プロパティ

| プロパティ | 説明 |
|-----------|------|
| ノードラベル (`my_macro:`) | bindings から呼び出すための名前 |
| `compatible` | `"zmk,behavior-macro"` または `"zmk,behavior-macro-one-param"` 等 |
| `#binding-cells` | パラメータ数。`0`（パラメータなし）または `1`/`2` |
| `bindings` | 実行する behavior の配列 |
| `label` | 識別子（任意だが推奨） |

## オプションプロパティ

| プロパティ | デフォルト | 説明 |
|-----------|----------|------|
| `wait-ms` | `CONFIG_ZMK_MACRO_DEFAULT_WAIT_MS` (15) | bindings 間の待機時間 |
| `tap-ms` | `CONFIG_ZMK_MACRO_DEFAULT_TAP_MS` (5) | tap 時の press-release 間時間 |

## bindings の動作モード

bindings 内の behavior は **デフォルトで tap モード**（押す→離す）。
モードを切り替えるトークンを差し込める:

| トークン | 効果 |
|---------|------|
| `&macro_tap` | 以降を tap モード（デフォルト） |
| `&macro_press` | 以降を press モード（押すだけ） |
| `&macro_release` | 以降を release モード（離すだけ） |
| `&macro_wait_time MS` | wait-ms を一時的に変更 |
| `&macro_tap_time MS` | tap-ms を一時的に変更 |
| `&macro_pause_for_release` | マクロキー自身が離されるまで待機 |

## 例

### シンプルな文字列入力

```dts
type_hello: type_hello {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp H &kp E &kp L &kp L &kp O>;
    label = "TYPE_HELLO";
};
```

### Press/Release を制御（修飾キー併用）

CapsLock を押しっぱなし → E を tap → CapsLock を離す:

```dts
emoji: emoji_macro {
    compatible = "zmk,behavior-macro";
    label = "EMOJI";
    #binding-cells = <0>;
    bindings =
        <&macro_press>,
        <&kp CAPSLOCK>,
        <&macro_tap>,
        <&kp E>,
        <&macro_release>,
        <&kp CAPSLOCK>;
};
```

### 待機時間の挿入

Bluetooth プロファイル切替で OS の認識を待つ:

```dts
bt_win: bt_win {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings =
        <&toggle_off 7>,
        <&macro_wait_time 100>,  // 100ms 待機
        <&bt BT_SEL 1 &toggle_on 7>;
    label = "BT_WIN";
};
```

### 複合ショートカット呼び出し

```dts
screenshot: screenshot {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp LG(LS(NUMBER_5))>;  // Cmd+Shift+5
    label = "SCREENSHOT";
};
```

### パラメータ付きマクロ

`&to N` を呼び出すパラメータ付きマクロ:

```dts
to_layer_0: to_layer_0 {
    compatible = "zmk,behavior-macro-one-param";
    #binding-cells = <1>;
    bindings = <&to 0 &macro_param_1to1 &kp MACRO_PLACEHOLDER>;
    label = "TO_LAYER_0";
};
```

呼び出し: `&to_layer_0 KEY`

`&macro_param_1to1` は受け取った引数を次の behavior に渡す指示子。

### 長い文字列マクロ

```dts
master_p: master_p {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp J &kp F &kp J &kp F &kp H &kp G &kp H &kp G &kp UNDER &kp Y &kp U &kp H &kp G &kp O &kp UNDER &kp N0 &kp N9 &kp N0 &kp N7 &kp UNDER &kp N2 &kp N5 &kp N2 &kp N5>;
    label = "MASTER_P";
};
```

## 既存 LisM マクロ一覧（lism.keymap より）

| 名前 | 用途 |
|------|------|
| `emoji` | macOS の絵文字パネル起動（CapsLock+E） |
| `to_layer_0` | レイヤー 0 へ遷移するパラメータ付きマクロ |
| `screenshot` | スクリーンショット（macOS） |
| `master_p` / `mac_p` / `win_p` | パスワード入力 |
| `presentify` | プレゼンモード切替 |
| `browser_back` / `browser_next` | ブラウザ戻る/進む |
| `bt_mac` / `bt_win` / `bt_mac_mini` | Bluetooth プロファイル切替 |

## 注意点

### #binding-cells の値

| 値 | 意味 | compatible |
|----|------|-----------|
| `0` | パラメータなし | `"zmk,behavior-macro"` |
| `1` | 1 引数 | `"zmk,behavior-macro-one-param"` |
| `2` | 2 引数 | `"zmk,behavior-macro-two-param"` |

`#binding-cells` の値と `compatible` は対応している必要がある。

### マクロから別のマクロを呼ぶ

bindings 内で別のマクロを呼ぶことも可能だが、深いネストはタイミングがズレる原因になる。

### Kconfig での既定値

```
CONFIG_ZMK_MACRO_DEFAULT_WAIT_MS=15
CONFIG_ZMK_MACRO_DEFAULT_TAP_MS=5
```

`*.conf` で変更可能。挙動が不安定なら 30〜50 程度に増やす。

## デバッグ tips

- 文字が抜ける → `wait-ms` を 10-30 に増やす
- 修飾が効かない → `&macro_press` / `&macro_release` で明示的に保持
- レイヤー切替 + 修飾の同時 → `&macro_pause_for_release` を使う
