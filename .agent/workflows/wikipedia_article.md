---
description: Wikipedia記事自動生成ワークフロー
---

# Wikipedia記事生成プロセス

このワークフローは、指定された主題についてWikipedia記事を生成するための一連のタスクを実行します。
各ステップは `tasks/` ディレクトリ内の定義ファイルに基づき実行されます。

## 前提条件
- `knowledge/` ディレクトリにWikipediaの方針ファイル（削除の方針など）が存在すること。
- 調査対象の「主題名（subject_name）」が決定していること。

## 実行ステップ

### 1. 情報源調査 (Research)
**タスク定義**: [tasks/research_subject.md](file:///Users/yusukekikuchi/Documents/AI_system/github/wikipedia/tasks/research_subject.md)
- **目的**: 主題に関する信頼できる情報源（報道機関、専門機関等）を収集し、構造化された調査レポートを作成します。
- **出力**: `outputs/{subject_name}/research_report.md`
- **重要**: 宣伝的な情報源や信頼性の低いブログ等は除外します。

### 2. 記事執筆 (Draft)
**タスク定義**: [tasks/draft_article.md](file:///Users/yusukekikuchi/Documents/AI_system/github/wikipedia/tasks/draft_article.md)
- **目的**: 調査レポートを入力とし、MediaWiki記法で記事ドラフトを執筆します。
- **入力**: `outputs/{subject_name}/research_report.md`
- **出力**: `outputs/{subject_name}/draft.txt`
- **重要**: 「〜と思われる」等の独自研究を避け、中立的な記述を心がけます。

### 3. 事実検証 (Verify)
**タスク定義**: [tasks/verify_facts.md](file:///Users/yusukekikuchi/Documents/AI_system/github/wikipedia/tasks/verify_facts.md)
- **目的**: 執筆されたドラフトに含まれる事実とURLの有効性を検証します。
- **入力**: `outputs/{subject_name}/draft.txt`
- **出力**: `outputs/{subject_name}/verification_report.md`
- **重要**: リンク切れや情報の誤りを厳密にチェックします。

### 4. URL確認 (Check URLs)
**タスク定義**: [tasks/check_urls.md](file:///Users/yusukekikuchi/Documents/AI_system/github/wikipedia/tasks/check_urls.md)
- **目的**: 記事ドラフトに含まれる全てのURLの有効性（ステータスコード200）を確認します。
- **入力**: `outputs/{subject_name}/draft.txt`
- **出力**: `outputs/{subject_name}/url_check/report.md`
- **重要**: リンク切れ（404等）がある場合は代替URLを探すか、修正を提案します。

### 5. 記事修正 (Refine)
**タスク定義**: [tasks/refine_article.md](file:///Users/yusukekikuchi/Documents/AI_system/github/wikipedia/tasks/refine_article.md)
- **目的**: 検証レポートの指摘事項に基づき、ドラフトを修正して最終版を完成させます。
- **入力**: `outputs/{subject_name}/draft.txt`, `outputs/{subject_name}/verification_report.md`, `outputs/{subject_name}/url_check/report.md`
- **出力**: `outputs/{subject_name}/final.txt`
- **重要**: 検証レポート自体を記事に含めないよう注意してください。
