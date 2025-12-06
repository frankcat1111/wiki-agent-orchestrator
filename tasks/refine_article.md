task: refine_article
version: 1
owner: Content-Team
priority: P0
entry:
  goal: "ファクトチェック結果に基づき、記事ドラフトを修正・完成させる"
  inputs:
    - name: mediawiki_draft
      type: text
      required: true
    - name: verification_report
      type: markdown
      required: true
    - name: subject_name
      type: string
      required: true
      description: "記事の主題名（出力ファイルパス生成に使用）"
  outputs:
    - name: final_article
      type: text
      file_path: "outputs/{subject_name}/final.txt"
      acceptance:
        - "指摘された虚偽情報が修正または削除されている"
        - "無効なURLが除去または代替URLに差し替えられている"
        - "MediaWiki構文が維持されている"
  constraints:
    - "検証レポート自体を出典として引用しないこと"
tools:
  - name: web_search
    intent: search
    description: "修正に必要な追加情報の取得（必要な場合のみ）"
file_output:
  instruction: "最終記事は outputs/{subject_name}/final.txt に保存すること。{subject_name} フォルダが存在しない場合は自動作成する。"
prompts:
  system: |
    あなたはWikipedia記事の最終編集者（Refinement Agent）です。
    提供された「記事ドラフト」と「ファクトチェック結果」に基づき、記事を修正してください。

    # ルール
    1. ファクトチェックで「虚偽」「不正確」と判定された箇所を修正、または記述を削除してください。
    2. ファクトチェックで「リンク切れ」「信頼性低」と判定されたURLを削除、または代替URLに差し替えてください。
    3. 検証レポート自体（「ファクトチェックの結果によると〜」等の記述）を記事内に含めないでください。
    4. 出力は完全な MediaWiki 構文のみで行い、Markdownを含めないでください。

    # プロセス
    1. 検証結果の理解: 修正が必要な箇所を特定する。
    2. 記事の修正: 事実に基づき記述を改める。
    3. 最終出力: 修正済みの記事全文を出力する。
  user_template: |
    Draft Article:
    {{mediawiki_draft}}

    Fact Check Report:
    {{verification_report}}

    Please output the corrected final article.