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
| B.1 | Windows のシステム言語設定 + Microsoft IME のキーボードレイアウトを **「英語キーボード (101/104 キー)」** に変更 | 設定画面で「英語キーボード」と表示される、かつ Win 接続時に LisM で記号が US 配列で出る | A.7 | cc:WIP（システム言語は US 変更済みだが、Win 11 の Microsoft IME UI に「ハードウェア キーボード レイアウト」項目が見当たらず詰まり） |
| B.2 | Windows にサインアウト / 再起動して設定を反映 | 反映後、LisM の `default_layer` で英字が出る | B.1 | cc:WIP（B.1 待ち） |
| B.3 | LisM で `mark_layer` K 位置（`&kp PIPE`）を押して Win でも `\|` が出ることを確認 | Win で `\|` が出る | B.2 | cc:TODO（要実機） |

> **B.1 の詰まりメモ (2026-05-13)**: Windows 11 の Microsoft IME UI から「ハードウェア キーボード レイアウト」項目が消えている。「以前のバージョンの Microsoft IME を使う」トグルを ON にしてサインアウト/インしても項目が出ない。検証で LisM の `mark_layer` W 位置押下時に `@` が出ることから、IME が依然 JIS として解釈している（W 位置の `&kp DOUBLE_QUOTES` = `Shift+'` が JIS で `@` を出すのと一致）。
>
> **次回検討する選択肢**:
> 1. Google IME / Mozc UT / ATOK のいずれかをインストール（サードパーティ IME ならプロパティでキーボードレイアウト変更可能）
> 2. レジストリ直接編集（`HKLM\SYSTEM\CurrentControlSet\Services\i8042prt\Parameters` の `LayerDriver JPN=kbd101.dll` 等）
> 3. Phase B を断念して `win_*_layer` を全部保持する方針にシフト。現状の Phase A ロールバック済みの状態で main にマージし、二重管理を継続。

#### Phase C: keymap 共通化（win_mark / win_function だけ削除）

**方針**: 修飾キー順序やデスクトップ移動関連の OS 差を保つため、`win_default_layer` / `win_arrow_layer` / `win_util_layer` は**保持**する。記号レイヤーと数字ファンクションレイヤーだけ Mac と共有する。

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| C.1 | `#define WIN_MARK_LAYER 8` / `#define WIN_FUNCTION_NUMBER_LAYER 10` を削除（レイヤー番号も詰める） | `WIN_MARK_LAYER`, `WIN_FUNCTION_NUMBER_LAYER` が keymap から消える | B.3 | cc:TODO |
| C.2 | `win_default_layer` の `&lt 8 LANG2` → `&lt 1 LANG2`、`&lt 10 DELETE` → `&lt 3 DELETE` に修正（共通の `mark_layer` / `function_number_layer` を参照） | win_default_layer が共有レイヤーを呼ぶ | C.1 | cc:TODO |
| C.3 | `win_mark_layer` ブロックを削除 | keymap から `win_mark_layer` が消える | C.2 | cc:TODO |
| C.4 | `win_function_number_layer` ブロックを削除 | keymap から `win_function_number_layer` が消える | C.3 | cc:TODO |
| C.5 | 残った `win_arrow_layer` / `win_util_layer` のレイヤー番号 (現在 9, 11) を詰めて、対応する `&lt` 参照も修正 | レイヤー番号が連番、`&lt` 整合 | C.4 | cc:TODO |
| C.6 | `combos` の `layers = <0 1 2 ... 11>` を新しい範囲に修正 | combos が正しいレイヤー範囲を指す | C.5 | cc:TODO |
| C.7 | `make all` 成功、Mac/Win 両方で実機動作確認（特に記号レイヤーが Win でも動くこと） | 5 種ビルド成功、両 OS で動作 | C.6 | cc:TODO（要実機） |

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
