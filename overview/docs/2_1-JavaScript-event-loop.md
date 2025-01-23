# 非同期性の補足資料

JavaScript の Node.js 実行時のライフサイクル：

1. コード読み込み

- JavaScript → Parser → AST（抽象構文木）作成

![JavaScript コード読み込み](/overview/svg/2_1_JavaScript-Event-Loop/js-parsing-flow.svg)

2. コンパイル

- AST → Ignition（V8 のインタプリタ）が Bytecode を生成
- Bytecode 実行開始

3. 最適化

- TurboFan（最適化コンパイラ）がホットコードを検出
- Bytecode → 中間表現（IR） → 機械語に変換

4. 実行

- 最適化された機械語コードを実行
- 非最適化コードは Bytecode のまま実行
- 必要に応じて最適化/最適化解除を繰り返す

## JavaScript の非同期処理の実装フロー：

1. コードの記述

```javascript
// 非同期関数の宣言
async function fetchData() {
  console.log("開始");

  // 非同期処理
  const result = await someAsyncOperation();

  console.log("完了");
  return result;
}
```

2. V8 エンジンの処理

- コードをパース → AST 生成
- Bytecode に変換
- イベントループへの登録

3. 実行時の流れ

```javascript
// メインスレッド
console.log("処理1"); // 同期処理

fetchData() // 非同期処理
  .then((data) => {
    console.log(data);
  });

console.log("処理2"); // 同期処理
```

出力順序：

```
処理1
開始
処理2
完了
[データ]
```

このフローで V8 は：

1. 同期コードを即時実行
2. 非同期処理をキューに登録
3. イベントループで非同期処理を管理
4. 完了時にコールバックを実行
