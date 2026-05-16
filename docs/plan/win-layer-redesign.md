# keymap 共通化と Win 用 OS 差オーバーレイ設計（v5 確定版）

最終更新: 2026-05-16
対応 Plans: Phase G (`Plans.md` 参照)

## ゴール

**Mac / Win とも US 配列で運用する**前提で、

- 記号系レイヤーは Mac/Win で **完全共有**
- `win_mark_layer` / `win_function_number_layer` は **削除**
- `win_arrow_layer` / `win_util_layer` は **薄い OS 差オーバーレイ** として再構築
- `conditional_layers` で「`win_default_layer` (7) が on のとき」だけオーバーレイを自動 on
- `default_layer` / `win_default_layer` の修飾キー配置は **現状維持**

---

## なぜ共通化できるのか

両 OS が US 配列を期待しているので、ZMK が送る HID コード（US 配列基準）を Mac/Win は同じように解釈する。
→ 記号レイヤー・F1-F12・数字・矢印などは **そのまま共有できる**。
→ 差が出るのは「OS ショートカット」だけ（行頭ジャンプ、デスクトップ移動など）。

---

## OS 差吸収のアーキテクチャ

### 全体図

```
                    ┌─────────────────────┐
       Mac 接続時:  │ default_layer (0)   │ ← 基底
                    └─────────────────────┘
                              ↓
                    使うレイヤー: 1, 2, 3, 6
                    （Mac 仕様の HID が送られる）

                    ┌─────────────────────┐
       Win 接続時:  │ default_layer (0)   │ ← 基底
                    │ win_default_layer(7)│ ← Win フラグ兼用、常時 on
                    └─────────────────────┘
                              ↓
                    arrow_layer (2) を発動すると
                    conditional_layers が
                    win_arrow_overlay (8) を自動 on
                              ↓
                    オーバーレイのキーが優先される
                    （HOME/END などが送られる）
```

### キーポイント

- `win_default_layer` (7) を **「Win 用 OS フラグ」として兼用**。`bt_mac` / `bt_win` マクロが toggle するので、Win 接続時のみ on になる。
- `conditional_layers` で「7 + 2 が両方 on のとき → 8 も on」「7 + 6 が両方 on のとき → 9 も on」を宣言。
- オーバーレイ (8, 9) は **差分だけ書く**。残りは全部 `&trans` で下のレイヤー（2 や 6）が透過して見える。

---

## レイヤー構成（最終）

| # | レイヤー名 | 役割 | 中身 |
|---|-----------|------|------|
| 0 | `default_layer` | Mac 用基底 | 既存維持 |
| 1 | `mark_layer` | 記号 | 既存維持（共有） |
| 2 | `arrow_layer` | 矢印 + Mac 仕様の行頭/行末 | 既存維持（`LG(LEFT)` / `LG(RIGHT)`） |
| 3 | `function_number_layer` | F1-F12 / 数字 | 既存維持（共有） |
| 4 | `mouse_layer` | マウス | 既存維持 |
| 5 | `scroll_layer` | スクロール | 既存維持 |
| 6 | `util_layer` | メディア / BT / Mac デスクトップ移動 | 既存維持（`RC(LEFT)` 等） |
| 7 | `win_default_layer` | Win 用基底 + OS フラグ | `&lt` 参照先を共通レイヤーに変更 |
| **8** | **`win_arrow_overlay`** | **Win 用 arrow 差分** | **新規（差分のみ）** |
| **9** | **`win_util_overlay`** | **Win 用 util 差分** | **新規（差分のみ）** |

combos の `layers = <0 1 2 3 4 5 6 7>` → `<0 1 2 3 4 5 6 7 8 9>` に拡張。

---

## OS 差マッピング詳細

### `win_arrow_overlay` (#8) の中身

`arrow_layer` (2) と同じキー位置で、Win 用に差分だけ書く。

| 物理位置 | Mac (`arrow_layer`) | **Win オーバーレイ (`win_arrow_overlay`)** |
|---------|--------------------|--------------------------------|
| A 位置 (row1 左1) | `&kp LG(LEFT)` | **`&kp HOME`** |
| F 位置 (row1 左4) | `&kp LG(RIGHT)` | **`&kp END`** |
| その他全部 | （省略） | **`&trans`**（下の arrow_layer が見える） |

### `win_util_overlay` (#9) の中身

`util_layer` (6) と同じキー位置で、Win 用に差分だけ書く。

| 物理位置 | Mac (`util_layer`) | **Win オーバーレイ (`win_util_overlay`)** |
|---------|--------------------|--------------------------------|
| H 位置 (row1 右1) | `&kp RC(LEFT)` | **`&kp LG(LC(LEFT))`**（デスクトップ左） |
| J 位置 (row1 右2) | `&kp RC(DOWN)` | **`&kp LG(D)`**（デスクトップ表示） |
| K 位置 (row1 右3) | `&kp RC(UP)` | **`&kp LG(TAB)`**（タスクビュー） |
| L 位置 (row1 右4) | `&kp RC(RIGHT)` | **`&kp LG(LC(RIGHT))`**（デスクトップ右） |
| その他全部 | （省略） | **`&trans`**（メディア・BT・輝度・Studio は共通レイヤーが見える） |

### IME 切替

`default_layer` / `win_default_layer` 両方で **`LANG1` / `LANG2`** を使う（Mac と同じ HID）。
Win 側でも `LANG1` / `LANG2` を IME 切替として受け取れる前提（要実機検証）。NG なら後で `LA(GRAVE)` (Alt+`) などに切り替える。

---

## `win_default_layer` の `&lt` 参照先付け替え

### 現状

```
&lt 8 LANG2   ← MARK_LAYER 相当（win_mark_layer 参照）
&lt 9 SPACE   ← ARROW_LAYER 相当（win_arrow_layer 参照）
&lt 10 DELETE ← FUNCTION_NUMBER_LAYER 相当（win_function_number_layer 参照）
&lt 11 LANG1  ← UTIL_LAYER 相当（win_util_layer 参照）
```

### 変更後

```
&lt 1 LANG2   ← MARK_LAYER (共通)
&lt 2 SPACE   ← ARROW_LAYER (共通)  ※ conditional で #8 が自動 on
&lt 3 DELETE  ← FUNCTION_NUMBER_LAYER (共通)
&lt 6 LANG1   ← UTIL_LAYER (共通)  ※ conditional で #9 が自動 on
```

---

## `conditional_layers` 設定

`/ { conditional_layers { ... } }` ノードを追加:

```dts
conditional_layers {
    compatible = "zmk,conditional-layers";

    win_arrow_overlay {
        if-layers = <7 2>;
        then-layer = <8>;
    };

    win_util_overlay {
        if-layers = <7 6>;
        then-layer = <9>;
    };
};
```

`win_default_layer` (7) は常時 on（Win 接続中）なので、`arrow_layer` (2) が活性化したら自動で 8 も on、`util_layer` (6) が活性化したら自動で 9 も on になる。

---

## 削除する `#define`

```
#define WIN_MARK_LAYER 8           ← 削除
#define WIN_FUNCTION_NUMBER_LAYER 10 ← 削除
```

残す（番号変更）:

```
#define WIN_ARROW_LAYER 8    （旧 9 → 8）  または WIN_ARROW_OVERLAY にリネーム
#define WIN_UTIL_LAYER 9     （旧 11 → 9） または WIN_UTIL_OVERLAY にリネーム
```

---

## 実装順序

1. **G.1.1**: `win_default_layer` の `&lt` 参照先を `1/2/3/6` に変更
2. **G.1.2**: `win_mark_layer` と `win_function_number_layer` を削除
3. **G.1.3**: `win_arrow_layer` をオーバーレイ仕様に書き換え（HOME/END + その他 trans）
4. **G.1.4**: `win_util_layer` をオーバーレイ仕様に書き換え（デスクトップ移動 + その他 trans）
5. **G.1.5**: `#define` を整理（番号を 8, 9 に詰める）
6. **G.1.6**: `conditional_layers` ノードを追加
7. **G.1.7**: `combos` の `layers` を `<0..9>` に修正
8. **G.1.8**: default_layer 系の修飾キー配置に diff がないか確認

---

## 検証項目（G.2）

### Mac 側
- 記号レイヤー: 既存動作維持
- arrow_layer: `LG(LEFT)` / `LG(RIGHT)` で行頭・行末ジャンプ
- util_layer: `RC(LEFT)` 等で Mission Control 操作
- オーバーレイは Win フラグが off なので発動しない

### Win 側
- 記号レイヤー: Mac と同じキー位置で同じ記号が出る（US/US なので HID 共通）
- arrow_layer + win_default で **HOME / END** が出る
- util_layer + win_default で **デスクトップ移動 / タスクビュー / デスクトップ表示** が出る
- IME 切替: `LANG1` / `LANG2` が Win IME ON/OFF として効くか確認

---

## レビュー（2026-05-16 確定済み）

- [x] Mac/Win 両方 US 配列前提
- [x] OS 差吸収方式: `conditional_layers` + オーバーレイレイヤー
- [x] OS 判定: `win_default_layer` (7) の on/off
- [x] 行頭/行末 (Win): HOME / END
- [x] デスクトップ移動 (Win): 左右 LG(LC(←/→))、上 LG(TAB)、下 LG(D)
- [x] IME 切替: LANG1 / LANG2（Mac と同じ HID）
- [x] レイヤー番号: 0〜9 に整理
