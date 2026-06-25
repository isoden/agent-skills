# agent-skills

testing library に関する agent skills を作成するリポジトリ。

## 外部コンテンツの引用・参照ポリシー（重要）

Kent C. Dodds 氏のブログ記事をはじめとする外部コンテンツを skill に取り込む際は、以下を厳守する。

### 前提：Kent C. Dodds 氏のコンテンツのライセンス状況
- サイトフッター: `All rights reserved © Kent C. Dodds`（全著作権保有）
- ブログのソースリポジトリ [kentcdodds/kentcdodds.com](https://github.com/kentcdodds/kentcdodds.com) の `LICENSE.md`:
  「This material is available for **private, non-commercial** use under the GPL version 3」。
  ワークショップ用途・商用利用は `me@kentcdodds.com` への許諾依頼が必要。
- → **Creative Commons のような自由再利用ライセンスではない。**

### 原則：表現は守られるが、アイデア・概念・方法論は守られない
著作権で保護されるのは「具体的な表現（文章そのもの）」であって、アイデア・概念・方法論は保護されない。この切り分けで判断する。

### ✅ やってよいこと
- **概念・原則を自分の言葉で説明する**（例: Testing Trophy、実装詳細ではなくユーザー振る舞いをテストする、`getByRole` 優先 等）。これは知識の再構成であり自由。
- **記事を出典としてリンク・参照する**（"Further reading" として URL を案内）。
- **短い一文を出典付きで引用する**（短文 + 著者名・出典 URL の明確な帰属。引用の要件を満たすこと）。

### ⚠️ やってはいけないこと
- 記事本文を段落単位でコピペして skill に埋め込む（複製にあたる。non-commercial 限定 & All rights reserved なので侵害リスク）。
- 記事を翻訳して丸ごと載せる（二次的著作物の作成にあたる）。
- 記事のフルテキストを RAG 用途等で同梱する（必要なら `me@kentcdodds.com` に許諾を取る）。

### skill の推奨構成
```
skill 本体  : 原則・ベストプラクティスを「自分の言葉」で記述（コピペしない）
References  : 出典記事を "Further reading" リンクとして列挙
引用       : 原文が必要な箇所のみ、短文 + 帰属付きで最小限
```
