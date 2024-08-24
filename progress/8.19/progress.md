## 進捗報告(8/19)

### 先週やったこと

- Motivating Exampleの改良
- アノテーションの付け方の調査
- 中間発表のスライド構成を考える

### Motivating Example

- victim_functionが繰り返し実行されることで大きなオーバーヘッドが発生.
- 16行目が秘密情報にアクセスしないなら、15行目と22行目を削除して良い.

```c=
#define ARRAY1_SIZE 16

typedef struct {
    uint8_t array1[ARRAY1_SIZE];
    uint8_t not_secret_data[100];
    uint8_t secret_data[100];
} MyData;

MyData data;
uint8_t array2[256 * 512];
uint8_t temp = 0;

void victim_function(int x) {  // 0 <= x <= 100
    if (x < ARRAY1_SIZE) {
        register_interlock();
        temp &= array2[data.array1[x] * 512];
    }
}

void foo(int a) {
    while (0 <= a && a <= 100) {
        register_interlock();
        victim_function(a);
        a = a + 1;
    }
}
```

### 手動でのアノテーション情報埋め込み

- wat2wasmで`--enable-annotations`フラグを有効にすることで、オリジナルのカスタムセクションを埋め込める.
- カスタムセクションに秘密情報のシンボル情報(どの関数のどの変数か)を埋め込む.
  - wasmでは関数やローカル変数はインデックス番号で扱う.
  - サブセクション名は`secret_関数インデックス`
  - 各サブセクションはアノテーションを付与した変数のインデックス列を持つ
- 一度WAT形式に変換すると、デバッグ情報が失われているのが怖い
  - デバッグ情報を保持するオプションがあるのか？

WAT形式

```=
(module
  (@custom "secret_0" "0,1")  ;; オリジナルのカスタムセクション追加
  (func $addTwoLocals (result i32)
    (local $var1 i32)
    (local $var2 i32)
    (local.set $var1 (i32.const 5))
    (local.set $var2 (i32.const 10))
    local.get $var1
    local.get $var2
    i32.add
    return
  )
)
```

WASMバイナリ

```=
Contents of section Custom:
000002b: 0873 6563 7265 745f 3030 2c31            .secret_00,1
```

### attributeによるアノテーション付与(Clang)

- 関数に対しての属性付与はwasmまで落ちることをLLVMの実装から確認.
  - カスタムセクション内に"secret"サブセクションが作成され、属性を付与した関数のインデックスが入る.
- 変数に対しても同様にやれば、比較的簡単に実装できるか？

C++

```=c
#define SECRET [[clang::annotate("secret")]]

int foo(int param) {
  int var = param;
  return var;
}

SECRET int var(int param, int param2) {
  int var = param + param2;
  return var;
}

SECRET int fuzz(int param, int param2, int param3) {
  int var = param + param2 + param3;
  return var;
}

int main() {
  int hello = foo(10);
  int goodby = var(30, 40) + fuzz(10, 20, 30);
  return 0;
}
```

Wasm

```=
Contents of section Custom:
0004dd5: 1e6c 6c76 6d2e 6675 6e63 5f61 7474 722e  .llvm.func_attr.
0004de5: 616e 6e6f 7461 7465 2e73 6563 7265 7405  annotate.secret.
0004df5: 0000 0004 0000 00                        .......
```

### 中間発表構成

- 研究概要
  - 目的、既存研究の課題、提案手法
- 背景
  - Wasm, Spectre攻撃の概要
  - Wasm実行環境におけるSpectre攻撃
- Swivelにおける防御手法
  - Register interlocking
  - 問題点
    - MotivatingExample
    - オーバーヘッドの大きさを示すグラフ
- 提案手法
  - 秘密情報にアノテーションを付与
  - テイント解析、区間抽象解釈の利用
  - 実装の全体像
- 現在の進捗状況

### 今週の予定

- ~~アノテーション付与の実装~~
  - LLVMバックエンド、Wasmtimeフロントエンドの修正
  - 無理そうなら手動でのアノテーション方法などに切り替え
- 中間発表スライド作成
- Wasm(またはCranelift IR)を解析する上での技術的課題を調査する
  - Tough Call [ISSTA '23]
  - oo7を Cranelift IRに単純適用してみて、上手くいかない理由を調査
  - Motivating Exampleを技術的課題を含めるように改良
