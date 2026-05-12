# zmk-keymap-editor スキル作成計画

作成日: 2026-05-10

## 目的・背景

zmk-config-LisM 専用に、ZMK の keymap・config・build 設定を安全に編集するための Claude Code skill を作る。
公式 ZMK ドキュメント（<https://zmk.dev/docs/keymaps>, <https://zmk.dev/docs/zmk-cli>）を一次情報として、
最新の devicetree 構文と ZMK CLI コマンドを反映する。

## スコープ

**含むもの**:

- keymap 編集（レイヤー追加/削除、キーバインド変更、display-name 設定）
- 高度機能（combos、macros、hold-tap、mod-tap、layer-tap、sticky-key）
- `*.conf` (Kconfig) 編集（スリープ、RGB、Bluetooth など）
- `build.yaml` 管理（バリエーション追加、snippet/cmake-args 設定）
- `west.yml` / `config/west.yml` 管理（モジュール追加・更新）
- ZMK CLI 連携（`zmk module add`, `zmk update`, `zmk keyboard list` などのラッパー）
- ビルド検証（オプション、`build-firmware` skill 連携）

**含まないもの**:

- ZMK 本体のソース改変
- 物理レイアウト（`lism.json`）の大幅な再設計
- 新規キーボード定義の作成（既存 LisM 前提）

## 重要な技術的発見

ZMK CLI（`zmk` コマンド）は keymap編集を **「エディタで開く」機能のみ** 提供しており、
構造化API（プログラマティックなレイヤー追加・キー差し替え）は持たない。

→ スキルは **二層構成** にする:

1. **ZMK CLI ラッパー層**: リポジトリ操作（`zmk module add`, `zmk update`, `zmk config` など）
2. **devicetree 編集層**: keymap/combos/macros を Edit/Write で直接編集（CLI 非対応領域）

## 優先度マトリクス

### Required（最優先）

- keymap のレイヤー定義の理解と編集
- キーバインド差し替え（`&kp X` → `&kp Y`）
- combos の追加/編集
- ZMK 公式キーコード（`dt-bindings/zmk/keys.h`）リファレンス組み込み

### Recommended（推奨）

- macros / hold-tap / mod-tap の追加
- `*.conf` 編集（Kconfig オプション）
- `build.yaml` バリアント追加
- ZMK CLI ラッパー（`zmk module add`, `zmk update`）
- ビルド検証フラグ（`build-firmware` skill と連携）

### Optional（任意）

- keymap-drawer 連携で視覚化
- ZMK Studio 対応版の自動生成

## TDD 採用判断

**採用しない**（このスキルは構成ファイル編集 skill であり、テスト対象がない）。
代わりに **検証ステップ** として以下を skill 内で必須化:

- 編集後に `git diff` を必ず提示
- devicetree 構文の括弧/セミコロン整合チェック
- オプションフラグでビルド検証（`make single` or `mise run dc-exec make single`）

## Phase 1: スキル骨格と基本 keymap 編集

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 1.1 | `.claude/skills/zmk-keymap-editor/` ディレクトリと `SKILL.md` の骨格作成 | SKILL.md が skill-creator フォーマット (description, when_to_use, instructions) を満たす | - | cc:TODO |
| 1.2 | LisM ハードウェア情報を `references/lism-layout.md` に固定（42キー配置、エンコーダー、左右 col 番号） | キー位置 0-41 が config/lism.keymap と一致して記述されている | 1.1 | cc:TODO |
| 1.3 | ZMK keymap 基礎リファレンスを `references/keymap-basics.md` に作成（include、レイヤー定義、bindings、key-position） | 公式ドキュメント (<https://zmk.dev/docs/keymaps>) の最新仕様が反映されている | 1.1 | cc:TODO |
| 1.4 | キーコード一覧を `references/keycodes.md` に作成（基本キー、修飾、メディア、Bluetooth、レイヤー切替） | `dt-bindings/zmk/keys.h` の主要キーコードが網羅されている | 1.1 | cc:TODO |
| 1.5 | レイヤー追加/削除/キー差し替えの手順を SKILL.md に記述 | `&kp A` を `&kp B` に変える例が動作する | 1.2, 1.3, 1.4 | cc:TODO |

## Phase 2: 高度機能（combos / macros / hold-tap）

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 2.1 | combos リファレンスを `references/combos.md` に作成（key-positions, timeout-ms, layers, slow-release, require-prior-idle-ms） | 公式 (<https://zmk.dev/docs/keymaps/combos>) の全パラメータを記述 | 1.3 | cc:TODO |
| 2.2 | macros リファレンスを `references/macros.md` に作成（wait-ms, tap-ms, macro_press/tap/release, macro_pause_for_release） | 公式 (<https://zmk.dev/docs/keymaps/behaviors/macros>) の全モードを記述 | 1.3 | cc:TODO |
| 2.3 | hold-tap リファレンスを `references/hold-tap.md` に作成（4 flavor、tapping-term-ms、quick-tap-ms、bindings） | 公式 (<https://zmk.dev/docs/keymaps/behaviors/hold-tap>) の全 flavor を記述 | 1.3 | cc:TODO |
| 2.4 | combos 追加手順を SKILL.md に記述 | LisM の左右キー位置を使った combo 追加例が動作 | 2.1, 1.2 | cc:TODO |
| 2.5 | macro 追加手順を SKILL.md に記述 | 文字列入力 macro の追加例が動作 | 2.2 | cc:TODO |
| 2.6 | hold-tap カスタム behavior の追加手順 | 既存 lt/mt 以外のカスタム hold-tap 定義例が動作 | 2.3 | cc:TODO |

## Phase 3: ビルド設定（conf / build.yaml / west.yml）

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 3.1 | Kconfig (`*.conf`) 主要オプションを `references/kconfig.md` に整理（スリープ、Bluetooth、RGB、エンコーダー、バッテリー） | 現在の `lism_left.conf` / `lism_right.conf` の全項目が説明されている | 1.1 | cc:TODO |
| 3.2 | `build.yaml` スキーマを `references/build-yaml.md` に整理（board, shield, snippet, cmake-args, artifact-name） | LisM の全バリアント (trackball x2 + Studio) が例として記述されている | 1.1 | cc:TODO |
| 3.3 | `west.yml` モジュール管理手順を `references/west-yml.md` に整理 | LisM の依存（zmk, paw3222, rgbled-widget, charge-indicator, keyboards-LisM）が例示されている | 1.1 | cc:TODO |
| 3.4 | conf 編集手順を SKILL.md に記述（左右同期ルールも含む） | スリープ時間変更例が動作 | 3.1 | cc:TODO |
| 3.5 | build.yaml バリアント追加手順を SKILL.md に記述 | 新規 artifact-name の追加例が動作 | 3.2 | cc:TODO |

## Phase 4: ZMK CLI ラッパーとビルド検証

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 4.1 | ZMK CLI コマンドリファレンスを `references/zmk-cli.md` に作成（`zmk module add/remove/list`, `zmk update`, `zmk keyboard list`, `zmk config`） | 公式 (<https://zmk.dev/docs/zmk-cli>) の全コマンドが記述されている | - | cc:TODO |
| 4.2 | ZMK CLI が未インストールの場合の検出と案内（`uv tool install zmk` の提示） | `zmk --version` が無い環境でも代替手順を案内できる | 4.1 | cc:TODO |
| 4.3 | モジュール追加フローを SKILL.md に記述（`zmk module add` → `west.yml` 確認 → `make setup-west`） | 新規モジュール追加の例が動作 | 4.1, 3.3 | cc:TODO |
| 4.4 | ビルド検証オプション（`--verify` フラグ）を SKILL.md に記述 | フラグ ON 時のみ `build-firmware` skill を呼び出す | 1.5 | cc:TODO |
| 4.5 | 編集後の必須検証ステップ（`git diff`、devicetree 構文チェック） | SKILL.md に明記され、出力例がある | 1.5 | cc:TODO |

## Phase 5: 仕上げと検証

| Task | 内容 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 5.1 | スキル全体の when_to_use トリガーを最適化（"keymap編集" "コンボ追加" "ZMK" 等） | 自然言語トリガーが網羅されている | 1.5, 2.6, 3.5, 4.5 | cc:TODO |
| 5.2 | 既存 `build-firmware` skill との役割分担を明記 | 両 skill の README に相互参照あり | 4.4 | cc:TODO |
| 5.3 | empirical-prompt-tuning skill で skill 動作を検証 | サブエージェントが期待通りに動作 | 5.1 | cc:TODO |
| 5.4 | 実際に keymap 編集を 1 件試して動作確認（例: 新規 combo 追加） | git diff が想定通り、ビルドが通る | 5.1 | cc:TODO |

## 参照ドキュメント

- ZMK CLI: <https://zmk.dev/docs/zmk-cli>
- Keymaps 基本: <https://zmk.dev/docs/keymaps>
- Combos: <https://zmk.dev/docs/keymaps/combos>
- Macros: <https://zmk.dev/docs/keymaps/behaviors/macros>
- Hold-Tap: <https://zmk.dev/docs/keymaps/behaviors/hold-tap>
- Build/Flash: <https://zmk.dev/docs/development/build-flash>
- ZMK Studio: <https://zmk.dev/docs/features/studio>
- 既存スキル: `.claude/skills/build-firmware/`
- 既存 keymap: `config/lism.keymap`, `config/lism.json`, `config/lism_*.conf`
