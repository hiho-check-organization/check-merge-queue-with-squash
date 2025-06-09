# GitHub マージキューとスカッシュマージの動作検証

[VOICEVOX/voicevox_engine#1734](https://github.com/VOICEVOX/voicevox_engine/issues/1734) で報告された問題を検証するためのプロジェクトです。

## 問題の背景

以下の状況で潜在的な問題が発生する可能性があります：

> もしかしたら、親設定（Settings > Pull Requests）の Allow merge commits がオンの状態で merge queue の method を ff merge にしたあと、親設定の Allow merge commits をオフにしたとき、UI 上は squash になるけど実際は ff merge される状況になるのかもしれない

## 用語と構成要素

### 設定項目

- **リポジトリの親設定**（Settings > Pull Requests）
  - Allow merge commits: マージコミットを許可するかどうか
  - Allow squash merging: スカッシュマージを許可するかどうか
  - Allow rebase merging: リベースマージを許可するかどうか
- **マージキューの設定**（ブランチ保護ルールの一部）
  - マージ方法: Fast-Forward、Squash などを指定できる

### マージ方法の違い

- **FF（Fast-Forward）マージ**: コミット履歴が直線的、元のコミットメッセージが保持される
- **スカッシュマージ**: 複数のコミットが1つに圧縮され、PRタイトルがコミットメッセージとなる

## 検証シナリオ

### シナリオ1: 設定変更による不整合検証

親設定とマージキューの設定間に不整合が生じた場合の動作を検証します。

#### 初期設定

1. **親設定**
   - Allow merge commits: **オン**
   - Allow squash merging: オン
   - Allow rebase merging: オン（任意）

2. **マージキュー設定**
   - マージ方法: **Fast-Forward**

#### PR #1 検証 (基準検証)

1. ブランチ作成と変更
   ```bash
   git checkout -b feature-1
   echo "# 変更 1" >> test-file-1.md
   git add test-file-1.md
   git commit -m "Add test file 1"
   git push -u origin feature-1
   ```

2. **PR作成と確認**
   - UIのマージ方法を確認（FF マージと表示されるはず）
   - スクリーンショット撮影

3. **マージ実行とログ確認**
   - マージキューでマージ
   - `git log --graph --oneline -n 5`でFF マージを確認

#### 設定変更

1. **親設定変更**
   - Allow merge commits: **オフ**
   - その他の設定は変更なし

2. **マージキュー設定**: 変更しない

#### PR #2 検証 (不整合検証)

1. ブランチ作成と変更
   ```bash
   git checkout -b feature-2
   echo "# 変更 2" >> test-file-2.md
   git add test-file-2.md
   git commit -m "Add test file 2"
   git push -u origin feature-2
   ```

2. **PR作成と確認**
   - UIのマージ方法を確認（スカッシュマージと表示されるはず）
   - スクリーンショット撮影

3. **マージ実行とログ確認**
   - マージキューでマージ
   - `git log --graph --oneline -n 5`で実際のマージ方法を確認
   - **結果判定**:
     - FFマージされた場合: 仮説が正しい
     - スカッシュマージされた場合: 仮説が誤り

### シナリオ2: マージキュー設定保存と親設定変更の影響

親設定を変更した後、マージキュー設定を保存し直さない場合と保存し直した場合の違いを検証します。

#### 初期設定

1. **親設定**
   - Allow merge commits: **オン**のみ（他をオフ）

2. **マージキュー設定**
   - マージ方法: **Fast-Forward**

#### 親設定変更

1. **親設定変更**
   - Allow merge commits: **オフ**
   - Allow squash merging: **オン**

2. **マージキュー設定**: 変更・保存しない

#### PR #3 検証 (設定保存なし)

1. ブランチ作成と変更
   ```bash
   git checkout -b feature-3
   echo "# 変更 3" >> test-file-3.md
   git add test-file-3.md
   git commit -m "Add test file 3"
   git push -u origin feature-3
   ```

2. **PR作成と確認**
   - UIのマージ方法を確認
   - スクリーンショット撮影

3. **マージ実行とログ確認**
   - `git log --graph --oneline -n 5`
   - 実際のマージ方法を記録

#### マージキュー設定の再保存

1. **マージキュー設定を変更なしで保存**
   - マージ方法: 変更せずに保存のみ実行

#### PR #4 検証 (設定再保存後)

1. ブランチ作成と変更
   ```bash
   git checkout -b feature-4
   echo "# 変更 4" >> test-file-4.md
   git add test-file-4.md
   git commit -m "Add test file 4"
   git push -u origin feature-4
   ```

2. **PR作成と確認**
   - UIのマージ方法を確認
   - スクリーンショット撮影

3. **マージ実行とログ確認**
   - `git log --graph --oneline -n 5`
   - 実際のマージ方法を記録

## 結果記録と分析

### マージ方法の判別

1. **FFマージの特徴**:
   - コミット履歴が直線的
   - コミットメッセージが元のブランチのコミットと同一

   例:
   ```
   * abcd123 Add test file 1  (HEAD -> main, origin/main)
   * efgh456 Initial commit
   ```

2. **スカッシュマージの特徴**:
   - コミットが圧縮される
   - コミットメッセージはPRタイトル + PR番号の形式

   例:
   ```
   * ijkl789 Add test file 2 (#2)  (HEAD -> main, origin/main)
   * abcd123 Add test file 1
   * efgh456 Initial commit
   ```

### 結果テンプレート

```
## シナリオX 検証結果

### PR #Y 検証

#### 設定状態
- 親設定: Allow merge commits [オン/オフ], Allow squash merging [オン/オフ]...
- マージキュー設定: [Fast-forward/Squash]

#### UI表示
- マージボタン表示: [Merge/Squash and merge/Rebase and merge]

#### 実際のマージ結果
- git log 出力: [出力をペースト]
- 実行されたマージ方法: [FF マージ/スカッシュマージ]

#### 結論
- UI表示と実際のマージ方法の一致: [一致/不一致]
- 仮説の検証: [支持される/支持されない]
```

### 仮説検証のポイント

1. **UI表示と実際の動作の一致**:
   - UIが「Squash and merge」表示で実際にスカッシュされるか
   - UIが「Merge」表示で実際にFFマージされるか

2. **設定の相互作用**:
   - 親設定変更後、マージキュー設定を保存し直さなくても設定が反映されるか
   - 親設定で無効化した方法がマージキューでも無効になるか
