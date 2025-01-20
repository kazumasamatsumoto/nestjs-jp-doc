# イントロダクションの章

## 100 文字以内の要約

NestJS は、TypeScript ベースの Node.js フレームワークで、Angular 風のアーキテクチャを採用し、Express/Fastify 上で動作します。OOP、FP、FRP を組み合わせ、スケーラブルで保守性の高いサーバーサイドアプリケーションの開発を可能にします。

## はじめに

Nest (NestJS)は、効率的でスケーラブルな Node.js サーバーサイドアプリケーションを構築するためのフレームワークです。プログレッシブ JavaScript を使用し、TypeScript で構築され、完全にサポートしています（ただし、純粋な JavaScript でのコーディングも可能）。また、OOP（オブジェクト指向プログラミング）、FP（関数型プログラミング）、FRP（関数型リアクティブプログラミング）の要素を組み合わせています。

内部では、Nest は Express（デフォルト）のような堅牢な HTTP サーバーフレームワークを使用し、オプションで Fastify を使用するように設定することもできます。

Nest は、これらの一般的な Node.js フレームワーク（Express/Fastify）の上に抽象化レベルを提供しますが、開発者に直接それらの API を公開しています。これにより、開発者は基盤となるプラットフォームで利用可能な無数のサードパーティモジュールを使用する自由を得られます。

## フィロソフィー

近年、Node.js のおかげで、JavaScript はフロントエンドとバックエンドの両方のアプリケーションにおいてウェブの「共通言語」となりました。これにより、Angular、React、Vue などの素晴らしいプロジェクトが生まれ、開発者の生産性を向上させ、高速で、テスト可能で、拡張可能なフロントエンドアプリケーションの作成を可能にしました。しかし、Node（およびサーバーサイド JavaScript）には優れたライブラリ、ヘルパー、ツールが多数存在するものの、アーキテクチャという主要な問題を効果的に解決するものはありませんでした。

Nest は、開発者やチームが高度にテスト可能で、スケーラブルで、疎結合で、保守が容易なアプリケーションを作成できるように、すぐに使えるアプリケーションアーキテクチャを提供します。このアーキテクチャは、Angular から大きな影響を受けています。

## インストール

始めるには、**Nest CLI**でプロジェクトをスキャフォールドするか、**スタータープロジェクトをクローン**する（どちらも同じ結果になります）という 2 つの方法があります。

Nest CLI でプロジェクトをスキャフォールドするには、以下のコマンドを実行します。これにより新しいプロジェクトディレクトリが作成され、初期の Nest コアファイルとサポートモジュールがディレクトリに配置され、プロジェクトの従来の基本構造が作成されます。初めてのユーザーには**Nest CLI**での新規プロジェクト作成が推奨されます。**はじめの一歩**ではこのアプローチで進めていきます。

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

**ヒント**
より厳格な機能セットで TypeScript プロジェクトを作成するには、`nest new`コマンドに`--strict`フラグを付けてください。

**代替手段**
あるいは、**Git**で TypeScript スタータープロジェクトをインストールする場合：

```bash
$ git clone https://github.com/nestjs/typescript-starter.git project
$ cd project
$ npm install
$ npm run start
```

**ヒント**
Git の履歴なしでリポジトリをクローンしたい場合は、**degit**を使用できます。

ブラウザを開いて`http://localhost:3000/`にアクセスしてください。

JavaScript バージョンのスタータープロジェクトをインストールする場合は、上記のコマンドシーケンスで`javascript-starter.git`を使用してください。

また、コアパッケージとサポートパッケージをインストールして、まったく新しいプロジェクトを始めることもできます。その場合、プロジェクトのボイラープレートファイルは自分で設定する必要があることに注意してください。最低限必要な依存関係は以下の通りです：`@nestjs/core`、`@nestjs/common`、`rxjs`、`reflect-metadata`。完全なプロジェクトの作成方法については、この短い記事をご覧ください：
[**ゼロから NestJS アプリを作成する 5 つのステップ！**](https://dev.to/micalevisk/5-steps-to-create-a-bare-minimum-nestjs-app-from-scratch-5c3b)

## イメージ画像

![NestJSの全体像](/introduction/svg/nest-architecture-updated.svg)

## ゼロから NestJS アプリを作成する 5 つのステップ

### はじめに

通常の`npx @nestjs/cli new`コマンド（基本的には TypeScript スターター・プロジェクトをブートストラップする）を使用せずに、依存関係、スクリプト、ソースコードを完全にコントロールしながら、新しい NestJS 標準アプリケーションを作成する方法を説明します。

このガイドの目的は、標準的な（ExpressJS ベースの HTTP サーバー）NestJS アプリを開始するために本当に必要なものを示すことです。

### 使用するツール

- node.js v18 - ランタイム
- npm v9 - パッケージマネージャー
- npx v9 - NPM パッケージのバイナリを実行するユーティリティ
- bash シェル - ターミナル関連。ここでは UNIX のヒアドキュメント機能を使用してファイルを作成しますが、VS Code などの IDE で代用可能です。

もちろん、これらの新しいバージョンを使用することも可能です。

### 最終的なファイル構造

```
.
├── nest-cli.json
├── package.json
├── package-lock.json
├── src
│   ├── app.module.ts
│   └── main.ts
├── tsconfig.build.json
└── tsconfig.json
```

### 手順 1: NPM プロジェクトのセットアップ

```bash
## アプリのコードを格納する新しいディレクトリを作成
mkdir my-nestjs-app
## そのディレクトリに移動
cd $_
## NPMプロジェクトを開始
npm init --yes
```

### 手順 2: 必須の依存関係をインストール

本番環境の依存関係:

```bash
npm install reflect-metadata @nestjs/common @nestjs/core
# Express as HTTPライブラリとして使用する場合：
npm install @nestjs/platform-express
```

開発用の依存関係:

```bash
npm install --save-dev typescript @types/node @nestjs/cli @nestjs/schematics
```

### 手順 3: package.json に npm スクリプトを追加

```bash
# NPMが生成したtestスクリプトを削除
npm pkg delete scripts.test
# プロジェクトビルド後の正しいエントリーファイルにmainエントリを変更
npm pkg set main="dist/src/main"
# ビルドスクリプトを定義
npm pkg set scripts.build="nest build"
# 開発用の監視モードスクリプトを定義
npm pkg set scripts.start:dev="nest start --watch"
# 本番環境用の実行スクリプトを定義
npm pkg set scripts.start:prod="node ."
```

### 手順 4: 必要なファイルの作成

src ディレクトリ、tsconfig.json、tsconfig.build.json、nest-cli.json などの設定ファイルを作成します。

はい、続きを翻訳させていただきます。

### 手順 4（続き）: 必要なファイルの作成

各設定ファイルの内容:

**tsconfig.json**:

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2021",
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./",
    "baseUrl": "./",
    "skipLibCheck": true,
    "incremental": true
  }
}
```

**tsconfig.build.json**:

```json
{
  "extends": "./tsconfig.json",
  "exclude": ["node_modules", "test", "dist", "**/*spec.ts"]
}
```

**nest-cli.json**:

```json
{
  "$schema": "https://json.schemastore.org/nest-cli",
  "collection": "@nestjs/schematics",
  "monorepo": false,
  "sourceRoot": "src",
  "entryFile": "main",
  "language": "ts",
  "generateOptions": {
    "spec": false
  },
  "compilerOptions": {
    "tsConfigPath": "./tsconfig.build.json",
    "webpack": false,
    "deleteOutDir": true,
    "assets": [],
    "watchAssets": false,
    "plugins": []
  }
}
```

**注意**: このチュートリアルでは Jest を使用していないため、`generate`コマンドを使用する際に spec ファイルを生成しないように設定しています。ただし、Jest は必須ではありません。TS デコレータで動作する他のテストフレームワークを使用することも可能です。

最小限のアプリケーションのエントリーポイント（**src/main.ts**）:

```typescript
import { NestFactory } from "@nestjs/core";
import type { NestExpressApplication } from "@nestjs/platform-express";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  await app.listen(process.env.PORT || 3000);
}
bootstrap();
```

### 手順 5: アプリケーションの実行 🎊

**開発環境での実行**:

```bash
npm run start:dev
```

**本番環境での実行**:

```bash
# 本番用ビルド
npm run build
# node_modulesディレクトリを削除
rm -rf node_modules
# ロックファイルを変更せずに本番用依存関係のみをインストール
npm ci --omit=dev
# mainエントリを使用してアプリを起動
node .
```

### 最終確認

すべてが正しく設定されていれば、以下のような依存関係構造になっているはずです：

```bash
$ npm ls --depth=0
my-nestjs-app@1.0.0 /tmp/my-nestjs-app
├── @nestjs/cli@9.1.5
├── @nestjs/common@9.2.1
├── @nestjs/core@9.2.1
├── @nestjs/platform-express@9.2.1
├── @types/node@18.11.13
├── reflect-metadata@0.1.13
└── typescript@4.9.4
```

3 つの開発用依存関係と 4 つの本番用依存関係、そして 3 つの NPM スクリプトのみの、最小限の構成となっています。

すべての手順を bash スクリプトにまとめたものは、こちらで確認できます：
https://github.com/micalevisk/create-nest/blob/main/create-nestjs-from-scratch.sh
