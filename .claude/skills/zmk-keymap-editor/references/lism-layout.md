# LisM 物理レイアウト

## 概要

- **キー数**: 42（左 21 + 右 21）
- **行**: 4 行（row 0-3）
- **列**: row 0-2 は左右各 5 列、row 3 は親指クラスター含む左右各 6 列
- **エンコーダー**: 1 個（alps,ec11、`encoder` という識別子）

## 物理配置（lism.json 由来）

`config/lism.json` の `layout` 定義を元にした物理位置:

```
Row 0:  col 0  1  2  3  4              col 8  9  10 11 12
Row 1:  col 0  1  2  3  4              col 8  9  10 11 12
Row 2:  col 0  1  2  3  4              col 8  9  10 11 12
Row 3:  col 0  1  2  3  4  5    col 7  col 8  9  10 11 12
```

- 左半分: col 0-5（row 3 のみ col 5 まで使用、row 0-2 は col 0-4）
- 右半分: col 7-12（row 3 のみ col 7 から、row 0-2 は col 8 から）
- col 6 は未使用（中央の隙間）

## key-position マップ

ZMK の `bindings` は配列の出現順で 0 始まりの key-position が割り当てられる。
LisM の `keymap` ノードでは以下の順番:

```
左半分                  右半分
 0  1  2  3  4         5  6  7  8  9       (row 0)
10 11 12 13 14        15 16 17 18 19       (row 1)
20 21 22 23 24        25 26 27 28 29       (row 2)
30 31 32 33 34 35  36 37 38 39 40 41       (row 3)
```

### 視覚化

```
┌──┬──┬──┬──┬──┐         ┌──┬──┬──┬──┬──┐
│ 0│ 1│ 2│ 3│ 4│         │ 5│ 6│ 7│ 8│ 9│  Row 0
├──┼──┼──┼──┼──┤         ├──┼──┼──┼──┼──┤
│10│11│12│13│14│         │15│16│17│18│19│  Row 1
├──┼──┼──┼──┼──┤         ├──┼──┼──┼──┼──┤
│20│21│22│23│24│         │25│26│27│28│29│  Row 2
├──┼──┼──┼──┼──┴──┐   ┌──┴──┼──┼──┼──┼──┤
│30│31│32│33│34│35│   │36│37│38│39│40│41│  Row 3
└──┴──┴──┴──┴──┴──┘   └──┴──┴──┴──┴──┴──┘
```

## エンコーダー

- 識別子: `encoder`
- ドライバ: `alps,ec11`
- レイヤーごとの `sensor-bindings` プロパティで動作を定義

例:

```dts
default_layer {
    bindings = < ... >;
    sensor-bindings = <&encoder_msc SCRL_DOWN SCRL_UP>;
};
```

## bindings の物理配置

`config/lism.keymap` の bindings は以下の様に **改行で行を区切って** 記述するのが慣習:

```
default_layer {
    bindings = <
&kp Q             &kp W         &kp E             &kp R        &kp T                                       &kp Y        &kp U   &kp I      &kp O      &kp P
&mt LCTRL A       &kp S         &kp D             &kp F        &kp G                                       &kp H        &kp J   &kp K      &lt 2 L    &kp RETURN
&mt LEFT_SHIFT Z  &kp X         &kp C             &kp V        &kp B                                       &kp N        &kp M   &mkp LCLK  &mkp RCLK  &mt LEFT_SHIFT SLASH
&kp LCTRL         &kp LEFT_ALT  &mt LEFT_WIN ESC  &lt 1 LANG2  &lt 2 SPACE  &lt 3 DELETE    &kp BACKSPACE  &lt 6 LANG1  &trans  &mac_p     &master_p  &kp TAB
    >;
};
```

行ごとの bindings 数:

- row 0-2: 左 5 + 右 5 = **10 トークン**
- row 3: 左 6 + 右 6 = **12 トークン**
- 合計: **42 トークン**

## 親指クラスター（Row 3）

LisM の row 3 は親指で押す 6 キー × 2 で構成:

- 左: col 0-5（ペダル方向に向かって）
- 右: col 7-12

LisM では親指クラスターに以下のような重要キーが割り当てられている:

- LCTRL / LWIN（OS 修飾）
- LEFT_ALT
- ESC（mt LEFT_WIN ESC）
- LANG2（英数）
- SPACE（lt 2 SPACE = レイヤータップ）
- DELETE（lt 3 DELETE）
- BACKSPACE
- LANG1（かな）
- TAB

## key-position 早見表（用途別）

| 用途 | 推奨 key-position |
|------|------------------|
| 親指コンボ | 33-35, 36-38（左右親指で隣接した2キー） |
| ホームロウコンボ | 13-14, 15-16（人差し指側のホームロウ） |
| 上段コンボ | 1-2, 7-8（数字記号入力用） |
| 左右クロスコンボ | 4-5（左右両方を組み合わせて新規キー） |

## 既存コンボ参照例（lism.keymap より）

- `tab`: key-positions = `<11 12>` → 左 row 1 の D, F 位置
- `shift_tab`: `<1 2>` → 左 row 0 の W, E 位置
- `homerow-click`: `<14 15>` → 左 row 1 末端 + 右 row 1 先頭
- `amical`: `<24 25>` → 左 row 2 末端 + 右 row 2 先頭

## 参照

- 物理レイアウト: `config/lism.json`
- ハードウェア定義（外部モジュール）: `_west/zmk-keyboards-LisM/boards/shields/lism/`
