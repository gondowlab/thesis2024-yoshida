## 進捗報告(6/17)

### 今週やったこと

- 論文紹介の準備
  - Fuzzm: Finding Memory Bugs through Binary-Only Instrumentation and Fuzzing of WebAssembly (arXiv'21)
- 論文調査
  - That's a Tough Call: Studying the Challenges of Call Graph Construction for WebAssembly (ISSTA'23)

### コールグラフ構築の課題

- https://hackmd.io/N9hYOS6QSnGoHyI7_R13Yw?both
- 型が少ない
- ホストコードの解析が必要
- ソースコードレベルの概念を表現する構造がない

### 研究テーマ

ラインタイムのSpectre防御の効率化

#### ~~Wasmバイナリに対するSpectreガジェット探索~~

#### PKUWA[CCS'23]をSpectre防御できるように拡張する

- 問題提起
  - 既存のランタイム(wasmtimeなど)では、全てのリニアメモリアクセスに対し、Spectre防御を適用する.
  - かつそれらは投機実行を抑制するため、実行時のオーバーヘッドが大きい
- 提案手法
  - PKUWAを利用してSpectre脆弱性を含む関数のメモリアクセス範囲を限定する
  - 投機実行は抑制しないが、メモリアクセス時に権限のチェックが必要になる(どちらが早いのかは疑問)
- PKUWAの問題
  - Spectreに対処できていないらしい
  - 手動で関数のメモリアクセス範囲を指定する必要がある
    - 大規模なアプリケーションの場合、Spectre脆弱性を含む関数を手動で見つけるのは大変
    - ガジェット探索と組み合わせて使うのが良いかも

### 今後の予定

- PKUWA[CCS'23]をちゃんと理解する
  - Spectreに対処できていないのか？
- 研究のアイデアの深掘り
