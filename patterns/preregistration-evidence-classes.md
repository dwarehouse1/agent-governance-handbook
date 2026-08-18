# 事前登録と証拠クラス — Preregistration & Evidence Classes

> **EN:** Classify every result by how it was produced — exploratory, locked/walk-forward, or live — and allow promotion in one direction only. An agent's impressive backtest is exploratory by definition and can never be promoted by re-running it on the same data.

## 解決する問題

エージェントは「結果を見てから基準を調整する」ことを、悪意なく、気づかれずに行う。探索中に見えた良い結果と、事前に固定した基準で出た良い結果は、**同じ数値でも証拠としての価値がまったく違う**。この区別を運用に埋め込まないと、探索的なまぐれが確証として意思決定に流れ込む。

## パターン

### 証拠クラス(3段階)

```text
EXPLORATORY   — 結果閲覧後の分析を含む一切。仮説生成にのみ使用可。
                確証・投入判断の根拠として使用禁止。
LOCKED        — 仮説・パラメータ・評価関数・kill基準を固定(ハッシュロック)した後、
                ロック時点より未来のデータ/入力のみで評価したもの。
LIVE          — 事前登録済みプロトコルでの実運用記録。
                実コスト込みの唯一の一次証拠。
```

- 昇格は EXPLORATORY → LOCKED → LIVE の**一方向のみ**
- 結果を見た後の基準変更・仮説の狭め直しは、新IDの新仮説として最初から登録する(claim narrowing禁止)
- どのクラスの証拠かを、すべての数値報告に必ず添える

### 事前登録

- 仮説は検証前にレジストリへ登録し、仕様(対象・定義・評価統計量・コストモデル・kill閾値)を記述したspecファイルの**sha256を記録**する。ロック後のspec変更は改竄として検出される
- 登録前に同一データで行った予備計算は、すべてEXPLORATORYとして申告する

### レジストリの最小スキーマ

```csv
hypothesis_id,family,registered_utc,spec_path,spec_sha256,evidence_class,status,kill_condition_summary
```

## エージェント運用での使い方

- エージェントが出したすべての数値に証拠クラスをラベルさせる。ラベルのない数値は EXPLORATORY として扱う(fail-closed)
- specのロックは人間が行う(ハッシュを取ってレジストリに記録)。エージェントにロックとロック対象の生成を両方任せると、ロックの意味が消える
- LOCKED評価を走らせる実装には、ロック日時・spec sha256を定数として埋め込み、出力(scorecard)に自動で刻印させる
- 機構のテスト実行(ロック前データでの smoke test)には「これは証拠ではない」ラベルを強制的に付与し、保存経路を分ける

## 失敗モード

- **暗黙の再ロック**: 「specを少し直してもう一度ロック」を繰り返すと、実質的に結果を見ながらの探索になる。spec改訂は新IDで
- **クラスのインフレ**: 探索結果をスライドや記事で引用するうちにLOCKED相当として扱われだす。引用時のクラス明記を規約にする
- **ロックだけして未来を待たない**: ロック後、過去データへの当てはめで「walk-forwardした」と称する。LOCKEDの定義は「ロック時点より未来の入力のみ」

## テンプレート

- [templates/HYPOTHESIS_REGISTRY.template.csv](../templates/HYPOTHESIS_REGISTRY.template.csv)
- [templates/GOVERNANCE.template.md](../templates/GOVERNANCE.template.md)

## 出所

EDGE研究プログラム(非公開)のガバナンス文書「証拠クラス」「事前登録」節と仮説レジストリ運用。原型はBPL研究プログラムの事前登録→予測ロック→検証規律、および「結果閲覧後の訂正解析を事前登録確認実験として表現することの禁止」則。
