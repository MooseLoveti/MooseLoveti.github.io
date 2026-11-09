---
title: C++ の constexpr（コンパイル時に値を計算する）
date: 2026-11-07
category: Dev
topic: C++
tags: [dev, cpp, constexpr, reversing]
summary: constexpr はコンパイル時に値を計算できることを示す指定子。static_assert と組み合わせると、条件を満たさないとコンパイルエラーになるチェックが書ける
---

## 概要

`constexpr` は、その変数や関数の値をコンパイル時に計算できることを示す C++ の指定子（C++11 から）。

- `constexpr` 変数: 値がコンパイル時に確定していなければならない。
- `constexpr` 関数: 引数がコンパイル時に分かっていれば、コンパイル時に計算される（実行時に普通の関数として呼ぶこともできる）。必ずコンパイル時に計算させたいときは、C++20 の `consteval` を使う。

```cpp
constexpr int square(int x) { return x * x; }

constexpr int a = square(4);        // コンパイル時に 16 になる
static_assert(a == 16, "wrong");    // コンパイル時に検査。偽ならコンパイルエラー
```

`static_assert` はコンパイル時に条件を検査し、偽ならコンパイルを失敗させる。`constexpr` で計算した値と組み合わせると、「正しい入力のときだけコンパイルが通る」プログラムが書ける。

## リバーシングでの見方

- 入力（文字列など）を `constexpr` で変換し、`static_assert` で期待値と比べる形のチェッカーは、判定の処理がすべてソースコードの中にあり、実行ファイルには残らない。ソースを読み、同じ計算を Python などで再現して逆算する。
- テンプレートや `std::integer_sequence` を使った計算は読みにくいが、やっていることは配列の計算であることが多い。1 文字ずつの値を並べた配列として読み替えると整理しやすい。
