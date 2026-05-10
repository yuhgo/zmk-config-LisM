---
name: build-firmware
description: キーマップ修正後にDev Containerでファームウェアをビルドします。lism.keymapの変更後にファームウェア生成が必要な場合に使用。
---

> **役割分担**: keymap や *.conf, build.yaml の **編集** は `zmk-keymap-editor` skill が担当する。
> このスキルは編集後の **ビルド** 専用。「ビルドして」「ファームウェア作って」と依頼があった時のみ使用する。

## ファームウェアビルド

キーマップ（`config/lism.keymap`）を修正した後、Dev Containerを使ってファームウェアをビルドします。

## 実行手順

### 1. Dev Containerを起動

```bash
mise run dc-up
```

### 2. ビルドを実行

対話的に選択する場合:
```bash
mise run dc-exec make single
```

よく使うビルドターゲット:
- `4` = lism_right_trackball（トラックボール付き右手）
- `7` = lism_right_trackball_studio（ZMK Studio対応版）

直接ビルドする場合（右手トラックボール版）:
```bash
echo "4" | mise run dc-exec make single
```

全ビルド:
```bash
mise run dc-exec make all
```

### 3. 生成されるファイル

ビルド成果物は `firmware_builds/` に出力されます:
- `lism_right_trackball.uf2` - 通常版
- `lism_right_trackball_studio.uf2` - ZMK Studio対応版

### 4. キーボードへの書き込み

1. キーボードをブートローダーモードにする（リセットボタン2回押し）
2. マウントされたドライブに `.uf2` ファイルをコピー
3. 自動的に書き込まれて再起動

## 注意事項

- Dockerが起動している必要があります
- 初回は `mise trust` と `mise install` が必要です
