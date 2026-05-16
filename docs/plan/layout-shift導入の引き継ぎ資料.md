# Layout Shift 導入の引き継ぎ資料

最終更新: 2026-05-12

このドキュメントは「Win 記号レイヤーを Mac と共通化する（zmk-layout-shift 導入）」計画を中断した時点での状態を、次のセッションが拾えるようにまとめたもの。

---

## TL;DR — 今ここから始めるなら

1. 接続復旧済み・旧 keymap で動作確認済み（main ブランチ）
2. layout-shift 再導入は **必ず `lism_right.conf` (central) だけ** に CONFIG を入れる。`lism_left.conf` (peripheral) には入れない
3. `&kp` の一括上書き (`layout_shift_kp_override.dtsi`) は **使わない**。`mark_layer` の記号キーだけ `&kpls` に書き換える方針で
4. **必ず `CONFIG_LAYOUT_SHIFT_PERSISTENT_STATE=n`** を付けて Settings 領域への書き込みを止める（split pairing と競合する疑いが強い）

---

## 現在のリポジトリ状態（main ブランチ）

| 項目 | 値 |
|---|---|
| ブランチ | `main` |
| 最新コミット | `0a8695e Merge remote-tracking branch 'upstream/main'` |
| keymap | 旧 357 行（fork 独自の Mac/Win 分離、bt_mac/bt_win、master_p 等を持つ）を upstream merge 後の新環境に移植 |
| ZMK 本体 | `v0.3.0` 固定（`config/west.yml`） |
| board | `seeeduino_xiao_ble`（旧 `xiao_ble` から変更） |
| shield 構造 | `boards/shields/lism/`（旧 `_west/zmk-keyboards-LisM/` モジュールから本リポに統合） |
| 動作確認 | 両側書き込み + ペアリングまで完了、青 LED |

### 退避済みの作業（stash / 別ブランチ）

- `stash@{0}`: build.yaml と .serena/project.yml のローカル微調整（upstream merge 前に退避、merge 後 unstash していない）
- `stash@{1}`: layout-shift JIS アプローチの試作（`&kpls` 化 + `bt_mac`/`bt_win` に `&tog_ls_off/on` 埋め込み）。ただし**旧 keymap 構造**に対する変更なので、新 keymap には**そのまま適用できない**
- `feat/zmk-layout-shift-jis` ブランチ: 上記試作の commit 履歴と Plans.md / 詳細計画 docs を保持

参考用にどちらも消さない方が無難。

---

## 一連の流れと得られた知見

### Phase A: layout-shift 導入の最初の試み（feat/zmk-layout-shift-jis ブランチ）

**やったこと**:
1. `config/west.yml` に `kot149/zmk-layout-shift v1` を追加
2. **左右両方の conf に `CONFIG_LAYOUT_SHIFT_TARGET_JIS=y` を追加** ← これが後の元凶
3. `lism.keymap` に `layout_shift.dtsi` と `layout_shift_kp_override.dtsi` を include（`&kp` 一括上書き）
4. `bt_mac` マクロに `&tog_ls_off`、`bt_win` マクロに `&tog_ls_on` を埋め込み
5. `win_mark_layer` / `win_function_number_layer` を削除し、Mac 用 `mark_layer` / `function_number_layer` を Win でも共有
6. レイヤー番号を 0–9 に詰めて `&lt` 参照を整理

**結果**: 全 7 種ビルド成功。FLASH 32% 程度。

**実機で出た問題**: Mac の **Mission Control (Ctrl+↑)** と **Spaces 切替 (Ctrl+←/→)** が動かなくなった。書き込み直後の起動から発生。

### Phase B: 原因切り分けと部分修正

**ソースコードを読んで発見した点**:

`_west/zmk-layout-shift/src/behavior_layout_shift_key_press.c` を分析:

- `layout_shift_kp_override.dtsi` は `&kp` を **すべて** layout-shift behavior 経由に置換する
- JIS シフト OFF 状態でも、`raise_zmk_keycode_state_changed_from_encoded()` 経由でキーが送信される
- このパスは ZMK 標準の `behavior-key-press` とは別経路で、`mask_unwanted_modifiers` などの追加処理が挟まる
- 結果として `RC(LEFT_ARROW)` のような **修飾付きキー** で modifier 抑制が起きるリスクがある

**`_west/zmk-layout-shift/src/layouts/layout_jis.h`**:
- 全マッピングが `OPTIONAL_CTRL | OPTIONAL_ALT | OPTIONAL_GUI`
- = 「Ctrl/Alt/GUI が付いていてもマッピング対象」になる
- → JIS シフト ON 時に Cmd+`[` のような修飾付き記号入力も変換されてしまう

**`_west/zmk-layout-shift/Kconfig`**:
- `CONFIG_LAYOUT_SHIFT_PERSISTENT_STATE` のデフォルトが `y`
- → 一度 ON にすると再起動後も保持される
- 内部で `settings_save_one("layout_shift/state", ...)` を呼ぶ。**ZMK の Settings 領域を使う**

**対処**:
- `layout_shift_kp_override.dtsi` の include を削除
- `mark_layer` の各記号キーを `&kpls` に書き換え（修飾キー単体や Cmd+Space は `&kp` のまま）
- `combos` の `minus`/`plus`/`asterisk`/`slash` も `&kpls` 化

→ ビルド成功。

### Phase C: 深刻な問題発生

**実機で**:
1. 左キーボードが繋がらなくなった（LED が青ではなく**紫**）
2. 右の Spaces 右切替 (`RC(RIGHT)`) が効かない
3. main ブランチに戻して再ビルドしても**直らない**

**対処を試みた**:
- 両側に `settings_reset-xiao_ble-zmk.uf2` で Settings 領域消去
- main ブランチでファームを再生成して書き込み直し
- → **直らない**

### Phase D: 真の原因発見

サブモジュール調査で:

```
$ cd _west/zmk-keyboards-LisM && git log -1
e94aeb5 Add README with project archive notice
```

**`zmk-keyboards-LisM` リポはアーカイブされていた**。README に「今後の開発・更新は `zmk-config-LisM` へ統合」と書かれていた。

upstream 比較で:

```
$ git log main..upstream/main --oneline
5034cac Update README with DYA Studio branch info
a66af8f [Draw] zmk-rgbled-widgetをzmkバージョン(v0.3)に合わせる
86e8923 zmk-rgbled-widgetをzmkバージョン(v0.3)に合わせる
24dbc42 ローカルビルド時間の説明修正
4a6b50a ローカルビルドへ並列処理を追加
7f1bbe0 add .gitattributes
847c55c shield名修正
23568ff cmake-args指定方法修正
4434a4c zmk-keyboards-LisM統合
```

**upstream が ZMK v0.3 対応 + 大改修を済ませていて、fork main が 9 コミット遅れていた**。これによって:

- **古い ZMK main** と **古い zmk-keyboards-LisM main** の組み合わせで動いていた split BLE が、`west update` で zmk main が新しくなったことで API が変わり、特に左 (peripheral) との pairing が成立しなくなった可能性が高い
- layout-shift 導入はこの**潜在的な不安定**を顕在化させた引き金にすぎなかった可能性が高い

### Phase E: upstream merge と復旧

**手順**:
1. `git stash push` で build.yaml / .serena の変更を退避（merge を通すため）
2. `git merge upstream/main --no-edit` で 9 コミット取り込み
3. conflict 解決: `config/lism.keymap`、`build.yaml`、`.devcontainer/devcontainer.json`、`README.md` は `--theirs`（upstream 側）を採用
4. merge commit `0a8695e` を作成
5. `make setup-west` で west workspace を v0.3 構成に更新
6. `make all` で新形式 .uf2 を生成
7. 両側を **新 `settings_reset-seeeduino_xiao_ble-zmk.uf2`** で初期化
8. 左 ← `lism_left_peripheral_non_trackball.uf2`、右 ← `lism_right_central_trackball.uf2`
9. PC の Bluetooth から旧 LisM 登録を削除 → 再ペアリング
10. **接続復旧、青 LED**

### Phase F: 旧 keymap の移植

upstream の新 `lism.keymap` は **upstream 作者による完全な書き直し**（135 行、Mac/Win 分離なし、bt_mac/bt_win なし、master_p なし）になっていた。

ユーザーの選択により、`ff12009` 時点の旧 357 行 keymap を新環境に上書き:

```bash
git show ff12009:config/lism.keymap > config/lism.keymap
```

→ **ZMK v0.3 / 新 board / 新 shield 構造でも無修正で全ビルド成功**。実機動作も問題なし（接続復旧確認済み）。

---

## 環境差分まとめ（旧 → 新）

| 項目 | 旧 | 新（現在） |
|---|---|---|
| ZMK 本体 revision | `main`（rolling） | `v0.3.0` 固定 |
| zmk-rgbled-widget | `main` | `v0.3` |
| board | `xiao_ble` | `seeeduino_xiao_ble` |
| shield 配置 | `_west/zmk-keyboards-LisM/boards/shields/lism/` | `boards/shields/lism/`（自リポ内） |
| build.yaml の artifact-name | `lism_left_non_trackball.uf2` | `lism_left_peripheral_non_trackball.uf2`（peripheral/central 明示） |
| settings_reset | `settings_reset-xiao_ble-zmk.uf2` | `settings_reset-seeeduino_xiao_ble-zmk.uf2` |
| central/peripheral 役割 | ファーム実行時に決定 | **build 時に固定**（`-DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y` を central 側にだけ付ける） |
| TB / non-TB の指定 | `overlay-path` で個別に切替 | **snippet** で切替（`trackball-central` / `non-trackball-peripheral` 等） |

---

## layout-shift を再導入する際の方針

### やるべきこと

1. **必ず `lism_right.conf` (central) だけ** に以下を入れる:
   ```
   CONFIG_LAYOUT_SHIFT=y
   CONFIG_LAYOUT_SHIFT_TARGET_JIS=y
   CONFIG_LAYOUT_SHIFT_PERSISTENT_STATE=n   # ← 必須
   ```
2. `lism_left.conf` (peripheral) には**何も追加しない**
3. `config/west.yml` に `kot149/zmk-layout-shift v1` を追加（`zmk-driver-paw3222` などと同じ並び）
4. `config/lism.keymap` の include に **`<layout_shift.dtsi>` のみ** を追加（`layout_shift_kp_override.dtsi` は使わない）
5. `mark_layer` の各記号キーを `&kpls <記号>` に書き換える（`&kp LEFT_SHIFT` などの修飾キー単体は `&kp` のまま）
6. `combos` の `minus`/`plus`/`asterisk`/`slash` も `&kpls` 化
7. `bt_mac` マクロ先頭に `&tog_ls_off`、`bt_win` マクロ先頭に `&tog_ls_on` を追加（プロファイル切替 = レイアウトシフト切替）

### やってはいけないこと（地雷）

| 地雷 | 起きること |
|---|---|
| 左右両方の conf に `CONFIG_LAYOUT_SHIFT_TARGET_JIS=y` を入れる | peripheral の Settings 領域で split pairing と layout-shift state が競合する疑い |
| `layout_shift_kp_override.dtsi` を include する | `&kp` 全体が layout-shift behavior 経由になり、`RC(LEFT)` 等の修飾付きキーで modifier mask が誤発火する |
| `CONFIG_LAYOUT_SHIFT_PERSISTENT_STATE=y`（デフォルト）のまま | 一度 ON にしたら再起動後も持続。Settings 領域への書き込みで split pairing と競合するリスク |
| Win 仮想デスクトップなど OS 固有ショートカットを `&kpls` 化 | layout-shift は記号変換専用。`LG(LC(LEFT))` 等を `&kpls` にしても意味がないし副作用リスクあり |

### 検証順序

1. **conf と west.yml だけ追加** → ビルド → 動作変化なしを確認（layout-shift behavior は登録されるが、誰も `&kpls` を呼ばないので無効と等価）
2. **bt_mac/bt_win に tog_ls_off/on 追加** → ビルド → bt_mac / bt_win の切替が以前と変わらず動くか確認
3. **mark_layer の記号キー 1〜2 個だけ `&kpls` 化** → ビルド → 実機で記号がちゃんと出るか確認
4. **問題なければ mark_layer 全体と combos も `&kpls` 化**
5. **Win (JIS) で `bt_win` 切替 → mark_layer を試打** → Mac と同じ記号が出るか確認
6. **Mac で `bt_mac` 切替 → Mission Control / Spaces 切替が効くか必ず確認**（前回の最大の問題点）

---

## ブランチ運用の提案

### 案 1: feat/zmk-layout-shift-jis を捨てて新規ブランチで再挑戦（推奨）

旧ブランチは「旧 keymap 構造 + 失敗した kp 上書きアプローチ」が混ざっていて、新 keymap には流用できない。

```bash
git checkout -b feat/layout-shift-v2
# 上記「再導入の方針」に従って実装
```

### 案 2: 旧 feat ブランチを保持して別ブランチに作業

`feat/zmk-layout-shift-jis` と `stash@{1}` は履歴として残す。新 keymap 構造ベースで別ブランチを切る（案 1 と実質同じだが、過去の試行を意図的に保存）。

### 案 3: 旧ブランチで unstash して手動マージ

`stash@{1}` を `feat/zmk-layout-shift-jis` に pop し、旧 keymap 部分を新 keymap に手動で書き戻す。**コンフリクトと作業量が大きい**ので非推奨。

---

## 復旧時に学んだ書き込み手順（恒久的に有効）

接続が壊れたときの「確実な復旧手順」:

1. 両キーボードを**新形式** `settings_reset-seeeduino_xiao_ble-zmk.uf2` で初期化
2. **左 (peripheral)** ← `lism_left_peripheral_non_trackball.uf2`（あなたの構成は左 TB なし）
3. **右 (central)** ← `lism_right_central_trackball.uf2`（右 TB あり）
4. PC 側の Bluetooth で **既存の LisM 登録を必ず削除**
5. 右 → 左の順で電源 ON、split 自動ペアリング
6. PC でペアリング → 青 LED

---

## 関連リソース

- 詳細計画（旧）: `docs/plan/win記号レイヤーをmacと共通化する計画.md`
- 旧フォーマット時の比較表: `docs/report/mac-win-mark-layer-comparison.md`
- 退避中のブランチ: `feat/zmk-layout-shift-jis`
- 退避中の stash: `stash@{1}`（layout-shift 試作）、`stash@{0}`（build.yaml ローカル微調整）
- zmk-layout-shift 公式: https://github.com/kot149/zmk-layout-shift
- upstream: https://github.com/4mplelab/zmk-config-LisM

---

## チェックリスト（次セッション冒頭で確認したいこと）

- [ ] `git branch --show-current` が `main` か
- [ ] `git status` が clean か（必要なら stash pop / 新ブランチ作成）
- [ ] `_west/` が存在し、`make all` でビルド通るか
- [ ] 実機の LED が青で、両側とも繋がる状態か（紫が出たら過去 Phase C と同じ症状）
- [ ] `firmware_builds/` 配下に `lism_left_peripheral_*` と `lism_right_central_*` の最新ビルドがあるか
