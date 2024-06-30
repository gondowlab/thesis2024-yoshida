## 進捗報告(6/17)

## 今週やったこと

- 論文の調査
  - Put Your Memory in Order: Efficient Domain-based Memory Isolation for WASM Applications(CCS'23)
  - Research on WebAssembly Runtimes: A Survey(arXiv'24)

## PKUWA[CCS'23]

### 目的

- intel MPKを利用してWasmのリニアメモリの分割を行う.

### Wasmのリニアメモリ構成

<img src="https://hackmd.io/_uploads/BktnnHFSR.png" width=60%> <br>

- リニアメモリは、ランタイムが予約された仮想メモリ空間からマップする単一の連続したバイト配列
- 主にヒープ部、データ部、スタック部の3つの部分から構成される
- データセクションは異なる権限のデータ（読み取り専用データと書き込み可能データなど）を区別せず、このセクションのデータはすべて読み取り可能.
- 異なるセクション間にマッピングされていないメモリがなく、スタックセクションからデータセクションまでの空間は連続している.

**脆弱なコード例:**

```c=
// Somewhere in the binary
char buf[32];
// Stack-based buffer overflow
scanf("%[^\n]", buf);
...
// Write "constant" string into "constant" file
FILE *f = fopen("file.txt", "a");
fprintf(f, "Append constant text.");
fclose(f);
```

### intel MPKのwasmへの適用

- MPKはページ単位でのアクセス権限の管理を行なっており、Wasmはmmapとページアラインドメモリをサポートしていないため、Wasmのリニアメモリと互換性がない.
- 論文では新しいメモリ分離モデルを導入し、Wasmのリニアメモリとページベースの保護モデルのミスマッチに対処する.

### intel MPK(Memory Protection Keys)

- プロセス内のメモリアクセス権限を効率的に制御する仕組み
- protection keyと呼ばれるページテーブルエントリ(PTE)内のフィールドと、Appから触れられるPKRUレジスタを使用する.
- https://logmi.jp/tech/articles/324700
- **Protection key**
  - page tabelを拡張したフィールドに設定
  - 最大16($4^2 = 16$)個のキーを用いて、各ページテーブルをタグ付けできる.

![図1](https://hackmd.io/_uploads/rJRl5i5BA.png)

- **PKRUレジスタ** - 各protection keyに対応するアクセス権限を設定するレジスタ - メモリ・アクセスのたびにPKRUとPTEのprotection keyを比較し、許可をチェックする. - 16(protection key) \* 2(Access Disable, Write Disable)で設定する.
  ![スクリーンショット 2024-06-15 14.53.21](https://hackmd.io/_uploads/SyYinjqr0.png)

### リニアメモリのドメイン分割

![スクリーンショット 2024-06-15 20.53.47](https://hackmd.io/_uploads/HyP6gbsSA.png)

- リニアメモリを複数のドメインに分割し、各ドメインへの関数のアクセス権を制御する.
- intel MPKを利用してリニアメモリの各セグメントを最大16個のドメインに分割する.

## 研究テーマ

ラインタイムのSpectre防御の効率化

- Wasmバイナリに対するSpectreガジェット探索

  - 既存のSpectreガジェット探索をWasmに適用できるようにする
  - ファジングやコンコリック実行をWasmに適用する研究は既に存在している
  - 課題が見当たらず、テーマになるのか微妙

- PKUWA[CCS'23]をSpectre防御できるように拡張する
  - Swivelと同様にFaaSプラットフォームでのWasmの利用を想定
  - intel MPKを使用して外部のサンドボックスのリニアメモリをアクセス不可能にする.
  - 単純な解決だと、(ランタイム上のサンドボックスの数) \* (各リニアメモリ内のセグメント数)だけPkeyを使ってドメインを分割すれば良い.
    - 16個では足りないのでlibmpkなどを使うか?
  - ガジェット探索をして脆弱な関数を特定のドメインにのみアクセスできるようにする.
    - PKUWAを自動化させただけ？

### 今後の予定

- 研究のアイデアを深ぼる
