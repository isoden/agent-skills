# agent-skills

Testing Library に関する agent skills を作成し、保守するリポジトリ。

## リポジトリ構成

```
skills/                         配布対象の agent skills
  testing-library-philosophy/   RTL の思想とベストプラクティス
    SKILL.md
    references/*.md
.claude/skills/                 このプロジェクト専用のツール
  skill-ab-eval/                スキルの有用性を A/B で検証するツール
README.md                       プロジェクト概要（人間向け）
CLAUDE.md                       このファイル（Claude 向けの作業指針）
```

- `skills/` 配下は**外部配布される汎用スキル**。プロジェクト固有の情報は入れない。
- `.claude/skills/` 配下は**このリポジトリの開発を助けるローカルツール**。配布しない。

## スキル作成の方針

### 役割分担：判断は prose、決定論は linter

- スキルでは**人間/エージェントの判断が必要なこと**（何をテストするか、どの query が意味的に妥当か等）を扱う。
- **機械的に検出できる決定論的ルール**（await 漏れ、`screen` 使用、`findBy` vs `waitFor` 等）は `eslint-plugin-testing-library` / `eslint-plugin-jest-dom` に委譲し、prose で再教育しない。
- 最重要の判断（linter の死角）は、参照ファイルに埋めず **SKILL.md 本体の先頭付近**に「Non-negotiables」として昇格し、発火率を上げる。

### スキルの推奨構成

```
SKILL.md    : 中核原則 + Non-negotiables + 各 reference への導線
references/ : トピック別に分割。各ファイル末尾に "Further reading" の出典リンク
引用        : 原文が必要な箇所のみ、短文 + 帰属付きで最小限
```

### 言語

スキルの記述は**英語**（配布対象のため）。このリポジトリのドキュメント（README / CLAUDE）は日本語。

## 外部コンテンツの引用と参照のポリシー（重要）

Kent C. Dodds 氏のブログ記事をはじめとする外部コンテンツを skill に取り込む際は、以下を厳守する。

### 前提：Kent C. Dodds 氏のコンテンツのライセンス状況

- サイトフッター: `All rights reserved © Kent C. Dodds`（全著作権保有）
- ブログのソースリポジトリ [kentcdodds/kentcdodds.com](https://github.com/kentcdodds/kentcdodds.com) の `LICENSE.md`:
  「This material is available for **private, non-commercial** use under the GPL version 3」。
  ワークショップ用途や商用利用は `me@kentcdodds.com` への許諾依頼が必要。
- → **Creative Commons のような自由再利用ライセンスではない。**

### 原則：表現は守られるが、アイデアや概念、方法論は守られない

著作権で保護されるのは「具体的な表現（文章そのもの）」であって、アイデアや概念、方法論は保護されない。この切り分けで判断する。

### ✅ やってよいこと

- **概念や原則を自分の言葉で説明する**（例: Testing Trophy、実装詳細ではなくユーザー振る舞いをテストする、`getByRole` 優先 など）。これは知識の再構成であり自由。
- **記事を出典としてリンク・参照する**（"Further reading" として URL を案内）。
- **短い一文を出典付きで引用する**（短文 + 著者名と出典 URL の明確な帰属。引用の要件を満たすこと）。

### ⚠️ やってはいけないこと

- 記事本文を段落単位でコピペして skill に埋め込む（複製にあたる。non-commercial 限定 & All rights reserved なので侵害リスク）。
- 記事を翻訳して丸ごと載せる（二次的著作物の作成にあたる）。
- 記事のフルテキストを RAG 用途等で同梱する（必要なら `me@kentcdodds.com` に許諾を取る）。

## スキルの検証

スキルを新規に作成または編集したら、`/skill-ab-eval`（`.claude/skills/skill-ab-eval`）で**有用性を A/B 検証**できる。スキルあり/なしで同じタスクをエージェントに解かせ、ブラインド採点して control vs treatment を比較する。スキルの価値は多くの場合「平均の底上げ」ではなく「悪い出力に転落する確率（regression）の低下」に出るため、回帰率に注目する。
