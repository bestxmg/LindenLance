# GCC: Reducing Needless Value Copies in the Analyzer and Middle-End

**Status:** Posted to `gcc-patches@gcc.gnu.org`, 24 September 2026. Reviewed
positively by GCC maintainers David Malcolm and Martin Jambor.
*(Update once merge is confirmed: commit link here.)*

## Motivation

Move semantics are a part of C++ that experienced engineers still get
subtly wrong in large, long-lived codebases. Rather than review by hand, I
audited the GCC tree with clang-tidy's `performance-*` checks —
`performance-unnecessary-value-param`, `performance-move-const-arg`, and
related checks that trace whether a by-value parameter is ever mutated, or
whether a `std::move` target is actually reachable at the call site. That's
an exhaustive, per-parameter check that neither compiler warnings nor human
review are built to do consistently across a codebase this size.

## The series

Five patches — four are correctness/consistency fixes with no individual
performance claim, and one has a measured win:

1. **`analyzer: avoid deep-copying program_state when creating an
   exploded_node`** — `program_state` owns a whole `region_model`. Every
   time the analyzer's exploded-graph engine created a new `exploded_node`,
   it was deep-copying that `program_state` by value. Gave `program_state`
   a move-assignment operator and threaded the move through
   `point_and_state` and `exploded_node`, so the copy only happens where a
   real copy is actually needed.
2. **`analyzer, diagnostics: add missing std::move for by-value sinks`**
3. **`diagnostics: take HTML tag names as const char * in source-printing`**
4. **`gcc, analyzer: drop std::move calls that have no effect`**
5. **`range-op, tree-ssa-ccp, fold-const, ipa-cp: bind wide_int / widest_int
   by reference`**

17 files changed, 101 insertions, 48 deletions across `gcc/analyzer`,
`gcc/diagnostics`, and several middle-end passes (`range-op`,
`tree-ssa-ccp`, `fold-const`, `ipa-cp`).

## Measuring it properly

For patch 1, a single timing run isn't trustworthy. `cc1 -fanalyzer` was run
on `libiberty/cp-demangle.c` 8 times, alternating between the patched and
unpatched binary on every run — not all of one binary's trials first — to
cancel out drift from thermal throttling, cache state, and background load.
Median time dropped from 63.98s to 60.10s, a **6.06% improvement**, with no
overlap between the two sets across all 8 pairs.

## Testing

- `x86_64-pc-linux-gnu`, `--enable-languages=c,c++,lto`, `--disable-bootstrap`
- `check-gcc analyzer.exp/tree-ssa.exp/ipa.exp` on trunk (`492fbdbf99d`):
  16820+10537 pass, with the same 2 pre-existing failures as unpatched
  trunk — an unrelated `-Wanalyzer-symbol-too-complex` depth boundary and a
  known `ivopts` regression — i.e. the series introduces no regressions.

## Review

- **David Malcolm** (GCC analyzer maintainer): "nice to see a measurable
  performance win in the analyzer for this," with line-by-line feedback
  across all 5 patches.
- **Martin Jambor** (GCC, SUSE): approved the `ipa-cp` hunks, offered to
  push patch 1 on its own.

[Cover letter and full review thread →](<PENDING LINK>)
