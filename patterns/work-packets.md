# ワークパケット — Work Packets

> **EN:** Every agent work session commits against one machine-validated "work packet" — a structured declaration of what boot path the task follows, what it will change, how it will be validated, and what counts as done. A scaffolder emits a schema-valid skeleton with every narrative field prefixed `REPLACE:` so honesty stays on the session.

## 解決する問題

エージェントに自由記述で作業させると、(1)何を変えるつもりだったのかが事後に分からない、(2)「ついで」の変更が混入する、(3)完了条件が宣言されないまま作業が漂う、(4)検証が実行されたかどうかが確認できない。人間のコードレビューだけでこれを抑えるのは、エージェントの出力速度に対してスケールしない。

## パターン

1. **1コミット=1パケット**: 作業単位ごとに構造化された宣言ファイル(JSON等)を作り、コミット時にバリデータが機械検査する
2. **boot path(作業種別)による分岐**: タスクを種別(ドキュメント整理/バリデータ変更/製品実装/挙動修理/ガバナンス変更/継続作業…)に分類し、種別ごとに必須フィールドと必読文書のセットを変える。軽い作業に重い様式を課さない・重い作業に軽い様式で通さない
3. **パケットの中身**(一般化した必須要素):
   - 種別・リスククラス・影響範囲・変更対象パスのクラス
   - 何をするか(concept)と何をしないか(スコープ境界)
   - 実行する検証コマンド(挙動バリデータ/構造バリデータ)の列挙
   - 完了契約(何が満たされたら完了か)と、停止する場合の正当な停止理由の列挙
   - 積み残し(deferred work)の明示 — 空でもフィールド自体は必須
4. **scaffold + REPLACE規約**: 骨格生成ツールがスキーマ適合のスケルトンを吐き、**すべての記述値に `REPLACE:` 接頭辞を付ける**。書き手は実際の変更内容で置換しない限りバリデータを通れない。様式作成の負荷を消しつつ、内容の正直さは書き手(エージェント)に残す
5. **fail-closedな検査**: パケットが欠落・不正・スキーマ不一致ならコミット不可([[fail-closed-gates]])

## なぜ効くか

- パケットは**事前宣言**なので、エージェントの「やったことの事後正当化」ではなく「やる前の計画」を検査できる
- 停止理由が列挙型(ユーザー指示待ち/ブロック/検証不能/スコープ境界到達…)なので、「なんとなく終わった」を排除できる
- 人間のレビューは、diffの全行ではなくパケットとdiffの整合に集中できる

## エージェント運用での使い方(最小構成)

フル実装(スキーマ+バリデータ+scaffolder)は重い。最小構成はこれだけでも機能する:

1. [templates/WORK_PACKET.template.json](../templates/WORK_PACKET.template.json) をリポジトリに置く
2. エージェントへの標準指示に「作業開始前にパケットを埋める。REPLACE:を残したままコミットしない」を入れる
3. CIかpre-commitフックで、パケットの存在と `REPLACE:` 残存を機械検査する(grep 1本で始められる)

## 失敗モード

- **パケットの事後作成**: 作業後に埋めると、ただの作業報告になる。事前宣言であることに価値がある
- **様式の固定費が高すぎる**: 全作業に最重量の様式を課すと、様式を満たすこと自体が目的化する。boot path分岐が対策
- **プレースホルダの黙認**: `REPLACE:` が残ったままのコミットを一度でも通すと、様式全体が飾りになる

## テンプレート

- [templates/WORK_PACKET.template.json](../templates/WORK_PACKET.template.json)

## 出所

個人開発ファクトリーOS(非公開)のワークパケット機構 — スキーマ検証バリデータ、boot path別の骨格生成ツール(scaffold)、`REPLACE:` プレースホルダ規約、完了契約・停止理由の列挙型。製品固有の内容は含まない。
