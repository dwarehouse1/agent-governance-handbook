# Known issues and errata — <project>

地位: 正本の問題台帳。負の結果・旧文言・旧成果物を削除せず、以後どの証拠として読めるかを制限する。

| ID | artifact | issue | correction / boundary | consequence |
|---|---|---|---|---|
| ERR-001 | <欠陥のある成果物(文書名・関数名・台帳行)> | <何が誤りか。1行で機序まで> | <正しくは何か / 旧成果物が依然有効な範囲の境界> | <下流への影響: 何を撤回するか、何が使用禁止になるか> |

## Evidence classification

<欠陥の影響を受ける旧成果物の証拠格をここで再分類する。例:>

- `<artifact>`: `EXPLORATORY_POST_OUTCOME_CORRECTIVE_ANALYSIS`(結果閲覧後の訂正解析。確証には使えない)
- `<artifact>`: `INTERNAL_DRAFT_WITH_ERRATA`(エラータ前提の内部草稿)

## Reopening rules

問題を `RESOLVED` にする条件:

1. 文言修正だけでは不可。訂正した成果物、回帰テスト、claim/下流への影響記述を**同一チェンジセットで**示す
2. 再現性の欠陥(非決定的シード等)はコードの修正だけでは既存結果を救済しない。再実行と成果物比較を要する
3. 原理的に回収不能な旧成果物は「永久に探索格」と明示し、打ち切る
