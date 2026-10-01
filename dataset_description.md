# BatchTrace dataset description

Synthetic operational records for an inverse schedule-recovery task: a
multi-product batch plant runs two-week production campaigns; only coarse
shared-tank-farm readings survive; the executed unit-level batch schedule
must be reconstructed.

## Source

Fully synthetic, generated for this challenge (BatchTrace). No external data
archive is used or reproduced. License: CC0 1.0 Universal (public domain
dedication). See SOURCE_AND_LICENSE.md. The unique source page for this
dataset is the public description page referenced in SOURCE_AND_LICENSE.md.

## Shipped files

- `raw/cases.jsonl` — the complete campaign corpus: 3,600 line-delimited
  JSON records, one per production campaign (2,400 train, 1,200 test).
  Each record carries the campaign's family tag, layout, regime flag,
  executed schedule (keyed `unit,shift`), offtake curves, and historian
  readings. `prepare.py` derives the solver-facing public files and the
  private answer key from this file; the raw record retains the answer
  material needed for that derivation.
- `dataset_description.md` — this document.
- `SOURCE_AND_LICENSE.md` — source statement and license.

## Fields (raw/cases.jsonl)

- `id` (string): opaque campaign id (`bt_NNNNN`), assigned from a seeded
  permutation; carries no ordering or family signal.
- `split` (string): `train` or `test`.
- `family` (string): `base`, `heldout`, or `rush` (test-only families).
  Private evaluator metadata; never shipped to solvers in public files.
- `layout` (string): plant layout tag (constant `A`).
- `regime` (string): `base` or `rush` operating regime. Private metadata.
- `sched` (object): the executed schedule, keys `unit,shift`, values recipe
  index or -1 (idle). This is the gold target; prepare.py moves it to the
  private answer key for test campaigns.
- `demand` (object): `m7` and `m8` offtake per shift (12 values each).
- `obs` (object): historian readings, keys `material,shift` for shifts 6
  and 12, values = quantized noisy tank levels in material units.

Derived public fields (documented in the challenge README): `units`
(eligibility map), `demand` (string form), `obs` (string form), and
`schedule` (submission-grammar string, train only).

## Intended use

Training and evaluating models for inverse reconstruction of executed
operational schedules from sparse aggregated physical readings — a
sequence-to-sequence transduction task over a fixed horizon. Not for:
profile inference about real plants, companies, or individuals (the corpus
is synthetic; no real entity is represented); de-anonymization research.

## Known limitations

- The corpus is synthetic by construction: it captures the statistical
  structure of batch-plant operation (campaigning, changeover preferences,
  demand response, calendar pressure) but is not a simulation of any
  specific real plant, and real plant historians may carry richer or
  noisier signals.
- Historian readings are quantized (8% of tank capacity) with additive
  noise (sigma 1.2% of capacity); only two snapshots per campaign exist
  (after shifts 6 and 12). This sparsity is deliberate: it is what makes
  the schedule non-identifiable from balances alone.
- The offtake curve is shipped exactly (no observation error); only tank
  levels are noisy.
- Family tags, regime flags, and the executed schedules of test campaigns
  exist only in the raw corpus and the private answer key; public files
  never carry them.

## Split and integrity

2,400 labeled train campaigns / 1,200 unlabeled test campaigns in three
test families of 400 (base, heldout, rush). Split assigned by construction
before id minting; ids from a seeded permutation; emitted row order
shuffled with an independent seed; all shipped input serializations unique
(verified 3,600/3,600); train/test id overlap zero (verified). The private
answer key is case-contiguous in two row-balanced halves (600/600) with an
explicit board column.
