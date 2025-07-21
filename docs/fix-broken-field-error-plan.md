# Fix Plan: Broken Field Serialization Error

## 問題の概要
`get_bookmarks`ツールで「missing field 'broken' at line 1 column 1324」というJSONシリアライゼーションエラーが発生している。

## 原因分析
- `Bookmark`構造体の`broken`フィールドが`bool`型（必須フィールド）として定義されている（types.rs:195行目）
- 実際のRaindrop APIレスポンスには`broken`フィールドが含まれていない場合がある
- serdeのデシリアライゼーション時に必須フィールドが見つからないためエラーになる

## 解決案の比較

### 案1: `broken`フィールドをオプショナルにする ✅ (推奨)
```rust
pub broken: Option<bool>,
```

**メリット：**
- APIレスポンスの実際の仕様に合わせた柔軟な対応
- `broken`フィールドの有無を明示的に判断できる
- 将来的なAPI変更にも柔軟に対応可能

**デメリット：**
- 既存のコードで`broken`フィールドを使用している箇所の修正が必要

### 案2: デフォルト値を設定する
```rust
#[serde(default)]
pub broken: bool,
```

**メリット：**
- 既存のコードへの影響が最小限
- 後方互換性を保ちやすい

**デメリット：**
- フィールドが存在しない場合は常に`false`となる
- APIの実際の状態を正確に反映しない可能性

## 採用する解決案
**案1（オプショナル化）を採用します。**

理由：
1. Raindrop APIの実際の仕様に忠実
2. `broken`フィールドの有無を明示的に判断でき、より正確なデータ表現が可能
3. 将来的なAPI変更にも柔軟に対応可能
4. エラーの根本原因を解決

## 実装詳細

### 1. 型定義の変更
`src/raindrop/types.rs`の`Bookmark`構造体を修正：
```rust
// Before
pub broken: bool,

// After
pub broken: Option<bool>,
```

### 2. テストケースの更新
- types.rs:614行目: `"broken": false` → `"broken": false`（そのまま、Option型でも正しく動作）
- types.rs:627行目: `assert!(!bookmark.broken);` → `assert_eq!(bookmark.broken, Some(false));`

### 3. 影響範囲の確認
`broken`フィールドを参照している箇所を確認し、必要に応じて修正：
- 現在の実装では`broken`フィールドは読み取りのみで、直接操作していない
- `ExportOptions`構造体にも`broken`フィールドがあるが、これは別の用途（フィルタリング）なので影響なし

### 4. エラーハンドリングの改善
デバッグのため、JSONデシリアライゼーションエラー時により詳細な情報を出力：
```rust
// client.rs:57行目付近
serde_json::from_str::<T>(&text).map_err(|e| {
    debug!("Failed to deserialize response: {}", e);
    debug!("Response text: {}", text);
    RaindropMcpError::JsonSerialization(e)
})
```

## テスト計画
1. 既存のユニットテストが全て通ることを確認
2. `broken`フィールドが含まれないAPIレスポンスのテストケースを追加
3. 実際のRaindrop APIに対してget_bookmarksを実行し、エラーが解消されることを確認

## リスク評価
- **低リスク**: `broken`フィールドは主に読み取り専用で使用されており、ビジネスロジックへの影響は最小限
- 既存のAPIレスポンスで`broken`フィールドが含まれている場合も、Option型は正しく処理される