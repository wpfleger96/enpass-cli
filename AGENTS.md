# enpass-cli Agent Notes

This file is **dev-branch only** — it must never be committed to `master` or any branch
opened as a PR to `hazcod/enpass-cli` upstream.

## Fork workflow

Two remotes:
- `origin` → `wpfleger96/enpass-cli` (personal fork)
- `upstream` → `hazcod/enpass-cli` (canonical)

Every fix or feature follows this pattern:

1. Create a worktree from `upstream/master` (upstream's default branch):
   ```
   git worktree add .worktrees/<sanitized-branch> -b wpfleger96/<type>/<slug> upstream/master
   ```
2. Implement and commit in the worktree.
3. Push to fork and open a PR: `wpfleger96:<branch>` → `hazcod/enpass-cli:master`.
4. **Also merge into `dev`** so the fix is available for dogfooding immediately:
   ```
   git checkout dev
   git merge wpfleger96/<type>/<slug> --no-ff -m "chore: merge <slug> into dev"
   git push origin dev
   ```

The `dev` branch is always the running version for local dogfooding from homelabconfigs.

## Stacking PRs

When a fix targets code that only exists in an open upstream PR (not yet in
`upstream/master`), base the fix branch on that PR's branch instead of `upstream/master`:

```
git worktree add .worktrees/<sanitized-branch> -b wpfleger96/fix/<slug> wpfleger96/<base-branch>
```

Still open the upstream PR targeting `hazcod/enpass-cli:master`. GitHub recomputes
the diff dynamically — once the base PR merges, the stacked PR's diff shrinks to
just the new changes.

Note the stacking relationship in both PR descriptions and cross-reference with
`Related: #N` or `Stacked on #N`.

Currently open upstream PRs (update as PRs merge):
- (none)

## Build & test

```bash
make build                                         # produces ./enpass-cli binary
go test -v $(go list ./... | grep -v /vendor/)     # run all tests
golangci-lint run --config=.github/golangci.yml    # lint
```

## Key code notes

- **Single-file CLI:** The entire CLI lives in `cmd/enpasscli/main.go` (~908 lines). Commands are string constants dispatched through a flat switch.
- **Adding a command:** 3 touch points in `main.go`: constant (~line 39), command map entry (~line 51), switch case (~line 875).
- **Test vault:** `test/vault.enpassdb`, password `"absolutely-No-clue"`. All existing tests use this.
- **Upstream uses `master`** not `main` as the default branch.

## Commit trailers

Git is configured with the user's own identity (`Will Pfleger`). Normal global AGENTS.md
trailer rules apply — include `Co-authored-by` and `Signed-off-by` when the agent is
the commit author.

## Worktree naming

Branch `wpfleger96/fix/some-slug` → worktree `.worktrees/wpfleger96-fix-some-slug`
(replace `/`, `\`, `:` with `-`).
