# ZMK キーコード一覧

公式: <https://zmk.dev/docs/keymaps/list-of-keycodes>

`#include <dt-bindings/zmk/keys.h>` で利用可能。`&kp KEYCODE` の KEYCODE 部分に使う。

## 文字キー

### 英字

```
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
```

### 数字

| キー | 別名 |
|------|------|
| `N1` | `NUMBER_1`, `NUM_1` |
| `N2` | `NUMBER_2` |
| ... | ... |
| `N0` | `NUMBER_0` |

### 記号

| キーコード | 記号 | 別名 |
|-----------|------|------|
| `EXCLAMATION` | ! | `EXCL` |
| `AT_SIGN` | @ | `AT` |
| `HASH` | # | `POUND`, `NUMBER_SIGN` |
| `DOLLAR` | $ | `DLLR` |
| `PERCENT` | % | `PRCNT` |
| `CARET` | ^ | `CRRT` |
| `AMPERSAND` | & | `AMPS` |
| `ASTERISK` | * | `STAR` |
| `LEFT_PARENTHESIS` | ( | `LPAR` |
| `RIGHT_PARENTHESIS` | ) | `RPAR` |
| `MINUS` | - | `KP_MINUS` |
| `EQUAL` | = | `EQL` |
| `PLUS` | + | `KP_PLUS` |
| `LEFT_BRACKET` | [ | `LBKT` |
| `RIGHT_BRACKET` | ] | `RBKT` |
| `BACKSLASH` | \ | `BSLH` |
| `SEMICOLON` | ; | `SEMI` |
| `SINGLE_QUOTE` | ' | `SQT`, `APOS` |
| `DOUBLE_QUOTES` | " | `DQT` |
| `COMMA` | , | - |
| `PERIOD` | . | `DOT` |
| `SLASH` | / | `FSLH` |
| `GRAVE` | ` | - |
| `TILDE` | ~ | - |
| `PIPE` | \| | - |
| `UNDERSCORE` | _ | `UNDER` |
| `COLON` | : | - |

## 制御キー

| キーコード | 説明 |
|-----------|------|
| `RETURN` / `ENTER` | Enter |
| `ESCAPE` / `ESC` | Esc |
| `BACKSPACE` / `BSPC` | Backspace |
| `TAB` | Tab |
| `SPACE` | Space |
| `DELETE` / `DEL` | Delete |
| `CAPSLOCK` / `CAPS` | Caps Lock |

## 矢印キー

| キーコード | 別名 |
|-----------|------|
| `UP_ARROW` | `UP` |
| `DOWN_ARROW` | `DOWN` |
| `LEFT_ARROW` | `LEFT` |
| `RIGHT_ARROW` | `RIGHT` |

## ナビゲーション

| キーコード | 説明 |
|-----------|------|
| `HOME` | Home |
| `END` | End |
| `PAGE_UP` / `PG_UP` | PgUp |
| `PAGE_DOWN` / `PG_DN` | PgDn |
| `INSERT` / `INS` | Ins |
| `PRINTSCREEN` / `PSCRN` | PrtSc |
| `SCROLLLOCK` / `SLCK` | Scroll Lock |
| `PAUSE_BREAK` / `PAUSE` | Pause |

## ファンクションキー

`F1` - `F12` および `F13` - `F24` まで。

## 修飾キー

| 左 | 右 | 説明 |
|----|----|------|
| `LEFT_CONTROL` / `LCTRL` / `LCTL` | `RIGHT_CONTROL` / `RCTRL` / `RCTL` | Ctrl |
| `LEFT_SHIFT` / `LSHIFT` / `LSHFT` | `RIGHT_SHIFT` / `RSHIFT` / `RSHFT` | Shift |
| `LEFT_ALT` / `LALT` | `RIGHT_ALT` / `RALT` | Alt / Opt |
| `LEFT_GUI` / `LGUI` / `LWIN` / `LMETA` | `RIGHT_GUI` / `RGUI` / `RWIN` / `RMETA` | Cmd / Win |

### 修飾の組み合わせ

修飾は関数のように包む:

```dts
&kp LS(A)         // Shift + A
&kp LC(LS(C))     // Ctrl + Shift + C
&kp LG(LS(N4))    // Cmd + Shift + 4 (macOS スクリーンショット)
&kp LS(LC(LG(SPACE)))  // Cmd + Ctrl + Shift + Space
```

## 国際/言語キー

| キーコード | 説明 |
|-----------|------|
| `LANG1` | macOS: かな (日本語入力 ON) |
| `LANG2` | macOS: 英数 (英数入力 ON) |
| `INTERNATIONAL_1` / `INT1` | バックスラッシュ・アンダーバーキー（JIS） |
| `INTERNATIONAL_3` / `INT3` | 円記号・パイプキー（JIS） |
| `INTERNATIONAL_4` / `INT4` | 変換キー |
| `INTERNATIONAL_5` / `INT5` | 無変換キー |

## メディアキー（C_ プレフィックス）

| キーコード | 説明 |
|-----------|------|
| `C_VOLUME_UP` / `C_VOL_UP` | 音量↑ |
| `C_VOLUME_DOWN` / `C_VOL_DN` | 音量↓ |
| `K_MUTE` / `C_MUTE` | ミュート |
| `C_PLAY_PAUSE` / `C_PP` | 再生/一時停止 |
| `C_NEXT` | 次曲 |
| `C_PREVIOUS` / `C_PREV` | 前曲 |
| `C_STOP` | 停止 |
| `C_BRIGHTNESS_INC` / `C_BRI_UP` | 明るさ↑ |
| `C_BRIGHTNESS_DEC` / `C_BRI_DN` | 明るさ↓ |

## マウスキー (`<dt-bindings/zmk/mouse.h>` 必要)

| 用途 | キーコード | 例 |
|------|-----------|---|
| クリック | `LCLK` / `RCLK` / `MCLK` | `&mkp LCLK` |
| 移動 | `MOVE_UP` / `_DOWN` / `_LEFT` / `_RIGHT` | `&mmv MOVE_UP` |
| スクロール | `SCRL_UP` / `_DOWN` / `_LEFT` / `_RIGHT` | `&msc SCRL_DOWN` |

## Bluetooth キー (`<dt-bindings/zmk/bt.h>` 必要)

| キーコード | 説明 |
|-----------|------|
| `BT_SEL N` | プロファイル N を選択 (0-4) |
| `BT_CLR` | 現在のプロファイルをクリア |
| `BT_CLR_ALL` | 全プロファイルをクリア |
| `BT_NXT` / `BT_PRV` | 次/前のプロファイル |

例: `&bt BT_SEL 0`, `&bt BT_CLR`

## ポインティング (`<dt-bindings/zmk/pointing.h>` 必要)

| キーコード | 説明 |
|-----------|------|
| `SCRL_UP` / `SCRL_DOWN` | スクロール上下 |
| `SCRL_LEFT` / `SCRL_RIGHT` | スクロール左右 |

## レイヤー切替（番号で指定）

```dts
&mo 1        // レイヤー 1 を hold 中だけ ON
&lt 2 SPACE  // hold でレイヤー 2、tap で SPACE
&to 0        // レイヤー 0 に切り替え（永続）
&tog 3       // レイヤー 3 の ON/OFF を切り替え
&sl 4        // 次の 1 キーだけレイヤー 4
```

## LisM でよく使う組み合わせ

`config/lism.keymap` で実際に使われている例:

| 用途 | 記述 |
|------|------|
| Ctrl + A | `&mt LCTRL A` (mod-tap) |
| Cmd + Space (IME) | `&kp LG(SPACE)` |
| 日本語切替 | `&kp LANG1` / `&kp LANG2` |
| Cmd+Ctrl+Shift+Space | `&kp LS(LC(LG(SPACE)))` |
| macOS スクリーンショット | `&kp LG(LS(N5))` |
| 範囲選択 (Cmd+Shift+矢印) | `&kp LG(LEFT)` |
| Ctrl+矢印（Spaces 切替） | `&kp RC(LEFT_ARROW)` |

## キーコード命名の判断

迷ったら以下の順に試す:

1. 完全名（`LEFT_BRACKET`）
2. 略号（`LBKT`）
3. ASCII 風（`LBRC`）

公式の最新 keys.h: <https://github.com/zmkfirmware/zmk/blob/main/app/include/dt-bindings/zmk/keys.h>
