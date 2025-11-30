# Layer-Tap (`&lt`) 動作不良 調査レポート

## 問題の概要

左側キーボードの `&lt 1 LANG2` キーが動作しない。Layer-Tap を指定しているキーのみが送信されない状況。

## 現在のキーマップ設定

### `&lt` の設定（13行目）

```dts
&lt { quick-tap-ms = <300>; };
```

### `&mt` の設定（8-11行目）

```dts
&mt {
    tapping-term-ms = <300>;
    flavor = "balanced";
};
```

### 問題のキーバインディング（159行目）

```dts
&lt 5 DELETE      &kp LEFT_ALT  &mt LEFT_WIN ESC  &lt 1 LANG2  &lt 2 SPACE  &lt 3 DELETE    &kp BACKSPACE  &lt 6 LANG1  &trans  &trans     &trans     &kp TAB
```

## ZMK公式ドキュメントからの情報

### Hold-Tap Behavior (`&lt`)

- **タップ時**: 指定したキーコード（例: `LANG2`）を送信
- **ホールド時**: 指定したレイヤー（例: レイヤー1）を有効化

### デフォルト設定値

| パラメータ | デフォルト値 | 説明 |
|-----------|-------------|------|
| `tapping-term-ms` | 200ms | タップとホールドを区別する時間 |
| `quick-tap-ms` | -1（無効） | 連続タップ時に即座にタップとして扱う時間 |
| `flavor` | `"tap-preferred"` | タップ/ホールドの判定方法 |
| `retro-tap` | false | ホールド後にタップを送信するか |

### `flavor` の種類

| Flavor | 説明 |
|--------|------|
| `tap-preferred` | 他のキーを押さない限りタップとして扱う |
| `balanced` | タップ/ホールドをバランスよく判定 |
| `hold-preferred` | ホールドを優先 |
| `tap-unless-interrupted` | 割り込みがない限りタップ |

### `LANG2` キーコードの互換性

| OS | 対応状況 |
|----|---------|
| Windows | ⭐ 対応 |
| Linux | ⭐ 対応 |
| Android | ❌ 非対応 |
| macOS | ❔ 未確認 |
| iOS | ❔ 未確認 |

## 考えられる原因

### 1. `&lt` の設定不足

現在の `&lt` 設定には `quick-tap-ms` のみが設定されており、以下が未設定：

- `tapping-term-ms`: デフォルト200msが適用されている
- `flavor`: デフォルト `"tap-preferred"` が適用されている

`&mt` では `tapping-term-ms = <300>` と `flavor = "balanced"` が設定されているため、`&lt` と `&mt` で動作に差異が生じている可能性がある。

### 2. タップ判定のタイミング問題

- デフォルトの `tapping-term-ms = 200ms` では、キーを押してから離すまでの時間が200msを超えるとホールドとして扱われる
- ユーザーのタイピング速度によっては、意図せずホールドと判定されている可能性

### 3. `flavor` の影響

- `"tap-preferred"` では、他のキーを同時に押すとホールドとして扱われることがある
- `"balanced"` に変更することで改善する可能性

### 4. 分割キーボード固有の問題

- 左右間の通信遅延により、タップ判定に影響が出ている可能性
- 左側のキーが右側（セントラル）に正しく伝達されていない可能性

## 確認事項と検証結果

### 質問1: `&lt 1 LANG2` をタップした時、レイヤー1が一瞬有効になっているか？

**回答**: `&lt` を使用しているすべてのキーにおいて、**長押ししてもレイヤーが有効にならず、タップ時のキーが連続で入力される**

### 質問2: 他の `&lt` キーの動作確認

**回答**: 上記の通り、すべての `&lt` キーで同様の問題が発生

### 質問3: コンボでの `LANG2` 送信確認

**回答**: 未確認

---

## 問題の分析

### 現在の `&lt` 設定（適用済み）

```dts
&lt {
    quick-tap-ms = <300>;
    tapping-term-ms = <300>;
    flavor = "balanced";
};
```

### 症状の特徴

| 操作 | 期待される動作 | 実際の動作 |
|------|---------------|-----------|
| タップ | キーコード送信（例: LANG2） | キーコード送信 ✅ |
| 長押し | レイヤー有効化 | **キーコードが連続入力される** ❌ |

### 重要な発見

**ホールドが全く機能していない**ことが確認されました。これは単純な設定の問題ではなく、以下のいずれかの可能性があります：

1. **`quick-tap-ms` による意図しない動作**
   - `quick-tap-ms = <300>` は「前回のタップから300ms以内に再度押すと即座にタップとして扱う」設定
   - しかし、**最初の長押し**でもホールドが効かないなら、この設定だけでは説明できない

2. **キーリピート（OS側の設定）との混同**
   - OSのキーリピート機能により、キーを押し続けると連続入力される
   - ZMKのホールドが正しく機能していれば、レイヤー切り替え中はキーコードは送信されないはず

3. **ファームウェアのビルド/書き込みの問題**
   - 設定変更が正しくビルドされていない
   - 古いファームウェアが残っている

4. **分割キーボード固有の問題**
   - 左右間の通信で `&lt` の状態が正しく同期されていない

## 追加の確認事項

### 確認1: 長押しの時間を確認

**質問**: キーを押し続ける時間はどのくらいですか？

- `tapping-term-ms = <300>` なので、**300ms以上**押し続ける必要があります
- 300ms = 0.3秒 は意外と長いです。意識的に長く押してみてください

### 確認2: `&mt` の動作確認

**質問**: `&mt LEFT_SHIFT Z` や `&mt LCTRL A` は正常に動作しますか？
  - `&mt` は問題なく動作する
  - `&lt` だけが動作しない

- 長押しで Shift/Ctrl が有効になるか？
- これが動作するなら、`&lt` 固有の問題
- これも動作しないなら、hold-tap 全般の問題

### 確認3: ファームウェアの再ビルド確認

**質問**: 設定変更後、以下を実行しましたか？

```bash
make clean
make
```

そして、生成された `.uf2` ファイルを**左右両方**に書き込みましたか？

### 確認4: コンボでの検証

**質問**: キーポジション 21-22 同時押しで `LANG2` が送信されますか？

```dts
en {
    bindings = <&kp LANG2>;
    key-positions = <21 22>;
};
```

- 送信される場合: `LANG2` キーコード自体は問題なし、`&lt` の問題
- 送信されない場合: `LANG2` キーコードまたはOS側の問題

### 確認5: ZMK Studio での確認（該当する場合）

ZMK Studio対応ファームウェアを使用している場合、Studio上でキーマップが正しく反映されているか確認してください。

## 推奨される修正案

### 案1: `quick-tap-ms` を無効化してテスト

`quick-tap-ms` が問題を引き起こしている可能性があるため、一時的に無効化：

```dts
&lt {
    tapping-term-ms = <300>;
    flavor = "balanced";
    // quick-tap-ms を削除（デフォルト -1 = 無効）
};
```

### 案2: `tapping-term-ms` を短くしてテスト

300ms が長すぎる可能性があるため、200ms に変更：

```dts
&lt {
    tapping-term-ms = <200>;
    quick-tap-ms = <200>;
    flavor = "balanced";
};
```

### 案3: `retro-tap` を有効化

```dts
&lt {
    tapping-term-ms = <300>;
    quick-tap-ms = <300>;
    flavor = "balanced";
    retro-tap;
};
```

`retro-tap` を有効にすると、ホールド後にキーを離した時にタップアクションが送信される。

### 案4: カスタム hold-tap behavior を定義

```dts
/ {
    behaviors {
        lt_custom: layer_tap_custom {
            compatible = "zmk,behavior-hold-tap";
            #binding-cells = <2>;
            tapping-term-ms = <300>;
            quick-tap-ms = <300>;
            flavor = "balanced";
            bindings = <&mo>, <&kp>;
        };
    };
};
```

使用例: `&lt_custom 1 LANG2`

## 参考リンク

- [ZMK Hold-Tap Behavior](https://zmk.dev/docs/keymaps/behaviors/hold-tap)
- [ZMK List of Keycodes - Language](https://zmk.dev/docs/keymaps/list-of-keycodes#language)
- [ZMK Troubleshooting](https://zmk.dev/docs/troubleshooting)

## 次のステップ

1. 上記の確認事項を検証
2. 修正案1を適用してビルド・テスト
3. 改善しない場合は修正案2または3を検討
