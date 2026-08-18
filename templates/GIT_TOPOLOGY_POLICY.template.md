# Git Topology And Worktree Policy(テンプレート)

Updated: YYYY-MM-DD

この文書がローカルのブランチ・worktreeトポロジーの唯一の正本。並行するGitポリシー文書や、手書きのworktree台帳を作らない。実態は常にGitから導出する:

```powershell
git worktree list --porcelain
git branch --format="%(refname:short) %(upstream:short)"
```

## 正準状態

- 正準ローカルブランチは `main`、upstream は `origin/main`
- 正常状態は「非bare・非detachedのプライマリworktree1つ + ローカルブランチ `main` 1本」
- 作業は原則プライマリ `main` 上で行い、1作業単位=1コミットにまとめる
- 追加worktree/ブランチは下記の登録制レーンのみ。ブランチ名に `backup`・`detached` を含めることを禁止する

## 登録制の専用レーン

2本目のworktreeが必要な場合、作業開始**前**にレジストリ(例: `tmp/control/git_topology_exceptions.json`)へ登録する:

```json
{
  "branch": "lane/example-20260101",
  "worktree_path": "<absolute path>",
  "owner": "<agent/task identifier>",
  "packet_id": "<work packet id>",
  "scope_root": "<the only directory this lane may change>",
  "created_from_commit": "<sha>",
  "reason": "why sequential work on main is insufficient",
  "created_at": "YYYY-MM-DDTHH:MM:SS+00:00",
  "lifecycle": "active",
  "cleanup_contract": "merge_no_ff_then_retire"
}
```

- 同時に存在できる追加レーンは最大1つ
- レーンの変更は `scope_root` の内側、`main` 側の変更はその外側に限定(木の分割)
- 未登録・不正・スコープ違反のレーンは、理由の正当性に関わらず違反(fail-closed)

## 統合とクリーンアップ

1. レーン側で作業を完了・コミットし、worktreeをクリーンにする
2. レジストリのlifecycleを `retiring` に変え、ブランチHEADを凍結記録する
3. `main` 側で `git merge --no-ff --no-commit <lane-branch>` により統合(fast-forward・squash・cherry-pickは統合と認めない — マージ親の証拠が残らない)
4. 統合後、worktreeを `git worktree remove` で撤去し、ブランチを削除し、レジストリから消し、トポロジー検査を再実行する
5. 統合済みブランチの温存(「一応残す」)は違反

## 複数タスクが `main` を共有する場合

- 統合責任者を1タスクに固定し、最終コミットはそこだけが作る
- 各タスクは重複しないファイルスコープを持ち、自分のパスだけをstageする
- 他タスクと同じファイルを書いていると気づいたら、上書きせず停止して引き継ぐ

## 復旧

破壊的クリーンアップの前は、ブランチ温存ではなく検証済みbundleで退避する:

1. 追跡済みの作業をコミットとして回収可能にする
2. 未追跡・ignored・リンク先データを別途棚卸しする
3. `git bundle create` → `git bundle verify` を通す
4. bundleのSHA-256・元ref・作成時刻を記録してから元を消す

## push境界

ローカル統合はpush・リモートブランチ削除を含意しない。リモートへの各操作は、対象と行為を特定したユーザーの現行明示指示を要する。force-pushは常に禁止。
