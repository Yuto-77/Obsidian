

## 要件定義

- プロジェクト概要
	- プロジェクト名：AI Review Bot
	- 開発目的
		- Git/GitHub運用の練習を兼ねて、自動レビューの仕組みを構築する
		- Google AI Studioを活用して、PR内容を自動で要約、評価、記録する
		- 学習進捗をGoogleSheetsで可視化する
	- 開発目標
		- どのGitHubリポジトリでも簡単に導入できる共通レビューBot
		- 機能追加・拡張が容易なモジュール設計（AIモデル変更・通知・分析機能などを追加可能）

## 基本設計

### システム構成概要
GitHub PR作成
   │
   ▼
(1) Webhook → GAS doPost()
   │
   ▼
(2) GitHub API → PR詳細とdiff取得
   │
   ▼
(3) Gemini API → レビュー文生成
   │
   ▼
(4) GitHub API → コメント投稿
   │
   ▼
(5) Sheets API → データ記録



  

  

## 機能要件

### 必須機能
  
| 機能ID | 機能名      | 概要                              | 使用技術           |
| ---- | -------- | ------------------------------- | -------------- |
| F1   | PRイベント検知 | GitHub Webhookを利用してPR作成時にGASを起動 | GitHub Webhook |
|F2|PR情報取得|PRのタイトル・本文・変更内容(diff)を取得|GitHub REST API|
|F3|AIレビュー生成|PR情報をAIに渡してレビュー文を生成|Google AI Studio API|
|F4|コメント投稿|生成されたレビューをGitHubのPRに自動コメント|GitHub REST API|
|F5|進捗記録|PR情報とAIレビューの要約をGoogle Sheetsに保存|Google Sheets API|
 
### 拡張機能

| **機能ID** | **機能名**   | **概要**                            |
| -------- | --------- | --------------------------------- |
| E1       | タスク提示     | タスク一覧をSheets                      |
| E2       | Discord通知 | レビュー完了後にDiscordに送信                |
| E3       | AIモデル切替   | レビュー精度・トーンを切り替え（例：厳しめ、優しめ）        |
| E4       | 進捗分析      | Sheetsのデータをもとに週/月単位のレポート自動生成      |
| E5       | 設定UI      | Google Workspace Add-onで設定管理をGUI化 |

  

### 非機能要件

|項目|要件|
|---|---|
|拡張性|各処理を独立した関数・モジュールとして設計（差し替え容易）|
|保守性|コードを`github.gs` / `ai_review.gs` / `sheets.gs` に分割|
|セキュリティ|トークンはScript Propertiesに安全に保存|
|信頼性|例外処理・ログ出力を全API呼び出しに実装|
|汎用性|任意のGitHubリポジトリからのWebhook呼び出しに対応|
|実行制限|GAS実行時間（6分以内）に収まる処理フローを設計|

### データ要件
#### スプレッドシート出力項目
| 項目名     | 型      | 説明              |
| ------- | ------ | --------------- |
| 日付      | Date   | レビュー日時          |
| タイトル    | String | PRのタイトル         |
| 本文要約    | String | PR本文またはAI要約     |
| AIコメント  | String | AIが生成したレビュー文    |
| 進捗スコア   | Number | 到達度を0〜100で記録    |
| コメントURL | String | GitHub上のコメントリンク |


### 外部インターフェース要件

|項目|対象|使用技術|備考|
|---|---|---|---|
|API1|GitHub REST API|認証：Personal Access Token|PR取得・コメント投稿|
|API2|Google AI Studio API|API Key|レビュー生成|
|API3|Google Sheets API|OAuth認証|記録・読み取り|
|API4|GitHub Webhook|Secret Token|PR作成・更新トリガー|
### 運用・拡張設計ポリシー

|項目|方針|
|---|---|
|機能追加|各機能は個別の`.gs`ファイルに定義し、メイン処理から呼び出す|
|環境変数|`ScriptProperties` に格納（トークン、シートID、AIキーなど）|
|テスト|各関数を独立テスト可能に（例：`testGithubConnection()`）|
|ログ|`Logger.log` で実行ログを出力、必要に応じてSheetsにも保存|
|デプロイ|Webアプリとしてデプロイ（Webhook URL固定）|
### 成果物（Deliverables）

- Google Apps Script プロジェクト（共通AIレビューBot）
    
- Google Sheets（進捗・記録管理）
    
- GitHub Webhook設定手順書
    
- 将来的にREADME.md（導入ガイド兼ドキュメント）
  
  