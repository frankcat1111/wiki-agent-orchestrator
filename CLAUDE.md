# CLAUDE.md - Wikipedia Article Generation Orchestrator

## 概要

**目的**: 指定された主題（人物・企業）に関するWikipedia記事を、公式方針（NPOV、検証可能性、独自研究の禁止）に準拠した状態で自動生成する。

**成功指標**:
- MediaWiki構文エラー率 0%
- 参照リンク（出典）の有効性 100%
- Wikipedia削除方針（宣伝・特筆性不足）への抵触回避

**非目標**: Wikipediaへの自動投稿（あくまでドラフト生成までをスコープとする）

**制約**:
- 存命人物の伝記（BLP）における厳格な出典要求
- Web検索APIのレート制限およびトークン長制限

---

## Agent Orchestration

**Must**: All tasks must pass through the orchestrator workflow before execution.

### Core Principle

**Do NOT use pre-defined abstract agents.** Instead:
1. Create specialized agents from actual task requirements
2. Integrate existing agents when synergy improves outcomes
3. Let the agent pool evolve through continuous improvement

---

## ディレクトリ構造

```
.
├── .claude/agents/                    # エージェント管理
│   ├── orchestrator.md                # オーケストレーター定義
│   ├── _template.md                   # 新規エージェントテンプレート
│   ├── manifests/                     # スキルシート（メタデータ + メトリクス）
│   │   └── {agent-name}.yaml
│   └── pool/                          # エージェントプール
│       ├── specialized/               # タスク特化エージェント
│       ├── integrated/                # 統合エージェント
│       └── elite/                     # ハイパーエリートエージェント
│
├── knowledge/                         # Wikipedia方針ナレッジベース
│   ├── Wikipedia削除の方針.md
│   └── Wikipedia即時削除の方針.md
│
├── tasks/                             # タスク定義
│   ├── research_subject.md
│   ├── draft_article.md
│   ├── verify_facts.md
│   ├── refine_article.md
│   └── check_urls.md
│
└── outputs/{主題名}/                  # 生成物出力
    ├── research_report.md             # 調査レポート
    ├── draft.txt                      # 記事ドラフト
    ├── verification_report.md         # 検証レポート
    ├── final.txt                      # 最終記事
    └── url_check_report.md            # URL検証レポート
```

---

## Wikipedia Article Generation Pipeline

### 専門エージェント一覧

| Agent | 役割 | 主要タスク | 入力 | 出力 | 所有者 |
|---|---|---|---|---|---|
| **Researcher** | 情報収集・構造化 | `research_subject` | 主題名、基礎情報 | 構造化調査レポート | Content |
| **Drafter** | 記事執筆（MediaWiki） | `draft_article` | 調査レポート | 記事ドラフト | Engineering |
| **Auditor** | ファクトチェック | `verify_facts` | 記事ドラフト | 検証レポート | QA |
| **Editor** | 修正・最終化 | `refine_article` | ドラフト + 検証レポート | 最終記事 | Content |
| **UrlChecker** | URL生存確認 | `check_urls` | 最終記事 | URL検証レポート | QA |

### 実行フロー

```
タスク受信: 主題名（人物/企業）
     ↓
┌─────────────────────────────────────────────────────┐
│ Step 1: Researcher                                  │
│   - Web検索による情報収集                            │
│   - 出典の信頼性評価                                 │
│   - 構造化調査レポート作成                           │
│   → outputs/{主題名}/research_report.md             │
└─────────────────────────────────────────────────────┘
     ↓
┌─────────────────────────────────────────────────────┐
│ Step 2: Drafter                                     │
│   - MediaWiki形式での記事執筆                        │
│   - Wikipedia方針準拠チェック                        │
│   - 出典の適切な配置                                 │
│   → outputs/{主題名}/draft.txt                      │
└─────────────────────────────────────────────────────┘
     ↓
┌─────────────────────────────────────────────────────┐
│ Step 3: Auditor                                     │
│   - 事実関係の検証                                   │
│   - 出典との整合性確認                               │
│   - ハルシネーション検出                             │
│   → outputs/{主題名}/verification_report.md         │
└─────────────────────────────────────────────────────┘
     ↓
┌─────────────────────────────────────────────────────┐
│ Step 4: Editor                                      │
│   - 検証結果に基づく修正                             │
│   - 文体・構成の最適化                               │
│   - 最終品質確認                                     │
│   → outputs/{主題名}/final.txt                      │
└─────────────────────────────────────────────────────┘
     ↓
┌─────────────────────────────────────────────────────┐
│ Step 5: UrlChecker                                  │
│   - 全参照URLの生存確認                              │
│   - HTTPステータスコード検証                         │
│   - リンク切れレポート作成                           │
│   → outputs/{主題名}/url_check_report.md            │
└─────────────────────────────────────────────────────┘
```

### フェイルオーバー

- `Auditor` が重大な虚偽（Critical Hallucination）を検出した場合 → `Researcher` へ戻し再調査（Human Loop 推奨）
- `WebSearch` タイムアウト時 → 既知のナレッジベースのみで構成せず、欠損箇所を明記して終了

---

## General Agent Orchestration

### Workflow

```
Task Received
     ↓
Scan pool/ for existing agents
     ↓
Calculate coverage rate against task requirements
     ↓
┌─────────────────────────────────────────────────────┐
│ Coverage 90%+  → Use existing agent                 │
│ Coverage 60-90% → Create integrated agent           │
│ Coverage <60%  → Create new specialized agent       │
└─────────────────────────────────────────────────────┘
     ↓
Execute task with selected/created agent
     ↓
Update manifests/ with metrics
     ↓
Promote to elite/ if qualified
```

### Decision Matrix

| Coverage Rate | Action | Save Location |
|---------------|--------|---------------|
| **90%+** | Use existing agent directly | - |
| **60-90%** | Merge source agents into integrated agent | `pool/integrated/` |
| **Below 60%** | Create new specialized agent | `pool/specialized/` |

### Agent Creation Rules

When creating a new agent:

1. **Copy `_template.md`** structure
2. **Define in YAML frontmatter**:
   ```yaml
   ---
   name: task-domain-specialist
   description: One-line description
   tools: Read, Grep, Glob, Edit, Write, Bash
   model: opus
   ---
   ```
3. **Write detailed system prompt** in body
4. **Create skill sheet** in `manifests/{agent-name}.yaml`
5. **Save agent** to appropriate `pool/` subdirectory

### Integration Rules

When merging agents:

1. **Identify source agents** with partial coverage
2. **Combine capabilities**, eliminate redundancy
3. **Resolve conflicts** between source agents
4. **Record lineage** in skill sheet `parent_agents` field
5. **Save to `pool/integrated/merged-{source1}-{source2}.md`**

---

## ランタイム構成

### 接続ツール

| ツール | 用途 | 使用エージェント |
|--------|------|------------------|
| `WebSearch` | リアルタイム情報取得、URL生存確認 | Researcher, UrlChecker |
| `ContentScraper` | 参照先コンテンツの本文抽出 | Researcher, Auditor |
| `UrlChecker` | HTTPステータスコード確認 | UrlChecker |

### レート制限
- 各エージェントあたり 60 RPM（WebSearch依存）

### 機密情報の扱い
- 出力に含まれる個人情報は公開情報（著名人等）に限る
- 非公開情報は `SafetyFilter` で除外

---

## メトリクスと評価

### Wikipedia Article Quality Metrics

| メトリクス | 説明 | 目標値 |
|------------|------|--------|
| `citation_quality_score` | (学術・大手メディア出典数 / 全出典数) | ≥ 0.7 |
| `hallucination_rate` | ファクトチェックでのNG判定率 | ≤ 0.1 |
| `syntax_errors` | Markdown混入などの構文エラー数 | 0 |

### Agent Evolution Metrics

After every task execution, update the agent's skill sheet:

```yaml
metrics:
  usage_count: 5        # Increment
  success_rate: 0.85    # (successes / usage_count)
  last_used: 2025-01-15 # Current date
```

### Elite Promotion

An agent qualifies for elite status when:
- `usage_count >= 5`
- `success_rate >= 0.8`

Action: Move from `specialized/` or `integrated/` to `elite/`

### ログ出力

| ログ名 | 内容 |
|--------|------|
| `research_sources` | 採用した出典URLとその信頼性スコア |
| `syntax_errors` | Markdown混入などの構文エラー数 |
| `verification_result` | ファクトチェックでの指摘項目数 |

---

## 安全性・方針

### 拒否基準

- 犯罪予告、ヘイトスピーチ、爆発物製造等の違法コンテンツ
- 特筆性が著しく低い（一般人）のプライバシー暴露

### コンプライアンス

- Wikipedia「削除の方針」「即時削除の方針」をシステムプロンプトにコンテキストとして注入
- 参照: `knowledge/` ディレクトリ内のポリシー文書

---

## 評価手法

| 評価種別 | 方法 |
|----------|------|
| **オフライン** | 既存の良質なWikipedia記事を再現できるかの回帰テスト |
| **オンライン** | 生成された記事を人間がレビューし、修正に要した工数を計測（編集距離） |

---

## Quick Reference

| Situation | Action |
|-----------|--------|
| Wikipedia記事作成依頼 | Pipeline実行（Researcher → UrlChecker） |
| 新規タスク受信 | `pool/` スキャン、カバレッジ計算 |
| Perfect match exists | Use existing agent |
| Partial matches | Create integrated agent |
| No good match | Create specialized agent |
| Task completed | Update `manifests/` metrics |
| High-performing agent | Promote to `elite/` |
| Critical Hallucination検出 | Researcherへ戻し再調査 |

---

## 変更管理

- プロンプト変更時は `version` をインクリメントし、`tasks/` 内の定義を更新
- エージェント統合時は `parent_agents` フィールドに継承元を記録

---

## Reference Files

| ファイル | 説明 |
|----------|------|
| `.claude/agents/orchestrator.md` | オーケストレーター完全定義 |
| `.claude/agents/_template.md` | エージェント定義テンプレート |
| `.claude/agents/manifests/_template.yaml` | スキルシートテンプレート |
| `tasks/*.md` | 各タスクの詳細定義 |
| `knowledge/*.md` | Wikipedia方針ドキュメント |
