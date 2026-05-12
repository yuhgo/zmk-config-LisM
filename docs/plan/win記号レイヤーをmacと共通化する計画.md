# Win 記号レイヤーを Mac と共通化する計画

作成日: 2026-05-10

## 背景

現在 `config/lism.keymap` では、Mac (US配列) 用と Windows (JIS配列) 用で記号レイヤーを別々に手書きしている。
Win 側は JIS のキーコードに合わせてゴリゴリに書き換えてあり、メンテが二重になっている。
特に `win_mark_layer` の K 位置は `LS(INT1)` で `_` が出力されてしまうバグもある（Mac の `|` と一致しない）。

## ゴール

[zmk-layout-shift](https://github.com/kot149/zmk-layout-shift) モジュールを導入し、
**記号レイヤーは US キーコードで 1 本に統一**したまま、JIS マシンでも正しい記号が出るようにする。

修飾キーの位置（LWIN/LCTRL の入れ替え等）は OS ごとに違うので、`win_default_layer` などの修飾キー配置はそのまま残す。

## 採用する方針（ヒアリング結果）

1. **記号レイヤーだけ共通化**: `win_mark_layer` / `win_function_number_layer` の中身を Mac と同じ US キーコードに揃える。`win_default_layer` 等の修飾キー配置はそのまま残す
2. **`&kp` を `&kpls` で透過上書き**: `layout_shift_kp_override.dtsi` を include して、既存 `&kp` 記述を変えずに JIS 対応
3. **bt_mac/bt_win マクロで自動切替**: プロファイル切替がそのままレイアウトシフト切替を兼ねる（手動操作不要）

## Phase 1: モジュール導入と基盤整備

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 1.1 | `config/west.yml` に `kot149/zmk-layout-shift` (revision: v1) を追加 | west.yml diff が公式 README 通り | - | cc:TODO |
| 1.2 | `make setup-west` 相当で _west/ にモジュール取得確認 | _west/zmk-layout-shift/ が存在 | 1.1 | cc:TODO |
| 1.3 | `config/lism_left.conf` / `config/lism_right.conf` に `CONFIG_LAYOUT_SHIFT_TARGET_JIS=y` を追加 | 両 conf が同期 | 1.2 | cc:TODO |
| 1.4 | `config/lism.keymap` の include 群に `<layout_shift.dtsi>` と `<layout_shift_kp_override.dtsi>` を追加（behaviors.dtsi より下） | include 順が公式注意点通り | 1.3 | cc:TODO |
| 1.5 | この時点で一度ビルド (`mise run dc-exec make single` で右側のみ) | .uf2 生成成功・既存挙動が変わらない | 1.4 | cc:TODO |

## Phase 2: bt_mac / bt_win マクロにレイアウトシフト切替を組み込む

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 2.1 | `bt_mac` マクロに `&tog_ls_off` を追加（toggle_off 7 の前後どちらかに配置検討） | Mac プロファイル選択で JIS シフト OFF | 1.5 | cc:TODO |
| 2.2 | `bt_win` マクロに `&tog_ls_on` を追加 | Win プロファイル選択で JIS シフト ON | 1.5 | cc:TODO |
| 2.3 | レイアウトシフトの初期状態を US (OFF) と仮定し、起動直後の挙動を確認 | 初回ペアリングで Mac 側の記号が崩れない | 2.1, 2.2 | cc:TODO |

## Phase 3: Win 専用記号レイヤーを Mac と統合

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 3.1 | `win_default_layer` のレイヤー番号参照を点検（lt 8/9/10/11 → mark/arrow/function/util を mac と共有する番号に変更するか検討） | 設計判断記録 docs/plan に追記 | 1.5 | cc:TODO |
| 3.2 | `win_mark_layer` を削除し `win_default_layer` から `mark_layer` (1) を参照 | win 側で & や [ ] が Mac と同じ位置で出る | 3.1, 2.2 | cc:TODO |
| 3.3 | `win_function_number_layer` を削除し `function_number_layer` (3) を参照（`LS(MINUS)` → `EQUAL` に戻る点を確認） | win 側で `=` が同じ位置で出る | 3.1, 2.2 | cc:TODO |
| 3.4 | `win_arrow_layer` 統合判断（HOME/END vs LG(LEFT/RIGHT) は OS ごとに違うので **残す** 想定。判断を docs に記録） | 残す/統合の理由を明記 | 3.1 | cc:TODO |
| 3.5 | `win_util_layer` 統合判断（LG(LC(LEFT)) など Win 固有ショートカットは残す想定） | 残す/統合の理由を明記 | 3.1 | cc:TODO |
| 3.6 | レイヤー番号定義 (`#define WIN_MARK_LAYER` 等) のうち削除したものを掃除 | 未使用 define が消える | 3.2, 3.3 | cc:TODO |
| 3.7 | `combos.layers` の対象から削除レイヤー番号を外す | combos が壊れない | 3.6 | cc:TODO |

## Phase 4: 検証

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 4.1 | `mise run dc-exec make all` でビルド全部通過 | 全 .uf2 生成成功 | 3.7 | cc:TODO |
| 4.2 | 実機 Mac で記号レイヤーの全キーが期待通り出力 | docs/report/mac-win-mark-layer-comparison.md の Mac 出力列と一致 | 4.1 | cc:TODO |
| 4.3 | 実機 Win (JIS) で記号レイヤーの全キーが Mac と同じ記号を出力 | 同 docs の Mac 出力列と一致（特に K 位置 `\|`） | 4.1 | cc:TODO |
| 4.4 | bt_mac/bt_win 切替で JIS シフトが自動で ON/OFF になる | 切替直後の試打で記号が即座に切替 | 4.2, 4.3 | cc:TODO |
| 4.5 | docs/report/mac-win-mark-layer-comparison.md を「統合済み」状態に更新 or アーカイブ | docs と実装が一致 | 4.4 | cc:TODO |

## レイヤー番号設計（Phase 3.1 の判断結果）

### 統合後の番号マップ

ZMK の keymap レイヤー番号は **定義順に自動採番** されるため、削除した分は詰める。

| 番号 | レイヤー | 共有 | 備考 |
|---|---|---|---|
| 0 | DEFAULT | Mac | Mac 用 default |
| 1 | MARK | **Mac/Win 共有** | layout-shift で JIS 対応 |
| 2 | ARROW | Mac 専用 | LG(LEFT/RIGHT) は Mac 固有 |
| 3 | FUNCTION_NUMBER | **Mac/Win 共有** | layout-shift で JIS 対応 |
| 4 | MOUSE | 共通 | OS 非依存 |
| 5 | SCROLL | Mac 専用 | RC(LEFT) など Mac 固有 |
| 6 | UTIL | Mac 専用 | Mac 固有ショートカット |
| 7 | WIN_DEFAULT | Win | toggle_on/off で全体切替するため最上位 |
| 8 | WIN_ARROW | Win 専用 | HOME/END (JIS でも記号でなくナビゲーション) |
| 9 | WIN_UTIL | Win 専用 | LG(LC(LEFT)), LG(TAB) など Win 固有 |

旧 WIN_MARK (8) と WIN_FUNCTION_NUMBER (10) は削除し、後続レイヤーは番号を詰める。

### win_default_layer の `&lt` 番号書き換え

| 親指キー | 旧 | 新 | 飛び先 |
|---|---|---|---|
| LANG2 (左) | `&lt 8 LANG2` | `&lt 1 LANG2` | MARK 共有 |
| SPACE (左) | `&lt 9 SPACE` | `&lt 8 SPACE` | WIN_ARROW（番号詰めにより 9→8） |
| DELETE (左) | `&lt 10 DELETE` | `&lt 3 DELETE` | FUNCTION_NUMBER 共有 |
| LANG1 (右) | `&lt 11 LANG1` | `&lt 9 LANG1` | WIN_UTIL（番号詰めにより 11→9） |

### arrow / util を統合しない理由

`zmk-layout-shift` は「同じ記号を JIS に翻訳する」モジュールであって、
「OS 固有のショートカット (Cmd+← vs Home) を翻訳する」ものではない。
arrow / util レイヤーは記号入力ではなく OS 機能呼び出しなので、Win 用の別レイヤーが必要。

## リスクと注意点

- **include 順**: `layout_shift_kp_override.dtsi` は `behaviors.dtsi` より **下** に書く必要がある（公式 README）
- **修飾つきキー非対応**: `LG(LC(LEFT))` のような修飾つき通常キーはレイアウトシフトされないので、Win の `win_util_layer` でこれを残しているのは正しい判断
- **Swap Ctrl/Cmd は使わない**: 修飾キーの入れ替えは現状の `win_default_layer` でハードコードしているので、`CONFIG_LAYOUT_SHIFT_TARGET_SWAP_CTRL_CMD` には触れない
- **初期状態**: モジュールの初期状態が ON か OFF かを Phase 2.3 で必ず確認する（公式仕様の確認 or 実機テスト）
- **Combos / macros 内の `&kp`**: kp_override で combos 内の `&kp PLUS` 等も自動的にシフトされるか確認（特に `combos.plus` `combos.minus` `combos.asterisk` `combos.slash`）
