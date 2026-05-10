# build.yaml スキーマ

GitHub Actions のビルドマトリクスとローカルビルド (`make`) の両方で使われる定義ファイル。

## 基本構造

```yaml
---
include:
  - board: <board_name>
    shield: <shield_name>
    artifact-name: <output_name>
    overlay-path: <path1>;<path2>
    snippet: <snippet_name>
    cmake-args: <flags>
```

`include` 配下に各ビルドターゲットを並べる。

## フィールド

### `board` (必須)

ターゲット MCU ボード。LisM では `xiao_ble` を使用。

主要ボード:

| board | 説明 |
|-------|------|
| `xiao_ble` | Seeed Studio XIAO nRF52840 BLE（LisM デフォルト） |
| `nice_nano_v2` | nice!nano v2 |
| `nrfmicro_13` | nRFMicro v1.3 |

### `shield` (必須または board のみ)

ボードに組み合わせるシールド。スペース区切りで複数指定可能:

```yaml
shield: lism_left rgbled_adapter
```

複数 shield を指定すると、各 shield の overlay/conf がマージされる。

### `artifact-name` (推奨)

GitHub Actions で生成される `.uf2` ファイル名のベース。
ローカルビルド (`firmware_builds/`) でもこの名前が使われる。

```yaml
artifact-name: lism_right_trackball_studio
```

### `overlay-path` (LisM 必須)

devicetree overlay ファイルのパス。`;` 区切りで複数:

```yaml
overlay-path: zmk-keyboards-LisM/boards/shields/lism/trackball_r.overlay;zmk-rgbled-widget/boards/shields/rgbled_adapter/boards/xiao_ble_zmk.overlay
```

LisM では以下のオーバーレイが用意されている:

| パス | 用途 |
|------|------|
| `zmk-keyboards-LisM/boards/shields/lism/trackball_l.overlay` | 左、トラックボール付き |
| `zmk-keyboards-LisM/boards/shields/lism/trackball_r.overlay` | 右、トラックボール付き |
| `zmk-keyboards-LisM/boards/shields/lism/non_trackball_l.overlay` | 左、トラックボール無し |
| `zmk-keyboards-LisM/boards/shields/lism/non_trackball_r.overlay` | 右、トラックボール無し |
| `zmk-rgbled-widget/boards/shields/rgbled_adapter/boards/xiao_ble_zmk.overlay` | RGB LED アダプター |

### `snippet`

Zephyr スニペット名。ZMK Studio に必須:

```yaml
snippet: studio-rpc-usb-uart
```

### `cmake-args`

CMake に渡すフラグ:

```yaml
cmake-args: -DCONFIG_ZMK_STUDIO=y
```

複数指定:

```yaml
cmake-args: -DCONFIG_ZMK_STUDIO=y -DEXTRA_CONF_FILE=studio.conf
```

## LisM の現行ビルドマトリクス

`build.yaml` で定義されている artifact:

| # | artifact-name | 種別 |
|---|--------------|------|
| 1 | `lism_left_non_trackball` | 左・トラックボール無し |
| 2 | `lism_left_trackball` | 左・トラックボール付き |
| 3 | `lism_right_non_trackball` | 右・トラックボール無し |
| 4 | `lism_right_trackball` | 右・トラックボール付き |
| 5 | `settings_reset-xiao_ble-zmk` | 設定リセット用 |
| 6 | `lism_right_non_trackball_studio` | 右・トラックボール無し・Studio |
| 7 | `lism_right_trackball_studio` | 右・トラックボール付き・Studio |

ZMK Studio 対応版は **右側のみ** 提供されている（中央側がホスト接続するため）。

## 新規 artifact 追加例

### 例 1: トラックボールあり左の Studio 版を追加

```yaml
- board: xiao_ble
  shield: lism_left rgbled_adapter
  artifact-name: lism_left_trackball_studio
  snippet: studio-rpc-usb-uart
  cmake-args: -DCONFIG_ZMK_STUDIO=y
  overlay-path: zmk-keyboards-LisM/boards/shields/lism/trackball_l.overlay;zmk-rgbled-widget/boards/shields/rgbled_adapter/boards/xiao_ble_zmk.overlay
```

> 注: 通常 ZMK Studio は中央側にしか必要ない。LisM では右側が中央なので、左側 Studio 版を作っても動作的には意味がない場合が多い。

### 例 2: デバッグ用設定の追加（追加 conf を読ませる）

`config/lism_right_debug.conf` を別途作って、それを反映:

```yaml
- board: xiao_ble
  shield: lism_right rgbled_adapter
  artifact-name: lism_right_trackball_debug
  cmake-args: -DEXTRA_CONF_FILE=lism_right_debug.conf
  overlay-path: zmk-keyboards-LisM/boards/shields/lism/trackball_r.overlay;zmk-rgbled-widget/boards/shields/rgbled_adapter/boards/xiao_ble_zmk.overlay
```

## 編集時の注意

### overlay-path のスペル

`_west/` 配下の相対パス。スペル誤りに敏感。LisM の典型ミス:

- ❌ `non-trackball` (ハイフン)
- ✅ `non_trackball` (アンダースコア)

### shield の順序

LisM では `shield: lism_left rgbled_adapter` のように:
1. キーボードシールド
2. アクセサリーシールド

の順で書く慣習がある。

### YAML のクォート不要

`board` や `shield` の値はクォート不要。`artifact-name` も同様。

### 順序保持

`include` 配列の順番は GitHub Actions のジョブ順 = ローカル `make` の順序と一致する。

## ビルドスクリプトの動作

LisM のビルドスクリプト:

- `scripts/build-matrix.sh` — 全エントリを順次ビルド（FILTER_MODE 対応）
- `scripts/build-single.sh` — `build.yaml` から対話選択してビルド
- `scripts/lib/build-helpers.sh` — overlay 解決、複数 shield 対応

`build.yaml` のフォーマットを変更する場合、これらスクリプトも参照する。

## 公式ドキュメント

- ZMK build options: <https://zmk.dev/docs/development/build-flash>
- GitHub Actions Build: <https://github.com/zmkfirmware/zmk/blob/main/.github/workflows/build-user-config.yml>
- Studio: <https://zmk.dev/docs/features/studio>
