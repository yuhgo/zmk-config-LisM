# Win 側を US キーボード扱いにする方針変更

最終更新: 2026-05-12

## 経緯

`feat/layout-shift-v2` で `kot149/zmk-layout-shift v1` を導入し、Phase 1〜3.1 まで進めた段階で実機検証を実施した結果、**Win 側で layout-shift が期待通りに動かない**ことが判明した。

### 検証結果

| 試験 | 入力 | Win JIS 環境での表示 |
|---|---|---|
| 物理 JIS キーボード（内蔵 / USB）で `¥` 単押し | scancode 0x7D / HID INT3 | `¥` ✅正常 |
| LisM (BLE) で `&kp INT3` | HID INT3 (0x89) | `_` ❌異常 |
| LisM (BLE) で `&kp LS(INT3)` | HID INT3 + Shift | `_` ❌異常 |
| LisM (BLE) で `&kpls PIPE` (layout-shift v1 経由 = `LS(INT3)` 送信) | HID INT3 + Shift | `_` ❌異常 |

### 原因

**Windows は Bluetooth キーボード (LisM) を「英語キーボード」として認識している**ため、`INT3` などの JIS 固有の HID コードを送っても期待された記号にならない。

これは `Get-WinUserLanguageList` のシステム言語設定とは独立に、各物理キーボード単位でドライバが割り当てられる Windows の仕様による。

### 切り分けで除外した可能性

- Win のシステム言語: `LanguageTag = ja`、ハードウェアキーボードレイアウト = 「日本語キーボード (106/109 キー)」
- IME (ATOK / Google IME) によるキー乗っ取り: IME OFF (直接入力) でも `_` が出る
- リマップツール: PowerToys / AutoHotKey 等は使っていない

## 新方針

`layout-shift` モジュールを完全に外し、**Win 側のシステム設定を US キーボード扱い**にすることで、Mac と Win で同じ `keymap` を共有する。

### 採用しなかった案

| 案 | 不採用理由 |
|---|---|
| AutoHotKey / PowerToys で LisM だけ JIS 扱い | デバイス別ルールが事実上不可能 / 結果ベースの逆変換で複雑化 |
| レジストリで LisM だけ JIS ドライバ (`kbd106.dll`) を割り当て | 手順複雑、OS 挙動を壊すリスク |
| layout-shift v1 を fork して JIS テーブルをパッチ | LisM が Win に US 認識されている時点で JIS HID コードは無効、パッチでは解決しない |
| 現状の `win_*_layer` 二重管理を継続 | メンテ二重コスト、Win の記号配列が崩れたままで実用性低い |

## 実装計画

### Phase A: layout-shift モジュールのロールバック

| Task | 内容 | DoD |
|---|---|---|
| A.1 | `mark_layer` の検証用キーを元に戻す（現状 K 位置 `&kp INT3` → `&kp PIPE` に戻す。J 位置 `&kpls SLASH` → `&kp SLASH` に戻す） | mark_layer に `&kpls` が残らない |
| A.2 | `bt_mac` から `&tog_ls_off`、`bt_win` から `&tog_ls_on` を削除 | 2 マクロが Phase 0 状態に戻る |
| A.3 | `config/lism.keymap` から `#include <layout_shift.dtsi>` を削除 | keymap diff が include 1 行削除のみ |
| A.4 | `config/lism_right.conf` から `CONFIG_LAYOUT_SHIFT*` 3 行を削除 | right.conf が Phase 0 状態に戻る |
| A.5 | `config/west.yml` から `kot149` remote / `zmk-layout-shift` project を削除 | west.yml diff が remote 2 行 + project 3 行削除 |
| A.6 | `_west/zmk-layout-shift/` を `make setup-west` で再生成（手動削除ではない） | `_west/zmk-layout-shift/` が消えている |
| A.7 | `make all` 成功、Mac で実機動作確認 | 5 種ビルド成功、Mac で既存挙動が壊れていない |

### Phase B: Win 側を US キーボード設定に変更（ゆうご さん作業）

| Task | 内容 | DoD |
|---|---|---|
| B.1 | Windows 「設定 > 時刻と言語 > 言語と地域 > 日本語 > 言語のオプション > ハードウェア キーボード レイアウト」を **「英語キーボード (101/104 キー)」** に変更 | 設定画面で「英語キーボード」と表示される |
| B.2 | Windows にサインアウト / 再起動して設定を反映 | LisM で記号を打つと US 配列通りに出る |
| B.3 | LisM で **`mark_layer` K 位置（`&kp PIPE`）を押すと Win でも `\|` が出る**ことを確認 | Win で `\|` が出る |

### Phase C: keymap の共通化

| Task | 内容 | DoD |
|---|---|---|
| C.1 | `win_mark_layer` を削除し、`mark_layer` を共有 | keymap から `win_mark_layer` が消える |
| C.2 | `win_arrow_layer` を削除し、`arrow_layer` を共有（必要なら Mac/Win 共通の `Cmd/Ctrl + ←` 等を整理） | `win_arrow_layer` 削除 |
| C.3 | `win_function_number_layer` を削除し、`function_number_layer` を共有 | `win_function_number_layer` 削除 |
| C.4 | `win_util_layer` を削除し、`util_layer` を共有（Mac/Win で異なる Mission Control / 仮想デスクトップ系のキーだけ別マクロ化を検討） | `win_util_layer` 削除、または Mac/Win 差分マクロ |
| C.5 | `win_default_layer` を削除し、`default_layer` を共有 | `win_default_layer` 削除 |
| C.6 | レイヤー番号を 0〜6 に詰める、`#define` を整理 | レイヤー数が減って keymap が短くなる |
| C.7 | `bt_mac` / `bt_win` から `&toggle_off/on 7`（win_default_layer トグル）を削除 | 2 マクロは BT_SEL だけになる |
| C.8 | `make all` 成功、Mac/Win 両方で実機動作確認 | 5 種ビルド成功、両 OS で動作 |

### Phase D: クリーンアップ

| Task | 内容 | DoD |
|---|---|---|
| D.1 | `docs/report/mac-win-mark-layer-comparison.md` を新方針に合わせて更新（または削除） | docs が現状 keymap と一致 |
| D.2 | `feat/layout-shift-v2` を main にマージ、不要ブランチと stash を整理 | main 取り込み、退避物の処遇記録 |

## リスクと注意

### Win を US 扱いにする副作用

- Win PC に接続している**他の物理 JIS キーボード**（内蔵 / USB 外付け）の記号配列も全部 US 配列扱いになる
- ゆうご さん回答: 「LisM しか使わないので OK」

### 物理キー印字との不一致

- Win に接続される LisM は US 配列前提なので、物理キーキャップ印字（OEM 表記）と画面に出る記号が一致するように keymap を設計する必要がある
- これは元から `mark_layer` 設計時に意識していた点なので、`mark_layer` の `&kp` 定義をそのまま使えば問題なし

### Mac 側への影響

- Mac は元から US 扱いなので何も変わらない
- Phase 0〜1 で確認した Mission Control / Spaces の動作も変わらない
