## 進捗報告(6/10)

### 予定

- 研究のアイデアを深ぼる
- Spectre防御の効率化に関する論文を調べる

### 研究のアイデア

- WasmランタイムのSpectre防御の効率化
  - WasmtimeのSpectre対策は全てのヒープアクセスに対して適用される
    - 他のランタイムでも似たような保守的な対策をしていると予想
  - Spectreガジェット探索を行い、攻撃者が悪用可能なヒープアクセスに対してのみ防御手法が適用されるようにする
- Swivelよりも軽量かつ高いポータビリティを目指したい
  - Swivelはマスク処理をバイパスする投機的なパスを禁止するために、コンパイラを改造している. その代わり、広範囲なSpectre亜種の防御が可能.
  - アイデアは軽量かつ様々なランタイムに適用できるものにしたい.
  - Wasmバイナリに対してSpectreガジェット探索を行うことができれば、多くのランタイムで利用できるのでは？

### 眺めた論文

- Fuzzm: Finding Memory Bugs through Binary-Only Instrumentation and Fuzzing of WebAssembly(arXiv'21)
  - Wasmバイナリ向けのファザーを使用してメモリ脆弱性を検出する
  - AFLをWasmバイナリ向けに改良
    - Wasmプログラムの実行中にAFL互換のカバレッジ情報を収集できるようにする
    - ランタイム上で実行されるWasmの解析を既存のAFLと比較して十分早いものにする

### 既存のSpectreガジェット探索

|                |  手法  | Variant | 　Scalability | 精度 | LVI,MDS,PORT | OSS | 　キャッシュモデル化 |
| :------------: | :----: | :-----: | :-----------: | :--: | :----------: | :-: | :------------------: |
|     oo7[1]     |  STA   |   PHT   |       ✕       |  ✕   |      ✕       |  ◯  |         　-          |
| Spectector[2]  |   SE   |   PHT   |       ✕       |  ✕   |      ✕       |  ✕  |         　-          |
| KLEESpectre[3] |   SE   |   PHT   |       ✕       |  ✕   |      ✕       |  ◯  |         　◯          |
|  SpecFuzz[4]   |  F　   |   PHT   |       ◯       |  △   |      ✕       |  ◯  |         　✕          |
|  SpecTaint[5]  | F, DTA |   PHT   |       ◯       |  △   |      ✕       |  ◯  |         　-          |
|   Kasper[6]    | F, DTA |   PHT   |       ◯       |  △   |      ◯       |  ◯  |         　✕          |

### メモ

- 単純にWasmバイナリに対して既存のSpectreガジェット探索を適用するだけでは、研究テーマにならないのでは？
- wasmランタイムのSpectre防御効率化という線は良いと思う
- intel MPKを使用して外部のサンドボックスを読み取れないようにできないか？

  - こうすれば、SLHを挿入する必要がなくなる
  - swivelですでにやられていることでは?
  - 静的解析することで、ドメインの切り替えを最小限にできないか？
  - もしくはPKUWAを使用してより細かなメモリ分割をできないか？(秘密の部分だけ外部からアクセス不可にするなど)

- 各サンドボックスに対して個別のPkeyを対応しなければならず、最大１６個までしか対応できない(他のドメインのデータに不正にアクセスすることを防ぐため).

### 今後の予定

- 研究のアイデアを深ぼる

### 関連研究

[1] Low-overhead defense against spectre attacks via program analysis(IEEE Transactions on Software Engineering'19)
[2] Spectector: Principled detection of speculative information flows(SP'20)
[3] KLEESpectre: Detecting information leakage through speculative cache attacks via symbolic execution(TOSEM'20)
[4] SpecFuzz: Bringing Spectre-type vulnerabilities to the surface(USENIX'20)
[5] SpecTaint:　Speculative Taint Analysis for Discovering Spectre Gadgets(NDSS'21)
[6] KASPER: Scanning for Generalized Transient Execution Gadgets in the Linux Kernel(NDSS'22)
