### Hi, I'm Linden 👋

C++ engineer focused on **database systems and performance-critical software**.

- 🔧 3+ years building database kernel components — query processing, HTAP,
  columnar storage, and performance optimisation (Huawei, GaussDB)
- 🎓 Completing a Master of Information Technology at the University of
  Waikato, New Zealand
- 🧠 Interested in query engines, storage engines, and systems programming in C++
- 📍 Based in the Waikato, New Zealand — open to C++ / backend roles

#### Selected Open Source Contributions

**GCC — [Reduce needless value copies in the analyzer and a few hot middle-end paths](<PENDING: gcc-patches thread or merged-commit link>)**

5-patch series to `gcc-patches@gcc.gnu.org`, found by auditing the tree
with clang-tidy's `performance-*` checks for missed moves and needless
by-value parameters. Patch 1 gives `program_state` a move-assignment
operator and threads the move through `point_and_state` and
`exploded_node`, removing a deep copy of the whole region model on every
exploded-graph node — a measured **6.06% speedup** on `-fanalyzer`
(63.98s → 60.10s median, 8 interleaved runs, no overlap). Reviewed
positively across all 5 patches by GCC analyzer maintainer
**David Malcolm** and GCC maintainer **Martin Jambor**.
*(Status: reviewed on gcc-patches — link pending confirmation.)*

→ [Full write-up](./gcc-analyzer-move-semantics.md)

📫 linden.lance.developer@gmail.com
