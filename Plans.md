# zmk-config-LisM Plans.md

最終更新: 2026-05-12

このリポジトリのアクティブなタスクを管理する。詳細な計画は `docs/plan/` 配下の個別ファイルに記述する。

---

## アクティブな計画

### Win 側を US キーボード扱いにして keymap を Mac と共通化

詳細: [docs/plan/Win側USキーボード化に方針変更.md](docs/plan/Win側USキーボード化に方針変更.md)

`feat/layout-shift-v2` で zmk-layout-shift v1 を導入し Phase 1〜3.1 まで進めたが、実機検証で **Win は LisM (BLE) を US キーボードとして認識している** ことが判明した。layout-shift の JIS HID コード (`LS(INT3)` 等) は Win に届いても期待された記号を出さないため、layout-shift 方式は採用不可。

新方針: **Win 側のハードウェアキーボードレイアウトを「英語キーボード (101/104 キー)」に変更** することで、Mac と Win で同じ keymap を共有する。

#### Phase A: layout-shift をロールバック

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| A.1 | `mark_layer` の検証用キーを元に戻す（K 位置 `&kp INT3` → `&kp PIPE`、J 位置 `&kpls SLASH` → `&kp SLASH`） | mark_layer に `&kpls` / `&kp INT3` が残らない | - | cc:完了 |
| A.2 | `bt_mac` から `&tog_ls_off`、`bt_win` から `&tog_ls_on` を削除 | 2 マクロが Phase 0 状態に戻る | A.1 | cc:完了 |
| A.3 | `config/lism.keymap` から `#include <layout_shift.dtsi>` を削除 | keymap diff が include 1 行削除のみ | A.2 | cc:完了 |
| A.4 | `config/lism_right.conf` から `CONFIG_LAYOUT_SHIFT*` 3 行を削除 | right.conf が Phase 0 状態に戻る | A.3 | cc:完了 |
| A.5 | `config/west.yml` から `kot149` remote / `zmk-layout-shift` project を削除 | west.yml diff が remote 2 行 + project 3 行削除 | A.4 | cc:完了 |
| A.6 | `make setup-west` で workspace を再生成（`_west/zmk-layout-shift/` を west に消させる） | `_west/zmk-layout-shift/` が消えている（実際は west が stale ディレクトリとして残すが、ビルドには無影響） | A.5 | cc:完了（注釈: stale dir 残置） |
| A.7 | `make all` 成功、Mac で実機動作確認（既存挙動が壊れていない） | 5 種ビルド成功、Mac で動作 | A.6 | cc:WIP（ビルド成功、実機動作確認は ゆうご さん側待ち） |

#### Phase B: Win 側を US キーボード設定に変更（ゆうご さん作業）

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| B.1 | Windows「設定 > 時刻と言語 > 言語と地域 > 日本語 > 言語のオプション > ハードウェア キーボード レイアウト」を **「英語キーボード (101/104 キー)」** に変更 | 設定画面で「英語キーボード」と表示される ~~| A.7 | cc:TODO（要ゆうご作業） |
| B.2 | Windows にサインアウト / 再起動して設定を反映 | 反映後、LisM の `default_layer` で英字が出る | B.1 | cc:TODO（要ゆうご作業） |
| B.3 | LisM で `mark_layer` K 位置（`&kp PIPE`）を押して Win でも `\|` が出ることを確認 | Win で `\|` が出る | B.2 | cc:TODO（要実機） |

#### Phase C: keymap の共通化（win_*_layer の削除）

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| C.1 | `win_mark_layer` を削除し、`mark_layer` を共有 | keymap から `win_mark_layer` が消える | B.3 | cc:TODO |
| C.2 | `win_arrow_layer` を削除し、`arrow_layer` を共有 | `win_arrow_layer` 削除 | C.1 | cc:TODO |
| C.3 | `win_function_number_layer` を削除し、`function_number_layer` を共有 | `win_function_number_layer` 削除 | C.2 | cc:TODO |
| C.4 | `win_util_layer` を削除し、`util_layer` を共有（Mac/Win 差分のキー＝Mission Control / 仮想デスクトップ系は別マクロ化を検討） | `win_util_layer` 削除、または Mac/Win 差分マクロ | C.3 | cc:TODO |
| C.5 | `win_default_layer` を削除し、`default_layer` を共有 | `win_default_layer` 削除 | C.4 | cc:TODO |
| C.6 | レイヤー番号を 0〜6 に詰める、`#define` を整理、`&lt` 参照を整理 | レイヤー数が減って keymap が短くなる | C.5 | cc:TODO |
| C.7 | `bt_mac` / `bt_win` から `&toggle_off/on 7`（win_default_layer トグル）を削除 | 2 マクロは BT_SEL だけになる | C.6 | cc:TODO |
| C.8 | `make all` 成功、Mac/Win 両方で実機動作確認 | 5 種ビルド成功、両 OS で動作 | C.7 | cc:TODO（要実機） |

#### Phase D: クリーンアップ

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| D.1 | `docs/report/mac-win-mark-layer-comparison.md` を新方針に合わせて更新（または削除） | docs が現状 keymap と一致 | C.8 | cc:TODO |
| D.2 | `feat/layout-shift-v2` を main にマージ、不要ブランチ (`feat/zmk-layout-shift-jis`) と stash (`stash@{0}` `stash@{1}`) を整理 | main 取り込み、退避物の処遇記録 | D.1 | cc:TODO |

---

## アーカイブ

完了した計画は以下に移動する。

- **Win 記号レイヤーを Mac と共通化（旧計画 v2 / zmk-layout-shift 採用）** — 2026-05-12 中断・破棄。理由: Win は LisM (BLE) を US キーボードとして認識するため、JIS HID コード (`LS(INT3)` 等) を送っても期待された記号が出ない。詳細は `docs/plan/Win側USキーボード化に方針変更.md` の「検証結果」参照。Phase 0.2〜1.3 までは実機 OK だったが、Phase 3.1 で Win 側に変換が効かないことを確認して方針変更。
- Win 記号レイヤーを Mac と共通化（旧計画 v1 / `feat/zmk-layout-shift-jis`）— 2026-05-12 中断・破棄。詳細経緯は `docs/plan/layout-shift導入の引き継ぎ資料.md` の Phase A〜E を参照。
- zmk-keymap-editor スキル作成（2026-05-10 完了 / 詳細: `docs/plan/zmk-keymap編集スキル作成計画.md`）
