# git topologyポリシー — エージェントにリポジトリを壊させない

> **EN:** Agents get one canonical branch and (at most) one explicitly registered work lane. Topology state lives in Git itself, extra lanes require a registry entry with a full lifecycle contract, and a validator fails closed on any deviation. No backup branches, no detached heads, no worktree graveyards.

## 解決する問題

エージェントは git 操作を大量に行い、失敗すると「とりあえずブランチを切る」「バックアップブランチを作る」「古いworktreeを放置する」で回復しようとする。数週間で、どれが正かわからないブランチ・worktreeの墓場ができる。**リポジトリのトポロジー(ブランチとworktreeの形)そのものを統制対象にしない限り、これは必ず起きる。**

## パターン

1. **正準状態を1文で定義する**: 「正準ブランチは main、worktreeは1つ、ローカルブランチも1つ」— 正常形が明文化されて初めて逸脱が検出できる
2. **状態はGitから導出する**: worktree一覧やブランチ状態を手書きの台帳で管理しない。`git worktree list --porcelain` 等の生出力が唯一の実態。手書き台帳は必ず実態と乖離する
3. **追加レーンは登録制**: 2本目のworktree/ブランチが必要な場合は、専用レジストリに理由・所有者・対象範囲・作成時コミット・クリーンアップ契約を登録してから作る。未登録の追加トポロジーは(理由が正当でも)違反
4. **命名の禁止トークン**: `backup`、`detached` などをブランチ名に含めることを機械的に禁止する。「念のためコピー」文化の芽を名前レベルで摘む
5. **統合は証拠が残る形だけ**: レーンの取り込みは `git merge --no-ff` に限定(fast-forwardはマージ親の証拠を残さない)。rebase・cherry-pick・手動コピーによる「実質的な統合」を認めない
6. **ライフサイクル契約**: レーンは 作成→作業→凍結→統合→撤去 の状態機械を持ち、統合済みブランチの温存(「一応残しておく」)自体を違反とする
7. **バリデータで機械検査**: 以上をポリシー文書内の契約ブロック+バリデータで検査し、逸脱はfail-closedで停止([[fail-closed-gates]])

## 複数エージェントが同一ブランチで働く場合

ブランチを増やさずに並行作業させる場合の規律:

- 統合責任者(integration owner)を1タスクに固定し、最終コミットはそこだけが作る
- 各タスクに**重複しないファイルスコープ**を事前に割り当てる
- 各タスクは自分のスコープのパスだけをstageする
- 他タスクと同じファイルを書いていると気づいたタスクは、上書きせず停止して引き継ぐ

## 復旧の規律

- 破壊的なクリーンアップの前は、ブランチではなく**検証済みgit bundle**で退避する(`git bundle verify` を通し、SHA-256・元ref・作成時刻を記録)
- bundleは未追跡・ignoredファイルを含まない。Git外のデータは別途保全計画を立ててから消す

## 失敗モード

- **「一時的」ブランチの恒久化**: TTLやクリーンアップ契約のないレーンは残り続ける。作成時に撤去条件まで登録させる
- **レジストリを恒久権限と誤読**: レーン登録は実行時状態であって、恒久的な許可ではない。壊れた・不整合な登録エントリは追加トポロジーを正当化しない
- **push権限との混同**: ローカル統合はpush・リモート削除を含意しない。リモートへの操作は別途の明示指示を要求する(本ハンドブック全体の原則: 公開・外部反映は常に人間の明示指示)

## テンプレート

- [templates/GIT_TOPOLOGY_POLICY.template.md](../templates/GIT_TOPOLOGY_POLICY.template.md)

## 出所

個人開発ファクトリーOS(非公開)のGit Topology And Worktree Policy — 契約ブロック、登録制の専用worktreeレーン、`--no-ff`統合契約、bundle復旧規律、禁止トークン。製品固有情報は含まない。
