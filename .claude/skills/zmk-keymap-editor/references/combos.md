# ZMK Combos

公式: <https://zmk.dev/docs/keymaps/combos>

複数のキーを同時に押すことで、別の動作を発火させる仕組み。

## 基本構造

`combos` ノード（`compatible = "zmk,combos"`）の中に各 combo を定義する:

```dts
/ {
    combos {
        compatible = "zmk,combos";

        my_combo {
            bindings = <&kp ESC>;
            key-positions = <0 1>;
        };
    };
};
```

## プロパティ

### 必須

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `bindings` | behavior | 発火する behavior。`&kp`, `&mo`, `&bt`, `&mt`, `&lt` など全 behavior 利用可能 |
| `key-positions` | array<int> | 同時に押すキー位置の配列 |

### オプション

| プロパティ | 型 | デフォルト | 説明 |
|-----------|-----|----------|------|
| `timeout-ms` | int | 50 | この時間内に key-positions 全部が押されないと発火しない |
| `layers` | array<int> | （全レイヤー） | 発火対象レイヤー。指定するとそれ以外では発火しない |
| `slow-release` | bool | false | true: 全キー解放まで bindings をリリースしない<br>false: 1キー離した時点でリリース |
| `require-prior-idle-ms` | int | (なし) | この時間以内に non-modifier キーが押された場合は発火しない（誤爆防止） |

## 例

### 全レイヤーで動くシンプルなコンボ

```dts
combos {
    compatible = "zmk,combos";

    esc {
        bindings = <&kp ESC>;
        key-positions = <0 1>;
    };
};
```

### 特定レイヤーのみ有効

```dts
combos {
    compatible = "zmk,combos";

    parens {
        bindings = <&kp LEFT_PARENTHESIS>;
        key-positions = <2 3>;
        layers = <0 1>;  // レイヤー 0 と 1 のみ
    };
};
```

### グローバル layers 指定

`combos` ノード自体に `layers = <...>;` を書くと、その下の全 combo に共通適用される（lism.keymap で使用）:

```dts
combos {
    compatible = "zmk,combos";
    layers = <0 1 2 3 4 5 6 7 8 9 10 11>;  // 全 12 レイヤー対象

    tab {
        bindings = <&kp TAB>;
        key-positions = <11 12>;
    };
};
```

個別 combo で `layers` を上書きすることもできる。

### 誤爆防止

タイピング中の意図しない発火を防ぎたい場合:

```dts
slash {
    bindings = <&kp SLASH>;
    key-positions = <16 17>;
    require-prior-idle-ms = <150>;  // 150ms 以内のキー押下があったら発火しない
};
```

### マクロ呼び出し

`bindings` には事前定義したマクロも使える:

```dts
my_emoji_combo {
    bindings = <&emoji>;  // macros で定義した &emoji
    key-positions = <30 41>;
};
```

## key-position の指定

LisM のキー位置は [lism-layout.md](lism-layout.md) を参照。

例: 親指の隣接 2 キーで発火 → `key-positions = <34 35>;`

### 重複コンボ

部分的・完全に重複する key-positions を持つコンボは複数定義可能:

```dts
ab {
    bindings = <&kp X>;
    key-positions = <0 1>;
};
abc {
    bindings = <&kp Y>;
    key-positions = <0 1 2>;
};
```

ZMK は最大マッチ（`abc`）を優先する。

## 既存 LisM コンボ一覧（lism.keymap より）

| 名前 | key-positions | 動作 | 意図 |
|------|--------------|------|------|
| `tab` | 11 12 | TAB | 左 row 1 中央2キー |
| `shift_tab` | 1 2 | LS(TAB) | 左 row 0 中央2キー |
| `minus` | 2 3 | MINUS | 左上段右寄り |
| `plus` | 6 7 | PLUS | 右上段左寄り |
| `asterisk` | 12 13 | ASTERISK | 左 row 1 |
| `slash` | 16 17 | SLASH | 右 row 1 |
| `homerow-click` | 14 15 | LS(LC(LG(SPACE))) | ホームロウ中央 |
| `homerow-scroll` | 4 5 | LS(LC(LG(LA(SPACE)))) | row 0 中央 |
| `amical` | 24 25 | LS(LG(SPACE)) | row 2 中央 |

## デバッグ tips

- timeout-ms が短すぎると発火しない → 50-80 が標準、それでもダメなら徐々に上げる
- 発火しすぎる場合は `require-prior-idle-ms` を追加
- 親指コンボは押しやすいので timeout を短めにできる

## 制限

- 1 キーは複数のコンボに参加可能
- コンボは layer 単位で個別有効化可能
- bindings には 1 つの behavior しか書けない（複数動作はマクロを呼び出す）
