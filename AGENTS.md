# AGENTS

## 概要
- **目的**: 指定された主題（人物・企業）に関するWikipedia記事を、公式方針（NPOV、検証可能性、独自研究の禁止）に準拠した状態で自動生成する。
- **成功指標**:
  - MediaWiki構文エラー率 0%
  - 参照リンク（出典）の有効性 100%
  - Wikipedia削除方針（宣伝・特筆性不足）への抵触回避
- **非目標**: Wikipediaへの自動投稿（あくまでドラフト生成までをスコープとする）
- **制約**:
  - 存命人物の伝記（BLP）における厳格な出典要求
  - Web検索APIのレート制限およびトークン長制限



## エージェント一覧
| Agent | 役割 | 主要タスク | 入力 | 出力 | 所有者 |
|---|---|---|---|---|---|
| **Researcher** | 情報収集・構造化 | `research_subject` | 主題名、基礎情報 | 構造化調査レポート (`outputs/{主題名}/research_report.md`) | Content |
| **Drafter** | 記事執筆（MediaWiki） | `draft_article` | 調査レポート | 記事ドラフト (`outputs/{主題名}/draft.txt`) | Engineering |
| **Auditor** | ファクトチェック | `verify_facts` | 記事ドラフト | 検証レポート (`outputs/{主題名}/verification_report.md`) | QA |
| **Editor** | 修正・最終化 | `refine_article` | ドラフト + 検証レポート | 最終記事 (`outputs/{主題名}/final.txt`) | Content |
| **UrlChecker** | URL生存確認 | `check_urls` | 最終記事 | URL検証レポート (`outputs/{主題名}/url_check_report.md`) | QA |

## ランタイム構成
- **接続ツール**:
  - `WebSearch`: リアルタイム情報の取得、URL生存確認
  - `ContentScraper`: 参照先コンテンツの本文抽出
  - `UrlChecker`: HTTPステータスコード確認
- **レート制限**: 各エージェントあたり 60 RPM（WebSearch依存）
- **機密情報の扱い**: 出力に含まれる個人情報は公開情報（著名人等）に限る。非公開情報は `SafetyFilter` で除外。

## ルーティングとオーケストレーション
- **フロー**: 完全順次実行（Sequential Pipeline）
  1. `Researcher`: 情報収集と出典評価
  2. `Drafter`: MediaWiki形式での執筆
  3. `Auditor`: 生成物のURL生存確認・事実検証
  4. `Editor`: 検証結果に基づく修正
  5. `UrlChecker`: 最終成果物のリンク切れチェック

- **フェイルオーバー**:
  - `Auditor` が重大な虚偽（Critical Hallucination）を検出した場合 → `Researcher` へ戻し再調査（Human Loop 推奨）
  - `WebSearch` タイムアウト時 → 既知のナレッジベースのみで構成せず、欠損箇所を明記して終了

## 観測
- **ログ**:
  - `research_sources`: 採用した出典URLとその信頼性スコア
  - `syntax_errors`: Markdown混入などの構文エラー数
  - `verification_result`: ファクトチェックでの指摘項目数
- **メトリクス**:
  - `citation_quality_score`: (学術・大手メディア出典数 / 全出典数)
  - `hallucination_rate`: ファクトチェックでのNG判定率
- **出力ファイル構造**:
  - 全ての出力ファイルは `outputs/{主題名}/` ディレクトリ内に保存
  - ファイル名: `research_report.md`, `draft.txt`, `verification_report.md`, `final.txt`, `url_check_report.md`

## 安全性・方針
- **拒否基準**:
  - 犯罪予告、ヘイトスピーチ、爆発物製造等の違法コンテンツ
  - 特筆性が著しく低い（一般人）のプライバシー暴露
- **コンプライアンス**: Wikipedia「削除の方針」「即時削除の方針」をシステムプロンプトにコンテキストとして注入

## 評価
- **オフライン**: 既存の良質なWikipedia記事を再現できるかの回帰テスト
- **オンライン**: 生成された記事を人間がレビューし、修正に要した工数を計測（編集距離）

## 変更管理
- プロンプト変更時は `version` をインクリメントし、`tasks/` 内の YAML を更新