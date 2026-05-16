# west.yml モジュール管理

`config/west.yml` は West（Zephyr のビルドツール）のマニフェストファイル。
ZMK 本体と外部モジュールの依存関係を定義する。

## ファイルの場所

```
config/west.yml         (実体)
_west/config/west.yml   (シンボリックリンク、Westワークスペース)
```

## 基本構造

```yaml
manifest:
  remotes:
    - name: <alias>
      url-base: https://github.com/<owner>
  projects:
    - name: <repo>
      remote: <alias>
      revision: <branch_or_tag>
  self:
    path: config
```

## 現在の LisM 設定（実体）

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: sekigon-gonnoc
      url-base: https://github.com/sekigon-gonnoc
    - name: caksoylar
      url-base: https://github.com/caksoylar
    - name: 4mplelab
      url-base: https://github.com/4mplelab
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: zmk-driver-paw3222
      remote: sekigon-gonnoc
      revision: main
    - name: zmk-rgbled-widget
      remote: caksoylar
      revision: main
    - name: zmk-feature-charge-indicator
      remote: 4mplelab
      revision: main
    - name: zmk-keyboards-LisM
      remote: 4mplelab
      revision: main
      path: zmk-keyboards-LisM
  self:
    path: config
```

## 各モジュールの役割

| プロジェクト | 役割 |
|------------|------|
| `zmk` | ZMK ファームウェア本体 |
| `zmk-driver-paw3222` | PAW3222 トラックボールドライバ |
| `zmk-rgbled-widget` | RGB LED ステータスインジケーター |
| `zmk-feature-charge-indicator` | 充電インジケーター |
| `zmk-keyboards-LisM` | LisM ハードウェア定義（dtsi, overlay 等） |

## remotes プロパティ

GitHub の owner（または GitLab 等）を別名でまとめる:

```yaml
remotes:
  - name: zmkfirmware
    url-base: https://github.com/zmkfirmware
```

新規モジュール追加時、別 owner なら remote を追加する。

## projects プロパティ

各依存モジュールを定義:

| フィールド | 説明 |
|-----------|------|
| `name` | リポジトリ名 |
| `remote` | remotes で定義した別名 |
| `revision` | ブランチ名・タグ・コミット SHA |
| `import` | 別の west.yml を取り込む（ZMK 本体で必要） |
| `path` | クローン先のパス（省略可） |

### revision のオプション

| 値 | 例 |
|----|---|
| ブランチ名 | `main`, `master`, `dev` |
| タグ | `v0.1.0` |
| コミット SHA | `abc123...` (40 文字) |

**安定性が必要な場合はタグまたは SHA で固定** する。LisM は現在すべて `main` のため、上流変更で破壊される可能性あり。

## 新規モジュール追加

### 方法 A: ZMK CLI を使う（推奨）

```bash
zmk module add https://github.com/owner/repo
```

CLI が自動で west.yml を更新し、必要なら remote も追加する。

### 方法 B: 手動編集

1. `config/west.yml` の `projects:` 配下に追加:

```yaml
- name: zmk-some-module
  remote: owner-alias  # remote 未定義なら remotes に追加
  revision: main
```

2. Westワークスペースを更新:

```bash
make setup-west
```

または

```bash
mise run dc-exec make setup-west
```

3. `config/lism_*.conf` で必要な `CONFIG_*` を有効化

4. `build.yaml` の `overlay-path` で必要なら追加

## モジュール削除

```bash
zmk module remove
```

または手動で `west.yml` から該当エントリを削除し、`make setup-west` で再構築。

## モジュール更新

すべてのモジュールを最新化:

```bash
zmk update
```

または:

```bash
cd _west
west update
```

特定の revision を変更したい場合は west.yml の `revision:` を編集してから `west update`。

## トラブルシューティング

### 依存解決失敗

```bash
rm -rf _west
make setup-west
```

`_west/` を削除して再構築。

### キャッシュ問題

```bash
cd _west && west forall -c "git fetch --all"
west update --rebase
```

### overlay が見つからない

`overlay-path` の prefix は **west.yml の各 project name** と一致する必要がある:

```yaml
projects:
  - name: zmk-keyboards-LisM
    path: zmk-keyboards-LisM   # ← ここのパス
```

```yaml
overlay-path: zmk-keyboards-LisM/boards/shields/...
            # ↑ project の path と先頭一致
```

## 公式ドキュメント

- West manifests: <https://docs.zephyrproject.org/latest/develop/west/manifest.html>
- ZMK modules: <https://zmk.dev/docs/development/new-shield#building-from-zmk-config-folder>
- ZMK CLI: <https://zmk.dev/docs/zmk-cli>
