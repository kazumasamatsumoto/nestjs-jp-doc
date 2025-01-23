# 非同期性の補足資料

JavaScript の Node.js 実行時のライフサイクル：

![全体の流れ](/overview/svg/2_1_JavaScript-Event-Loop/js-execution-flow.svg)

1. コード読み込み

- JavaScript → Parser → AST（抽象構文木）作成

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

![JavaScript サンプル](/overview/svg/2_1_JavaScript-Event-Loop/v8-full-pipeline.svg)

## JavaScript の非同期処理の実装フロー：

![JavaScriptの非同期処理のイメージ](/overview/svg/2_1_JavaScript-Event-Loop/async-split-flow.svg)

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

![JavaScriptの非同期処理のイメージ](/overview/svg/2_1_JavaScript-Event-Loop/fetch-data-stack.svg)
