<div align="center">

# AutoHarness Skill

**Language / 言語:**　[🇯🇵 日本語](#-日本語) | [🇺🇸 English](#-english)

</div>

---

## 🇯🇵 日本語

> 🔧 **Claude Code** のためのプロジェクト固有コード検証ハーネス自動生成スキル

### 概要

**AutoHarness** は、[DeepMind の AutoHarness 論文（arXiv:2603.03329）](https://arxiv.org/abs/2603.03329v1) のアイデアを Claude Code のコーディングタスクに応用したスキルです。

プロジェクトを自動解析し、「このプロジェクトで有効なコード変更とは何か」を定義する **自然言語ルール** と **実行可能な検証スクリプト** の両方を生成・維持します。これにより、Claude Code が生成するコードの精度を継続的に向上させます。

### 特徴

| 機能 | 説明 |
|------|------|
| 🔍 **プロジェクト自動解析** | 言語・フレームワーク・Lint・型チェック・テストコマンドを自動検出 |
| 📝 **ルール自動生成** | `.claude/rules/harness.md` に命名規則・禁止パターン・検証条件を生成 |
| 🐍 **検証スクリプト生成** | `harness_check.py` で型チェック・Lint・テストを一括実行、JSON 形式で結果出力 |
| 🔄 **継続的な改善** | エラーやフィードバックをもとにハーネスを自律的に更新 |

### インストール

**前提条件**

- [Claude Code](https://claude.ai/code) がインストールされていること
- Python 3.10 以上

**セットアップ**

```bash
claude skills add https://github.com/shintaro-amaike/autoharness-skill
```

### 使い方

#### 初回セットアップ

Claude Code のチャットで以下のコマンドを入力します：

```
/autoharness-init
```

AutoHarness がプロジェクトを解析し、以下のファイルを自動生成します：

```
your-project/
├── .claude/
│   └── rules/
│       ├── harness.md          # 自然言語ルール
│       └── harness_check.py    # 検証スクリプト
└── CLAUDE.md                   # harness.md への参照を自動追記
```

#### ハーネスの更新

```
/autoharness-update
```

以下のタイミングで実行します（Claude Code が自律的に実行する場合もあります）：

- 型エラー・テスト失敗・Lint エラーが発生したとき
- ユーザーが「違う」「こうしてほしかった」などフィードバックをしたとき
- コード生成タスクが一段落したとき

#### 検証スクリプトの実行

```bash
# 特定のファイルを検証
python .claude/rules/harness_check.py src/main.py src/utils.py

# 使い方を確認
python .claude/rules/harness_check.py --help
```

出力例：

```json
{
  "passed": true,
  "results": {
    "typecheck": { "passed": true, "errors": [] },
    "lint":      { "passed": true, "errors": [] },
    "test":      { "passed": true, "errors": [] }
  }
}
```

### 仕組み

| 論文の概念 | このスキルでの対応 |
|---|---|
| `is_legal_action()` | `harness_check.py` の各チェック |
| `propose_action()` | Claude Code のコード生成 |
| Refinement loop | `/autoharness-update` の反復呼び出し |
| Environment feedback | エラーメッセージ・テスト結果 |

### 参考文献

- [AutoHarness: arXiv:2603.03329](https://arxiv.org/abs/2603.03329v1)
- [Claude Code Documentation](https://docs.claude.ai/code)

<div align="right">

[⬆ トップに戻る](#autoharness-skill)

</div>

---

## 🇺🇸 English

> 🔧 Auto-generate project-specific code validation harnesses for **Claude Code**

### Overview

**AutoHarness** is a Claude Code skill that applies ideas from [DeepMind's AutoHarness paper (arXiv:2603.03329)](https://arxiv.org/abs/2603.03329v1) to coding tasks.

It automatically analyzes your project and generates both **natural language rules** and **executable validation scripts** that define what constitutes a valid code change — continuously improving the accuracy of Claude Code's output.

### Features

| Feature | Description |
|---------|-------------|
| 🔍 **Auto Project Analysis** | Detects language, framework, lint, type checker, and test commands automatically |
| 📝 **Rule Generation** | Generates naming conventions, forbidden patterns, and validation conditions in `.claude/rules/harness.md` |
| 🐍 **Validation Script** | `harness_check.py` runs type checks, lint, and tests in one command with JSON output |
| 🔄 **Continuous Improvement** | Autonomously updates the harness based on errors and user feedback |

### Installation

**Prerequisites**

- [Claude Code](https://claude.ai/code) installed
- Python 3.10 or higher

**Setup**

```bash
claude skills add https://github.com/shintaro-amaike/autoharness-skill
```

### Usage

#### Initial Setup

Type the following command in the Claude Code chat:

```
/autoharness-init
```

AutoHarness analyzes your project and generates the following files:

```
your-project/
├── .claude/
│   └── rules/
│       ├── harness.md          # Natural language rules
│       └── harness_check.py    # Validation script
└── CLAUDE.md                   # Auto-updated with reference to harness.md
```

#### Updating the Harness

```
/autoharness-update
```

Run at any of the following moments (Claude Code also runs this autonomously):

- When type errors, test failures, or lint errors occur
- When the user gives feedback like "that's wrong" or "I wanted it this way"
- When a code generation task is complete

#### Running the Validation Script

```bash
# Validate specific files
python .claude/rules/harness_check.py src/main.py src/utils.py

# Show help
python .claude/rules/harness_check.py --help
```

Example output:

```json
{
  "passed": true,
  "results": {
    "typecheck": { "passed": true, "errors": [] },
    "lint":      { "passed": true, "errors": [] },
    "test":      { "passed": true, "errors": [] }
  }
}
```

### How It Works

| Paper Concept | Skill Equivalent |
|---|---|
| `is_legal_action()` | Each check in `harness_check.py` |
| `propose_action()` | Claude Code's code generation |
| Refinement loop | Iterative calls to `/autoharness-update` |
| Environment feedback | Error messages and test results |

### References

- [AutoHarness: arXiv:2603.03329](https://arxiv.org/abs/2603.03329v1)
- [Claude Code Documentation](https://docs.claude.ai/code)

<div align="right">

[⬆ Back to top](#autoharness-skill)

</div>

---

## License

MIT License
