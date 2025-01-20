# 100 文字要約

NestJS のコントローラーについての解説文書です。コントローラーはクライアントからのリクエストを処理し、レスポンスを返す役割を持ち、デコレータを使用して実装します。ルーティング、HTTP メソッド、パラメータ処理など、基本的な機能を提供します。

# 全体像

![全体像](/overview/svg/controller-concept-map-improved.svg)

# コントローラー

コントローラーは、クライアントからの着信リクエストを処理し、レスポンスを返す役割を担います。

コントローラーの目的は、アプリケーションへの特定のリクエストを受け取ることです。ルーティングメカニズムが、どのコントローラーがどのリクエストを受け取るかを制御します。通常、各コントローラーは複数のルートを持ち、異なるルートで異なるアクションを実行できます。

基本的なコントローラーを作成するには、クラスとデコレータを使用します。デコレータはクラスに必要なメタデータを関連付け、Nest がルーティングマップ（リクエストと対応するコントローラーの紐付け）を作成できるようにします。

> ヒント
> バリデーション機能が組み込まれた CRUD コントローラーを素早く作成するには、CLI の CRUD ジェネレーターを使用できます：`nest g resource [name]`

## ルーティング

以下の例では、基本的なコントローラーを定義するために必要な`@Controller()`デコレータを使用します。オプションのルートパスプレフィックスとして`cats`を指定します。`@Controller()`デコレータでパスプレフィックスを使用することで、関連するルートのグループ化が容易になり、コードの重複を最小限に抑えることができます。例えば、cat エンティティとのやり取りを管理する一連のルートを`/cats`ルートの下にグループ化することができます。

```typescript
// cats.controller.ts
import { Controller, Get } from "@nestjs/common";

@Controller("cats")
export class CatsController {
  @Get()
  findAll(): string {
    return "This action returns all cats";
  }
}
```

> ヒント
> CLI を使用してコントローラーを作成するには、`$ nest g controller [name]`コマンドを実行するだけです。

`findAll()`メソッドの前の`@Get()`HTTP リクエストメソッドデコレータは、特定のエンドポイントに対するハンドラーを作成するよう Nest に指示します。エンドポイントは、HTTP リクエストメソッド（この場合は GET）とルートパスに対応します。

ルートパスはどのように決定されるのでしょうか？ハンドラーのルートパスは、コントローラーに宣言された（オプションの）プレフィックスと、メソッドのデコレータで指定されたパスを連結して決定されます。すべてのルートにプレフィックス（`cats`）を宣言し、デコレータにパス情報を追加していないため、Nest は GET `/cats`リクエストをこのハンドラーにマッピングします。

前述したように、パスにはコントローラーのパスプレフィックス（オプション）とリクエストメソッドデコレータで宣言されたパス文字列の両方が含まれます。例えば、`cats`というパスプレフィックスと`@Get('breed')`デコレータを組み合わせると、`GET /cats/breed`のようなリクエストに対するルートマッピングが生成されます。

上記の例では、このエンドポイントに GET リクエストが行われると、Nest はリクエストを私たちが定義した`findAll()`メソッドにルーティングします。メソッド名は完全に任意であることに注意してください。ルートにバインドするメソッドを宣言する必要はありますが、Nest はメソッド名に特別な意味を持たせません。

このメソッドは 200 のステータスコードと、この場合は単なる文字列であるレスポンスを返します。なぜこれが起こるのでしょうか？説明するために、まず Nest がレスポンスを操作するために採用している 2 つの異なるオプションを紹介します：

### 標準（推奨）

この組み込みメソッドを使用する場合：

- リクエストハンドラーが JavaScript のオブジェクトや配列を返すと、自動的に JSON にシリアライズされます。
- JavaScript のプリミティブ型（文字列、数値、真偽値など）を返す場合、Nest はシリアライズを試みずに値をそのまま送信します。
- これによりレスポンス処理が簡単になります：値を返すだけで、残りは Nest が処理します。

さらに、レスポンスのステータスコードは、POST リクエスト（201 を使用）を除いて、デフォルトで常に 200 です。ハンドラーレベルで`@HttpCode(...)`デコレータを追加することで、この動作を簡単に変更できます。

### ライブラリ固有

- ライブラリ固有（例：Express）のレスポンスオブジェクトを使用できます。
- これはメソッドハンドラーのシグネチャで`@Res()`デコレータを使用して注入できます（例：`findAll(@Res() response)`）。
- このアプローチでは、そのオブジェクトが公開するネイティブのレスポンス処理メソッドを使用できます。
- 例えば、Express では`response.status(200).send()`のようなコードでレスポンスを構築できます。

> 警告
> Nest はハンドラーが`@Res()`または`@Next()`を使用していることを検出すると、ライブラリ固有のオプションを選択したと判断します。両方のアプローチを同時に使用すると、このルートに対して標準アプローチは自動的に無効になり、期待通りに動作しなくなります。両方のアプローチを同時に使用するには（例：クッキー/ヘッダーの設定のみにレスポンスオブジェクトを注入し、残りはフレームワークに任せる場合）、`@Res({ passthrough: true })`デコレータで passthrough オプションを true に設定する必要があります。

## リクエストオブジェクト

ハンドラーは多くの場合、クライアントリクエストの詳細にアクセスする必要があります。Nest は基盤となるプラットフォーム（デフォルトでは Express）のリクエストオブジェクトへのアクセスを提供します。ハンドラーのシグネチャに`@Req()`デコレータを追加することで、Nest にリクエストオブジェクトを注入するよう指示できます。

```typescript
// cats.controller.ts
import { Controller, Get, Req } from "@nestjs/common";
import { Request } from "express";

@Controller("cats")
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return "This action returns all cats";
  }
}
```

> ヒント
> Express 用の型定義（上記の例の`request: Request`パラメータなど）を利用するには、`@types/express`パッケージをインストールしてください。

リクエストオブジェクトは HTTP リクエストを表し、クエリ文字列、パラメータ、HTTP ヘッダー、本文などのプロパティを持っています。ほとんどの場合、これらのプロパティを手動で取得する必要はありません。代わりに、すぐに使える`@Body()`や`@Query()`などの専用デコレータを使用できます。以下は、提供されているデコレータと、それらが表すプラットフォーム固有のオブジェクトの一覧です：

- `@Request()`、`@Req()` → `req`
- `@Response()`、`@Res()`\* → `res`
- `@Next()` → `next`
- `@Session()` → `req.session`
- `@Param(key?: string)` → `req.params` / `req.params[key]`
- `@Body(key?: string)` → `req.body` / `req.body[key]`
- `@Query(key?: string)` → `req.query` / `req.query[key]`
- `@Headers(name?: string)` → `req.headers` / `req.headers[name]`
- `@Ip()` → `req.ip`
- `@HostParam()` → `req.hosts`

* 基盤となる HTTP プラットフォーム（Express や Fastify など）間での型の互換性のため、Nest は`@Res()`と`@Response()`デコレータを提供しています。`@Res()`は単に`@Response()`のエイリアスです。両者とも、基盤となるネイティブプラットフォームのレスポンスオブジェクトインターフェースを直接公開します。

## リソース

先ほど、cats リソースを取得するエンドポイント（GET ルート）を定義しました。通常、新しいレコードを作成するエンドポイントも提供したいと思います。そのために、POST ハンドラーを作成しましょう：

```typescript
// cats.controller.ts
import { Controller, Get, Post } from "@nestjs/common";

@Controller("cats")
export class CatsController {
  @Post()
  create(): string {
    return "This action adds a new cat";
  }

  @Get()
  findAll(): string {
    return "This action returns all cats";
  }
}
```

これだけです。Nest は標準的な HTTP メソッド全てにデコレータを提供しています：`@Get()`、`@Post()`、`@Put()`、`@Delete()`、`@Patch()`、`@Options()`、`@Head()`です。さらに、`@All()`はこれら全てを処理するエンドポイントを定義します。

## ワイルドカードルート

NestJS ではパターンベースのルートもサポートされています。例えば、アスタリスク（\*）をワイルドカードとして使用して、パスの末尾の任意の文字の組み合わせにマッチさせることができます。

以下の例では、`findAll()`メソッドは、`abcd/`で始まるルートに対して、その後に続く文字数に関係なく実行されます：

```typescript
@Get('abcd/*')
findAll() {
  return 'This route uses a wildcard';
}
```

`'abcd/*'`というルートパスは、`abcd/`、`abcd/123`、`abcd/abc`などにマッチします。ハイフン（-）とドット（.）は文字列ベースのパスでは文字通りに解釈されます。

ルート中央でアスタリスクを使用する場合、Express は名前付きワイルドカード（例：`ab{*splat}cd`）を必要としますが、Fastify はこれを全くサポートしていません。

## ステータスコード

前述のように、レスポンスのステータスコードは、POST リクエスト（201）を除いて、デフォルトで常に 200 です。ハンドラーレベルで`@HttpCode(...)`デコレータを追加することで、この動作を簡単に変更できます：

```typescript
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```

> ヒント
> `HttpCode`は`@nestjs/common`パッケージからインポートします。

多くの場合、ステータスコードは静的ではなく、様々な要因に依存します。その場合、ライブラリ固有のレスポンスオブジェクト（`@Res()`を使用して注入）を使用するか、エラーの場合は例外をスローすることができます。

## ヘッダー

カスタムレスポンスヘッダーを指定するには、`@Header()`デコレータを使用するか、ライブラリ固有のレスポンスオブジェクト（`res.header()`を直接呼び出す）を使用します：

```typescript
@Post()
@Header('Cache-Control', 'no-store')
create() {
  return 'This action adds a new cat';
}
```

> ヒント
> `Header`は`@nestjs/common`パッケージからインポートします。

## リダイレクト

レスポンスを特定の URL にリダイレクトするには、`@Redirect()`デコレータを使用するか、ライブラリ固有のレスポンスオブジェクト（`res.redirect()`を直接呼び出す）を使用できます。

`@Redirect()`は 2 つの引数（`url`と`statusCode`）を取り、どちらもオプションです。省略した場合、`statusCode`のデフォルト値は 302（Found）です：

```typescript
@Get()
@Redirect('https://nestjs.com', 301)
```

## サブドメインルーティング

`@Controller`デコレータは`host`オプションを取ることができ、これにより着信リクエストの HTTP ホストが特定の値に一致することを要求できます：

```typescript
@Controller({ host: "admin.example.com" })
export class AdminController {
  @Get()
  index(): string {
    return "Admin page";
  }
}
```

> 警告
> Fastify はネストされたルーターをサポートしていないため、サブドメインルーティングを使用する場合は、デフォルトの Express アダプターを使用することをお勧めします。

ルートパスと同様に、`hosts`オプションでもトークンを使用してホスト名のその位置の動的な値を取得できます。以下の`@Controller()`デコレータの例でこの使用法を示します。このように宣言されたホストパラメータには、メソッドのシグネチャに追加する`@HostParam()`デコレータを使用してアクセスできます：

```typescript
@Controller({ host: ":account.example.com" })
export class AccountController {
  @Get()
  getInfo(@HostParam("account") account: string) {
    return account;
  }
}
```

## スコープ

異なるプログラミング言語のバックグラウンドを持つ人々にとって、Nest ではほとんどすべてのものが着信リクエスト間で共有されていることを知るのは予想外かもしれません。データベースへの接続プール、グローバル状態を持つシングルトンサービスなどがあります。Node.js は、各リクエストが別のスレッドで処理されるマルチスレッドステートレスモデルに従っていないことを覚えておいてください。したがって、シングルトンインスタンスの使用は、私たちのアプリケーションにとって完全に安全です。

ただし、コントローラーのリクエストベースのライフタイムが望ましい動作となるエッジケースがあります。例えば、GraphQL アプリケーションでのリクエストごとのキャッシュ、リクエストトラッキング、マルチテナンシーなどです。スコープの制御方法については[こちら]で学ぶことができます。

## 非同期性

私たちは現代の JavaScript を愛しており、データの抽出のほとんどが非同期であることを知っています。そのため、Nest は非同期関数をサポートし、うまく連携します。

> ヒント
> async/await 機能についての詳細は[こちら]で学べます。

すべての非同期関数は Promise を返す必要があります。これは、Nest が自身で解決できる遅延値を返すことができることを意味します。例を見てみましょう：

```typescript
@Get()
async findAll(): Promise<any[]> {
  return [];
}
```

上記のコードは完全に有効です。さらに、Nest のルートハンドラーはより強力で、RxJS の Observable ストリームを返すこともできます。Nest は自動的にソースをサブスクライブし、（ストリームが完了したら）最後に発行された値を取得します：

```typescript
@Get()
findAll(): Observable<any[]> {
  return of([]);
}
```

上記の両方のアプローチが機能し、要件に合わせて使用することができます。

## リクエストペイロード

これまでの POST ルートハンドラーの例では、クライアントのパラメータを受け取っていませんでした。ここで`@Body()`デコレータを追加して、この問題を解決しましょう。

ただし最初に（TypeScript を使用している場合）、DTO（Data Transfer Object）スキーマを定義する必要があります。DTO は、データがネットワーク上でどのように送信されるかを定義するオブジェクトです。DTO スキーマは TypeScript インターフェースか、単純なクラスを使用して定義できます。興味深いことに、ここではクラスの使用を推奨します。なぜでしょうか？

- クラスは JavaScript ES6 標準の一部であり、コンパイルされた JavaScript で実際のエンティティとして保持されます。
- 一方、TypeScript インターフェースはトランスパイル時に削除されるため、Nest は実行時にそれらを参照できません。
- これは重要です。なぜなら、Pipes などの機能は、実行時に変数のメタタイプにアクセスできる場合に追加の可能性を提供するためです。

では、CreateCatDto クラスを作成してみましょう：

```typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

これには 3 つの基本的なプロパティしかありません。その後、新しく作成した DTO を CatsController で使用できます：

```typescript
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}
```

> ヒント
> ValidationPipe は、メソッドハンドラーが受け取るべきでないプロパティをフィルタリングできます。この場合、受け入れ可能なプロパティをホワイトリストに登録でき、ホワイトリストに含まれていないプロパティは結果のオブジェクトから自動的に削除されます。CreateCatDto の例では、ホワイトリストは name、age、breed プロパティです。詳細は[こちら]で学べます。

## エラー処理

エラーの処理（つまり、例外の扱い）については、[こちら]の別の章で説明しています。

## 完全なリソースサンプル

以下は、基本的なコントローラーを作成するために利用可能な様々なデコレータを使用した例です。このコントローラーは、内部データにアクセスし操作するためのいくつかのメソッドを公開しています：

```typescript
import {
  Controller,
  Get,
  Query,
  Post,
  Body,
  Put,
  Param,
  Delete,
} from "@nestjs/common";
import { CreateCatDto, UpdateCatDto, ListAllEntities } from "./dto";

@Controller("cats")
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return "This action adds a new cat";
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(":id")
  findOne(@Param("id") id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(":id")
  update(@Param("id") id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(":id")
  remove(@Param("id") id: string) {
    return `This action removes a #${id} cat`;
  }
}
```

> ヒント
> Nest CLI は、すべてのボイラープレートコードを自動生成してくれるジェネレーター（スキーマ）を提供しており、これらの作業を避け、開発者体験をよりシンプルにすることができます。この機能についての詳細は[こちら]で読むことができます。

## 始めるために

上記のコントローラーが完全に定義されていても、Nest はまだ CatsController が存在することを知らず、結果としてこのクラスのインスタンスを作成しません。

コントローラーは常にモジュールに属しているため、`@Module()`デコレータ内の`controllers`配列に含めます。root AppModule 以外のモジュールをまだ定義していないため、CatsController を導入するために AppModule を使用します：

```typescript
// app.module.ts
import { Module } from "@nestjs/common";
import { CatsController } from "./cats/cats.controller";

@Module({
  controllers: [CatsController],
})
export class AppModule {}
```

`@Module()`デコレータを使用してモジュールクラスにメタデータを付加し、これにより Nest は簡単にどのコントローラーをマウントする必要があるかを認識できます。

## ライブラリ固有のアプローチ

ここまでは、レスポンスを操作する Nest の標準的な方法について説明してきました。レスポンスを操作する 2 つ目の方法は、ライブラリ固有のレスポンスオブジェクトを使用することです。特定のレスポンスオブジェクトを注入するには、`@Res()`デコレータを使用する必要があります。違いを示すために、CatsController を以下のように書き直してみましょう：

```typescript
import { Controller, Get, Post, Res, HttpStatus } from "@nestjs/common";
import { Response } from "express";

@Controller("cats")
export class CatsController {
  @Post()
  create(@Res() res: Response) {
    res.status(HttpStatus.CREATED).send();
  }

  @Get()
  findAll(@Res() res: Response) {
    res.status(HttpStatus.OK).json([]);
  }
}
```

このアプローチは機能し、実際にレスポンスオブジェクトの完全な制御（ヘッダーの操作、ライブラリ固有の機能など）を提供することで、ある意味でより柔軟性を提供しますが、注意して使用する必要があります。

一般的に、このアプローチはより明確さに欠け、いくつかの欠点があります：

- コードがプラットフォーム依存になる（基礎となるライブラリによってレスポンスオブジェクトの API が異なる可能性がある）
- テストが難しくなる（レスポンスオブジェクトのモックが必要になるなど）

また、上記の例では、Interceptors やデコレータなどの Nest の標準的なレスポンス処理に依存する機能との互換性が失われます。これを修正するには、以下のように passthrough オプションを true に設定できます：

```typescript
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.status(HttpStatus.OK);
  return [];
}
```

これで、ネイティブのレスポンスオブジェクトと対話できる（例：特定の条件に応じてクッキーやヘッダーを設定する）一方で、残りの処理はフレームワークに任せることができます。
