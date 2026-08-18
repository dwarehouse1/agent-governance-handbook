# Claude Code / AIエージェントの運用統制ハンドブック

**Agent Governance Handbook — field-tested patterns for keeping AI agents honest, auditable, and unable to wreck your repo or your research.**

AIエージェント(Claude Code 等)を研究・開発の実務で回すときの**運用統制パターン集**です。すべて、著者が実際に運用している3つの非公開プログラム(市場研究・科学研究・個人開発ファクトリー)の実働記録から一般化したもので、机上の提案は含みません。各パターンは「解決する問題 → 機構 → 失敗モード → すぐ使えるテンプレート」の形で書かれています。

- 対象読者: AIエージェントに実作業(コード・分析・研究)をさせていて、**検証とレビューがボトルネック**になり始めた人
- 特に: レビュアーが自分しかいない個人開発者・小規模チーム

## なぜこれが要るのか — AIエージェントの暴走防止は「賢い指示」では実現しない

エージェントの生成は安く速くなった。すると失敗の形が変わる:

- もっともらしい偽の発見を高速に持ってくる(→ [ケーススタディ](#ケーススタディ-実話)は勝率88%に見えた期待値負けの実話)
- 試行が安いので、十分な回数試せば何もないところからでも「有望な結果」が出る(多重比較)
- ブランチ・worktreeを増殖させ、リポジトリのトポロジーを壊す
- 過去の欠陥成果物をコンテキストとして再摂取し、誤りの上に積む

これらはプロンプトの工夫では止まらない。**エージェントの外側に置かれた、fail-closedな検査点と台帳**だけが止める。本ハンドブックはその検査点と台帳の設計パターン集である。

## パターン一覧

| パターン | 一言で | 主な出所 |
|---|---|---|
| [ワークパケット](patterns/work-packets.md) | 1コミット=1つの機械検証される事前宣言。`REPLACE:`規約で様式負荷と正直さを両立 | ファクトリーOS |
| [fail-closedゲート](patterns/fail-closed-gates.md) | 制御成果物の欠落・不正・未登録は「続行」ではなく「停止」に倒す | 全プログラム共通 |
| [独立replay](patterns/independent-replay.md) | 決定に使う数値は、producerのコードを共有しない独立実装で再導出一致するまで引用禁止 | EDGE / BPL |
| [多重性台帳](patterns/multiplicity-ledger.md) | 全試行(失敗・無効含む)を追記専用台帳に記録。台帳外の試行は存在しない | EDGE |
| [kill記録](patterns/kill-log.md) | kill条件は事前登録、killは理由つきで第一級記録。静かな放置を禁止 | EDGE / BPL |
| [エラータ台帳](patterns/errata-ledger.md) | 過去の自分の成果物の欠陥を追記専用で台帳化し、引用可能範囲を制限する | BPL |
| [事前登録と証拠クラス](patterns/preregistration-evidence-classes.md) | EXPLORATORY→LOCKED→LIVEの一方向昇格。ラベルなしの数値は最弱格 | EDGE / BPL |
| [git topologyポリシー](patterns/git-topology.md) | 正準1ブランチ+登録制レーン。エージェント開発のレビュー対象にリポジトリの「形」を含める | ファクトリーOS |

## ケーススタディ(実話)

**[勝率88%の「発見」が実弾投入前に死ぬまで — F4-H1 設計段階kill](case-studies/f4h1-design-stage-kill.md)**

AIエージェントの検証規律が実際に機能した記録。エージェントのバックテストが+0.30%/回・勝率88%を報告 → 全バリアント台帳の中の「勝率100%」が計測破綻を通報 → 独立監査で2つの計測アーティファクトを特定 → 正しいPnLでは−0.22%/回と判明し、実弾前にコストゼロでkill。数値はすべて運用台帳の記載どおり。

## テンプレート

[templates/](templates/) にそのままコピーして使えるファイルを置いている:

- [GOVERNANCE.template.md](templates/GOVERNANCE.template.md) — 証拠規律の正本の雛形
- [HYPOTHESIS_REGISTRY.template.csv](templates/HYPOTHESIS_REGISTRY.template.csv) / [MULTIPLICITY_LEDGER.template.csv](templates/MULTIPLICITY_LEDGER.template.csv) / [KILL_LOG.template.csv](templates/KILL_LOG.template.csv)
- [KILL_RECORD.template.md](templates/KILL_RECORD.template.md) / [ERRATA_LEDGER.template.md](templates/ERRATA_LEDGER.template.md)
- [WORK_PACKET.template.json](templates/WORK_PACKET.template.json) / [GIT_TOPOLOGY_POLICY.template.md](templates/GIT_TOPOLOGY_POLICY.template.md)
- [VERIFIER_CHECKLIST.md](templates/VERIFIER_CHECKLIST.md)

## 最小導入(今日から入れる3つ)

フル装備は要らない。順番はこれを勧める:

1. **多重性台帳CSVを1枚置く** — エージェントに「試行したら結果の良し悪しに関わらず1行追記」を標準指示に入れる
2. **kill条件の欄を仕様テンプレに足す** — 「何が観測されたら棄却か」を書けない仮説・機能は着手させない
3. **`REPLACE:` 残存チェックをpre-commit/CIに入れる** — ワークパケットの最小版。grep 1本で始まる

## Sanitization & provenance

公開にあたってのサニタイズ判断は [SANITIZATION_POLICY.md](SANITIZATION_POLICY.md) に明文化し、各ファイル末尾に出所を記録している。ロック済み仮説の詳細・製品内部情報・研究内容そのものは含まれない。

## License(案 — 確定前)

コード・テンプレート: MIT / 文書: CC-BY-4.0 を提案中([LICENSING_PROPOSAL.md](LICENSING_PROPOSAL.md))。ライセンス確定までの間、再配布はご遠慮ください。

---

# English summary

**What this is:** An operations-governance handbook for running AI agents (Claude Code and similar) on real research and development work. Every pattern is generalized from the author's three private, actively-operated programs (market research under strict evidence discipline, a scientific research program, and a solo-dev product factory) — nothing here is speculative.

**The core idea:** As agent generation gets cheap, the failure modes shift — plausible false discoveries, multiplicity (enough tries always "find" something), repo topology decay, re-ingestion of your own defective past artifacts. None of this is fixed by better prompting. It is fixed by **fail-closed checkpoints and append-only ledgers that live outside the agent**: work packets, independent replay (producer/verifier separation), multiplicity ledgers, pre-registered kill conditions, errata ledgers, evidence-class promotion rules, and a git topology policy.

**Flagship case study:** [An agent's backtest showed +0.30%/event at an 88% win rate](case-studies/f4h1-design-stage-kill.md). A 100%-win-rate variant in the multiplicity ledger flagged broken measurement; independent audit found two artifacts (moving-baseline self-fulfillment, stale-FX reference); corrected PnL was −0.22%/event. Killed at design stage, same day, zero cost. All numbers verbatim from the operational ledgers.

**Templates:** ready-to-copy files in [templates/](templates/). Start with three things: a multiplicity ledger, a kill-condition field in your specs, and a `REPLACE:`-placeholder check in pre-commit.

---

<!-- 相談導線(ユーザー確認待ち。REVIEW_SUMMARY.mdの3案から選択) -->
エージェント運用・検証体制の構築や監査のご相談: GitHubのIssueまたは [zenn.dev/sobani_dev](https://zenn.dev/sobani_dev) までどうぞ。
