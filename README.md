# turboquant — meta repo

Meta repo for `turboquant` — a manifest of 2 child repos under `V-Sekai-fire`, plus the
working agreements that apply across them.

Uses [mateodelnorte/meta](https://github.com/mateodelnorte/meta) and
[meta-git](https://github.com/mateodelnorte/meta-git).

## Contents

| file | what it is |
|---|---|
| `.meta` | the project manifest — 2 repos, both `V-Sekai-fire`: `turboquant-godot`, `turboquant-project` |
| `CLAUDE.md` | working agreements: hard constraints, how measurements are reported, how work is verified, blocklists |

## Use

```sh
npm i -g meta meta-git      # requires Node.js
git clone https://github.com/V-Sekai-fire/turboquant
cd turboquant
meta git clone              # materialise every child repo
meta git status             # status across all of them
meta exec "git log -1"      # run any command in each
```

`.meta` is a manifest, not a checkout: nothing is vendored here, and child directories are
gitignored.

### Naming

The repo is `turboquant`, not `.meta`. GitHub reserves leading-dot repository names for
`.github` and renames the rest, so a repo created as `.meta` arrives as the org name. The
manifest file inside it keeps the name `.meta`, which is what the tooling reads.

### Windows note

`meta`'s subcommands shell out to `meta-git` / `meta-project`, and that `spawn` fails on
Windows `.cmd` shims with `EINVAL`. Call the plugin binaries directly instead:

```sh
meta-git clone              # instead of: meta git clone
meta-project import <name> <url>
```

`meta init` is affected the same way; this repo's `.meta` was written in the format
`meta-project import` produces, and verified against it.

## Where the reasoning lives

`CLAUDE.md` states the constraints. The incidents behind them are in `weftspun/logbook`:

- `todo.md` — dated entries, what was measured, and retractions kept beside what they retract
- `PITFALLS.md` — ten recurring failure modes, each a mistake actually made here, what it
  cost, and the mechanism that now catches it

Open work is tracked as issues in the repo it belongs to, not as prose.
