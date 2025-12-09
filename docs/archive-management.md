# 完了済みプロジェクト保管庫の管理ガイド

## 概要

`完了済み/` ディレクトリは、Wikipedia記事生成プロジェクトの完了後の成果物を保管する場所です。

## トークン最適化のための除外設定

### ✅ 実施済みの設定

完了済みディレクトリは以下の3つのファイルで完全に除外されています：

| ファイル | 目的 | 効果 |
|---------|------|------|
| `.cursorignore` | Cursor AI用 | AIがコンテキストに読み込まない |
| `.claudeignore` | Claude Code用 | Claude Codeがコンテキストに読み込まない |
| `.gitignore` | Git管理除外 | バージョン管理から除外 |

### 除外パターン

各ファイルには以下のパターンが設定されています：

```
完了済み/
完了済み/**/*
**/完了済み/
**/完了済み/**/*
```

これにより、完了済みディレクトリとその全サブディレクトリ・ファイルが除外されます。

## トークン削減効果

| 項目 | サイズ |
|------|--------|
| 完了済みディレクトリ合計 | 220KB |
| プロジェクト数 | 7件 |
| **削減されたトークン** | **約220KB相当** |

## 保管されているプロジェクト

現在の完了済みプロジェクト：

1. VIALA自由が丘（レストラン）
2. 株式会社FrankPR（PR会社）
3. 高知アイス（製造会社）
4. 岩佐大輝（人物）
5. 塚原龍雲（人物）
6. エコプロコート株式会社（企業）
7. 芳月健太郎（人物）

## アーカイブの利用方法

### 方法1: 個別ファイルの参照

過去のプロジェクトを参照する必要がある場合、Read toolで直接読み込みます：

```
Read /path/to/完了済み/{プロジェクト名}/final.txt
```

### 方法2: 圧縮アーカイブ作成

長期保管のため圧縮：

```bash
cd /Users/yusukekikuchi/Documents/AI_system/github/wiki-agent-orchestrator
tar -czf archive/completed-$(date +%Y%m%d).tar.gz 完了済み/
```

### 方法3: 外部ストレージへのバックアップ

クラウドストレージやNASにバックアップ：

```bash
# Google Drive, Dropbox, OneDrive等にコピー
cp -r 完了済み/ /path/to/cloud/storage/wikipedia-archive/

# 確認後、ローカルから削除
rm -rf 完了済み/
```

## 新規プロジェクトの管理フロー

### 1. プロジェクト開始

```
outputs/{プロジェクト名}/
```

で作業を開始

### 2. プロジェクト完了

完了後、2つの選択肢：

**オプションA: 保管庫に移動**
```bash
mv outputs/{プロジェクト名} 完了済み/
```

**オプションB: 削除**（バックアップ済みの場合）
```bash
rm -rf outputs/{プロジェクト名}
```

### 3. 定期メンテナンス（推奨：月1回）

```bash
# 1. 完了済みを確認
ls -lh 完了済み/

# 2. 圧縮アーカイブ作成
tar -czf archive/completed-$(date +%Y%m%d).tar.gz 完了済み/

# 3. 外部バックアップ
cp archive/completed-*.tar.gz /path/to/backup/

# 4. ローカルから削除（オプション）
rm -rf 完了済み/*
```

## トラブルシューティング

### Q: 完了済みディレクトリがまだAIに読み込まれている

**確認方法:**
```bash
# .cursorignore, .claudeignoreの内容を確認
cat .cursorignore | grep 完了済み
cat .claudeignore | grep 完了済み
```

**解決方法:**
1. Claude Code / Cursor AIを再起動
2. 設定ファイルを再読み込み
3. キャッシュをクリア

### Q: Gitで完了済みディレクトリがトラッキングされている

**確認:**
```bash
git ls-files 完了済み/
```

**解決:**
```bash
# Gitトラッキングから削除（ファイル自体は保持）
git rm -r --cached 完了済み/
git commit -m "Remove 完了済み from git tracking"
```

### Q: ディスク容量を節約したい

**推奨アクション:**
```bash
# 1. 圧縮比を確認
tar -czf test.tar.gz 完了済み/
du -sh 完了済み/ test.tar.gz

# 2. 圧縮後のサイズが小さければ、圧縮版のみ保持
tar -czf archive/completed-archive.tar.gz 完了済み/
rm -rf 完了済み/
```

## ベストプラクティス

1. **定期的なアーカイブ**: 月1回、完了済みを圧縮してバックアップ
2. **外部バックアップ**: クラウドまたは外部ストレージに保存
3. **ローカルクリーンアップ**: バックアップ確認後、ローカルから削除
4. **ドキュメント化**: 各プロジェクトの完了日と成果をREADMEに記録

## 監視と最適化

### トークン使用量のモニタリング

```bash
# 完了済みディレクトリのサイズ監視
watch -n 3600 'du -sh 完了済み/'

# 月次レポート作成
echo "$(date): $(du -sh 完了済み/)" >> .archive-size-log
```

### 自動クリーンアップスクリプト（オプション）

```bash
#!/bin/bash
# monthly-archive.sh

ARCHIVE_DIR="archive"
COMPLETED_DIR="完了済み"
DATE=$(date +%Y%m%d)

# アーカイブディレクトリ作成
mkdir -p "$ARCHIVE_DIR"

# 圧縮
tar -czf "$ARCHIVE_DIR/completed-$DATE.tar.gz" "$COMPLETED_DIR/"

# バックアップ確認
if [ -f "$ARCHIVE_DIR/completed-$DATE.tar.gz" ]; then
  echo "✓ Archive created: completed-$DATE.tar.gz"
  echo "  Size: $(du -sh "$ARCHIVE_DIR/completed-$DATE.tar.gz" | cut -f1)"

  # オプション: ローカルから削除
  # read -p "Delete local 完了済み/? (y/N): " -n 1 -r
  # echo
  # if [[ $REPLY =~ ^[Yy]$ ]]; then
  #   rm -rf "$COMPLETED_DIR"/*
  #   echo "✓ Local files deleted"
  # fi
else
  echo "✗ Archive creation failed"
fi
```

## 参考

- [Token Optimization Guide](./token-optimization.md)
- [Wikipedia Pipeline Documentation](../CLAUDE.md)
