---
name: check_urls
description: 記事ドラフト内の全URLの有効性を確認する。
agent: UrlCheckerAgent
input:
  - outputs/{subject_name}/draft.txt
output:
  - outputs/{subject_name}/url_check/report.md
dependencies:
  - draft_article
---

# URL確認タスク

## 目的
記事ドラフトで引用されているすべてのURLが有効であり、アクセス可能（ステータスコード200 OKを返す）であることを検証します。

## 手順
1.  **URL抽出**: `draft.txt` ファイルを解析し、`<ref>` タグおよび外部リンクセクションで使用されているすべてのURLを抽出します。
2.  **URL検証**: 各URLに対してHEADまたはGETリクエストを実行し、ステータスコードを確認します。
    *   リダイレクトを追跡します。
    *   404 Not Found、403 Forbidden、5xx Server Errors、または接続タイムアウトを特定します。
3.  **レポート作成**: チェックしたすべてのURLとそのステータスをリストしたレポート `report.md` を `outputs/{subject_name}/url_check/` ディレクトリに生成します。
    *   URLがリンク切れの場合は、代替URLを提案するか、手動確認のためのマークを付けます。
    *   すべてのURLが有効な場合は、「すべてのURLが有効です」と記述します。

## 出力形式
出力は以下の構造を持つMarkdownファイル（`outputs/{subject_name}/url_check/report.md`）である必要があります：

```markdown
# URL検証レポート: {主題名}

## サマリー
- URL総数: {数}
- 有効なURL: {数}
- リンク切れURL: {数}

## 詳細
| URL | ステータス | 必要なアクション |
|---|---|---|
| https://example.com/article | 200 OK | なし |
| https://example.com/broken | 404 Not Found | 代替を探すか削除 |
```
