<p align="center">
  <img src="docs/images/hero-ja.svg" alt="furikaeri — AIエージェントと人×AIプロジェクトのための振り返りSkills" width="100%">
</p>

<p align="center">
  <a href="README.md">English</a> · <b>日本語</b>
</p>

<p align="center">
  <img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-F2A33A">
  <img alt="Agent Skills standard" src="https://img.shields.io/badge/Agent%20Skills-standard-7FB2E0">
  <img alt="Markdown only" src="https://img.shields.io/badge/runtime-Markdown%20only-7CC7A4">
  <img alt="Works with" src="https://img.shields.io/badge/works%20with-Claude%20%C2%B7%20ChatGPT%20%C2%B7%20Codex%20%C2%B7%20Pi%20%C2%B7%20OpenCode-B5835A">
</p>

**furikaeri**（振り返り）は、AIと一緒にものを作る人のための、振り返りの仕組みです。AIは、前のチャットで決めたことを覚えていません。そこで、あなたとAIが決めたことを書き残し、非公開のGitリポジトリ（**アーカイブ**）にためて、作業中にAIが確認できるようにします。

[Agent Skills](https://agentskills.io)（スキル）は、AIへの指示を書いたMarkdownファイルです。Claude・ChatGPT・Codex・Pi・OpenCode などのAIツールが、必要なときに読み込みます。furikaeri は、このスキルを4つ集めたものです。プログラムもデータベースも要りません。

## 4つのスキル

<p align="center">
  <img src="docs/images/skills-ja.svg" alt="4つのSkill：furikaeri-project（書く）、furikaeri-recall（思い出す）、furikaeri-archive（しまう）、furikaeri-harness（点検）" width="100%">
</p>

| スキル | 何をする？ | 頼みかたの例 |
|---|---|---|
| `furikaeri-project` | あなたとAIが決めたこと・作ったことを振り返って、レポートを書きます。アーカイブに保存もします。 | 「昨日の振り返りして」 |
| `furikaeri-recall` | 作業中に、アーカイブを見て過去の決定を確認します。読むだけです。 | 「なんでRedisやめたんだっけ？」 |
| `furikaeri-archive` | アーカイブを作って、手入れします。 | 「アーカイブを作って」 |
| `furikaeri-harness` | AIツール自身の働き方（使ったモデルやツール、同じことの繰り返し）を点検して、直し方を提案します。 | 「今週のAIの働き方を振り返って」 |

`furikaeri-harness` の harness（ハーネス）は、Claude Code や Codex のように、AIモデルを動かすツールのことです。

## はじめかた

**1. インストール**

```bash
npx skills add Aero123421/furikaeri-skills --skill '*' -g
```

どのAIツールに入れるかを聞かれます。

`npx` が使えない場合や、Claudeアプリ・ChatGPTで使う場合は、[ほかのインストール方法](docs/GUIDE.ja.md#ほかのインストール方法)を見てください。

**2. アーカイブを作る** AIにこう頼みます。

```text
furikaeri のアーカイブを github.com/<あなた>/furikaeri-archive に作って（非公開で）
```

レポートの言語、タイムゾーン、個人かチームか、をいくつか聞かれます。

**3. プロジェクトを追加する**

```text
アーカイブにプロジェクト「acme-api」を追加して。
リポジトリは github.com/<あなた>/acme-api。「acme」「API v2」とも呼んでる。
```

ほかの呼び名を書いておくと、次からそのプロジェクトを見つけやすくなります。

貼りつけ用の短い案内文も2つ出ます。[AIに思い出させる](#aiに思い出させる任意)を見てください。

**4. 振り返る**

```text
振り返りして
```

チャットにレポートが出ます。「はい」と答えると、アーカイブに保存されます。

> [!TIP]
> 新しいチャットでは、AIがまだアーカイブを知らないことがあります。依頼にアーカイブ名を添えてください。たとえば `振り返りして。アーカイブは github.com/<あなた>/furikaeri-archive` のように書きます。毎回書くのが面倒なら、[デフォルトに登録](docs/GUIDE.ja.md#アーカイブを一度だけ登録する)できます。

## 毎日自動で振り返る（任意）

<p align="center">
  <img src="docs/images/daily-ja.svg" alt="毎晩、定期タスクが起動し、昨日の会話を振り返り、Gitにレポートを保存する" width="100%">
</p>

定期タスクは、ClaudeやChatGPTが決めた時刻にプロンプトを自動で実行してくれる機能です。これを使うと、毎晩AIが振り返ってくれます。

1. **Claude** または **ChatGPT** で **Scheduled** を開き、新しいタスクを作ります。時刻は、ちょうどの時刻から少しずらして、**01:07** のようにします。
2. [`prompts/scheduled-daily.ja.md`](prompts/scheduled-daily.ja.md)（[English](prompts/scheduled-daily.md)）のプロンプトを貼り付けます。アーカイブ、ハンドル名（ファイル名に使う短い名前）、ソース（`claude` か `chatgpt`）を書き換えてください。
3. 初回の実行後に、レポートの **Coverage** 欄を見ます。タスクから何が見えて、何が見えなかったかが書いてあります。

両方のサービスで設定すると、プロジェクトごとに、サービス別のレポートが並びます。

Claude CodeやCodexで使うときは、[サービスごとの注意](prompts/scheduled-daily.ja.md#サービスごとの注意)を見てください。

## AIに思い出させる（任意）

読み返されない記録は、何も変えません。短い2行を貼っておくと、AIが先にアーカイブを見てくれます。

プロジェクトの `AGENTS.md` か `CLAUDE.md` 用です（そのプロジェクトでAIに守ってほしいことを書いておくファイルです。1つ目は英語のまま使えます）。

```markdown
## furikaeri
Before starting work or proposing a design here, check `projects/acme-api/INDEX.md` in github.com/you/furikaeri-archive (furikaeri-recall). It is a partial record: what I say now wins.
```

ChatGPTかClaudeのメモリー用です。

```text
プロジェクトの決定の一部は github.com/you/furikaeri-archive に記録している。そこにあるプロジェクトの作業を始めるときや、設計を提案する前には、まずその INDEX.md を見て。私が今言っていることと違ったら確認して。今の私の発言が優先。
```

プロジェクトを追加すると、`furikaeri-archive` がこの2行を、あなたのアーカイブ名を入れて出してくれます。

すると `furikaeri-recall` が、次の3つのときにプロジェクトの記録を見ます。作業を始める・再開するとき、AIが設計やツールを提案する前、「何を決めたっけ？」「なんで？」と聞かれたときです。

## ひとつだけ知っておいてほしいこと

アーカイブは、記録のすべてではありません。AIが見られて、書き残したことしか入っていません。会議や別のチャット、最近決めたことは、入っていないかもしれません。

だから、AIはアーカイブを「参考」として読みます。**いまのあなたの言葉が、いつでも優先です。** 記録と違うことを言うと、AIは1回だけ確認して、そのあとはあなたに従います。

## くわしい説明

すべて[ガイド](docs/GUIDE.ja.md)にあります。

- [ほかのインストール方法](docs/GUIDE.ja.md#ほかのインストール方法)：フォルダのコピー、Claudeアプリ、ChatGPT、AIツールごとの置き場所と呼び出し方
- [こんなふうに頼めます](docs/GUIDE.ja.md#こんなふうに頼めます)：頼みかたの例と、指定できる期間
- [全体の流れ](docs/GUIDE.ja.md#全体の流れ)：図で見る
- [furikaeri-harnessが見るもの](docs/GUIDE.ja.md#furikaeri-harnessが見るもの)
- [AIはアーカイブをどう扱うか](docs/GUIDE.ja.md#aiはアーカイブをどう扱うか)
- [アーカイブの中身と設定](docs/GUIDE.ja.md#アーカイブの中身と設定)
- [レポートの例](docs/GUIDE.ja.md#レポートの例)
- [大事にしていること](docs/GUIDE.ja.md#大事にしていること)
- [よくある質問](docs/GUIDE.ja.md#よくある質問)
- [リポジトリの構成](docs/GUIDE.ja.md#リポジトリの構成)

## コントリビュート

IssueやPRを歓迎します。特に助かるのは、ほかのAIツールがセッションログを置く場所（[`log-sources.md`](skills/furikaeri-harness/references/log-sources.md)）、レポートテンプレートの翻訳、定期タスクから何が見えて何が見えないかの報告です。

## ライセンス

[MIT](LICENSE)

<p align="center">
  <img src="docs/images/owl.svg" alt="furikaeriのフクロウ" width="120"><br>
  <sub>フクロウは首を約270°回せます。振り返りの名人です。</sub>
</p>
