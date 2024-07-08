## 進捗報告(7/8)

### 今週やったこと

- libmpk[USENIX' 19]の調査

### ドメイン数有限問題

- Spectre脆弱性を含む関数はそれぞれ個別のドメインを割り当てたいが、intel MPKは最大１６個のドメインしか作成できない
- libmpkを使用することで、制限なくドメインを作成できる

### libmpkの調査

- libmpkを使用することで、どの程度のオーバーヘッドがあるか？
- それらはプログラム解析で解決できるか？
- 工夫してドメイン数を減らせないか？
  - 良い方法が思いつかない

### タイトル案

- WebAssemblyランタイムにおけるSpectre攻撃に対する防御の効率化
- WebAssemblyにおけるSpectre攻撃の効率的な防御

### 今後の予定

- ドメイン数削減について考える.
- EPK: Scalable and Efficient Memory Protection Keys [USENIX'22]の調査.
  - libmpkより効率的なドメイン分離手法
- 保護対象について考える.
  - Wasmモジュールだけで良いのか？
