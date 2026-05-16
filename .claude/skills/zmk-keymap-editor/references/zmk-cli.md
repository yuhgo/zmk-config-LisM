# ZMK CLI コマンド

公式: <https://zmk.dev/docs/zmk-cli>

`zmk` コマンド（Python 製）で ZMK 設定リポジトリを管理する。

## インストール確認

```bash
zmk --version
```

未インストールの場合:

```bash
# uv (推奨)
uv tool install zmk

# pip
pipx install zmk
```

更新:

```bash
uv tool upgrade zmk
```

## 重要な制限

ZMK CLI は **keymap の中身（レイヤー、bindings、combos）を構造化編集する API は持たない**。
keymap 編集機能は実質的に `zmk code` が「エディタで開く」だけ。

このスキルでは:

- **devicetree 編集（keymap/combos/macros/behaviors）** → Edit/Write で直接編集
- **リポジトリ管理（モジュール、ボード、設定）** → ZMK CLI を使用

の二層で扱う。

## コマンド一覧

### リポジトリ初期化

```bash
zmk init
```

GitHub リポジトリの作成とローカルクローンを自動化する（新規セットアップ用）。
LisM は既にセットアップ済みのため通常不要。

### キーボード管理

| コマンド | 動作 |
|---------|------|
| `zmk keyboard add` | 新規キーボードをビルドターゲットに追加（ボード選択を含む） |
| `zmk keyboard remove` | ビルドから除外 |
| `zmk keyboard list` | サポート機器一覧 |
| `zmk keyboard new` | テンプレートから新規キーボード生成 |

`zmk keyboard add` は対話的に board/shield を選び、`build.yaml` を更新する。

### ファイル編集（エディタ起動）

| コマンド | 動作 |
|---------|------|
| `zmk code` | リポジトリをエディタで開く |
| `zmk code <keyboard>` | 該当 keymap ファイルを開く |
| `zmk code --conf <keyboard>` | conf ファイルを開く |
| `zmk code --build` | build.yaml を開く |

エディタは `zmk config core.editor <command>` で設定。
このスキルでは Claude Code 側で Edit ツールを使うため、`zmk code` は通常使わない。

### モジュール管理

| コマンド | 動作 |
|---------|------|
| `zmk module add [<url>]` | モジュールを `west.yml` に追加 |
| `zmk module remove` | モジュール削除 |
| `zmk module list` | インストール済みモジュール一覧 |
| `zmk update` | ZMK 本体・全モジュールを更新 |

例: 新規モジュール追加:

```bash
zmk module add https://github.com/some-owner/zmk-cool-feature
```

CLI が `west.yml` を編集し、必要なら remote を追加する。

### GitHub 連携

| コマンド | 動作 |
|---------|------|
| `zmk cd` | リポジトリディレクトリへ移動（cd 表示） |
| `zmk download` (`zmk dl`) | GitHub Actions ページをブラウザで開く |

### 設定管理

```bash
zmk config                     # 全設定表示
zmk config <name> <value>      # 設定値を変更
zmk config --path              # 設定ファイル位置確認
```

主要設定項目:

| 設定 | 用途 |
|------|------|
| `user.home` | デフォルトリポジトリパス |
| `core.editor` | テキストエディタコマンド |
| `core.explorer` | ファイルエクスプローラーコマンド |

## ヘルプ

```bash
zmk --help              # 全般
zmk <command> --help    # 個別コマンド
```

## このスキルでの使い分け

| 操作 | ZMK CLI | 直接編集 |
|------|---------|---------|
| keymap のレイヤー追加・bindings 変更 | ❌ | ✅ Edit |
| combos / macros / behaviors の追加 | ❌ | ✅ Edit |
| `*.conf` 編集 | ❌ | ✅ Edit |
| `build.yaml` バリアント追加 | ✅ `zmk keyboard add` | ✅ Edit |
| モジュール追加・削除 | ✅ `zmk module add/remove` | ⚠️ 手動編集も可 |
| ZMK 本体・モジュール更新 | ✅ `zmk update` | ❌ |
| サポートキーボード一覧 | ✅ `zmk keyboard list` | ❌ |
| GitHub Actions 確認 | ✅ `zmk dl` | ❌ |

ZMK CLI を使うとマニフェストフォーマットの整合性を CLI が保証してくれるため、
**モジュール追加** や **新規ビルドターゲット追加** では CLI 優先がおすすめ。

## ZMK CLI 未インストール時の代替

ZMK CLI を入れたくない・入れられない場合の代替手順:

### モジュール追加の代替

`config/west.yml` を直接編集:

```yaml
remotes:
  # 既存 ...
  - name: new-owner
    url-base: https://github.com/new-owner

projects:
  # 既存 ...
  - name: zmk-some-module
    remote: new-owner
    revision: main
```

その後:

```bash
make setup-west
```

### 更新の代替

```bash
cd _west
west update
```

または `_west/` を削除して `make setup-west`。

### ビルドターゲット追加の代替

`build.yaml` を直接編集（[build-yaml.md](build-yaml.md) 参照）。

## トラブルシューティング

### `zmk` コマンドが見つからない

```bash
which zmk
# 出力なし → uv tool list で確認
uv tool list

# パス通ってない場合
uv tool update-shell
```

### `zmk update` が失敗する

```bash
# _west/ を再構築
rm -rf _west
make setup-west
```

### `zmk keyboard list` で目的のキーボードが出ない

ZMK 本体の keyboards/ + 各モジュールの keyboards/ から検出される。
モジュールが入っていない場合は `zmk module add` で追加する。
