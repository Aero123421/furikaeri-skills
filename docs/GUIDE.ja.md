<p align="center">
  <a href="GUIDE.md">English</a> · <b>日本語</b>
</p>

# furikaeri ガイド

[README](../README.ja.md) に書ききれなかった、くわしい説明です。必要なところだけ読んでください。

- [全体の流れ](#全体の流れ)
- [ほかのインストール方法](#ほかのインストール方法)
- [こんなふうに頼めます](#こんなふうに頼めます)
- [furikaeri-harnessが見るもの](#furikaeri-harnessが見るもの)
- [AIはアーカイブをどう扱うか](#aiはアーカイブをどう扱うか)
- [アーカイブの中身と設定](#アーカイブの中身と設定)
- [レポートの例](#レポートの例)
- [大事にしていること](#大事にしていること)
- [よくある質問](#よくある質問)
- [リポジトリの構成](#リポジトリの構成)

## 全体の流れ

チャットやコミットを読んで、レポートをアーカイブに保存します。AIは、そのアーカイブを読み返します。

<p align="center">
  <img src="images/flow-ja.svg" alt="Claudeの会話、ChatGPTの会話、エージェントのログ、Gitの履歴がfurikaeri-projectに集まり、Gitのアーカイブに保存される。furikaeri-recallが作業中のAIと次の振り返りのためにそれを読み返す" width="100%">
</p>

- **アーカイブが記憶になります。** 各プロジェクトに `INDEX.md` があり、現状、決定ログ（決めたことの一覧）、プロジェクトの呼び名（別名・リポジトリ・キーワード。「照合ヒント」と呼びます）が書いてあります。振り返りはまずここを読むので、新しい決定なのか、前の決定を覆したのかを見分けられます。`furikaeri-recall` が、ふだんの作業にもこれを活かします。
- **アーカイブが自分の説明書を持っています。** アーカイブの一番上にある `FURIKAERI.md` に、設定・ファイルの置き場所・レポートのテンプレート・プライバシーのルールが書いてあります。スキルを入れていないAI（たとえばChatGPTの定期タスク）でも、これを読めば同じ形式で書けます。チームで形式を変えたいときは、このファイル1つを編集するだけです。
- **各サービスは自分の会話を振り返ります。** ClaudeはChatGPTの履歴を読めませんし、その逆も同じです。だから、それぞれが同じプロジェクトに自分のレポートを置き、並べて残します。
- **AIが読むのは少しだけです。** レポートはGitにたまります。AIが読むのは、索引と、その作業に必要な数本のレポートだけです。アーカイブ全体は読みません。
- **記録は一部だけ、と分かっています。** [AIはアーカイブをどう扱うか](#aiはアーカイブをどう扱うか)を見てください。

## ほかのインストール方法

コマンド1つのインストールは [README](../README.ja.md#はじめかた) にあります。[`skills` CLI](https://github.com/vercel-labs/skills) は、どのAIツール（Claude Code・Codex・OpenCode・Cursor など）に入れるかを聞いてきます。最初から指定するなら、そのコマンドに `-a claude-code -a codex` を付けてください。

### フォルダをコピーする

```bash
git clone https://github.com/Aero123421/furikaeri-skills.git
cd furikaeri-skills

# Claude Code（OpenCodeもこのフォルダを読みます）
mkdir -p ~/.claude/skills && cp -r skills/* ~/.claude/skills/

# Codex・Pi・OpenCode
mkdir -p ~/.agents/skills && cp -r skills/* ~/.agents/skills/
```

| AIツール | スキルの置き場所 | 呼び出し方 |
|---|---|---|
| Claude Code | `~/.claude/skills/` または `.claude/skills/` | `/furikaeri-project` |
| Codex | `~/.agents/skills/` または `.agents/skills/` | `$furikaeri-project` |
| Pi | `~/.agents/skills/` または `~/.pi/agent/skills/` | `/skill:furikaeri-project` |
| OpenCode | `~/.agents/skills/`・`~/.claude/skills/`・`~/.config/opencode/skills/` | 普通に頼めばスキルを名前で読み込みます |

ふつうの言葉で頼んでも、たいてい動きます。AIが説明文を見て、合うスキルを選ぶからです。反応しないときは、右の列の形で名前を指定してください。

### ClaudeアプリとChatGPT

スキルのフォルダをそれぞれzipにして、カスタムスキルとしてアップロードします。

```bash
cd skills && for s in furikaeri-*; do zip -r "../$s.zip" "$s"; done
```

- **Claude（claude.ai・デスクトップ・モバイル）**：Skillsの設定からzipをアップロードします。
- **ChatGPT**：プランがSkillsに対応していればアップロードします。対応していなければインストールは不要です。[定期実行プロンプト](../prompts/scheduled-daily.ja.md)とアーカイブの `FURIKAERI.md` だけで、やり方がすべて伝わります。

> [!TIP]
> `furikaeri-project`・`furikaeri-archive`・`furikaeri-harness` は、頼んだときだけ動く想定です。Claude Codeでは、これらの `SKILL.md` の先頭にある設定欄（frontmatter）に `disable-model-invocation: true` を足すと、それを強制できます（Claude Code独自の拡張で、標準仕様ではありません）。`furikaeri-recall` には足さないでください。作業中に自分から動く必要があるからです。

## こんなふうに頼めます

| 言い方 | スキル | 何が起きるか |
|---|---|---|
| 「振り返りして」「今日の振り返り」 | project | 今日、動きのあったすべてのプロジェクト |
| 「acme-api の先週の振り返り」 | project | 1つのプロジェクト、先週の月〜日 |
| 「10/20〜10/26を振り返ってPRにして」 | project | 期間指定、プルリクエスト（GitHubでの変更の提案）で公開 |
| 「今週のAIの働き方を振り返って」 | harness | AIツールの働き方：モデル・ツール・キャッシュ・サブエージェント |
| 「今月、スキルにできそうな作業ある？」 | harness | 繰り返している作業を、根拠つきで |
| （acme-apiで作業を始める） | recall | 記録された現状の要約を5行以内で、1セッションに1回 |
| 「なんでRedisやめたんだっけ？」 | recall | 決定と日付・出典。なければ「アーカイブには記録なし」 |
| 「プロジェクトを追加して」「アーカイブの索引を作り直して」 | archive | アーカイブの手入れ：プロジェクトの追加、一覧の作り直し |

**期間**：今日（デフォルト）・昨日・ここ1週間・先週（月〜日）・ここ1ヶ月・日付・`2026-10-01..2026-10-26` のような範囲。時刻はあなたのタイムゾーンで数えます。

## furikaeri-harnessが見るもの

AIツールが残しているセッションログ（Claude Code・Codex・Pi・OpenCode）を読みます。ログにある数字だけを数え、ないものは「unknown」と書きます。見るのは次のものです。

- **モデルとトークン。** どのモデルが動いて、どれだけの文章が出入りしたか。トークンは、AIが文章の量を数える単位です。
- **キャッシュ。** 入力のうち、読み直さずに使い回せた分。使い回せると、速くて安く済みます。
- **コンパクション。** 長くなったチャットを要約に縮めて、空きを作ること。そのとき何が抜け落ちたかも見ます。
- **ツール。** どのツールを呼んだか、エラー、やり直し、同じファイルを何度も読んだこと。
- **スキル。** どのスキルがいつ読み込まれたか、無駄に読み込まれたもの、必要なのに読み込まれなかったもの。
- **サブエージェント。** メインのAIが、別の作業を任せるために立ち上げる助手のAI。作業がだぶっていないか、結果が使われたかを見ます。
- **手戻り。** あなたが途中で止めたこと、「そうじゃなくて…」という訂正、同じ説明を2回したこと、権限の拒否。

発見ごとに、何が起きたか、どんな種類の問題か、そして次のどれかを書きます。

- `ADD`：足りないルールやスキルを追加する。
- `UPDATE`：ルールが間違っている、または古いので直す。
- `REMOVE`：ルールが邪魔なので消す。
- `NO_CHANGE`：そのままにする。指示は正しく、AIがうっかりしただけ、ということがよくあります。

また、スキルにできそうな繰り返し作業も挙げます。目安は、2つ以上のセッションで3回以上見られたことです。あなたのファイルは勝手に書き換えません。正確な変更案と、次回それが効いたか確かめる方法を提案するだけです。

チャットアプリ（claude.ai・ChatGPT）では、AIがトークンやキャッシュのデータを読めないので、数字のいらない部分だけを見ます。アーカイブを設定してあれば、harnessのレポートは `harness/` の下に保存され、次の点検ではまず最新のレポートを読んで、前回の提案が効いたかを確かめます。アーカイブがないと、この確認は同じチャットの中でしかできません。

設計を提案する前に `furikaeri-recall` が動いたかも、確かめられます。これを使うと、[AIに思い出させる](../README.ja.md#aiに思い出させる任意)の案内文を、数字を見ながら調整できます。

## AIはアーカイブをどう扱うか

アーカイブに入るのは、AIが見られて、書き残したことだけです。次のものは入っていません。

- 会議やSlackで決まったこと
- furikaeriの定期タスクがないチャットで決まったこと
- このスキルを入れていないAIとの会話で決まったこと
- 最後のレポートのあとに決まったこと

レポートもAIの解釈なので、質はまちまちです。すべてのスキルと、アーカイブの `FURIKAERI.md` は、次の4つのルールで記録を読みます。

- **記録にない ≠ 存在しない。** 「Xは決まっていない」とは言わず、「アーカイブにはXの記録がない」と言います。
- **記録 ≠ 今も正しい。** いまのあなたの発言がアーカイブより優先です。食い違いは「記録が古い」と考えて、確認します。
- **決定だけが決定。** `[interp]`（解釈）や `[proposal]`（誰も採用していない提案）を、決定として読み返しません。決定ログに入るのは、あなたが決めた・受け入れたものだけです。
- **短く、出どころを添えて、静かに。** recallが見せるのは「アーカイブより、2026-10-25までの記録」のように書いた数行だけです（日付は実際のもの）。食い違いは1回だけ伝え、関係ないときは何も言いません。

## アーカイブの中身と設定

```
furikaeri-archive/
├── FURIKAERI.md                    設定・パス・レポートのテンプレート・プライバシー
├── INDEX.md                        すべてのプロジェクトと、その呼び名のヒント
├── projects/
│   └── acme-api/
│       ├── INDEX.md                現状・決定ログ・未解決の問い
│       └── reports/2026-10/
│           ├── 2026-10-26_claude_you.md
│           └── 2026-10-26_chatgpt_you.md
└── harness/you/2026-10/            （任意）harnessのレポート
```

設定は `FURIKAERI.md` にあります。

```yaml
mode: personal          # personal（個人）| team（チーム）
language: ja            # レポートの言語
timezone: Asia/Tokyo
publish: direct         # direct（push）| pr（プルリクエスト）
new-projects: propose   # propose（提案だけ）| create（自動で作る）
meta-analysis: full     # full | collaboration-only | off
```

- `publish`：`direct` は、レポートをそのままアーカイブに push します。`pr` は、プルリクエストを作って、誰かの承認を待ちます。
- `new-projects`：どのプロジェクトにも当てはまらない作業があったとき、`propose` は一覧に載せて、あなたの確認を待つだけです。`create` は、自動でプロジェクトを作ります。
- `meta-analysis`：レポートが、あなたとAIの協働についてどこまで書くか。`full` は、あなた自身のパターンも書きます。`collaboration-only` は、人ではなくやり取りの様子だけを書きます。`off` は、この欄を書きません。

### アーカイブを一度だけ登録する

毎回アーカイブ名を言わなくて済むように、`furikaeri-project`・`furikaeri-recall`・`furikaeri-harness` の `SKILL.md` の先頭にある **Defaults** 欄に書いておけます。フォルダの場所は、[ほかのインストール方法](#ほかのインストール方法)の表にあります。

```markdown
- Archive: github.com/you/furikaeri-archive
- Author: you
- Timezone: Asia/Tokyo
```

`furikaeri-recall` にあるのは、Archiveの行だけです。Authorは、ファイル名に使う短いハンドル名です。

## レポートの例

```markdown
# acme-api — 2026-10-26 (claude)

## Decisions
| Decision | Status | Rationale | Evidence |
|---|---|---|---|
| RESTを維持し、GraphQLの試作はやめる | changed | それなしでレイテンシ目標を達成 | 14:20の会話, PR #42 |

## Alignment & drift
- [fact] 「エラー処理は最小限で」と頼んだが、AIがリトライの仕組みを追加した。
- [interp] AIのドリフト：10/20に決めた「新しい抽象化は入れない」が失われていた。

## Context that would be lost
- 却下：ゲートウェイでのキャッシュ — 認証ヘッダーがユーザーごとに違うため。
```

- **Status** は、`new`（新規）・`reaffirmed`（再確認）・`changed`（変更）・`reverted`（取り消し）のどれかです。
- **ドリフト**は、作業が少しずつ元の目的からずれていくことです。ずれは、あなたが意図して変えた・AIが制約を忘れた・新しい情報が出た・判断できない（アーカイブから見えない場所で決まった可能性がある）、のどれかに分けます。
- **Context that would be lost** は、gitに残らない背景です。却下した案とその理由、失敗した試み、次に試したいことなどです。
- すべての記述に `[fact]`（出典つき）・`[interp]`（解釈）・`[proposal]`（提案）のラベルが付きます。
- レポートの言語は、`FURIKAERI.md` の `language`（`en`・`ja` など）で決まります。`ja` にすれば日本語で書かれます。

このリポジトリ自体の設計セッションから作った実例が [`examples/`](../examples/projects/furikaeri-skills/reports/2026-10/2026-10-26_multi_aero.md) にあります。

## 大事にしていること

- **記憶より証拠。** 数字はログから取り、取れなければ「unknown」と書きます。「完了」はコミット・テスト・ファイルで確かめます。
- **事実・解釈・提案を分けて書きます。**
- **指示は少なく、鋭く。** AIのうっかりミスと、指示そのものの欠陥を分けて考えます。ルールを足すより、絞る・消すほうを優先します。
- **提案はしても、勝手に書き換えません。** あなたのスキル・`AGENTS.md`・`CLAUDE.md` は書き換えず、正確な変更案を提案します。
- **見えなかった範囲を隠しません。** 取れなかったログや、届かなかった会話は、推測で埋めずにそう書きます。
- **記録は一部だけ。** 参考にはしても、決めるのは今のあなたの言葉です。
- **人を採点しません。** 協働のメモは、見て分かるパターンだけを書きます。プライバシー設定に従い、チームメイトを評価しません。

## よくある質問

<details>
<summary><b>PythonやNode、サーバーは必要？</b></summary>

サーバーは要りません。スキルはMarkdownだけです。ログを分析するときは、AIがもともと使えるツール（`git`・`jq`・`sqlite3` など）で、その場限りのコマンドを実行します。Nodeが要るのは、コマンド1つのインストール（`npx`）だけです。フォルダのコピーなら要りません。
</details>

<details>
<summary><b>4つとも使わないとだめ？</b></summary>

いいえ。インストールすると4つとも入りますが、1つずつ使えます。`furikaeri-project` は、アーカイブがなくても動きます。その場合、レポートはチャットに出るだけです。`furikaeri-harness` は、単独で動きます。`furikaeri-recall` は、読むためのアーカイブが必要です。
</details>

<details>
<summary><b>データはどこに行く？</b></summary>

あなたが選んだアーカイブのリポジトリだけです。非公開にしておいてください。秘密情報・トークン・会話の生ログはコミットしません。
</details>

<details>
<summary><b>チームで1つのアーカイブを共有できる？</b></summary>

できます。`mode: team` と `publish: pr` にしてください。レポートには作成者のハンドル名が付き、他人のレポートは編集しません。`meta-analysis: collaboration-only` にすると、人ではなくやり取りのパターンだけを書きます。
</details>

<details>
<summary><b>アーカイブの記録が、私の指示より優先されることはある？</b></summary>

ありません。今のあなたの発言が優先です。記録された決定と違うときは、`furikaeri-recall` が1回だけ「まだそのままですか？ 別の場所で変えましたか？」と確認し、そのあとはあなたに従います。変更は次の振り返りで記録されます。
</details>

<details>
<summary><b>AGENTS.md やスキルを勝手に変えない？</b></summary>

変えません。`furikaeri-harness` は、正確な変更案と、次回それが効いたか確かめる方法を提案するだけです。決めるのはあなたです。
</details>

<details>
<summary><b>harnessの振り返りで古いセッションが見つからないのはなぜ？</b></summary>

ログを消すAIツールがあるからです。たとえばClaude Codeは、デフォルトで30日後に削除します（`cleanupPeriodDays`）。定期的に振り返るか、この設定を延ばしてください。
</details>

## リポジトリの構成

```
skills/
├── furikaeri-harness/   SKILL.md・references/log-sources.md
├── furikaeri-project/   SKILL.md
├── furikaeri-recall/    SKILL.md
└── furikaeri-archive/   SKILL.md・assets/（FURIKAERI.md、INDEX.md、PROJECT_INDEX.md）
prompts/                 scheduled-daily.md・scheduled-daily.ja.md
examples/                プロジェクト索引とレポートの実例
docs/                    GUIDE.md・GUIDE.ja.md・images/（イラスト）
```
