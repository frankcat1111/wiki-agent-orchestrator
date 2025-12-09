# トークン最適化ガイド

## 実施済みの改善策

### ✅ 1. .cursorignore の最適化
- 完了済みプロジェクト (220KB) を除外
- knowledge/ ディレクトリ (124KB) を除外
- outputs/ の大きなレポートファイルを除外
- システムファイル (.DS_Store等) を除外

**削減効果**: 約 350KB以上のトークン削減

### ✅ 2. .gitignore の作成
- 一時ファイルや生成物をGit管理から除外
- チーム開発時の不要なコンフリクトを回避

### ✅ 3. Knowledge ディレクトリの最適化
- README.md に要約版を追加
- エージェントは必要時のみ元ファイルを読み込む方式に

## 追加の最適化オプション

### オプション A: 完了済みプロジェクトのアーカイブ化

完了済みディレクトリを圧縮またはGitブランチに移動：

```bash
# 方法1: ZIP圧縮してアーカイブ
cd /Users/yusukekikuchi/Documents/AI_system/github/wiki-agent-orchestrator
tar -czf archive/completed-projects-$(date +%Y%m%d).tar.gz 完了済み/
rm -rf 完了済み/

# 方法2: Gitブランチに保存
git checkout -b archive/completed-projects
git add 完了済み/
git commit -m "Archive completed projects"
git checkout main
rm -rf 完了済み/
```

**削減効果**: 220KB

### オプション B: 出力ファイルのストリーミング生成

research_report.md が大きくなる問題に対処：

1. **段階的な出力**: 各セクションを別ファイルに分割
   ```
   outputs/{主題}/
   ├── research/
   │   ├── 01-basic-info.md
   │   ├── 02-history.md
   │   └── 03-sources.md
   └── final_report.md (統合版)
   ```

2. **要約版の生成**: 冗長な情報を削減した要約版を作成

**削減効果**: 調査レポートあたり 30-50%

### オプション C: エージェント定義の最適化

現在のエージェント定義ファイルを確認：

```bash
# エージェント定義のサイズを確認
find .claude/agents -type f -exec wc -c {} +
```

可能な最適化：
- 重複する説明の削除
- 例示コードの外部参照化
- マニフェストYAMLの軽量化

**削減効果**: 10-20KB

### オプション D: プロジェクト構造の見直し

現在のディレクトリ構造の問題点：
1. 完了済みプロジェクトがリポジトリに残っている
2. outputs/ と 完了済み/ で重複した概念

**推奨構造：**
```
outputs/
├── active/           # 進行中のプロジェクト（.cursorignoreで除外しない）
│   ├── 株式会社YAMABE/
│   └── 株式会社フィッシュパス/
└── archive/          # 完了プロジェクト（.cursorignoreで除外）
    ├── VIALA自由が丘/
    └── 芳月健太郎/
```

## モニタリング

### トークン使用量の追跡

定期的に以下を実行してファイルサイズを確認：

```bash
# 大きなファイルTop 20を表示
find . -type f -not -path '*/\.git/*' -exec wc -c {} + | sort -rn | head -20

# ディレクトリ別サイズ
du -sh */ | sort -hr
```

### ベストプラクティス

1. **新規プロジェクト開始時**
   - outputs/active/ に作成
   - 不要な中間ファイルは削除

2. **プロジェクト完了時**
   - 必要なファイルのみ outputs/archive/ に移動
   - または Git LFS / 外部ストレージに保存

3. **定期メンテナンス**
   - 月1回: archive/ の古いプロジェクトを圧縮
   - 四半期1回: knowledge/ の内容を最新化

## 効果測定

| 項目 | 改善前 | 改善後 | 削減率 |
|------|--------|--------|--------|
| knowledge/ | 124KB | 除外 | 100% |
| 完了済み/ | 220KB | 除外 | 100% |
| outputs/レポート | 各15KB | 除外 | 100% |
| **合計** | ~370KB | **~0KB** | **~100%** |

## トークン消費の継続的な監視

Claude Code使用時のトークン消費を監視：

```bash
# セッション開始時と終了時に実行
echo "Session start: $(date)" >> .token-usage.log
find . -type f -not -path '*/\.git/*' -exec wc -c {} + | awk '{sum+=$1} END {print "Total bytes:", sum}' >> .token-usage.log
```

## 参考資料

- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [Token Optimization Best Practices](https://docs.anthropic.com/claude/docs/reducing-token-usage)
