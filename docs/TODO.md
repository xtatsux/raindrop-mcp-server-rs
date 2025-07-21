# TODO: Fix Broken Field Serialization Error

## タスクリスト

### 1. コード修正
- [x] `src/raindrop/types.rs`の`Bookmark`構造体の`broken`フィールドを`Option<bool>`に変更
- [x] テストケースの更新（types.rs:627行目のアサーション修正）
- [x] エラーハンドリングの改善（client.rs:57行目付近にデバッグ情報追加）

### 2. テスト
- [x] 既存のユニットテストが全て通ることを確認（`cargo test`）
- [x] `broken`フィールドが含まれないJSONのテストケースを追加
- [x] linterチェック（`cargo clippy`）
- [x] フォーマットチェック（`cargo fmt --check`）

### 3. 動作確認
- [x] ビルドが成功することを確認（`cargo build`）
- [ ] 実際のRaindrop APIに対してget_bookmarksを実行し、エラーが解消されることを確認

### 4. ドキュメント
- [ ] 変更内容をREADMEに記載（必要に応じて）
- [ ] コミットメッセージの準備

### 5. 完了
- [ ] 全てのタスクが完了したことを確認
- [ ] プルリクエストの作成

## 進捗メモ
- 開始時刻: 2025-07-21
- ブランチ名: `fix/broken-field-serialization-error`

---

# TODO: Fix FullName Field Serialization Error

## タスクリスト

### 1. コード修正
- [x] `src/raindrop/types.rs`の`CreatorRef`構造体の`full_name`フィールドを`Option<String>`に変更
- [x] テストケースで`CreatorRef`を使用している箇所の確認と必要に応じた修正

### 2. テスト
- [x] `fullName`フィールドが含まれないコレクションのテストケースを追加
- [x] 既存のユニットテストが全て通ることを確認（`cargo test`）
- [x] linterチェック（`cargo clippy`）
- [x] フォーマットチェック（`cargo fmt --check`）

### 3. 動作確認
- [x] ビルドが成功することを確認（`cargo build`）
- [ ] 実際のRaindrop APIに対してget_collectionsを実行し、エラーが解消されることを確認

### 4. 完了
- [ ] 全てのタスクが完了したことを確認

## 進捗メモ
- 開始時刻: 2025-07-21
- 関連issue: コレクション一覧取得時のfullNameフィールドエラー

---

# TODO: Fix i32 Field Null Error

## タスクリスト

### Phase 1: Collection構造体の修正
- [x] `src/raindrop/types.rs`の`Collection`構造体の`sort`フィールドを`Option<i32>`に変更
- [x] `src/raindrop/types.rs`の`Collection`構造体の`count`フィールドを`Option<i32>`に変更
- [x] 既存のテストケースで`sort`と`count`を使用している箇所の修正

### Phase 2: テスト
- [x] `sort`と`count`フィールドがnullのコレクションのテストケースを追加
- [x] 既存のユニットテストが全て通ることを確認（`cargo test`）
- [x] linterチェック（`cargo clippy`）
- [x] フォーマットチェック（`cargo fmt --check`）

### Phase 3: 動作確認
- [x] ビルドが成功することを確認（`cargo build`）
- [ ] 実際のRaindrop APIに対してget_collectionsを実行し、エラーが解消されることを確認

### Phase 4: 追加対応（必要に応じて）
- [ ] AccessInfo構造体の`level`フィールドの修正（エラーが継続する場合）
- [ ] Group構造体の`sort`フィールドの修正（エラーが継続する場合）

### 完了
- [ ] 全てのタスクが完了したことを確認

## 進捗メモ
- 開始時刻: 2025-07-21
- 関連issue: コレクション一覧取得時のi32フィールドnullエラー

---

# TODO: Fix Important Field Serialization Error

## タスクリスト

### 1. コード修正
- [x] `src/raindrop/types.rs`の`Bookmark`構造体の`important`フィールドを`Option<bool>`に変更
- [x] 既存のテストケースで`important`を使用している箇所の修正

### 2. テスト
- [x] `important`フィールドが含まれないブックマークのテストケースを追加
- [x] 既存のユニットテストが全て通ることを確認（`cargo test`）
- [x] linterチェック（`cargo clippy`）
- [x] フォーマットチェック（`cargo fmt --check`）

### 3. 動作確認
- [x] ビルドが成功することを確認（`cargo build`）
- [ ] 実際のRaindrop APIに対してsearch_bookmarksを実行し、エラーが解消されることを確認

### 4. 完了
- [ ] 全てのタスクが完了したことを確認

## 進捗メモ
- 開始時刻: 2025-07-21
- 関連issue: search_bookmarks実行時のimportantフィールドエラー