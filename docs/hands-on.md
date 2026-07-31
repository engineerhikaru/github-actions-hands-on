# ハンズオン手順

このページのコマンドと YAML は、そのままコピーして使えます。

## 事前準備

- GitHub アカウント
- ブラウザ

作業はすべて GitHub の画面上で完結します。
ローカルで進めたい場合は、各ステップの「ローカルで進める場合」を読んでください。

## STEP 1: リポジトリを fork する

1. https://github.com/tooppoo/github-actions-hands-on をブラウザで開く
2. 右上の `Fork` を押す
3. Owner を自分のアカウントにして `Create fork`
4. **Actions タブを開き、`I understand my workflows, go ahead and enable them` を押す**

以降の作業は、すべて fork した側のリポジトリで行います。

### 4 番を飛ばさないこと

fork したリポジトリでは、workflow が最初は無効になっています。
他人の書いた workflow が、fork した瞬間に動くのを防ぐためです。
有効化するまでは、push しても何も起きません。

### ローカルで進める場合

```bash
git clone git@github.com:<自分のアカウント>/github-actions-hands-on.git
cd github-actions-hands-on
```

有効化のボタンはブラウザからしか押せません。
clone する前に、Actions タブで有効化を済ませてください。

## STEP 2: hello-world.yml を動かす

`.github/workflows/hello-world.yml` の `steps` を、次のように書き換えます。

```yaml
    steps:
      - run: echo "Hello, GitHub Actions!"
      - run: date
```

書き換えたファイル全体は次のようになります。

```yaml
name: CI

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  check:
    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
      - run: echo "Hello, GitHub Actions!"
      - run: date
```

### 手順

1. `.github/workflows/hello-world.yml` を開く
2. 鉛筆アイコンから編集する
3. `Commit changes...` を押し、**`Commit directly to the main branch`** を選んでコミットする
4. Actions タブを開き、実行されたことを確認する

### 確認するときのポイント

Actions タブの一覧に並ぶ名前は、ファイル名ではなく `name:` の値です。
このリポジトリでは `CI` と表示されます。

ログは Actions タブ → 実行（workflow run）→ `check`（job）→ 各 step、の順に開きます。
失敗したときは、赤くなった step のログを開いてください。

### 手動で実行する

`workflow_dispatch:` が書いてあるので、push しなくても実行できます。
Actions タブで `CI` を選び、`Run workflow` を押してください。

試行錯誤するときは、こちらのほうが速く回せます。

### ローカルで進める場合

```bash
git switch main
# .github/workflows/hello-world.yml を編集する
git add .github/workflows/hello-world.yml
git commit -m "echo を書き換える"
git push origin main
```

ブランチを切ると `on.push.branches` の条件から外れて動きません。
`main` に直接コミットしてください。

## STEP 3: README.md の中身を出力する

`steps` を、次のように書き換えます。

```yaml
    steps:
      - uses: actions/checkout@v7
      - run: cat README.md
```

コミットしたら、Actions タブで README.md の中身が出力されたことを確認してください。

### なぜ checkout が要るのか

workflow を動かすマシン（runner）は、毎回まっさらな状態で起動します。
リポジトリのファイルは、何もない状態です。

`actions/checkout` は、そのマシンにリポジトリの内容を配置する action です。
これを入れないと、`cat README.md` は「そんなファイルはない」で失敗します。

### 試してみる

余裕があれば、`- uses: actions/checkout@v7` の行を消してコミットしてみてください。
`cat: README.md: No such file or directory` で失敗します。

### uses の書き方

```yaml
      - uses: <owner>/<repo>@<ref>
```

`actions/checkout@v7` なら、`actions` という owner の `checkout` というリポジトリの `v7` を使う、という意味です。
公開されている action は [GitHub Marketplace](https://github.com/marketplace?type=actions) から探せます。

`@v7` のようにバージョンは固定してください。
固定しないと、action 側の更新で急に壊れることがあります。

## STEP 4: 自分の action を作る

`.github/actions/greet/action.yml` を新規作成します。

```yaml
name: greet
description: 挨拶を出力する

runs:
  using: composite
  steps:
    - shell: bash
      run: echo "Hello, GitHub Actions!"
```

`.github/workflows/hello-world.yml` の `steps` を、次のように書き換えます。

```yaml
    steps:
      - uses: actions/checkout@v7
      - uses: ./.github/actions/greet
```

2 つのファイルとも `main` へ直接コミットしてください。

### 手順（ブラウザの場合）

1. リポジトリのトップで `Add file` → `Create new file`
2. ファイル名の欄に `.github/actions/greet/action.yml` と入力する
   - `/` を打つとフォルダとして扱われるので、先に階層を作る必要はありません
3. 上の内容を貼り付けてコミットする
4. `.github/workflows/hello-world.yml` を開き、`steps` を書き換えてコミットする

### 詰まりやすいところ

- composite action の `run` step には `shell` の指定が必要です。
  書かないと `Required property is missing: shell` で失敗します。
  `uses` の step には逆に書けません。
- リポジトリ内の action は `checkout` の後でないと呼べません。
  `./.github/actions/greet` はファイルパスなので、runner 上にファイルが置かれている必要があります。

## 困ったとき

| 症状 | 原因として多いもの |
| --- | --- |
| push しても Actions タブに何も出ない | STEP 1 の有効化ボタンを押していない |
| コミットしたのに実行されない | ブランチを切ってコミットしている。`main` へ直接コミットする |
| `cat: README.md: No such file or directory` | `actions/checkout` の step がない |
| `Required property is missing: shell` | composite action の `run` step に `shell: bash` がない |
| `Can't find 'action.yml'` | パスが違う。`checkout` より前で呼んでいる場合も同じエラーになる |

## この先

今日は action を定義して動かすところまでを扱いました。
次に読むと世界が広がるものを挙げておきます。

- [inputs と outputs](https://docs.github.com/actions/sharing-automations/creating-actions/metadata-syntax-for-github-actions) — action に値を渡し、結果を受け取る
- [reusable workflow](https://docs.github.com/actions/using-workflows/reusing-workflows) — workflow ごと使い回す
- [secrets](https://docs.github.com/actions/security-guides/using-secrets-in-github-actions) — トークンなどを安全に渡す
- [matrix](https://docs.github.com/actions/using-jobs/using-a-matrix-for-your-jobs) — 複数の環境で同じ job を回す
- [キャッシュ](https://docs.github.com/actions/using-workflows/caching-dependencies-to-speed-up-workflows) — 依存関係の再取得を省く

公式ドキュメントは https://docs.github.com/actions にあります。
