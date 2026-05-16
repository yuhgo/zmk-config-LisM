# zmk-config-LisM Plans.md

最終更新: 2026-05-16

このリポジトリのアクティブなタスクを管理する。詳細な計画は `docs/plan/` 配下の個別ファイルに記述する。

---

## アクティブな計画

### Mac/Win 両方 US 配列前提で keymap を共通化（`conditional_layers` で OS 差吸収）

#### 背景・方針転換のサマリー

これまで「Win の記号レイヤーを Mac と共通化する」ために 4 回方針を試している:

| 試行 | 方針 | 結果 |
|------|------|------|
| v1 (`feat/zmk-layout-shift-jis`) | zmk-layout-shift (試作) で JIS/US 切り替え | 2026-05-12 破棄 |
| v2 (`feat/layout-shift-v2` 初期) | zmk-layout-shift v1 を導入し共通 keymap 化 | 2026-05-12 破棄。Win 側に変換が効かないことが判明 |
| v3 | Win 側 OS 設定を US キーボード化 | 2026-05-16 破棄。Win 11 Microsoft IME UI に項目がなく変更困難 |
| v4 | Win は JIS 配列のまま、JIS HID コードで Mac と揃える | 2026-05-16 破棄。前提誤り（Win も US 配列で使う） |
| **v5 (現行・確定)** | **Mac/Win とも US 配列前提**。記号系は完全共有、OS ショートカット差は `conditional_layers` で吸収 | **これから実装** |

#### 確定方針（2026-05-16 ゆうご さんからの指示）

- **Mac も Win も US 配列で運用**（Win は既に OS 設定済み）
- 記号 / F1-F12 / 数字 / 矢印などの **キー位置と機能はレイヤー単位で完全共有**
- OS ショートカット差は `conditional_layers` を使った **薄いオーバーレイレイヤー** で吸収
- OS 判定のトリガーは **`win_default_layer` (7) が on かどうか**
- IME 切替は **Mac と同じ HID（`LANG1` / `LANG2`）を送る**
- 行頭/行末ジャンプ (Win): `HOME` / `END`
- デスクトップ移動 (Win): 左右 `LG(LC(LEFT/RIGHT))`、上 `LG(TAB)`、下 `LG(D)`
- `default_layer` / `win_default_layer` の修飾キー配置は **現状維持**

#### ゴール（実装完了時に成立している状態）

1. `win_mark_layer` / `win_function_number_layer` は **完全削除**
2. `win_arrow_layer` / `win_util_layer` は **OS 差オーバーレイ（薄いレイヤー）** として再構築
3. `win_default_layer` の `&lt` 参照先は共通レイヤー (1, 2, 3, 6) に付け替え
4. `conditional_layers` 設定で「7 (win_default) + 2 (arrow)」「7 + 6 (util)」のときオーバーレイが自動 on
5. `default_layer` / `win_default_layer` の修飾キー配置は変更なし

---

### Phase A: layout-shift ロールバック（旧計画の残務）

| Task | 内容 | Status |
|------|------|--------|
| A.1〜A.6 | layout-shift ロールバック関連 | cc:完了 |
| A.7 | Mac で実機動作確認（既存挙動が壊れていない） | cc:完了（2026-05-16 G.0.5 と合わせて確認済み） |

---

### Phase G: keymap 共通化（**新規・本命**）

#### G.0 設計フェーズ

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| G.0.1 | `docs/plan/win-layer-redesign.md` を US/US 前提 + `conditional_layers` 方式で確定版に更新 | 確定方針が文書化されている | A.7 | cc:完了 |
| G.0.2 | HTML 版 (`win-layer-redesign.html`) を確定版に追従 | HTML が最新内容と一致 | G.0.1 | cc:完了 |
| G.0.3 | レイヤー番号体系を確定: 0〜7（既存）+ 8 (`win_arrow_overlay`) + 9 (`win_util_overlay`) | レイヤー番号表が確定 | G.0.2 | cc:完了 |
| G.0.4 | OS ショートカット差マッピングを確定: 行頭/行末 = HOME/END、デスクトップ = LG(LC(←/→)) + LG(TAB)/LG(D)、IME = LANG1/2 | 差分マッピングが確定 | G.0.3 | cc:完了 |
| G.0.5 | 設計レビュー（ゆうご さん確認） | OK が出る | G.0.4 | cc:完了（2026-05-16 確認済み） |

#### G.1 実装フェーズ

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| G.1.1 | `win_default_layer` の `&lt` 参照先を共通レイヤー (1/2/3/6) に付け替え | `&lt 1/2/3/6` に変更 | G.0.5 | cc:完了 |
| G.1.2 | `win_mark_layer` と `win_function_number_layer` の 2 ブロックを keymap から削除 | 2 レイヤーが消える | G.1.1 | cc:完了 |
| G.1.3 | `win_arrow_layer` を **薄いオーバーレイ**として書き直し: A 位置 `&kp HOME`、F 位置 `&kp END`、他は `&trans` | `win_arrow_layer` がオーバーレイ仕様 | G.1.2 | cc:完了 |
| G.1.4 | `win_util_layer` を **薄いオーバーレイ**として書き直し: H 位置 `LG(LC(LEFT))`、J 位置 `LG(D)`、K 位置 `LG(TAB)`、L 位置 `LG(LC(RIGHT))`、他は `&trans`（メディア / BT 系は trans で共通レイヤーが見える） | `win_util_layer` がオーバーレイ仕様 | G.1.3 | cc:完了 |
| G.1.5 | `#define WIN_MARK_LAYER` / `WIN_FUNCTION_NUMBER_LAYER` を削除し、`WIN_ARROW_LAYER` / `WIN_UTIL_LAYER` のレイヤー番号を 8 / 9 に詰める（または `WIN_ARROW_OVERLAY` / `WIN_UTIL_OVERLAY` にリネーム） | レイヤー番号が連番 | G.1.4 | cc:完了（OVERLAY にリネーム） |
| G.1.6 | `conditional_layers` を追加: `if-layers = <7 2>; then-layer = <8>;` と `if-layers = <7 6>; then-layer = <9>;` | conditional_layers ノードが存在 | G.1.5 | cc:完了 |
| G.1.7 | `combos` の `layers = <0 1 2 ... 11>` を `<0 1 2 3 4 5 6 7 8 9>` に修正 | combos のレイヤー範囲が正しい | G.1.6 | cc:完了 |
| G.1.8 | `default_layer` / `win_default_layer` の修飾キー配置に diff がないことを確認 | 修飾キー配置 diff なし | G.1.7 | cc:完了 |

#### G.2 検証フェーズ

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| G.2.1 | `make all` / `make all_studio` で 6 種すべてビルド成功 | ビルドログ全グリーン | G.1.8 | cc:完了（2026-05-16 Dev Container で確認） |
| G.2.2 | Mac 実機で全レイヤーが壊れていないことを確認 | Mac 側で記号・矢印・F1-F12・util 全動作 | G.2.1 | cc:TODO（要実機・ゆうご さん） |
| G.2.3 | Win 実機で共通レイヤーが期待通り動くことを確認（記号・F1-F12 が Mac と同じ位置で出る） | Win 側で記号系全レイヤー動作 | G.2.2 | cc:TODO（要実機・ゆうご さん） |
| G.2.4 | Win 実機で OS 差オーバーレイが効くことを確認: arrow_layer + win_default で HOME/END、util_layer + win_default で LG(LC(←/→))/LG(TAB)/LG(D) | OS 差動作 OK | G.2.3 | cc:TODO（要実機・ゆうご さん） |
| G.2.5 | Mac で同じキー位置を押したとき、Mac 側の動作（LG(LEFT)/RC(LEFT) 等）になることを確認（オーバーレイが Mac では発動しない） | Mac 側のショートカット動作 OK | G.2.4 | cc:TODO（要実機・ゆうご さん） |

---

### Phase F: クリーンアップ

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| F.1 | `docs/report/mac-win-mark-layer-comparison.md` を新方針 (Phase G 結果) に合わせて更新 | docs が現状 keymap と一致 | G.2.5 | cc:TODO |
| F.2 | 過去計画書（`Win側USキーボード化に方針変更.md` / `layout-shift導入の引き継ぎ資料.md`）を archive ディレクトリに移動、または「破棄」ステータスを明記 | 旧計画書の現状がわかる | F.1 | cc:TODO |
| F.3 | `feat/layout-shift-v2` を main にマージ、不要ブランチ (`feat/zmk-layout-shift-jis`) と stash (`stash@{0}` `stash@{1}`) を整理 | main 取り込み、退避物の処遇記録 | F.2 | cc:TODO |

---

### Phase H: レイヤー番号再配置（Win 不具合 hotfix）

実機検証で発覚した不具合への対応:
- **症状**: Win 側で記号レイヤー・BT 切替が効かない
- **原因**: ZMK は「レイヤー番号が大きい方が優先」のルール。`win_default_layer (7)` が共通レイヤー (1〜6) より上にあったため、Win 時に `mark_layer (1)` などを on にしても win_default_layer に上書きされて見えなかった
- **対策**: `win_default_layer` を 1 に下げ、共通レイヤーを 2〜7 に上げる

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| H.1 | `#define` のレイヤー番号を再配置（win_default_layer=1, arrow=2, mark=3, function=4, mouse=5, scroll=6, util=7） | #define が新番号 | G.2.1 | cc:完了 |
| H.2 | `&lt` / `&mo` / `&toggle_on/off` の番号参照を全部更新 | 参照が新番号と一致 | H.1 | cc:完了 |
| H.3 | `conditional_layers` の `if-layers` を更新（`<7 2>`→`<1 2>`, `<7 6>`→`<1 7>`） | conditional_layers が新番号 | H.2 | cc:完了 |
| H.4 | keymap ブロックの出現順序を新しい番号順に並び替え（win_default_layer を 2 番目、arrow_layer を 3 番目に） | ブロック順序＝レイヤー番号 | H.3 | cc:完了 |
| H.5 | 再ビルド（make all_studio で 6 種全部） | ビルド全グリーン | H.4 | cc:完了 |
| H.6 | Win 実機で記号レイヤー・BT 切替が機能することを確認 | Win で記号・BT 動作 | H.5 | cc:完了（2026-05-17 確認済み） |
| H.7 | Mac 実機で既存挙動が壊れていないことを確認 | Mac で全レイヤー動作 | H.6 | cc:TODO（要実機・ゆうご さん） |

### Phase I: トラックボール scroller 契約の維持（H 後の hotfix）

実機検証で発覚した追加不具合:
- **症状**: スクロールレイヤーが効かなくなった（H.6 で記号・BT は直ったが、スクロールが死んだ）
- **原因**: `_west/zmk-keyboards-LisM/boards/shields/lism/trackball_r.overlay` の `scroller { layers = <2>; }` で「レイヤー 2 が active のときトラックボール XY をスクロールに変換」がハードコードされていた。旧設計ではレイヤー 2 = `arrow_layer` だったが、Phase H で 2 = `mark_layer` に変わり、トラックボール契約が壊れた
- **対策**: Phase H で割り振った `mark_layer` と `arrow_layer` の番号を入れ替え、`arrow_layer = 2` の契約を維持する

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| I.1 | `#define` で `ARROW_LAYER` と `MARK_LAYER` の番号を入れ替え（arrow=2, mark=3） | #define が新番号 | H.5 | cc:完了 |
| I.2 | `&lt` 参照を更新（`&lt 2 SPACE`/`&lt 2 L`→arrow=2、`&lt 3 LANG2`→mark=3） | 参照が新番号と一致 | I.1 | cc:完了 |
| I.3 | `conditional_layers` の `<1 3>` を `<1 2>` に戻す（win+arrow→8） | conditional_layers が新番号 | I.2 | cc:完了 |
| I.4 | keymap ブロックの順序: `arrow_layer` を `mark_layer` より前に移動 | ブロック順序＝レイヤー番号 | I.3 | cc:完了 |
| I.5 | 再ビルド（make 通常版 + 必要なら all_studio） | ビルド全グリーン | I.4 | cc:完了 |
| I.6 | Win/Mac 実機で記号・BT・スクロール・OS 差オーバーレイ全部動くことを確認 | 全レイヤー動作 | I.5 | cc:完了（2026-05-17 確認済み） |

### Phase J: Win 用 LGUI ショートカット復活（mark_layer 差分）

実機検証で発覚した追加要件:
- **症状**: Phase G 前の `win_mark_layer` row4 右親指位置に `&kp LGUI`（Windows キー単体）があったが、`win_mark_layer` 削除時に消えた
- **元の挙動**: 左親指で `&lt 8 LANG2`（旧 win_mark）押下しながら右親指の `&lt 11 LANG1` 位置を押すと **LGUI**（スタートメニュー）が出る
- **Mac の同じ位置**: `&kp LG(SPACE)`（Spotlight / IME 切替）→ Win ではこのままだと Windows+Space = 言語切替になり、スタートメニューを開けない
- **対策**: `win_mark_overlay` (10) を新規追加し、`conditional_layers` で 1+3 のとき自動 on

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| J.1 | `win_mark_overlay` レイヤーブロックを追加（右親指位置 = `&kp LGUI`、他は `&trans`） | overlay ブロックが存在 | I.6 | cc:完了 |
| J.2 | `conditional_layers` に `win_mark_overlay` を追加（`if-layers = <1 3>; then-layer = <10>`） | conditional_layers に新エントリ | J.1 | cc:完了 |
| J.3 | `#define WIN_MARK_OVERLAY 10` を追加 | define が存在 | J.2 | cc:完了 |
| J.4 | `combos` の `layers` を `<0..10>` に拡張 | layers が新範囲 | J.3 | cc:完了 |
| J.5 | 再ビルド | ビルドグリーン | J.4 | cc:完了 |
| J.6 | Win 実機で「左親指 mark hold + 右親指で LGUI（スタートメニュー）」が出ることを確認 | LGUI 動作 | J.5 | cc:TODO（要実機・ゆうご さん） |

---

## 参考: 最終レイヤー構成

| # | レイヤー | 役割 | 備考 |
|---|---------|------|------|
| 0 | `default_layer` | Mac 用文字入力 + 修飾キー | 変更なし |
| 1 | `win_default_layer` | Win 用文字入力 + 修飾キー | OS 判定フラグを兼ねる（Phase H で番号変更） |
| **2** | **`arrow_layer`** | **矢印 + 行頭・行末ジャンプ (Mac 仕様)** | **Mac/Win 共有（差分は #8 で吸収）。トラックボール scroller 契約のためここで固定（Phase I）** |
| 3 | `mark_layer` | 記号 | Mac/Win 共有 |
| 4 | `function_number_layer` | F1-F12 / 数字 | Mac/Win 共有 |
| 5 | `mouse_layer` | マウス | 変更なし |
| 6 | `scroll_layer` | スクロール | 変更なし |
| 7 | `util_layer` | メディア / BT / 輝度 / Studio / デスクトップ移動 (Mac 仕様) | Mac/Win 共有（差分は #9 で吸収） |
| 8 | `win_arrow_overlay` | Win 用 arrow 差分（HOME/END） | `conditional_layers` で 1+2 のとき on |
| 9 | `win_util_overlay` | Win 用 util 差分（デスクトップ移動） | `conditional_layers` で 1+7 のとき on |
| **10** | **`win_mark_overlay`** | **Win 用 mark 差分（右親指 = `LGUI`）** | **`conditional_layers` で 1+3 のとき on（Phase J）** |

combos の `layers = <0 1 2 3 4 5 6 7 8 9 10>` に修正。

---

## アーカイブ

- **Win 側を JIS 配列前提で再設計 (v4)** — 2026-05-16 破棄。前提誤りが判明（Win も US 配列で運用）。
- **Win 側 OS を US キーボード扱いにする (v3)** — 2026-05-16 破棄。Win 11 Microsoft IME UI から項目消失。
- **Win 記号レイヤーを Mac と共通化（v2 / zmk-layout-shift 採用）** — 2026-05-12 中断・破棄。詳細: `docs/plan/layout-shift導入の引き継ぎ資料.md`。
- Win 記号レイヤーを Mac と共通化（v1 / `feat/zmk-layout-shift-jis`）— 2026-05-12 中断・破棄。
- zmk-keymap-editor スキル作成（2026-05-10 完了 / 詳細: `docs/plan/zmk-keymap編集スキル作成計画.md`）
