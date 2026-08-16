# turboquant — manifest repo

Manifest for the `turboquant` fleet — 3 child repos across 2 GitHub orgs, plus the working
agreements that apply across them.

Uses [repo](https://gerrit.googlesource.com/git-repo), the multi-repo tool from the Android
Open Source Project.

## Contents

| file | what it is |
|---|---|
| `default.xml` | the manifest — 3 projects, 2 remotes |
| `CLAUDE.md` | working agreements: hard constraints, how measurements are reported, how work is verified, blocklists |

## Use

```sh
brew install repo                 # or: curl the launcher into ~/.local/bin

mkdir turboquant-ws && cd turboquant-ws
repo init -u https://github.com/V-Sekai-fire/turboquant --partial-clone
repo sync                         # materialise every child repo
repo forall -c 'git log -1 --oneline'   # run any command in each
repo status                       # status across all of them
```

`--partial-clone` matters here: `turboquant-godot` is a Godot engine fork at 1.6 GB
packed (GitHub-reported; `misc/scripts/check_docs.py` holds it to 1.4-1.9 GB). Cloning it in full was
measured at 1.0-1.1 MB/s single-stream on 2026-08-15, which is 25 minutes. Partial clone fetches
blobs on demand (`--clone-filter` defaults to `blob:none`), which brings first sync down to a
couple of minutes. Add `--partial-clone-exclude=<name>` to keep full history for a given repo.

The manifest sets `sync-j="16"`, so `repo sync` fetches sixteen projects at a time without a
flag. That attribute is load-bearing: with no `sync-j` in the manifest and no `--jobs` on the
command line, repo sets `jobs_network = 1` and fetches serially. Parallelism is across repos,
never within one, so it does nothing for `turboquant-godot` on its own.

The manifest repo is not the workspace: `repo init` checks this repo out into `.repo/manifests`
and places child repos beside it in the workspace root. Nothing is vendored here.

### Branches tracked

| project | remote | revision |
|---|---|---|
| `turboquant-godot` | V-Sekai-fire | `feat/turboquant-on-master` |
| `turboquant-project` | V-Sekai-fire | `main` |
| `qwen38-mtp` | sudoingX | `master` |

Every project states its own `remote` and `revision`. The `<default>` element carries neither,
only `sync-j`, so a project that omits either fails at `repo init` rather than inheriting a
default nobody chose. It matters here: `turboquant-godot` has no `main` or `master` branch at
all, so an inherited default would have pointed at a branch that does not exist.

`repo sync` leaves each project on a detached HEAD at its manifest revision. Use `repo start
<branch> <project>` before making changes in a child repo.

### Why not `meta`

This manifest was a `.meta` file read by [mateodelnorte/meta](https://github.com/mateodelnorte/meta).
That project's last non-bot commit to its default branch was 2021-06-08 and its last npm
release was 2022-06-19. Two failure modes cost real time here: `meta git clone` cannot find its
own `meta-git` plugin when both are installed globally (it searches `node_modules/.bin` paths,
not `PATH`), and both that failure and a wrong-subcommand usage error **exit 0** — a silent
skip that reads exactly like a pass.

## Checks

```sh
python3 misc/scripts/check_docs.py              # every README claim, against default.xml and the remotes
python3 misc/scripts/check_docs.py --self-test  # plus: break each claim, require its check to fail
```

Eight checks: every path the README names exists (here, or in the repo it is attributed to),
the counts, the branch table, the explicit remote/revision invariant, the documented
`sync-j`, that every manifest revision exists on its remote, that `turboquant-godot` really
has no `main` or `master`, and that its size is inside the stated bound. Each one has a
negative control, so the gate is known to fail rather than assumed to work.

The hooks are wired with [prek](https://github.com/j178/prek):

```sh
prek install                      # pre-commit and pre-push
prek run --all-files              # the offline subset, now
prek run --hook-stage pre-push --all-files
```

Commit stage runs the offline checks so it stays fast; the network claims and the negative
controls run at pre-push, before anything leaves the machine. `--local-only` prints what it
deferred and where that runs, because a silent skip reads exactly like a pass.
`CLAUDE.md` explains why documentation is held to this.

## Where the reasoning lives

`CLAUDE.md` states the constraints. The incidents behind them are in `weftspun/logbook`:

- `todo.md` — dated entries, what was measured, and retractions kept beside what they retract
- `PITFALLS.md` — ten recurring failure modes, each a mistake actually made here, what it
  cost, and the mechanism that now catches it

Open work is tracked as issues in the repo it belongs to, not as prose.
