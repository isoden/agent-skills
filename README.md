# agent-skills

Testing Library 周辺の知識を AI エージェントに持たせるための **agent skills** を作成し、保守するリポジトリです。

## 配布スキル

### `testing-library-philosophy`

React Testing Library / user-event / jest-dom の思想とベストプラクティス、および Kent C. Dodds 氏のテストに関する考え方を、エージェントが実践できる形にまとめたスキルです。
設計の核は「判断は prose、決定論は linter」という役割分担にあります。

- このスキルが扱うこと（人間やエージェントの判断が要る領域）
  - 何を、どの層でテストするか（Testing Trophy、use-case coverage）
  - ユーザーの探し方に沿った query 選択（role → label → text、`data-testid` は最終手段）
  - 実ユーザー操作の再現（user-event）と意図が伝わる assertion（jest-dom）
  - 実装詳細への結合を避ける（false negative / false positive の回避）
  - モックのトレードオフ（MSW でネットワーク境界をモック）
  - テストの保守性（AHA = Avoid Hasty Abstraction、custom render、fewer-longer tests）
- このスキルが扱わないこと（決定論的かつ機械的に検出できるルール）
  - `await` 漏れ、`screen` 使用、`findBy` vs `waitFor`、`no-container` など
  - これらは [`eslint-plugin-testing-library`](https://github.com/testing-library/eslint-plugin-testing-library) / [`eslint-plugin-jest-dom`](https://github.com/testing-library/eslint-plugin-jest-dom) に委譲します。

#### 構成

```
skills/testing-library-philosophy/
├── SKILL.md                                  中核原則 + Non-negotiables + 各 reference への導線
└── references/
    ├── guiding-principles.md                 confidence / Testing Trophy / 何をテストするか
    ├── queries.md                            アクセシビリティ優先の query 選択
    ├── user-event.md                         実ユーザー操作の再現
    ├── jest-dom.md                           意図が伝わる assertion + eslint-plugin-jest-dom
    ├── avoiding-implementation-details.md     実装詳細への結合を避ける
    ├── common-mistakes.md                    判断レベルの落とし穴
    ├── test-maintainability.md               AHA / custom render / fewer-longer tests
    ├── mocking.md                            モックのトレードオフ / MSW
    ├── eslint-plugin.md                      決定論的チェックの linter への委譲
    └── further-reading.md                    出典リンク集
```

### `skill-ab-eval`（プロジェクト専用ツール）

スキルが実際にエージェントの挙動を変えるかを A/B で検証するツールです（`.claude/skills/skill-ab-eval`）。

- スキルなしの control 群と、スキルありの treatment 群に同一タスクを解かせる
- 出力を群を伏せてブラインド採点し、control と treatment を比較する
- スキルの価値は多くの場合「平均の底上げ」ではなく「悪い出力に転落する確率（regression）の低下」に出るため、回帰率を重視する

Claude Code 上で `/skill-ab-eval`、または「このスキルの有用性をチェックして」と依頼すると起動します。

## ライセンス

このリポジトリは [MIT License](./LICENSE)）で公開しています。
スキルの本文は外部記事の転載を含みません。
外部コンテンツの引用と参照のポリシーは [`CLAUDE.md`](./CLAUDE.md) を参照してください。

## インストール

[Agent Skills 仕様](https://agentskills.io/specification) の `skills/*/SKILL.md` 規約に準拠しているため、GitHub CLI（`gh skill install`）からそのままインストールできます。

```bash
# リポジトリから対象スキルをインストール
gh skill install isoden/agent-skills testing-library-philosophy

# 対象エージェント / スコープを指定する例
gh skill install isoden/agent-skills testing-library-philosophy --agent claude-code --scope project
```

## リポジトリ構成

```
agent-skills/
├── skills/                  ← `gh skill install` が自動検出（＝配布対象）
│   └── testing-library-philosophy/
└── .claude/skills/          ← 検出対象外（＝配布されないローカルツール）
    └── skill-ab-eval/
```

スキルを編集したら `skill-ab-eval` で効果を確認するワークフローを推奨します。
