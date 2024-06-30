## 進捗報告(6/3)

### 予定

- wasm(特にWasmtimeにおいて)のコンパイル、実行がどのように行われるか調査する
  - マスク処理、ガードページの実装
  - returnアドレス保護のためのスタックの分離
  - Spectre攻撃の対策
- 研究のアイデアについて考える
  - ユーザからアノテーションやソースコード上の情報を利用することでパフォーマンスを改善できないか？
  - Swivelではコンパイル時に、投機的な制御フローにおいても安全なメモリアクセスが行われるようなコードを生成するが、安全性は十分か？(That's a Tough Call[ISSTA'23])
  - Branch Target Buffer(BTB)をコンテキストスイッチの度にフラッシュするのはパフォーマンス上良くないのでは？

### Wasmtimeの調査

Wasmからコンパイルされたコードは小さなメモリ領域(リニアメモリ)が与えられ、すべてのリニアメモリアクセスは境界チェックされる.

#### Wasmtimeのメモリモード

参考:
https://docs.wasmtime.dev/contributing-architecture.html
https://docs.rs/wasmtime/latest/wasmtime/struct.Config.html

- static memory(static heap)

  - デフォルトの設定であり、コンパイル時に必要なメモリ領域が全て予約される.
  - メモリ領域が拡張されることがないので、ベースアドレスは固定である.
  - 4GBの予約済みアドレス領域があり、その後に、2GBのガードページが続きアクセス時にトラップが発生することが保証されている.
  - **境界外へのアドレスがガードページに遭遇することが証明された場合、生成されるコードにおいて境界チェックは省略されるが、そうでない場合は境界チェックを行うコードが生成される**
  - 境界チェックは単純な条件分岐命令(**Spectreでバイパスすることが可能か**)

- dynamic memory(dynamic heap)
  - コンパイル時にメモリ領域のサイズが確定しておらず、実行時に拡張が可能.
  - 環境がリニアメモリ用に6GBの仮想メモリ空間を確保できず、メモリを動的に割り当てたい場合に使用.
  - ヒープを拡張したときに異なるベースアドレスに再配置される場合がある.
  - **全てのパターンにおいて境界チェックを行うコードが発行される.**
  - ガードページもヒープのサイズが変更されたときに移動する.

#### Spectreに対する対策

- heap_access_spectre_mitigationという環境変数によって、ヒープアクセスに対するSpectre対策のコードが生成されるか決まる.
  - https://github.com/bytecodealliance/wasmtime/blob/23409ca7e184ef3b7d42fe17938caed470643d5d/cranelift/filetests/src/test_wasm/env.rs#L19-L24
- 適用する場合、境界チェックの条件に依存したセレクト命令が生成される。境界チェックが失敗した場合はnull、それ以外の場合はヒープアドレスを選択する.
- SLHと同じだと思われる

```c=
void victim_function(size_t x) {
  if (x < array1_size) {
    // 新たなデータ依存関係を作り、投機実行を抑制
    ptr = (x < array1_size) ? &array1[x] : NULL;
    secret = *ptr
    temp = array2[secret];
  }
}
```

- **ドキュメントの内容に矛盾する？**
  https://docs.wasmtime.dev/security.html?highlight=Spectre#spectre

  > Wasmtime's default configuration for linear memory means that bounds checks will not be present for memory accesses due to the reliance on page faults to instead detect out-of-bounds accesses. When Wasmtime is configured with "dynamic" memories, however, Cranelift will insert spectre mitigation for the bounds checks performed for all memory accesses.

- Swivelで述べられているマスク処理に関する記述が見当たらない

#### 生成されたアセンブリの確認

以下のコードをEmscripten + Wasmtime(Cranelift)でコンパイル

```=c
#include <stddef.h>
#include <stdint.h>

unsigned int array1_size = 16;
uint8_t unused1[64];
uint8_t array1[160] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16};

uint8_t unused2[64];
uint8_t array2[256 * 512];

char *secret = "This is the secret word.";

uint8_t temp = 0;

void victim_function(size_t x) {
  if (x < array1_size) {
    temp &= array2[array1[x] * 512];
  }
}
```

結果: file:///Users/yoshidakota/Applications/wasm_c/hello.explore.html

### 研究のアイデア

- WasmtimeのSpectre防御の効率化
  - WasmtimeのSpectre対策は全てのヒープアクセスに対して適用される.
  - ガジェット探索を行い、攻撃者が悪用可能なヒープアクセスに対してのみ適用されるようにする
  - Spectre-PHTのみをターゲットにする予定
- Swivelよりも軽量かつ高いポータビリティを目指したい
  - Swivelはマスク処理をバイパスする投機的なパスを禁止するために、コンパイラを改造している. その代わり、広範囲なSpectre亜種の防御が可能.
  - アイデアは軽量かつ様々なランタイムに適用できるものにしたい.

### 今後の予定

- 生成されたアセンブリを調査したい
  - 自分の調査結果に合致するコードが生成されているか確認
  - Swivelで述べられているマスク処理が本当にあるのか
- 研究のアイデアを深ぼる
