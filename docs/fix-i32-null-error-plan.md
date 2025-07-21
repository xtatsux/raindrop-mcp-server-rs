# Fix Plan: i32 Field Null Error

## 問題の概要
`get_collections`で「invalid type: null, expected i32 at line 1 column 1489」というJSONシリアライゼーションエラーが発生している。

## 原因分析
Raindrop APIレスポンスで、i32型として定義されているフィールドにnull値が含まれている。

調査の結果、以下のi32型フィールドがnullになる可能性がある：

### 1. Collection構造体（最も可能性が高い）
- `sort: i32` (types.rs:105行目) - コレクションの並び順
- `count: i32` (types.rs:107行目) - コレクション内のアイテム数

### 2. AccessInfo構造体
- `level: i32` (types.rs:130行目) - アクセスレベル
- Collection構造体の`access: Option<AccessInfo>`フィールド内

### 3. Group構造体
- `sort: i32` (types.rs:44行目) - グループの並び順
- User構造体の`groups: Option<Vec<Group>>`フィールド内

## 解決案

### 段階的アプローチ
エラー位置（column 1489）から判断して、Collection構造体のフィールドが最も疑わしいため、まずこれらを修正し、必要に応じて他のフィールドも対応する。

### Phase 1: Collection構造体の修正
```rust
// Before
pub sort: i32,
pub count: i32,

// After  
pub sort: Option<i32>,
pub count: Option<i32>,
```

### Phase 2: 必要に応じて追加修正
Phase 1で解決しない場合：
- `AccessInfo`の`level`フィールドをOption型に変更
- `Group`の`sort`フィールドをOption型に変更

## 実装詳細

### 1. 型定義の変更
`src/raindrop/types.rs`の`Collection`構造体を修正：
```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct Collection {
    #[serde(rename = "_id")]
    pub id: i64,
    pub title: String,
    pub description: Option<String>,
    pub color: Option<String>,
    pub public: Option<bool>,
    pub view: CollectionView,
    pub sort: Option<i32>,        // i32 → Option<i32>
    pub cover: Option<Vec<String>>,
    pub count: Option<i32>,        // i32 → Option<i32>
    pub expanded: Option<bool>,
    // ... rest of fields
}
```

### 2. テストケースの更新
既存のテストケースでsortとcountを使用している箇所：
- `test_collection_deserialization` (types.rs:552行目付近)
- 新しいコレクション作成時のデフォルト値設定

### 3. 新しいテストケースの追加
```rust
#[test]
fn test_collection_with_null_sort_and_count() {
    let json = json!({
        "_id": 999,
        "title": "Test Collection with Nulls",
        "view": "list",
        "sort": null,
        "count": null,
        "user": { "$id": 123 },
        "created": "2023-01-01T00:00:00Z",
        "lastUpdate": "2023-01-01T00:00:00Z"
    });
    
    let collection: Collection = serde_json::from_value(json).unwrap();
    assert_eq!(collection.id, 999);
    assert_eq!(collection.sort, None);
    assert_eq!(collection.count, None);
}
```

## 影響範囲の分析

### ビジネスロジックへの影響
1. **sort フィールド**
   - コレクションの表示順序に使用される可能性
   - デフォルト値として0を想定したロジックがある場合は要確認

2. **count フィールド**
   - コレクション内のアイテム数表示に使用
   - UIでの表示やページネーション計算に影響する可能性

### 対応が必要な箇所
- sortやcountを直接参照している箇所でのnullチェック追加
- デフォルト値の設定（例：`collection.sort.unwrap_or(0)`）

## リスク評価
- **中リスク**: 
  - これらのフィールドはUIやソート機能で使用される重要なフィールド
  - Option型への変更により、既存コードでのnullチェックが必要
  - ただし、コンパイル時にエラーとして検出されるため、見落としのリスクは低い

## テスト計画
1. 既存のユニットテストが全て通ることを確認
2. nullフィールドを含むコレクションのテストケースが成功することを確認
3. 実際のRaindrop APIに対してget_collectionsを実行し、エラーが解消されることを確認
4. sortとcountがnullの場合のUIでの表示を確認（必要に応じて）

## デフォルト値の推奨
- `sort`: 0（最初の位置）
- `count`: 0（空のコレクション）

使用例：
```rust
let sort_value = collection.sort.unwrap_or(0);
let item_count = collection.count.unwrap_or(0);
```