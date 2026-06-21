# 開発プロセスドキュメント — 構成ガイド

このリポジトリは **Claude Code + Superpowers + Serena** での個人開発において、
Claudeに開発プロセスを守らせるためのドキュメントテンプレートです。

---

## ドキュメント構成

```
プロジェクトルート/
│
├── CLAUDE.md                      # ★ Claude への行動指示（自動読み込み）
│
├── docs/
│   ├── OVERALL_PLAN.md            # 全体計画：承認済みロードマップ
│   ├── DRAFT_PLAN.md              # 未整理タスク候補：アイデア・改善要望の受け皿
│   ├── PROCESS.md                 # 開発プロセス全体の定義（7フェーズ）
│   ├── ARCHITECTURE.md            # アーキテクチャ・技術方針
│   ├── CODING_STANDARDS.md        # コーディング規約
│   └── DECISIONS.md               # 技術的意思決定ログ（ADR）
│
└── tasks/
    └── T-XXX_<機能名>.md          # タスク詳細：要件・設計・Todoリスト・テスト設計・テスト結果・フィードバック・マージ記録（永続保存）
```

---

## 各ファイルの役割

### CLAUDE.md（最重要）
Claude Code が会話開始時に**自動で読み込む**唯一のファイル。
ルール・禁止事項・フェーズチェックリストを記載することで Claudeの行動を制御する。

### docs/OVERALL_PLAN.md
承認済みのロードマップ。タスクの優先度・順序・スコープ・完了条件を定義する。
フィードバック後や要件変更時に更新する。

### docs/DRAFT_PLAN.md
未整理のタスク候補を気軽に追記する場所。
フィードバックで「次サイクルに回す」となった要望や思いつきを記録し、
整理後に OVERALL_PLAN.md のロードマップへ昇格させる。

### docs/PROCESS.md
7フェーズの開発プロセスを詳細に定義。
各フェーズで何をすべきか・承認ゲートの条件・完了条件を記載する。
一部フェーズでは Superpowers スキルを併用する（フェーズ3 実装: `test-driven-development` ／ フェーズ4 テスト: `systematic-debugging` ／ フェーズ7 次プロセス判断: `brainstorming`）。詳細・境界条件は同ファイルの「Superpowers スキルの併用」セクションを参照。
また、コードを伴うフェーズでは Serena MCP（LSP ベースのセマンティックなコード操作）を併用する（フェーズ2 設計のコード調査 ／ フェーズ3 実装 ／ フェーズ4 リグレッション）。詳細・境界条件・メモリ担当表は同ファイルの「Serena MCP の併用」セクションを参照。

### docs/ARCHITECTURE.md
技術スタック・ディレクトリ構成・設計原則を記載する。
Claudeは設計フェーズでこのファイルを参照し、方針に反する設計を避ける。新しいライブラリの採用・設計方針の変更がある場合は設計フェーズで更新し、実装後はフィードバックフェーズの最終判断後（フェーズ5 Step 6）にも必要に応じて更新する。

### docs/CODING_STANDARDS.md
命名規則・コメントルール・エラーハンドリング・テスト方針を記載する。
Claudeは実装フェーズでこのファイルを参照する。

### docs/DECISIONS.md
重要な技術的決定（Architecture Decision Record）を蓄積する。
「なぜそう決めたか」を後から追跡できる。

### tasks/T-XXX_<機能名>.md
タスクごとに作成する詳細記録ファイル（例: `T-001_ログイン機能.md`）。
要件・設計メモ・Todoリスト・テスト設計・テスト結果・フィードバック・マージ記録を全フェーズにわたって記録し、完了後も永続保存する。

---

## 運用フロー

```
1. OVERALL_PLAN.md のロードマップから次のタスクを選ぶ
2. Claude に「T-XXX のタスクを始めよう」と伝える
3. Claude が PROCESS.md に従って各フェーズを進める（T-XXX_<機能名>.md の新規作成を含む）
   要件定義 → 設計 → 実装 → テスト → フィードバック → マージ → 次プロセス判断
4. 完了後に OVERALL_PLAN.md の完了済み欄へ移動する
```

---

## Serena MCP のセットアップ

Serena は MCP サーバとして接続する。`uv`（Python パッケージランナー）経由で起動する。

1. `uv` を導入する（未導入の場合）。Python があれば `pip install uv` でよい。

   ```
   pip install uv
   ```

2. Serena をプロジェクトスコープで登録する（`.mcp.json` が生成される。初回 `uvx` は GitHub から Serena を取得するためネットワークが必要）。

   ```
   claude mcp add serena -s project -- uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant
   ```

3. 登録を確認する。`serena` が表示されれば登録完了（ツールが実際に使えるのは Claude Code のセッション再読込後）。

   ```
   claude mcp list
   ```

- `--context ide-assistant` は Claude Code 向けの推奨コンテキスト（編集・シェルは Claude Code 側が担当し、Serena はセマンティックなコード操作に集中する）。
- 併用の方針（どのフェーズで何に使うか・メモリの担当範囲）は `docs/PROCESS.md`「Serena MCP の併用」を参照。
- このリポジトリには `.mcp.json` を同梱しており、複製先ではそのまま接続設定のテンプレートとして使える。

---

## カスタマイズのポイント

| ファイル | カスタマイズ推奨箇所 |
|---|---|
| `CLAUDE.md` | プロジェクト概要・技術スタック・禁止事項 |
| `docs/OVERALL_PLAN.md` | プロジェクト目標・初期タスク定義 |
| `docs/ARCHITECTURE.md` | システム構成図・ディレクトリ構成・技術スタック表 |
| `docs/CODING_STANDARDS.md` | 命名規則・行数制限・テストルール |
| `docs/PROCESS.md` | 各フェーズのサマリーフォーマット |
