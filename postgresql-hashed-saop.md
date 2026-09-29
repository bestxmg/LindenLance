# PostgreSQL: Hashing a Parameterised ScalarArrayOpExpr

**Status:** Posted to `pgsql-hackers@lists.postgresql.org`, 9 September
2026. Awaiting review.

## Motivation

Since PostgreSQL 14, the executor can evaluate `col = ANY (array)` with a
hash table instead of a linear scan — but only when the array is a
`Const`. Every parameterised form stayed on the O(rows × N) path:

- `col = ANY ($1)` — a `Param`, the shape of a cached/generic plan
- `col IN ($1, ..., $N)` — an `ArrayExpr` of `Params`
- `col = ANY ($1::int[])` — an `ArrayCoerceExpr`
- `col = ANY (string_to_array($1, ','))` — a `FuncExpr`

These are exactly what real client drivers emit — JDBC's `setArray`,
psycopg's `= ANY(%s)`, asyncpg — and what anything that expands an `IN`
list into bind placeholders produces. A one-off custom plan folds the
array to a `Const` and gets the fast path; a cached generic plan never
does, which is why the slowdown tends to only show up after a statement
has run a few times. The original PG 14 thread named the non-`Const` case
as future work, deferred "from fear that we may slow down cases where the
expression is evaluated only once."

## The patch

Hashes any array argument the planner can prove is fixed for the whole
execution: no `Vars`, no volatile functions, no aggregate/grouping/window
functions, no sub-selects, and no `Params` other than `PARAM_EXTERN`. For
such an array, the executor compiles it as an independent sub-expression,
evaluates it once on the first row, builds the hash table, and reuses it.
`MIN_ARRAY_SIZE_FOR_HASHED_SAOP` is still enforced, now against the
run-time element count; a shorter array falls back to the existing linear
path. A standalone `ExprState` (a PL/pgSQL "simple expression," reused
across calls with different parameters) deliberately keeps the linear
path — the "one execution" proof needs an execution boundary it doesn't
have.

Not covered, because the array genuinely isn't fixed for the execution: a
`Var` (new array per row), a volatile function, an
aggregate/grouping/window value, a sub-select, or a `PARAM_EXEC`
(correlated / nestloop-inner) — these stay linear.

5 backend files changed, ~360 lines, plus regression tests. No new GUC,
syntax, or catalog change.

## Measuring it

Release build, no LLVM, generic plan forced, warm cache, one Ryzen 7 4800H
laptop. `SELECT count(*) FROM t WHERE v = ANY($1)`, 1,000,000 rows, median
of 9 runs:

| N | master | patched |
|---|---|---|
| 6 | 59 ms | 57 ms (below threshold: linear both) |
| 9 | 65 ms | 50 ms |
| 40 | 150 ms | 52 ms |
| 100 | 297 ms | 52 ms |
| 1000 | 2494 ms | 51 ms |

Patched time is flat in N once hashed — roughly **49× faster at N=1000**.
`IN ($1,...,$N)` sees 3.6× at N=9, 12× at N=60. No change on paths that
stay linear (N < 9, literal `IN`, pgbench).

**The PG 14 concern was single-execution overhead.** Plan built and run
once, fresh connection:

| rows | N | master | patched |
|---|---|---|---|
| 1 | 40 | 0.025 ms | 0.028 ms |
| 1 | 200 | 0.023 ms | 0.035 ms (+12 µs, worst seen) |
| 1000 | 100 | 0.316 ms | 0.085 ms |

Break-even is around 10 rows; worst case is +12 µs to build a 200-entry
hash table for a one-row scan.

## Open question, raised in the patch itself

`cost_qual_eval` charges the hashed cost for an opaque stable array using
`estimate_array_length()`'s default guess, which slightly overestimates
startup cost for an array that turns out short at run time — left for a
follow-up rather than folded in speculatively.

## Possible follow-ups (out of scope, noted for later)

- An uncorrelated `ARRAY(SELECT ...)` becomes an `InitPlan` whose
  `PARAM_EXEC` never changes — execution-stable in principle, only the
  planner needs to recognise it.
- A `PARAM_EXEC` array on a nestloop inner side is fixed per rescan, so it
  could be hashed once per rescan given a cost gate and rescan-aware
  invalidation of the cached table.

[Full post and thread →](https://www.mail-archive.com/pgsql-hackers@lists.postgresql.org/msg238384.html)
