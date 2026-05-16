# Kconfig (`*.conf`) 設定

公式: <https://zmk.dev/docs/config>

`config/lism_left.conf` と `config/lism_right.conf` で ZMK のコンパイル時設定を行う。

## ファイル形式

`KEY=VALUE` の単純な形式。`#` でコメント:

```
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000
# CONFIG_ZMK_LOG_LEVEL_DBG=y
```

## 左右同期ルール

LisM では **`lism_left.conf` と `lism_right.conf` は同一内容** で運用している。
変更時は両方を同じに更新する。
左右で意図的に異なる値が必要な場合のみ個別に編集する。

## 現在の LisM 設定（実体）

```
# Sleep
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_TIMEOUT=300000
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000

# encoder
CONFIG_EC11=y
CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y

# Battery
CONFIG_BT_BAS=y
CONFIG_ZMK_BATTERY_REPORTING=y
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y

# RGB LED widget
CONFIG_RGBLED_WIDGET=y
CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_HIGH=30
CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_CRITICAL=10

# debuging
#CONFIG_ZMK_LOG_LEVEL_DBG=y

# Charge Indicator feature
CONFIG_CHARGE_INDICATOR=y
```

## 主要オプション

### スリープ・電源管理

| オプション | デフォルト | 説明 |
|-----------|----------|------|
| `CONFIG_ZMK_SLEEP=y` | n | 非アクティブ時のスリープ機能を有効化 |
| `CONFIG_ZMK_IDLE_TIMEOUT` | 30000 | アイドル状態への移行時間（ms） |
| `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` | 900000 | スリープへの移行時間（ms） |

例: スリープを 1 時間に変更:

```
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=3600000
```

### Bluetooth

| オプション | 説明 |
|-----------|------|
| `CONFIG_BT_BAS=y` | バッテリーサービスを有効化 |
| `CONFIG_ZMK_BATTERY_REPORTING=y` | バッテリーレベル報告 |
| `CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y` | 実験的接続機能（不安定） |
| `CONFIG_ZMK_BLE_PASSKEY_ENTRY=y` | ペアリング時のパスキー入力 |

### 分割キーボード（Split）

| オプション | 説明 |
|-----------|------|
| `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y` | 中央側がペリフェラルのバッテリーをプロキシ |
| `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y` | ペリフェラルのバッテリーを取得 |

### エンコーダー

| オプション | 説明 |
|-----------|------|
| `CONFIG_EC11=y` | EC11 ロータリーエンコーダーサポート |
| `CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y` | グローバルスレッドトリガー（推奨） |

### マクロタイミング

| オプション | デフォルト | 説明 |
|-----------|----------|------|
| `CONFIG_ZMK_MACRO_DEFAULT_WAIT_MS` | 15 | マクロ実行間の待機時間 |
| `CONFIG_ZMK_MACRO_DEFAULT_TAP_MS` | 5 | マクロ tap の press-release 間 |

文字抜けがある場合は 25-50 程度に増やす:

```
CONFIG_ZMK_MACRO_DEFAULT_WAIT_MS=30
CONFIG_ZMK_MACRO_DEFAULT_TAP_MS=10
```

### コンボ・ホールドタップ関連

| オプション | デフォルト | 説明 |
|-----------|----------|------|
| `CONFIG_ZMK_COMBO_MAX_COMBOS_PER_KEY` | 5 | 1 キーが参加できるコンボ数 |
| `CONFIG_ZMK_COMBO_MAX_KEYS_PER_COMBO` | 4 | 1 コンボの最大キー数 |
| `CONFIG_ZMK_COMBO_MAX_PRESSED_COMBOS` | 4 | 同時にアクティブな最大コンボ数 |

コンボが多くて警告が出る場合は増やす:

```
CONFIG_ZMK_COMBO_MAX_COMBOS_PER_KEY=10
```

### USB / HID

| オプション | 説明 |
|-----------|------|
| `CONFIG_ZMK_USB=y` | USB HID を有効化（標準で y） |
| `CONFIG_ZMK_HID_REPORT_TYPE_NKRO=y` | NKRO（全キー同時押し） |
| `CONFIG_ZMK_HID_REPORT_TYPE_HKRO=y` | HKRO（6 キー同時押し、デフォルト） |

NKRO に切替:

```
CONFIG_ZMK_HID_REPORT_TYPE_NKRO=y
```

### ZMK Studio

| オプション | 説明 |
|-----------|------|
| `CONFIG_ZMK_STUDIO=y` | Studio サポート（通常は build.yaml の cmake-args で設定） |
| `CONFIG_ZMK_STUDIO_LOCK_BASE=y` | デフォルトで Studio をロック |

通常 build.yaml で:

```yaml
cmake-args: -DCONFIG_ZMK_STUDIO=y
```

を使う。

### マウス・ポインティング

| オプション | 説明 |
|-----------|------|
| `CONFIG_ZMK_POINTING=y` | ポインティング機能 |
| `CONFIG_ZMK_POINTING_SMOOTH_SCROLLING=y` | スムーススクロール |

### モジュール固有設定

| オプション | 説明 |
|-----------|------|
| `CONFIG_RGBLED_WIDGET=y` | RGB LED ステータスインジケーター |
| `CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_HIGH=30` | 高バッテリーしきい値 (%) |
| `CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_CRITICAL=10` | 危険バッテリーしきい値 (%) |
| `CONFIG_CHARGE_INDICATOR=y` | 充電インジケーター（4mplelab モジュール） |
| `CONFIG_PAW3222=y` | PAW3222 トラックボールドライバ（必要に応じて） |

### デバッグ

| オプション | 説明 |
|-----------|------|
| `CONFIG_ZMK_LOG_LEVEL_DBG=y` | デバッグログ出力（USB シリアル経由） |
| `CONFIG_ZMK_USB_LOGGING=y` | USB CDC でログ出力 |

問題が発生した場合のみコメントアウトを外す。

## 編集後の確認

1. `git diff config/lism_*.conf` で変更を確認
2. 左右同期されていること
3. ビルド検証（必要に応じて）

## 参照

- ZMK 設定全リスト: <https://zmk.dev/docs/config>
- システム設定: <https://zmk.dev/docs/config/system>
- BLE 設定: <https://zmk.dev/docs/config/bluetooth>
- パワーマネジメント: <https://zmk.dev/docs/config/power>
