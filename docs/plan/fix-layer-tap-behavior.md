# Layer-Tap (`&lt`) 動作不良 修正計画

## 概要

`&lt` (Layer-Tap) を使用したすべてのキーで、長押ししてもレイヤーが有効にならず、タップ時のキーが連続入力される問題を修正する。

## 問題の要約

| 項目 | 状態 |
|------|------|
| `&lt` (Layer-Tap) | 動作しない |
| `&mt` (Mod-Tap) | 正常動作 |
| 症状 | 長押しでレイヤー有効化されず、タップキーが連続入力 |

## 原因分析

### `&mt` と `&lt` の違い

| 項目 | `&mt` | `&lt` |
|------|-------|-------|
| bindings | `<&kp>, <&kp>` | `<&mo>, <&kp>` |
| デフォルト flavor | `hold-preferred` | `tap-preferred` |
| ホールド動作 | 修飾キー送信 | レイヤー切り替え |

### 現在の設定（`config/lism.keymap` 13-17行目）

```dts
&lt {
    quick-tap-ms = <300>;
    tapping-term-ms = <300>;
    flavor = "balanced";
};
```

### 推定原因

1. **`quick-tap-ms = <300>` の影響**
   - 連続入力が発生していることから、この設定が意図しない動作を引き起こしている可能性
   - 「前回のタップから300ms以内に再度押すと即座にタップとして扱う」設定

2. **`&mo` (momentary layer) との組み合わせの問題**
   - 分割キーボードでレイヤー状態はセントラル（右側）で管理される
   - 通信遅延や状態同期の問題の可能性

---

## 修正計画

### Phase 1: `quick-tap-ms` 無効化テスト

`quick-tap-ms` が問題を引き起こしている可能性があるため、まず無効化する。

#### タスク

- [x] `config/lism.keymap` の `&lt` 設定から `quick-tap-ms` を削除
  ```dts
  &lt {
      tapping-term-ms = <300>;
      flavor = "balanced";
  };
  ```
- [x] ファームウェアをビルド (`make clean && make`)
- [ ] 左右両方に `.uf2` を書き込み
- [ ] `&lt` キーの動作確認
  - [ ] `&lt 1 LANG2` の長押しでレイヤー1が有効になるか
  - [ ] タップで `LANG2` が送信されるか

#### 判定

- 改善した場合 → Phase 1 で完了
- 改善しない場合 → Phase 2 へ

---

### Phase 2: flavor を `hold-preferred` に変更

ホールドを優先する設定に変更する。

#### タスク

- [ ] `config/lism.keymap` の `&lt` 設定を変更
  ```dts
  &lt {
      tapping-term-ms = <300>;
      flavor = "hold-preferred";
  };
  ```
- [ ] ファームウェアをビルド (`make clean && make`)
- [ ] 左右両方に `.uf2` を書き込み
- [ ] `&lt` キーの動作確認
  - [ ] `&lt 1 LANG2` の長押しでレイヤー1が有効になるか
  - [ ] タップで `LANG2` が送信されるか

#### 判定

- 改善した場合 → Phase 2 で完了
- 改善しない場合 → Phase 3 へ

---

### Phase 3: カスタム behavior を定義

ZMKの標準 `&lt` をオーバーライドせず、新しい behavior を定義する。

#### タスク

- [ ] `config/lism.keymap` に新しい behavior を追加
  ```dts
  / {
      behaviors {
          ltp: layer_tap_custom {
              compatible = "zmk,behavior-hold-tap";
              #binding-cells = <2>;
              tapping-term-ms = <300>;
              flavor = "hold-preferred";
              bindings = <&mo>, <&kp>;
          };
      };
  };
  ```
- [ ] キーマップで `&lt` を `&ltp` に置き換え
  - [ ] `&lt 1 LANG2` → `&ltp 1 LANG2`
  - [ ] `&lt 2 SPACE` → `&ltp 2 SPACE`
  - [ ] `&lt 2 L` → `&ltp 2 L`
  - [ ] `&lt 3 DELETE` → `&ltp 3 DELETE`
  - [ ] `&lt 5 DELETE` → `&ltp 5 DELETE`
  - [ ] `&lt 6 LANG1` → `&ltp 6 LANG1`
- [ ] ファームウェアをビルド (`make clean && make`)
- [ ] 左右両方に `.uf2` を書き込み
- [ ] `&ltp` キーの動作確認

#### 判定

- 改善した場合 → Phase 3 で完了
- 改善しない場合 → 追加調査が必要

---

### Phase 4: 追加調査（Phase 3 でも改善しない場合）

#### タスク

- [ ] ZMK の issue/discussion を検索し、類似の問題がないか確認
- [ ] 分割キーボード固有の問題かどうかを切り分け
  - [ ] 右側（セントラル）のみで `&lt` をテスト
- [ ] ZMK のバージョンを確認し、既知のバグがないか確認
- [ ] 必要に応じて ZMK コミュニティに質問

---

## ビルド手順

```bash
# クリーンビルド
make clean
make

# 成果物の確認
ls -la firmware_builds/
```

## 書き込み手順

1. キーボードをブートローダーモードにする（リセットボタンをダブルクリック）
2. マウントされたドライブに `.uf2` ファイルをコピー
3. **左右両方**に書き込むこと

## 関連ファイル

- `config/lism.keymap` - キーマップ設定
- `docs/report.md` - 調査レポート

## 参考リンク

- [ZMK Hold-Tap Behavior](https://zmk.dev/docs/keymaps/behaviors/hold-tap)
- [ZMK Split Keyboards](https://zmk.dev/docs/features/split-keyboards)
