---
title: "Claude Codeを活用したNeovimでのGit運用フロー"
emoji: "🤖"
type: "tech"
topics: ["vim", "ai", "claude", "git"]
published: true
---

:::message
本記事には一部生成AIを利用しています。ただし筆者が責任を持って清書しておりますので、よろしくお願いいたします。
:::

# はじめに

こんにちは、フリーでフロントエンジニアをしている齋藤です。エディターはNeovimを愛用しています

今月2025年5月始めにClaudeのMaxプランが発表されましたね。MaxプランではClaudeを定額月$100で利用することができ、Claude CodeをAPIのトークン消費なしで利用することができます。個人的には今まで従量課金だったClaudeのAIエージェントをこの金額で利用できるというのはかなりお得になったかなという印象です

https://www.anthropic.com/pricing

筆者はこのClaude Maxプランを契約し、毎日使い倒しながらも試行錯誤中です

昨今AIの進化は非常に目覚ましいものがあります。特にソフトウェア開発においては、Gitワークフローの効率化が生産性向上に大きく貢献できるかと思っています。特にコミットメッセージの作成やPull Requestの準備といった作業は、とても重要でありながらも時間のかかる作業です。これらの作業は高品質なプロダクトの維持に欠かせません

私自身、日々のGitやGithubでの作業に多くの時間を費やしており、特に次のような課題を感じていました

- コミットメッセージの一貫性維持の難しさ
- GitHub Issueやプルリク作成でもっとコードの内容に沿った詳細な情報を記載したい
- プロジェクトごとに異なるテンプレートへの対応

本記事では、NeovimとAIエージェントであるClaude Codeを組み合わせることで、これらの課題をどのように解決し、日々のGit操作をよりスムーズに行うための方法をご紹介します

# 実現したこと

今回Claude Codeを利用して実現したことは主に3つです

- Github Issuesの自動作成
  - 主にOSSでの課題をメモするのに使用
  - 筆者はClaude Codeでの対話のメモリを頻繁にリセットするためIssueをストレージとして活用しています
- コミットメッセージの自動作成とコミットの実行
- プルリクの自動作成
- 変更のプッシュとリベース・マージの自動化

https://github.com/mikinovation/dotfiles/blob/9e093b2c5ab2a46e27c0c4c88f6e5a48abb650fb/config/nvim/plugins/claude-code.lua#L281-L289

これらのカスタムプロンプトをインタラクティブに組み立てて生成するような仕組みを構築しました。

以下はデモです。[quick-scope](https://github.com/unblevable/quick-scope)というNeovimのプラグインを導入するという簡単なタスクをやってみました

Gifの尺の関係で出力された内容を精査せず脳死でエンター連打していますがご容赦ください

GithubのIssue作成デモ
![](https://storage.googleapis.com/zenn-user-upload/18d75e4b97a5-20250518.gif)

gitのコミット作成デモ
![](https://storage.googleapis.com/zenn-user-upload/ff8a20a6fc76-20250518.gif)

Githubのプルリク作成デモ
![](https://storage.googleapis.com/zenn-user-upload/9d69f178ed69-20250518.gif)

Claude CodeとNeovimを連携するにあたってはclaude-code.nvimを利用しました。Claude Code自体はCLIのツールです。claude-code.nvimはClaude Codeを、Neovim上でTUIとして利用できるようにしたプラグインになります

https://github.com/greggh/claude-code.nvim

# Github CLIに最適化させたカスタムプロンプトの設計

このワークフローの中核は、GitHub CLI（gh）とClaude Codeへの最適化されたカスタムプロンプトの組み合わせです。

## GitHub CLIの活用

GitHub CLIは、コマンドラインからGitHubの様々な機能にアクセスするための公式ツールです。githubのssh接続を通すためだけに入れている方も多いでしょう。実際`gh auth login`はとても便利です

https://cli.github.com/

このCLIツールを使用すると、ブラウザを開くことなく以下のような操作が可能になります：

```bash
# GitHub Issueを作成する例
gh issue create --title "新機能の提案" --body "詳細な説明..."

# プルリクエストを作成する例
gh pr create --base main --head feature-branch --title "新機能の実装"

# PRへの自己アサイン
gh pr edit 123 --add-assignee "@me"
```

しかし、これらのコマンドを覚えたり、適切なオプションを指定したりする必要があります。そこでClaude Codeの出番です。

## カスタムプロンプトの設計

Claude Codeへのプロンプトは、タスクの種類（コミット、Issue、PR）ごとに最適化しています。例えば、PR作成のプロンプトは以下のようになっています：

```
> I'm going to create a pull request. I will use gh command. Please follow these
  instructions:
  - Create a PR in en language
  - Set PR status to open
  - Assign myself to the PR
  - Use 'main' as the base branch for the PR
  - With ticket reference: https://github.com/mikinovation/dotfiles/issues/175
  - Please follow the template format in .github/PULL_REQUEST_TEMPLATE.md
```

このカスタムプロンプトを生成するためにはワークフローの項目に沿っていくつかのインプット項目を埋める必要があります。

## Github Issue作成ワークフロー

Github Issue作成のワークフローでは日本語か英語かの言語選択のみを行います。すると以下のプロンプトが生成されます

```
> I'm going to create a GitHub issue. I will use gh command. Please follow these
  instructions:
  - Create an issue in en language
  - Use gh issue create command to create the issue
  - Please check if there are templates in .github/ISSUE_TEMPLATE and use the appropriate template
```

- Create an issue in en language
  - 日英の言語の設定を行う
- Use gh issue create command to create the issue
  - Github CLIを使うように指示
  - これがないと度々チャット上で課題をあげるのみになるケースが発生
- Please check if there are templates in .github/ISSUE_TEMPLATE and use the appropriate template
  - Github自体の仕組みであるフォーマットを用意して、一貫した形式でissueを作成させた方が読みやすい

## gitのコミット作成ワークフロー

Github Issue作成のワークフローでは日本語か英語かの言語選択のみを行います。すると以下のプロンプトが生成されます

```
> I'm going to create a git commit. Please follow these instructions:
  - Create a commit in en language
  - Follow conventional commits format (e.g., `feat:`, `fix:`, `chore:`)
  - Use git status to see what files have changed
  - Use git diff to understand the changes
  - Create the commit using git commit -m with an appropriate message
  - Do NOT add any AI attribution lines to the commit message
```

- Create a commit in en language
  - コミットを作成する際の言語の選択
- Follow conventional commits format (e.g., `feat:`, `fix:`, `chore:`)
  - feat等のどのようなコミットなのか分類
  - これがないとformatがバラバラになる
- Use git status to see what files have changed
  - リポジトリの全体像を把握させる
- Use git diff to understand the changes
  - git statusより具体的なファイルごとの差分を確認させる
- Create the commit using git commit -m with an appropriate message
  - これでほぼタイトルだけのコミットで安定
  - 筆者はdescriptionのような詳細なコミットは書かない派
- Do NOT add any AI attribution lines to the commit message
  - これがないと度々Co-Authored-By: Claude Codeのようなコミットメッセージを提案してくる
  - Claudeは「自分がコードを生成しましたよ」という自己主張が激しめだが、コミットメッセージのノイズになるため含めたくない

## Githubのプルリク作成ワークフロー

こちらの機能が本命です。先に出たGithub Issue作成とgitコミットの作成は、プルリク作成ワークフローを作って思いついた副産物になります

Githubのプルリク作成ワークフローでは以下の手順でワークフローを作成します

1. 記載する言語(日英)の選択
2. プルリクをドラフトで作成するか、オープンにして作成するか選択
3. ベースブランチの選択
4. Github IssueやNotion等チケットのリンクを入力

すると以下のようなプロンプトが生成されます

```
> I'm going to create a pull request. I will use gh command. Please follow these
  instructions:
  - Create a PR in en language
  - Set PR status to open
  - Assign myself to the PR
  - Use 'main' as the base branch for the PR
  - Before pushing, rebase from origin/main
  - With ticket reference: https://github.com/mikinovation/dotfiles/issues/174
  - Please follow the template format in .github/PULL_REQUEST_TEMPLATE.md
```

- Create a PR in en language
  - 記載する言語の指定
- Set PR status to open
  - プルリクをオープンにするか、ドラフトで作るか
- Assign myself to the PR
  - Github CLIで`--assignee @me`を指定して自身をアサインさせる
- Use 'main' as the base branch for the PR
  - ベースブランチをリモートブランチから選択する
- Before pushing, rebase from origin/main
  - プッシュ前にリベースを行うよう指示
- With ticket reference: `https://github.com/mikinovation/dotfiles/issues/174`
  - Github IssueやNotion等のチケット紐づけリンクを入力させる
  - Notion等のチケットであればリンクをそのまま張ってくれるし、Github Issueであればテンプレートのフォーマットに応じて`resolve #{issue_number}`みたいな記載を臨機応変に対応してくれるので便利
- Please follow the template format in .github/PULL_REQUEST_TEMPLATE.md
  - テンプレートを探して存在すれば、.github内のディレクトリを確認するようプロンプトを組み立てている

ちなみにベースブランチを取得する際にはNeovim側で必ずgit fetchを実行するようにしています
これに関して先日公開された、たけてぃさんの記事を参考にしました

https://www.takeokunn.org/posts/fleeting/20250518144557-local_git_branch_operation/

これで常にリモートブランチを参照したブランチの運用をすることができます。またリモートブランチとローカルブランチは別ものであるという意識が強くなりました

`~/.config/git/config`を作成し、設定は以下になっています。この設定内容もdotfilesで管理するようにしました

https://github.com/mikinovation/dotfiles/blob/fe37a210a5cb4c2a139c9d8988c40cb315971417/config/git/config#L1-L3

## Git Pushのワークフロー

このワークフローではベースブランチを選択するだけで、自動的に以下のようなプロンプトが生成されます

```
> I'm going to push changes. Please follow these instructions:
  - First, check if a pull request already exists for the current branch with Github CLI
  - If a PR exists, use git merge to update from origin/main before pushing
  - If no PR exists, use git rebase from origin/main before pushing
  - After the merge/rebase is successful, push the changes to origin
```

- First, check if a pull request already exists for the current branch with Github CLI
  - ベースブランチと現在のブランチを元にプルリクが存在するかどうかをghで確認
- If a PR exists, use git merge to update from origin/main before pushing
  - プルリクが存在すれば、mergeしてからpush
- If no PR exists, use git rebase from origin/main before pushing
  - プルリクが存在しなければ、rebaseしてからpush
- After the merge/rebase is successful, push the changes to origin
  - mergeやrebaseが成功すれば、そのまま変更内容をpush

# 実装詳細

改めて実装の詳細です。Neovimでどのようにインタラクティブなカスタムプロンプトの構築をしているのか興味のある方はご覧ください

もしコードの改善点や追加機能で良き案があれば、どしどしコメント頂きたいです

https://github.com/mikinovation/dotfiles/blob/fe37a210a5cb4c2a139c9d8988c40cb315971417/config/nvim/plugins/claude-code.lua#L47-L260

# FAQ

いくつか考えられる疑問もあると思うので補足しておきます
 
## なぜClaude Codeのスラッシュコマンドを使わないのか

Claude Codeには定型文となるタスクをスラッシュコマンドで実行する機能があります

```
#  ユーザー固有のスラッシュコマンド
/user:commit

# プロジェクト固有のスラッシュコマンド
/project:commit
```

ユーザー固有のスラッシュコマンドはホームディレクトリにコマンドを作成することが可能です。上記のコマンドの例でいうと`~/.claude/commands/commit.md`に作成することで実現可能です。project固有のスラッシュコマンドは`${project}/.claude/commands/commit.md`として各リポジトリに作成することができます

このcommit.mdの中で

```md
- コミットはプロジェクトでベースとして使用されている言語にあわせてください
```

のような指示を作ればいい話かもしれません。しかしプルリク作成のような複数の動的要素(ベースブランチやチケットリンク等)が含まれる場合、入力の抜け漏れが発生してしまいます。またプロジェクトでベースとして使用されている言語を不要に探索する必要が出てきてしまいます

更にいうとNeovimから直接キーマップで指定のプロンプトを実行できるようにした方が楽というのがあります

## claude -pコマンドは使わないのか

`-p`のオプションは`--print`オプションを意味します。これはclaude codeを使った対話式ではなく、一回こっきりのコマンドを実行するためのものです

コミットやプルリク作成といった重要な作業は必ず作業者の承認を得ながら進める必要があるため、`-p`オプションは使用していません

# まとめ

本記事では、Claude CodeとNeovimを組み合わせてGitワークフローを改善する方法を紹介しました。GitHub IssueやPRの作成、コミットメッセージの生成といった日々の作業を、AIの力を借りてより効率的に行えるようになりました。

特に、カスタムプロンプトを構築し、インタラクティブなワークフローを実現することで、一貫性のあるGit操作ができるようになりました。

今回の実装はあくまで一例です。まだまだ追加や改善した方がいいワークフローは発生するかと思っています。皆さんもご自身のワークフローに合わせて、プロンプトや機能をカスタマイズしてみてください。
