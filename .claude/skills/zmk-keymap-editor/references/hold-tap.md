# ZMK Hold-Tap Behavior

公式: <https://zmk.dev/docs/keymaps/behaviors/hold-tap>

押し続けると hold 動作、軽くタップすると tap 動作という、1 キーに 2 つの役割を持たせる behavior。

## 標準提供の hold-tap

ZMK には事前定義済みの hold-tap が 2 つある（`behaviors.dtsi` で定義済み）:

| behavior | 用途 |
|---------|------|
| `&mt MOD KEY` | mod-tap: hold で修飾、tap でキー |
| `&lt LAYER KEY` | layer-tap: hold でレイヤー、tap でキー |

例:

```dts
&mt LCTRL A    // 押し続けると Ctrl、タップで A
&lt 2 SPACE    // 押し続けるとレイヤー 2、タップで SPACE
```

## カスタム hold-tap の定義

`behaviors` ノード内で定義:

```dts
/ {
    behaviors {
        my_ht: my_hold_tap {
            compatible = "zmk,behavior-hold-tap";
            #binding-cells = <2>;
            flavor = "balanced";
            tapping-term-ms = <200>;
            quick-tap-ms = <150>;
            bindings = <&kp>, <&kp>;
        };
    };
};
```

呼び出し: `&my_ht LSHIFT A`（hold で LSHIFT、tap で A）

## 必須プロパティ

| プロパティ | 説明 |
|-----------|------|
| `compatible` | `"zmk,behavior-hold-tap"` |
| `#binding-cells` | `2`（hold と tap の 2 引数） |
| `bindings` | `<&hold_behavior>, <&tap_behavior>` 形式の 2 要素 |

## flavor（重要）

押下中に別キーが押された場合の挙動を決める。4 種類:

### `hold-preferred`

- `tapping-term-ms` 経過 **または** 別キーが押された時点で **即 hold** 確定
- メリット: hold 動作（修飾）が反応しやすい
- デメリット: 連続タイピング中に意図せず修飾が発動する誤爆あり

### `balanced` ★ LisM デフォルト

- `tapping-term-ms` 経過 **または** 別キーが押されて **離されるまで保持されたとき** hold 確定
- 別キーが先に離されたら tap 動作
- メリット: バランスが良い
- 設定例: `&mt { flavor = "balanced"; };` → LisM の `&mt` 全体に適用

### `tap-preferred`

- `tapping-term-ms` 経過するまで hold 確定しない
- 他のキー押下では hold にならない
- メリット: タイピング中の誤爆がほぼ起きない
- デメリット: 修飾としての反応が遅い

### `tap-unless-interrupted`

- 別キーが timeout 内に押されたら hold、そうでなければ tap
- `hold-preferred` の逆
- 修飾を素早く必要とする用途で有用

## オプションプロパティ

| プロパティ | デフォルト | 説明 |
|-----------|----------|------|
| `tapping-term-ms` | 200 | hold 判定までの保持時間 |
| `quick-tap-ms` | 0 (無効) | 直前のリリースから N ms 以内なら必ず tap（連打対応） |
| `require-prior-idle-ms` | 0 (無効) | 直前のキー押下から N ms 経たないと hold にならない |
| `flavor` | "hold-preferred" | 上記 4 種から選択 |
| `hold-trigger-key-positions` | (なし) | hold 動作を許可するキー位置を限定（positional hold-tap） |
| `hold-trigger-on-release` | false | hold-trigger 判定をリリース時に行う |

## 例

### Mod-Tap のグローバル flavor 設定

LisM では先頭で `&mt` の flavor を `balanced` に上書きしている:

```dts
&mt { flavor = "balanced"; };
```

これで全 `&mt MOD KEY` に balanced が適用される。

### カスタム layer-tap（quick-tap 付き）

```dts
fast_lt: fast_layer_tap {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <180>;
    quick-tap-ms = <150>;     // 連打時は tap 確定
    bindings = <&mo>, <&kp>;
};
```

呼び出し: `&fast_lt 2 SPACE`

### LSHIFT の hold-tap で別キーを発火

`bindings` の 1 つ目（hold）に修飾、2 つ目（tap）にキーを置くだけでなく、
特殊な behavior（マクロやコンボ）も設定可能:

```dts
shift_emoji: shift_emoji {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <0>;          // パラメータ取らない場合は 0
    flavor = "balanced";
    tapping-term-ms = <200>;
    bindings = <&kp LSHIFT>, <&emoji>;
};
```

ただし `#binding-cells = <0>` の場合は `bindings` 側を `<&kp LSHIFT>, <&emoji>` のように **完全な behavior 記述** にする。

### Positional Hold-Tap

「左手の hold-tap キーは右手のキーを押した時だけ hold 確定」のように、
hold 確定する隣接キーを限定できる（誤爆対策に強力）:

```dts
home_row_mods: home_row_mods {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <250>;
    quick-tap-ms = <175>;
    bindings = <&kp>, <&kp>;
    hold-trigger-key-positions = <5 6 7 8 9 15 16 17 18 19 25 26 27 28 29>;  // 右半分のキー
    hold-trigger-on-release;
};
```

## 関連 behavior（hold-tap 派生）

### tap-dance

複数回タップで異なる動作（lism.keymap で使用済み）:

```dts
double_shift: double_shift {
    compatible = "zmk,behavior-tap-dance";
    label = "DOUBLE_SHIFT";
    #binding-cells = <0>;
    tapping-term-ms = <200>;
    bindings = <&kp Z>, <&kp Z>;  // 1 回タップで Z、2 回タップで Z（= ダブルZ）
};
```

`bindings` の n 番目要素が n 回タップ時に発動。

### toggle-layer

layer の ON/OFF を制御:

```dts
toggle_on: toggle_layer_on {
    compatible = "zmk,behavior-toggle-layer";
    #binding-cells = <1>;
    display-name = "Toggle Layer On";
};
```

呼び出し: `&toggle_on 7`

## チューニングのコツ

| 症状 | 対処 |
|------|------|
| 連打中に修飾が発動する | `quick-tap-ms` を 100-200 で設定 |
| hold 反応が遅い | `tapping-term-ms` を 150-180 に下げる、または `flavor = "hold-preferred"` |
| タイピング中の誤爆 | `flavor = "tap-preferred"` または `hold-trigger-key-positions` で限定 |
| 親指キーで誤動作 | `flavor = "balanced"` + tapping-term-ms 200-250 |

## 既存 LisM カスタム behavior

`config/lism.keymap` の `behaviors` ノードに以下が定義済み:

- `lt_to_layer_0`: hold-tap の派生（`&mo` + `&to_layer_0`）
- `double_shift`: tap-dance（Z + Z）
- `layer3_backspace_td`: tap-dance（mo 3 + BACKSPACE）
- `tab_enter_td`: tap-dance（TAB + ENTER）
- `toggle_on` / `toggle_off`: toggle-layer
- `encoder_msc` / `encoder_kp`: sensor-rotate-var（エンコーダー）

新規 behavior を追加する場合、これらと衝突しないラベル名を使う。
