# mealie: Test Suite Reliability and Speed

Three merged PRs to [mealie](https://github.com/mealie-recipes/mealie), a
self-hosted recipe manager (Python/FastAPI backend, Vue frontend) —
cleaning up and speeding up its test suite.

## [#8033 — Fix all warnings in pytest](https://github.com/mealie-recipes/mealie/pull/8033)

Eliminated all 156 pytest warnings — not by suppressing them, but by
root-causing each category: Pydantic double-serialization in import
paths, a Starlette status-code rename (`HTTP_422_UNPROCESSABLE_ENTITY` →
`_CONTENT`), meal-plan dates built in local time instead of UTC (which
also fixed 4 pre-existing test failures as a side effect), a deprecated
SQLAlchemy `Inspector.from_engine()` call, a stale ORM collection cache in
a fixture teardown, and narrowly-scoped filters for an unmaintained
third-party dependency. 12 commits, verified with `task py:check` after
every one. 156 warnings → 0.

## [#8094 — Treat warnings as errors in pytest](https://github.com/mealie-recipes/mealie/pull/8094)

Once the warnings were gone, made sure they stay gone: added `"error"` to
`filterwarnings` so any new warning fails CI. Verified the mechanism
itself by temporarily reintroducing a known warning and confirming it
correctly failed 4 tests, before removing it again.

## [#8473 — Run tests in parallel with pytest-xdist](https://github.com/mealie-recipes/mealie/pull/8473)

Cut the Python test suite from ~4 minutes to ~1.5 minutes on a 16-core
machine by running tests concurrently (`pytest -n auto --dist loadfile`).
The real work was isolation, not just parallelism: per-worker SQLite data
directories and Postgres databases, so concurrent workers can't collide —
a maintainer specifically checked and confirmed that isolation story
before approving.

All three merged by maintainer **michael-genson**.
