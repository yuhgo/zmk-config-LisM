# macOS/Windows キー統一化計画

## 目的

macOSとWindowsで同じキー位置で同じ操作ができるようにする:
1. Mission Control (mac) / タスクビュー (win) を同じキーで
2. 仮想デスクトップ移動を同じキーで

## 現状分析

### 現在のレイヤー構造
- macOS用: `scroll_layer` (Layer 5), `util_layer` (Layer 6)
- Windows用: `win_util_layer` (Layer 11)

### 該当箇所
| レイヤー | 行番号 | 現在の設定 |
|---------|--------|-----------|
| scroll_layer | 289行目 | `RC(LEFT_ARROW)` `RC(DOWN)` `RC(UP_ARROW)` `RC(RIGHT)` |
| util_layer | 299行目 | `RC(LEFT_ARROW)` `RC(DOWN_ARROW)` `RC(UP_ARROW)` `RC(RIGHT)` |
| win_util_layer | 351行目 | `RC(LEFT_ARROW)` `RC(DOWN_ARROW)` `RC(UP_ARROW)` `RC(RIGHT)` |

### 問題点

macOS側（scroll_layer, util_layer）は正しい設定:
- `RC(LEFT)` / `RC(RIGHT)` = Control+Left/Right → 仮想デスクトップ移動
- `RC(UP)` = Control+Up → Mission Control

Windows側（win_util_layer）が間違い:
| 機能 | Windows正解 | 現在の設定 |
|------|-------------|-----------|
| タスクビュー | Win+Tab | Control+Up (間違い) |
| 仮想デスクトップ左 | Win+Ctrl+Left | Control+Left (間違い) |
| 仮想デスクトップ右 | Win+Ctrl+Right | Control+Right (間違い) |

## 実装計画

### 対象ファイル
- `config/lism.keymap`

### 変更内容

**351行目 (win_util_layer の2行目) を修正:**

変更前:
```
&trans          &kp C_VOLUME_DOWN     &kp K_MUTE    &kp C_VOLUME_UP       &trans                    &kp RC(LEFT_ARROW)  &kp RC(DOWN_ARROW)  &kp RC(UP_ARROW)  &kp RC(RIGHT)  &trans
```

変更後:
```
&trans          &kp C_VOLUME_DOWN     &kp K_MUTE    &kp C_VOLUME_UP       &trans                    &kp LG(LC(LEFT))    &kp RC(DOWN_ARROW)  &kp LG(TAB)       &kp LG(LC(RIGHT))  &trans
```

### 変更の詳細

| キー位置 | 変更前 | 変更後 | 機能 |
|---------|--------|--------|------|
| 右手2行目 1番目 | `RC(LEFT_ARROW)` | `LG(LC(LEFT))` | 仮想デスクトップ左移動 (Win+Ctrl+Left) |
| 右手2行目 2番目 | `RC(DOWN_ARROW)` | `RC(DOWN_ARROW)` | そのまま |
| 右手2行目 3番目 | `RC(UP_ARROW)` | `LG(TAB)` | タスクビュー (Win+Tab) |
| 右手2行目 4番目 | `RC(RIGHT)` | `LG(LC(RIGHT))` | 仮想デスクトップ右移動 (Win+Ctrl+Right) |

### キーコード説明
- `LG()` = Left GUI (Windows キー)
- `LC()` = Left Control
- `RC()` = Right Control
- `LG(LC(LEFT))` = Win + Ctrl + Left
- `LG(TAB)` = Win + Tab

## 検証方法

1. ファームウェアビルド: `make single` で `lism_right_trackball` を選択
2. ファームウェア書き込み: 生成された `.uf2` ファイルをキーボードに転送
3. 動作確認:
   - Windows PCに接続（`bt_win` で切り替え）
   - `win_util_layer` に切り替えて、2行目右手側キーをテスト:
     - 1番目: 仮想デスクトップ左移動
     - 3番目: タスクビュー表示
     - 4番目: 仮想デスクトップ右移動

## 結果

| 操作 | macOS キー | Windows キー | 同じキー位置 |
|------|-----------|-------------|-------------|
| Mission Control / タスクビュー | Control+Up | Win+Tab | 2行目3番目 |
| 仮想デスクトップ左 | Control+Left | Win+Ctrl+Left | 2行目1番目 |
| 仮想デスクトップ右 | Control+Right | Win+Ctrl+Right | 2行目4番目 |
