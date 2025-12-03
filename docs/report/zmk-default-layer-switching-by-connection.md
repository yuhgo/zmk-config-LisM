# ZMKキーボードで接続先に応じてデフォルトレイヤーを切り替える方法

## 調査日

2024年12月3日

## 出典

[ZMKキーボードで接続先に応じてデフォルトレイヤーを切り替える方法 - Zenn](https://zenn.dev/shakupan/articles/261ce435251607)

著者: shakupan ([@shakupan_](https://x.com/shakupan_))

---

## エグゼクティブサマリー

ZMKファームウェアを使用した自作キーボードにおいて、**Bluetooth接続先（PC、タブレット、スマホなど）に応じてデフォルトレイヤー（キーマップ）を自動的に切り替える方法**を解説した記事。

---

## 背景と課題

### ZMKの利点

- 無線運用を前提としたファームウェア
- 複数端末をペアリング可能
- キー1つで接続先を切り替えられる

### 課題

- QMK/VIAのようにキーマップを端末によって切り替える機能がない
- Windows/MacやPC/iPadを使い分けるシチュエーションで使い勝手が悪い
- 例: Macでは`Cmd+C`、Windowsでは`Ctrl+C`など、修飾キーの違いに対応できない

---

## 解決アプローチ

### コンセプト

**「接続先切り替えキーにトグルレイヤーをくっつけてマクロ化する」**

### 仕組み

1. 接続先を認識する複雑な実装ではなく、シンプルな方法で実現
2. PCからiPadへ接続を切り替える際、必ずiPadへの接続切り替えキーを押す
3. そこにトグルレイヤーを紐づけて任意のデフォルトレイヤーで固定
4. キーボードが接続先を認識する必要がない

---

## 実装手順

### 1. トグルレイヤーをON/OFFで明示的に指定できるように変更

通常のトグルレイヤーはON/OFF切り替え動作のため、明示的にON/OFFを指定できるよう`.keymap`ファイルのbehavior部分を拡張する。

```dts
/ {
    behaviors {
        toggle_on: toggle_layer_on {
            compatible = "zmk,behavior-toggle-layer";
            #binding-cells = <1>;
            display-name = "Toggle Layer On";
        };
        toggle_off: toggle_layer_off {
            compatible = "zmk,behavior-toggle-layer";
            #binding-cells = <1>;
            display-name = "Toggle Layer Off";
            toggle-mode = "off";
        };
    };
};
```

### 2. 機種毎のデフォルトレイヤーを定義

接続先0〜Nに合わせてレイヤー0〜Nを定義する。

| 接続先 | 端末 | レイヤー |
|--------|------|----------|
| 0 | Mac | 0 |
| 1 | Windows | 1 |
| 2 | iPad | 2 |

**重要**: ZMKは高いレイヤーが優先されるため、デフォルトレイヤーは他のレイヤーよりも低い位置に設定する必要がある。

### 3. 機種&レイヤー切り替えのマクロ定義

マクロの処理フロー：

1. 全てのデフォルトレイヤーのトグルをOFF
2. ちょっと待つ（wait時間を設定）
3. 接続先切り替えコマンドを実行（`&bt BT_SEL n`）
4. 接続先に紐づいたレイヤーのトグルをON（レイヤー0の場合は不要）

```dts
/ {
    macros {
        // Mac (接続先0, レイヤー0)
        bt_mac: bt_mac {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&toggle_off 1>, <&toggle_off 2>,
                       <&macro_wait_time 100>,
                       <&bt BT_SEL 0>;
        };

        // Windows (接続先1, レイヤー1)
        bt_win: bt_win {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&toggle_off 1>, <&toggle_off 2>,
                       <&macro_wait_time 100>,
                       <&bt BT_SEL 1>,
                       <&toggle_on 1>;
        };

        // iPad (接続先2, レイヤー2)
        bt_ipad: bt_ipad {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&toggle_off 1>, <&toggle_off 2>,
                       <&macro_wait_time 100>,
                       <&bt BT_SEL 2>,
                       <&toggle_on 2>;
        };
    };
};
```

### 4. キーマップに切り替えキーを設定

作成した切り替えマクロをキーマップに割り当てる。

```dts
&bt_mac   // Mac + レイヤー0に切り替え
&bt_win   // Windows + レイヤー1に切り替え
&bt_ipad  // iPad + レイヤー2に切り替え
```

---

## 技術的ポイント

### 1. toggle_on / toggle_off behavior

- 標準のトグルレイヤーを拡張
- 明示的にON/OFFを制御できるようにする
- `toggle-mode = "off"` で OFF 専用のビヘイビアを定義

### 2. マクロによる複合動作

- 接続先切り替え（`&bt BT_SEL n`）とレイヤー切り替えを1つのキーで実行
- wait時間を入れることで確実に処理を実行

### 3. レイヤー優先度

- ZMKは高い番号のレイヤーが優先される
- デフォルトレイヤーは低い番号に配置する必要がある

---

## 注意事項

- **電源ON時の不整合**: 電源ON時に接続先とデフォルトレイヤーが不整合になる可能性がある
- **解決方法**: 再度切り替えマクロを押せば解決

---

## LisM キーボードへの適用可能性

### 適用メリット

- Mac/Windows/iPadなど複数端末での使い分けが可能に
- 各端末に最適化されたキーマップを自動適用できる
- 例: MacではCmd、WindowsではCtrlを同じ物理キーに割り当て

### 実装時の考慮事項

1. 現在のレイヤー構成を確認し、デフォルトレイヤー用のスペースを確保
2. 各端末用のレイヤーを定義
3. BT切り替えキーをマクロに置き換え

---

## 関連リンク

- [moNa Discord サポートサーバー](https://discord.gg/SZ3EMRdk9N)
- [shakupan booth](https://shakupan.booth.pm)

---

## 信頼度評価

**高** - 実装例と手順が明確に示されており、moNaキーボードで実際に運用されている手法。
