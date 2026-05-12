# Mac/Windows Mark Layer 対応表

macのmarkレイヤー（US配列）とwindowsのmarkレイヤー（JIS配列）の対応表です。

## 目的

macのmarkレイヤーで入力できる記号と同じ記号を、windowsのmarkレイヤー（JIS配列）でも入力できるようにするための分析資料。

## 凡例

- **Mac（現在）**: macのmarkレイヤーで設定されているキーコード
- **Mac出力**: macのUS配列で実際に出力される記号
- **Win（現在）**: windowsのmarkレイヤーで設定されているキーコード
- **Win出力**: windowsのJIS配列で実際に出力される記号
- **修正要否**: macと同じ記号を出力するために修正が必要かどうか
- **修正後Win**: 修正が必要な場合の正しいキーコード

## JIS配列でのキーコード対応（重要）

JIS配列では、USキーコードと出力される記号の対応が異なります：

| USキーコード | JIS配列での出力 | 備考 |
|--------------|-----------------|------|
| `LBKT` (0x2F) | `@` | Pの右のキー |
| `RBKT` (0x30) | `[` | @の右のキー |
| `BSLH` / `BACKSLASH` (0x31) | `]` | [の右のキー |
| `NUHS` / `NON_US_HASH` | `]` | 同上（別名） |

## Row 1（上段）

| 位置 | Mac（現在） | Mac出力 | Win（現在） | Win出力 | 修正要否 | 修正後Win |
|------|-------------|---------|-------------|---------|----------|-----------|
| Q | `SQT` | `'` | `LS(N7)` | `'` | ✅ OK | - |
| W | `DOUBLE_QUOTES` | `"` | `LS(N2)` | `"` | ✅ OK | - |
| E | `EXCLAMATION` | `!` | `EXCLAMATION` | `!` | ✅ OK | - |
| R | `LEFT_BRACKET` | `[` | `RBKT` | `[` | ✅ OK | - |
| T | `LEFT_PARENTHESIS` | `(` | `LS(N8)` | `(` | ✅ OK | - |
| Y | `RIGHT_PARENTHESIS` | `)` | `LS(N9)` | `)` | ✅ OK | - |
| U | `RIGHT_BRACKET` | `]` | `BSLH` | `]` | ✅ OK | - |
| I | `PLUS` | `+` | `LS(SEMI)` | `+` | ✅ OK | - |
| O | `AMPERSAND` | `&` | `LS(N6)` | `&` | ✅ OK | - |
| P | `PERCENT` | `%` | `PERCENT` | `%` | ✅ OK | - |

## Row 2（中段）

| 位置 | Mac（現在） | Mac出力 | Win（現在） | Win出力 | 修正要否 | 修正後Win |
|------|-------------|---------|-------------|---------|----------|-----------|
| A | `AT_SIGN` | `@` | `LBKT` | `@` | ✅ OK | - |
| S | `SEMICOLON` | `;` | `SEMICOLON` | `;` | ✅ OK | - |
| D | `DOLLAR` | `$` | `DOLLAR` | `$` | ✅ OK | - |
| F | `trans` | - | `trans` | - | ✅ OK | - |
| G | `trans` | - | `trans` | - | ✅ OK | - |
| H | `HASH` | `#` | `HASH` | `#` | ✅ OK | - |
| J | `SLASH` | `/` | `SLASH` | `/` | ✅ OK | - |
| K | `PIPE` | `\|` | `LS(INT1)` | `_` | ❌ 要修正 | `LS(INT3)` |
| L | `BACKSLASH` | `\` | `INT1` | `\` | ✅ OK | - |
| RETURN | `AMPERSAND` | `&` | `LS(N6)` | `&` | ✅ OK | - |

## Row 3（下段）

| 位置 | Mac（現在） | Mac出力 | Win（現在） | Win出力 | 修正要否 | 修正後Win |
|------|-------------|---------|-------------|---------|----------|-----------|
| Z | `LEFT_SHIFT` | (修飾キー) | `LEFT_SHIFT` | (修飾キー) | ✅ OK | - |
| X | `trans` | - | `trans` | - | ✅ OK | - |
| C | `COLON` | `:` | `SQT` | `:` | ✅ OK | - |
| V | `COMMA` | `,` | `COMMA` | `,` | ✅ OK | - |
| B | `MINUS` | `-` | `MINUS` | `-` | ✅ OK | - |
| N | `EQUAL` | `=` | `LS(MINUS)` | `=` | ✅ OK | - |
| M | `PERIOD` | `.` | `PERIOD` | `.` | ✅ OK | - |
| , | `MINUS` | `-` | `MINUS` | `-` | ✅ OK | - |
| . | `trans` | - | `trans` | - | ✅ OK | - |
| / | `trans` | - | `trans` | - | ✅ OK | - |

## Row 4（親指行）

| 位置 | Mac（現在） | Mac出力 | Win（現在） | Win出力 | 修正要否 | 修正後Win |
|------|-------------|---------|-------------|---------|----------|-----------|
| 左1 | `trans` | - | `trans` | - | ✅ OK | - |
| 左2 | `trans` | - | `trans` | - | ✅ OK | - |
| 左3 | `trans` | - | `trans` | - | ✅ OK | - |
| 左4 | `trans` | - | `trans` | - | ✅ OK | - |
| 左5 | `trans` | - | `trans` | - | ✅ OK | - |
| 左6 | `trans` | - | `trans` | - | ✅ OK | - |
| 右1 | `trans` | - | `trans` | - | ✅ OK | - |
| 右2 | `LG(SPACE)` | Cmd+Space | `trans` | - | ❓ 要確認 | Win用IME切替？ |
| 右3 | `trans` | - | `trans` | - | ✅ OK | - |
| 右4 | `ASTERISK` | `*` | `LS(SQT)` | `*` | ✅ OK | - |
| 右5 | `CARET` | `^` | `EQUAL` | `^` | ✅ OK | - |
| 右6 | `GRAVE` | `` ` `` | `LS(LBKT)` | `` ` `` | ✅ OK | - |

## JIS配列での記号キーコード対応表（参考）

| 記号 | US配列キーコード | JIS配列キーコード |
|------|------------------|-------------------|
| `'` | `SQT` | `LS(N7)` |
| `"` | `DOUBLE_QUOTES` / `LS(SQT)` | `LS(N2)` |
| `@` | `AT_SIGN` / `LS(N2)` | `LBKT` |
| `[` | `LEFT_BRACKET` / `LBKT` | `RBKT` |
| `]` | `RIGHT_BRACKET` / `RBKT` | `BSLH` / `BACKSLASH` |
| `{` | `LEFT_BRACE` / `LS(LBKT)` | `LS(RBKT)` |
| `}` | `RIGHT_BRACE` / `LS(RBKT)` | `LS(BSLH)` |
| `(` | `LEFT_PARENTHESIS` / `LS(N9)` | `LS(N8)` |
| `)` | `RIGHT_PARENTHESIS` / `LS(N0)` | `LS(N9)` |
| `+` | `PLUS` / `LS(EQUAL)` | `LS(SEMI)` |
| `=` | `EQUAL` | `LS(MINUS)` |
| `:` | `COLON` / `LS(SEMI)` | `SQT` |
| `*` | `ASTERISK` / `LS(N8)` | `LS(SQT)` |
| `^` | `CARET` / `LS(N6)` | `EQUAL` |
| `&` | `AMPERSAND` / `LS(N7)` | `LS(N6)` |
| `\|` | `PIPE` / `LS(BACKSLASH)` | `LS(INT3)` |
| `\` | `BACKSLASH` | `INT1`（右Shift左）または `INT3`（BS左） |
| `` ` `` | `GRAVE` | `LS(LBKT)` |
| `~` | `TILDE` / `LS(GRAVE)` | `LS(EQUAL)` |
| `_` | `UNDERSCORE` / `LS(MINUS)` | `LS(INT1)` |
| `¥` | N/A | `INT3` |

## 要修正箇所まとめ

### K位置: `|` (パイプ) の入力

- **Mac**: `PIPE` → `|`
- **Win（現在）**: `LS(INT1)` → `_`（アンダースコア）を出力（**間違い**）
- **修正後Win**: `LS(INT3)` → `|`

### 右2位置: IME切り替え（検討事項）

- **Mac**: `LG(SPACE)` → Cmd+Space（Spotlight/IME切替）
- **Win（現在）**: `trans` → 何もなし
- **検討**: Win用のIME切替キーを設定するか？
  - `LA(GRAVE)` または `LC(SPACE)` など

## 補足: JIS配列の特殊キー

- `INT1`: JIS配列の `\` / `_` キー（右Shiftの左）
  - 単押し: `\`
  - Shift+: `_`
- `INT3`: JIS配列の `¥` / `|` キー（Backspaceの左）
  - 単押し: `¥`
  - Shift+: `|`

## 次のステップ

1. K位置の `LS(INT1)` を `LS(INT3)` に修正
2. （オプション）右2位置にWin用IME切替を設定
3. 実機でテストして動作確認
