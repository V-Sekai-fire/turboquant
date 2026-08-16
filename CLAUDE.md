# Working agreements

Standing constraints across every repo in this meta. Each carries a cost behind it; the incident sits in `weftspun/logbook` (`todo.md` for the
narrative, `PITFALLS.md` for the recurring failure modes and the guards that catch
them).

## Hard constraints

**Compute.** GPU work runs on RunPod, never on the local desktop GPU. Tear down after use,
then **double-check** the teardown. Anything not in a git repo is torn down after use — so
if it matters, it is committed and pushed before the machine goes away.

**Archive formats.** zstd, in parquet or standalone. **zip is not acceptable**, and neither
is gzip; recompress to `.zst` and verify payload hashes before deleting an original.
Tabular data is parquet + zstd.

**Normal form.** Parquet is in **Essential Tuple Normal Form**: interned vocabularies,
satellite relations rather than nullable columns, **no NULLs**, no derivable columns. A
value like `-1` for "no parent" is a value; a NULL is not.

**Data hygiene.** Training data only — validation and test splits are strictly held out from
training, tuning, and selection. Generative-model outputs never enter training corpora; that
is a quality rule, not a licensing one.

**Deployment.** glTF exports carry **pure data only** — skin weights, animation samplers,
morph targets. No runtime modifiers, drivers, constraints, or custom extensions. An export
that only looks right because the consumer runs our code is not portable.

**Skinning.** Dual-quaternion skinning is **blocklisted**. Delta Mush and Direct Delta Mush
are approved. Note DDM bakes the smoothing but not the pose dependence, so it suits renders
and baked clips and is not an option for live avatars.

**Pose sources.** From ANNY/SOMA's own pose library or synthetic. No scraped or third-party
pose references.

**Latents.** Stages pass latents; VAE decode happens once, at final output. Never
`encode(decode(z))`.

**Repo layout.** One standalone repo per model, not one repo with many model folders.

**Deliverables.** Video-ready assets land as PSD or a video/image intermediate with `.cff`
title and metadata, before any pod teardown. PSD because it carries lossless vector and
raster layers.

## How measurements are reported

Pair every physical measurement with a household-object equivalent. "4.3 mm" does not tell a
reader whether an error matters; "about three stacked pennies" does. Useful anchors: credit
card 0.76 mm, penny 1.52 mm, pencil 7 mm, AAA 10.5 mm, AA 14.5 mm, nickel 21.2 mm, golf ball
42.7 mm, adult wrist 57 mm, soda can 66 mm.

Where a script prints measurements repeatedly, give it a helper rather than relying on
recall.

## How work is verified

These recur often enough to state as rules:

1. **Measure the physical quantity, not the convenient proxy.** The proxy is always the one
   that is easy to read, and it lies at five sites here.
2. **A check that passes on known-broken input is decoration** — it certifies the defect.
   Every gate ships with a negative control asserting the broken input fails.
3. **A silent skip reads exactly like a pass.** An unmet precondition is a FAIL. Unchecked
   things are named and counted, never omitted.
4. **A number without a baseline is not a measurement.** Report the floor in the same table.
5. **State the detection floor.** A sampled check only sees defects larger than ~3/n. For a
   *fixed* population, enumerate rather than estimate.
6. **Conventions are data.** Parse rotation order, up axis, and units; never assume them.
7. **Bugs live at interfaces**, not inside components. Name the interfaces and check each.

## How the logbook is written

An entry records the **measurement** rather than the intention, and clips the experimental
apparatus — enough to re-run the test, not merely its conclusion.

**Retractions stay in place, next to what they retract.** Several entries exist only to
withdraw an earlier number, and that is the point: a reader who knows which roads are dead
ends is better off than one who only knows the current answer.

Documentation carries the same obligation. Where a README states a number, that number
should be machine-checked against live code (see `dataflow-coco-gemx/check_readme_claims.py`)
so drift fails a command rather than being discovered six months later.

## Documentation is antifragile

Robust documentation survives being wrong. Antifragile documentation gets **stronger** each
time it is wrong, because every error is converted into a check before the fix ships.

1. **One source of truth per fact, and it is not the prose.** A README states what the
   manifest, the schema, or the code already says. Where prose and artefact disagree, the
   artefact wins and the prose is the bug.
2. **Every claim is executable or it is decoration.** Counts, tables, branch names, file
   sizes, and version numbers are checked by a command that exits non-zero on drift. A claim
   no command can falsify does not belong in the document.
3. **A found error becomes a check, not just an edit.** Correcting the text is half the fix.
   The other half is the assertion that would have caught it, added in the same change. This
   is the whole mechanism: stress adds checks, so the harness ends up strongest exactly where
   the documentation has failed before.
4. **The checker ships with a negative control.** A doc gate that has never been shown to
   fail is certifying nothing. Each check is run once against deliberately broken input, and
   that run is part of the test, not a one-off done by hand.
5. **Never hedge to survive.** "Roughly", "should be", and "approximately" applied to a
   knowable number are ways of making a claim unfalsifiable so the check cannot fail. State
   the number and let the gate defend it; where a value is genuinely a range, state the range
   and check the bound.
6. **Prefer generated to maintained.** A table a script can emit from the artefact should be
   emitted, not typed. Hand-maintained duplication is where drift starts.

## Blocklists

Sources excluded from corpora, with the reason:

| source | reason |
|---|---|
| CMU mocap | provenance |
| Mixamo animation packs | licensing |
| posemaniacs | third-party pose scraping |
| CC-BY-SA | share-alike exposure |
| DeepFashion | re-export of a research-only corpus |
| AddBiomechanics `.b3d` as an identity source | lab volunteers — narrow and inequitable population |
| `caldata_*_jc.parquet` | pre-cut derivatives; use originals |
| EasyDiffusion outputs, seethrough PSDs | secondary generation |

`O:\Documents\Datasets\cosplay_photo_library` may be used for **validation only**, never
training.
