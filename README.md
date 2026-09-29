### Hi, I'm Linden 👋

C++ engineer focused on **database systems and performance-critical software**.

- 🔧 3+ years building database kernel components — query processing, HTAP,
  columnar storage, and performance optimisation (Huawei, GaussDB)
- 🎓 Completing a Master of Information Technology at the University of
  Waikato, New Zealand
- 🧠 Interested in query engines, storage engines, and systems programming in C++
- 📍 Based in the Waikato, New Zealand — open to C++ / backend roles

#### Selected Open Source Contributions

**openGauss — [Flattening the expression evaluation framework](https://gitee.com/opengauss/openGauss-server/pulls/3121)** · *merged*

The public counterpart to the Expression Flattening Computation Framework
on my résumé from Huawei's GaussDB team. Flattens the expression tree once
at init instead of re-walking it recursively on every evaluation. Merged
to `openGauss/openGauss-server` master, March 2023, 118 files.

→ [Full write-up](./opengauss-expression-flattening.md)

**PostgreSQL — [Hashing a parameterised ScalarArrayOpExpr](https://www.mail-archive.com/pgsql-hackers@lists.postgresql.org/msg238384.html)** · *work in progress*

Extended PG14's `col = ANY (array)` hash-table optimisation — previously
limited to constant arrays — to cover the parameterised forms real client
drivers actually emit (JDBC, psycopg, asyncpg bind parameters), which had
stayed on the linear-scan path since PG14. Up to **~49× faster** at
N=1000 (2494ms → 51ms), with the single-execution overhead the original
PG14 discussion worried about measured at worst +12µs. Posted to
`pgsql-hackers@lists.postgresql.org`, 9 September 2026.

→ [Full write-up](./postgresql-hashed-saop.md)

**GCC — Reduce needless value copies in the analyzer and a few hot middle-end paths** · *work in progress*

5-patch series to `gcc-patches@gcc.gnu.org`, found by auditing the tree
with clang-tidy's `performance-*` checks for missed moves and needless
by-value parameters. Patch 1 gives `program_state` a move-assignment
operator and threads the move through `point_and_state` and
`exploded_node`, removing a deep copy of the whole region model on every
exploded-graph node — a measured **6.06% speedup** on `-fanalyzer`
(63.98s → 60.10s median, 8 interleaved runs, no overlap). Reviewed
positively by GCC analyzer maintainer **David Malcolm** and GCC
maintainer **Martin Jambor**, revised in response to their feedback.
Posted 23 September 2026; awaiting push (no commit access).

→ [Full write-up](./gcc-analyzer-move-semantics.md)

**mealie — [Test suite reliability and speed](https://github.com/mealie-recipes/mealie)** · *3 PRs merged*

Eliminated all 156 pytest warnings by root-causing each one (not
suppressing them), locked that in by making warnings fail CI, then cut
the test suite from ~4 min to ~1.5 min with isolated parallel workers.

→ [Full write-up](./mealie-testing.md)

📫 linden.lance.developer@gmail.com
