# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

LisMキーボードのZMKファームウェア設定リポジトリ。Dev Container環境でローカルビルドを実行し、トラックボール有無・左右・ZMK Studio対応など複数バリエーションのファームウェアを生成します。

## リポジトリ管理

このリポジトリは4mplelabのリポジトリをフォークして管理しています。

### リモート設定

- **`origin`**: https://github.com/yuhgo/zmk-config-LisM.git（自分のリポジトリ）
- **`upstream`**: https://github.com/4mplelab/zmk-config-LisM.git（元のリポジトリ）

### 通常の作業フロー

```bash
# 自分のリポジトリにプッシュ
git push origin main

# 元のリポジトリから最新の変更を取得
git fetch upstream
git merge upstream/main  # または git rebase upstream/main
```

### 元のリポジトリの更新を追跡する場合

```bash
# upstreamの変更を確認
git fetch upstream

# upstreamの変更をマージ
git merge upstream/main
```

## ビルドコマンド

### 基本ビルド
```bash
# 全ファームウェアビルド（ZMK Studioなし）
make

# 全ファームウェアビルド（ZMK Studioあり）
make all

# 対話的に選択してビルド
make single

# ビルド成果物削除
make clean
```

### 開発環境セットアップ
```bash
# Westワークスペース初期化（通常は自動実行）
make setup-west

# 依存関係更新
git pull
make setup-west
```

## アーキテクチャ

### ビルドシステムの構造

**`build.yaml`** がビルド定義の中心
- 各エントリは `board`, `shield`, `overlay-path`, `artifact-name` 等を指定
- ZMK Studio対応版は `snippet: studio-rpc-usb-uart` と `cmake-args: -DCONFIG_ZMK_STUDIO=y` が追加される
- トラックボール/非トラックボールは `overlay-path` で切り替え
  - `zmk-keyboards-LisM/boards/shields/lism/trackball_*.overlay`
  - `zmk-keyboards-LisM/boards/shields/lism/non_trackball_*.overlay`

**Westワークスペース (`_west/`)** は Git管理外
- `config/west.yml` がマニフェスト（シンボリックリンク: `_west/config/west.yml`）
- 依存プロジェクト:
  - `zmk`: ZMKファームウェア本体（zmkfirmware/zmk）
  - `zmk-driver-paw3222`: トラックボールドライバ（sekigon-gonnoc）
  - `zmk-rgbled-widget`: RGBステータスウィジェット（caksoylar）
  - `zmk-feature-charge-indicator`: 充電インジケータ（4mplelab）
  - `zmk-keyboards-LisM`: LisMハードウェア定義（4mplelab）

**ビルドスクリプトの役割**
- `scripts/build-matrix.sh`: `build.yaml` の全エントリを順次ビルド、`FILTER_MODE` でStudio版フィルタリング
- `scripts/build-single.sh`: `build.yaml` エントリを対話選択してビルド
- `scripts/lib/build-helpers.sh`: overlay解決、複数shield対応、成果物コピーの共通ロジック
- `scripts/west-common.sh`: パス設定とツールチェック

**成果物の配置**
- `firmware_builds/` に `.uf2` ファイルが生成される
- `artifact-name` に応じたファイル名（例: `lism_right_trackball_studio.uf2`）

### キーマップとハードウェア設定

**キーマップファイル**: `config/lism.keymap`
- ZMK標準のdevicetreeフォーマット
- レイヤー定義、キーバインディング、ビヘイビア設定を含む
- **重要**: キーマップの編集は主にこのファイルで行う

**物理レイアウト定義**: `config/lism.json`
- ZMK Studioが使用するメタデータ
- 物理的なキー配置: 4行×13列（左右分割型で中央列は空き）
  - 左側: col 0-5、右側: col 7-12（col 6は未使用）
  - 合計42キー（各半分21キー）
- エンコーダー（ロータリーエンコーダー）定義:
  - identifier: "encoder"
  - compatible: "alps,ec11"

**左右別設定ファイル**: `config/lism_left.conf` と `config/lism_right.conf`
現在は両方とも同じ内容で、以下の機能を設定:

- **スリープ管理**:
  - アイドルタイムアウト: 5分（300000ms）
  - スリープ移行: 30分（1800000ms）

- **エンコーダー**:
  - EC11ロータリーエンコーダーサポート有効
  - グローバルスレッドトリガー使用

- **バッテリー関連**:
  - Bluetoothバッテリーサービス有効
  - 分割キーボードの中央（右側）がペリフェラル（左側）のバッテリーレベルを取得・プロキシ

- **RGB LEDステータス表示** (zmk-rgbled-widget):
  - バッテリーレベル表示: 高30%、危険10%

- **充電インジケーター** (zmk-feature-charge-indicator):
  - 充電状態の視覚的フィードバック

### Dev Container環境

**ベースイメージ**: `zmkfirmware/zmk-build-arm:stable`
- 必要ツール: `west`, `yq`（YAMLパース）
- `postCreateCommand` で自動セットアップ (`make setup-west`)

**VS Code拡張機能**:
- `ms-vscode.makefile-tools`: Makefile サポート
- `redhat.vscode-yaml`: YAML編集
- `zephyr-tools.zephyr-ide`: Zephyr IDE
- `spadin.zmk-tools`: ZMKツール（`.keymap` のシンタックスハイライト等）

### Dev Container CLI（VS Code以外での使用）

VS Code以外の環境でDev Containerを使用する場合は、Dev Container CLIを使用します。
このプロジェクトではmiseを使ってNode.jsと`@devcontainers/cli`を管理しています。

**前提条件**:
- Docker がインストールされていること
- mise がインストールされていること

**セットアップ手順**:
```bash
# 1. miseでこのディレクトリの設定を信頼
mise trust

# 2. Node.jsをインストール
mise install

# 3. @devcontainers/cli をインストール
mise run devcontainer-install
```

**コンテナの操作**:
```bash
# コンテナを起動
mise run dc-up

# コンテナ内でシェルを開く
mise run dc-shell

# コンテナ内でコマンドを実行（例：make）
mise run dc-exec make

# 全ファームウェアビルド
mise run dc-exec make all

# 単体ビルド（対話選択）
mise run dc-exec make single
```

**直接devcontainerコマンドを使用する場合**:
```bash
# コンテナを起動
devcontainer up --workspace-folder .

# コンテナ内でコマンドを実行
devcontainer exec --workspace-folder . make

# コンテナ内でシェルを開く
devcontainer exec --workspace-folder . /bin/bash
```

**コンテナの停止・削除**:
```bash
# 実行中のコンテナを確認
docker ps

# コンテナを停止
docker stop <コンテナ名またはID>

# コンテナを削除
docker rm <コンテナ名またはID>
```

## キーマップ編集ワークフロー

1. **キーマップ変更**: `config/lism.keymap` を編集
   - devicetree形式でレイヤー、キーバインディング、ビヘイビアを定義
   - 左右42キー + エンコーダーの定義

2. **設定変更**: 必要に応じて `config/lism_{left,right}.conf` を編集
   - スリープ、バッテリー、RGB LED等の動作設定

3. **ビルド**: `make` または `make single` で選択ビルド
   - 通常はキーマップ変更のみなら右側（`lism_right_*`）のみビルドでOK

4. **書き込み**: `firmware_builds/*.uf2` をキーボードに転送

## 重要な注意点

- **キーマップファイルの場所**: `config/lism.keymap` がこのリポジトリのメインキーマップ。外部モジュール `zmk-keyboards-LisM` にはハードウェア定義（.dtsi, overlay等）が含まれる
- **ワークスペース管理**: `_west/` は `.gitignore` で管理外。削除する場合は `rm -rf _west && make setup-west` で再構築
- **overlay-path の解決**: `_west/` 配下の相対パスで指定。スペル誤りに注意
- **ZMK Studio対応**: 右側（right）のファームウェアのみStudio対応版を提供。`config/lism.json` がStudioで使用される
- **ファイル形式**: `.uf2` が優先生成されるが、ターゲットによっては `.bin` の場合もある
- **左右設定の同期**: 現在 `lism_left.conf` と `lism_right.conf` は同一内容。左右で異なる設定が必要な場合のみ個別編集
