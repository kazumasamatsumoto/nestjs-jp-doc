# First Steps

## 100 文字要約

NestJS の基本的なセットアップ方法と初期構築手順を説明。TypeScript/JavaScript の互換性、プロジェクト構造、実行方法、コード品質管理ツールの使用方法まで、初心者向けの導入ガイド。

## はじめに

この一連の記事では、Nest の核となる基礎を学びます。Nest アプリケーションの基本的な構成要素に慣れるため、入門レベルで広範囲をカバーする基本的な CRUD アプリケーションを構築していきます。

## 言語

私たちは TypeScript を愛していますが、何よりも Node.js を愛しています。そのため、Nest は TypeScript と純粋な JavaScript の両方に対応しています。Nest は最新の言語機能を活用しているため、バニラ JavaScript で使用する場合は Babel コンパイラが必要です。

例示ではほとんど TypeScript を使用しますが、各コードスニペットの右上隅にある言語切り替えボタンをクリックすることで、いつでもバニラ JavaScript の構文に切り替えることができます。

はい、日本語訳の続きを提供させていただきます：

## 前提条件

オペレーティングシステムに Node.js（バージョン 20 以上）がインストールされていることを確認してください。

## セットアップ

Nest CLI を使用すると、新しいプロジェクトのセットアップは非常に簡単です。npm がインストールされていれば、OS のターミナルで以下のコマンドを実行して新しい Nest プロジェクトを作成できます：

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

**ヒント**
より厳格な TypeScript の機能セットで新しいプロジェクトを作成するには、`nest new`コマンドに`--strict`フラグを付けてください。

project-name ディレクトリが作成され、node モジュールといくつかの基本ファイルがインストールされ、src/ディレクトリが作成されて複数のコアファイルが配置されます。

![環境構築に関する図式化](/overview/svg/nestjs-setup-flow.svg)

```
src/
  app.controller.spec.ts
  app.controller.ts
  app.module.ts
  app.service.ts
  main.ts
```

これらのコアファイルの概要は以下の通りです：

- app.controller.ts: 単一のルートを持つ基本的なコントローラー
- app.controller.spec.ts: コントローラーのユニットテスト
- app.module.ts: アプリケーションのルートモジュール
- app.service.ts: 単一のメソッドを持つ基本的なサービス
- main.ts: NestFactory コア機能を使用して Nest アプリケーションインスタンスを作成するアプリケーションのエントリーファイル

main.ts には、アプリケーションをブートストラップする非同期関数が含まれています：

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

Nest アプリケーションインスタンスを作成するために、コアの NestFactory クラスを使用します。NestFactory は、アプリケーションインスタンスを作成できるいくつかの静的メソッドを公開しています。create()メソッドは、INestApplication インターフェースを実装するアプリケーションオブジェクトを返します。このオブジェクトは、後の章で説明するメソッド群を提供します。上記の main.ts の例では、単に HTTP リスナーを起動し、アプリケーションがインバウンドの HTTP リクエストを待ち受けられるようにしています。

なお、Nest CLI でスキャフォールドされたプロジェクトは、各モジュールを専用のディレクトリに保持するという規約に従うことを推奨する初期プロジェクト構造を作成します。

**ヒント**
デフォルトでは、アプリケーションの作成中にエラーが発生した場合、アプリは終了コード 1 で終了します。代わりにエラーをスローさせたい場合は、abortOnError オプションを無効にしてください（例：NestFactory.create(AppModule, { abortOnError: false })）。

はい、日本語訳の続きを提供させていただきます：

## プラットフォーム

Nest は、プラットフォームに依存しないフレームワークを目指しています。このプラットフォーム独立性により、開発者が様々な種類のアプリケーションで活用できる再利用可能な論理パーツを作成することが可能になります。技術的には、Nest はアダプターさえ作成すれば、任意の Node HTTP フレームワークで動作することができます。標準で 2 つの HTTP プラットフォームがサポートされています：express と fastify です。ニーズに最も合うものを選択できます。

- platform-express: Express は、node 用の有名な最小限の Web フレームワークです。本番環境で実績があり、コミュニティによって実装された多くのリソースを持つライブラリです。@nestjs/platform-express パッケージがデフォルトで使用されます。多くのユーザーにとって Express で十分であり、有効にするための特別な操作は必要ありません。

- platform-fastify: Fastify は、最大限の効率性と速度を提供することに重点を置いた、高性能で低オーバーヘッドのフレームワークです。使用方法については[こちら]で確認できます。

使用されるプラットフォームに関わらず、それぞれが独自のアプリケーションインターフェースを公開します。これらはそれぞれ NestExpressApplication と NestFastifyApplication として参照されます。

以下の例のように、NestFactory.create()メソッドに型を渡すと、app オブジェクトはその特定のプラットフォーム専用のメソッドを使用できるようになります。ただし、基盤となるプラットフォームの API にアクセスしたい場合を除き、型を指定する必要はありません：

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);
```

## アプリケーションの実行

インストールプロセスが完了したら、OS のコマンドプロンプトで以下のコマンドを実行して、インバウンドの HTTP リクエストを待ち受けるアプリケーションを起動できます：

```bash
$ npm run start
```

**ヒント**
開発プロセスを高速化するため（ビルドが 20 倍速く）、start スクリプトに-b swc フラグを付けて SWC ビルダーを使用できます：`npm run start -- -b swc`

このコマンドは、src/main.ts ファイルで定義されたポートで HTTP サーバーを待ち受けるアプリケーションを起動します。アプリケーションが実行されたら、ブラウザを開いて http://localhost:3000/にアクセスしてください。"Hello World!"メッセージが表示されるはずです。

ファイルの変更を監視するには、以下のコマンドでアプリケーションを起動できます：

```bash
$ npm run start:dev
```

このコマンドはファイルを監視し、自動的にサーバーを再コンパイルして再起動します。

## リンティングとフォーマット

CLI は、大規模な開発ワークフローを確実にスキャフォールドするための最善の努力を提供します。そのため、生成された Nest プロジェクトには、コードリンター（eslint）とフォーマッター（prettier）が事前にインストールされています。

**ヒント**
フォーマッターとリンターの役割の違いについて不明な場合は、[こちら]で違いを学べます。

最大限の安定性と拡張性を確保するため、基本の eslint と prettier cli パッケージを使用しています。この設定により、公式拡張機能による優れた IDE 統合が可能になります。

IDE が関係ない環境（継続的インテグレーション、Git フック等）のために、Nest プロジェクトには使用準備の整った npm スクリプトが付属しています：

```bash
# ESLintでリントと自動修正
$ npm run lint

# Prettierでフォーマット
$ npm run format
```

## NestJS の構成

![構成の図式化](/overview/svg/nestjs-basic-structure.svg)

## Express と Fastify の比較

### 100 文字要約

Express はエコシステムが充実し安定性が高く一般的な Web アプリに適しており、Fastify は高パフォーマンスと TypeScript サポートが強みで、高速性が求められるアプリに最適です。

![ExpressとFastifyの比較](/overview/svg/express-fastify-comparison.svg)

### Express

**登場背景（2010 年）**

- Node.js が 2009 年に登場した直後の時期で、Web アプリケーションフレームワークの需要が高まっていました
- TJ Holowaychuk によって開発され、Ruby on Rails の思想に影響を受けています
- 当時の Node.js エコシステムには、フルスタックな Web アプリケーションを構築するためのフレームワークが存在していませんでした

**発展の経緯**

1. 最小限の機能で始まり、"middleware"という概念を導入
2. 2014 年に StrongLoop に買収され、その後 IBM が StrongLoop を買収
3. 2016 年に Node.js Foundation に寄贈され、現在は OpenJSF（Open JavaScript Foundation）の下でメンテナンスされています

### Fastify

**登場背景（2016 年）**

- Node.js のエコシステムが成熟し、マイクロサービスアーキテクチャやハイパフォーマンスな API の需要が増加していた時期
- Matteo Collina と Tomas Della Vedova によって開発
- 既存のフレームワーク（Express 等）では対応できない以下のような課題に対応するために開発されました：
  - 高いスループットの要求
  - JSON スキーマによる堅牢な型検証の必要性
  - プラグインベースのアーキテクチャの需要
  - 非同期処理の効率的な扱い

**発展の経緯**

1. 当初からパフォーマンスを重視した設計
2. v1.0.0 が 2018 年にリリース
3. 現在も活発に開発が続けられ、Node.js の最新機能を積極的に取り入れています

### 主な違いが生まれた理由

- **アプローチの違い**：

  - Express: シンプルさと柔軟性を重視した設計（2010 年代初頭の Web 開発ニーズを反映）
  - Fastify: パフォーマンスと型安全性を重視した設計（現代のマイクロサービス時代のニーズを反映）

- **時代背景の違い**：
  - Express: Node.js 黎明期の汎用的な Web フレームワークとして設計
  - Fastify: マイクロサービス時代の高性能なサービス開発ニーズに応えるために設計

これらの歴史的背景が、両フレームワークの設計思想や特徴の違いとなって現れています。

![歴史的背景](/overview/svg/framework-history-timeline-final.svg)
