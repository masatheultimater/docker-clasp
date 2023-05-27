# docker-clasp-starter

GASを管理するための環境を簡単に整えるためのリポジトリ

# このリポジトリを使った GAS 開発の大まかな流れ

```mermaid
graph TD
A[リポジトリをクローン] --> B[Visual Studio CodeでRemote Containerを開始]
B --> C[コンテナ内で clasp create を実行]
C --> D[プロジェクトフォルダが作成される]
D --> E[コンテナ内で開発作業を行う]
E --> F[変更をコミットしGitHubにPush]
F --> G[GitHub Actionsが実行される]
G --> GA[GitHub Actionsで clasp push を実行]
GA --> H[.clasp.jsonが存在するフォルダに移動し `clasp push` を実行]
H --> I[Google Apps Scriptプロジェクトがデプロイされる]
```

# 使い方

## 1. リポジトリをクローンする

ローカルにクローンして、ディレクトリに移動。

```bash
git clone
cd docker-clasp-starter
```

## Visual Studio Code でコンテナを開く

Visual Studio Code プロジェクトを開く。

```bash
code .
```

Cmd + Shift + P でコマンドパレットを開き、`Remote-Containers: Reopen in Container`を選択。

コンテナが開いたら、 `postCreateCommand` によって自動で `npm install` される。

## コンテナ内で clasp をセットアップする

### clasp login

```bash
clasp login
```

## プロジェクトフォルダとプロジェクトを作成

```bash
npm run create [project_name]
```

これを実行することによって [clasp-script.sh](https://github.com/Studist/docker-clasp-starter/blob/main/clasp-script.sh) が動き、`project_name` という名前のフォルダが作成され、以下のツリー構造でプロジェクトが作成される。

```bash
project_name
├── src
│   └── appsscript.json
└── .clasp.json
```

そして、タイムゾーンが日本に設定される。
project_name は任意の名前を指定。すでに存在するフォルダ名を指定すると、エラーが発生。
直接 clasp を実行してしまうと、上記の処理が行われないので、必ず `npm run create [project_name]` を実行。

## 作成したフォルダに移動して開発

```bash
cd project_name
touch ./src/code.gs
```

Typescript を使う場合は、以下のような `./src/code.ts` を作成。

### Push したい場合

Git リポジトリを Push すると GitHubActions で `clasp push` が行われるが、コンテナ内から直接 `clasp push` を実行することもできる。
ただし、 `package.json` が `project_name` フォルダに存在しない場合エラーになるので、これも Clone と同様に `npm run push project_name` を実行。
どこのディレクトリにいても `project_name` のプロジェクトを明示的に Push できる。

## GitHub Actions 用の環境変数を設定する

コンテナ内で `clasp login` を実行すると、`/home/node/.clasprc.json.` が作成される。

このファイルを `cat` し、出力を全てコピーして GitHub のリポジトリの [Settings] => [Secrets and variables] => [Actions] => [New repository secret] に `CLASP_TOKEN` という名前で作成。

```bash
cat /home/node/.clasprc.json
```

## 開発が終わったらプッシュ

リポジトリをプッシュすると、GitHub Actions で `.clasp.json` があるフォルダを全て探して、それぞれのフォルダで `clasp push` が行われる。

# 補足

## コンテナ内で `clasp login` を実行する理由

`clasp login` は必ずコンテナ内で行わないといけないわけではない。別の端末などで作った `.clasprc.json` の内容を `CLASP_TOKEN` に設定してもいいし、手打ちでプロジェクトフォルダや `.clasp.json` を作ってもよい。
Clasp の開発環境セットアップや、プロジェクトの作成などを誰でもできるように、コンテナ内で行うように記載してある。

## `project1` フォルダはなんのためにある？

`project1` フォルダはツリー構造を示すためのサンプルプロジェクト。
不要であれば削除。

## TypeScript を使う場合

以下のドキュメントを参考に `tsconfig.json` を編集。
https://github.com/google/clasp/blob/master/docs/typescript.md
