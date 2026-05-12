# zmk-config-LisM Plans.md

最終更新: 2026-05-12

このリポジトリのアクティブなタスクを管理する。詳細な計画は `docs/plan/` 配下の個別ファイルに記述する。

---

## アクティブな計画

### Win 記号レイヤーを Mac と共通化（zmk-layout-shift v2 再導入）

詳細: [docs/plan/layout-shift導入の引き継ぎ資料.md](docs/plan/layout-shift導入の引き継ぎ資料.md)

前回 (`feat/zmk-layout-shift-jis`) は ① 左右両方の conf に CONFIG を入れた、② `layout_shift_kp_override.dtsi` で `&kp` を一括上書きした、③ `PERSISTENT_STATE=y` のままだった、の 3 点が複合して split pairing 崩壊と Mission Control / Spaces 不動を招いた。今回は「central だけ」「`&kpls` 個別書き換え」「`PERSISTENT_STATE=n`」で再挑戦する。

#### Phase 0: 準備

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 0.1 | `feat/layout-shift-v2` ブランチを main から作成 | `git branch --show-current` が `feat/layout-shift-v2` | - | cc:完了 |
| 0.2 | 現状の `make all` がクリーンに通り全 5 種 .uf2 が出ることを再確認（Studio 版は除外、Studio 含む 7 種が必要なら `make all_studio`） | `firmware_builds/` に 5 ファイル（左 ×2 + 右 ×2 + settings_reset） | 0.1 | cc:完了 |

#### Phase 1: layout-shift モジュール組み込み（無効状態で導入）

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 1.1 | `config/west.yml` に `kot149/zmk-layout-shift v1` を追加（`zmk-driver-paw3222` と同じ並び） | west.yml diff が公式 README 通り、`make setup-west` 後 `_west/zmk-layout-shift/` が存在 | 0.2 | cc:完了 |
| 1.2 | `config/lism_right.conf` に `CONFIG_LAYOUT_SHIFT=y` / `CONFIG_LAYOUT_SHIFT_TARGET_JIS=y` / `CONFIG_LAYOUT_SHIFT_PERSISTENT_STATE=n` を追加（**left.conf には何も追加しない**） | right.conf 差分のみ、left.conf 無変更 | 1.1 | cc:完了 |
| 1.3 | `config/lism.keymap` の include に `<layout_shift.dtsi>` のみ追加（`layout_shift_kp_override.dtsi` は include しない） | keymap 差分が include 1 行追加のみ | 1.2 | cc:完了 |
| 1.4 | `make all` が成功し、左右ペアリングと既存挙動に変化がないことを実機で確認（`&kpls` 未使用なので無効と等価） | 7 種ビルド成功＋実機で青 LED＋既存キー全部効く | 1.3 | cc:TODO（要実機） |

#### Phase 2: プロファイル切替に layout-shift トグルを連動

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 2.1 | `bt_mac` マクロ先頭に `&tog_ls_off`、`bt_win` マクロ先頭に `&tog_ls_on` を追加 | keymap diff が該当 2 マクロのみ | 1.4 | cc:TODO |
| 2.2 | `make all` 成功＋実機で `bt_mac`/`bt_win` 切替が以前と同じく動くこと（記号はまだ未変更なので変化なしのはず） | プロファイル切替が壊れない | 2.1 | cc:TODO（要実機） |

#### Phase 3: 段階的に `&kpls` 化（小さく試す）

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 3.1 | `mark_layer` の記号キーを 1〜2 個だけ `&kpls` に書き換え（修飾単体・Cmd+Space は `&kp` のまま） | 該当キーが Mac で従来通り出る | 2.2 | cc:TODO（要実機） |
| 3.2 | 問題なければ `mark_layer` 全体の記号キーを `&kpls` 化 | mark_layer の記号が Mac で全部出る | 3.1 | cc:TODO（要実機） |
| 3.3 | `combos` の `minus`/`plus`/`asterisk`/`slash` を `&kpls` 化 | コンボで対応記号が Mac で出る | 3.2 | cc:TODO（要実機） |

#### Phase 4: Win (JIS) での検証と回帰チェック

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 4.1 | Win (JIS) で `bt_win` 切替後 `mark_layer` を試打 | `docs/report/mac-win-mark-layer-comparison.md` の Mac 列と同じ記号が出る（特に K 位置の `\|`） | 3.3 | cc:TODO（要実機） |
| 4.2 | Mac で `bt_mac` 切替後 **Mission Control (Ctrl+↑) と Spaces 切替 (Ctrl+←/→) が効く** | 前回最大の地雷を踏んでいないことを確認 | 3.3 | cc:TODO（要実機） |
| 4.3 | `bt_mac`/`bt_win` 切替直後の試打で記号崩れがないこと（初期状態の正しさ） | 切替後に記号が即正しく出る | 4.1, 4.2 | cc:TODO（要実機） |
| 4.4 | 左キーボードが青 LED で繋がり続けること（Phase C 紫 LED 再発しない） | 数時間運用しても紫にならない | 4.1, 4.2 | cc:TODO（要実機） |

#### Phase 5: 後始末

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 5.1 | 不要になった `win_mark_layer` を削除して `mark_layer` を共有（任意・今回は keymap 修正範囲を最小化したいなら見送り可） | 削除した場合は keymap が短くなる、見送る場合は判断を docs に記録 | 4.3, 4.4 | cc:TODO |
| 5.2 | `docs/report/mac-win-mark-layer-comparison.md` を更新（実装と一致させる） | docs 内容が現 keymap と一致 | 5.1 | cc:TODO |
| 5.3 | `feat/layout-shift-v2` を main にマージし `feat/zmk-layout-shift-jis` ブランチと `stash@{0}` `stash@{1}` を整理（残すか消すかを記録） | main に取り込み済み、退避物の処遇が記録される | 5.2 | cc:TODO |

---

## アーカイブ

完了した計画は以下に移動する。

- Win 記号レイヤーを Mac と共通化（旧計画 v1 / `feat/zmk-layout-shift-jis`）— 2026-05-12 中断・破棄。詳細経緯は `docs/plan/layout-shift導入の引き継ぎ資料.md` の Phase A〜E を参照。新計画 v2 として再起動。
- zmk-keymap-editor スキル作成（2026-05-10 完了 / 詳細: `docs/plan/zmk-keymap編集スキル作成計画.md`）
