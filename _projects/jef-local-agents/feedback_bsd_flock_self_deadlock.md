---
name: feedback-bsd-flock-self-deadlock
description: "agent_lock() uses BSD flock which is per-open-file-description, not per-PID. Same process opening a new fd to the same lock file SELF-DEADLOCKS — never nest agent_lock calls in the same process."
metadata:
  node_type: memory
  type: feedback
  originSessionId: db6b198f-0bfd-4bfd-b3c6-f9243db6ae5b
---

# BSD flock self-deadlock — never nest `agent_lock` in the same process

`agent_lock.py` uses `fcntl.flock(fd, LOCK_EX | LOCK_NB)`. macOS implements
this as BSD flock semantics: locks are **per-open-file-description**, not
per-process. The same PID opening a new fd to the same lock file and
requesting LOCK_EX **blocks against its own outer lock** — silently, with
no timeout if `timeout_s=None`.

**Why:** `with agent_lock("trading-scan"): ... with agent_lock("trading-analyze"): ...`
in the same process opens TWO `fd`s to `/tmp/jef-local-agents-ollama.lock`.
Both want LOCK_EX. Second one blocks forever. Today's wedge: PID 58768 ran
for 5h21m with 0% CPU before being killed. launchd silently skipped 3
scheduled fires while it was wedged.

**How to apply:**

- **Never nest `agent_lock(...)` in the same process.** If outer scope
  already holds the lock, the inner caller does not need its own
  acquisition — flock serializes against OTHER processes, not against
  re-entry from the same one.
- When an inner module wants its own lock for standalone use (e.g.
  `python -m trading analyze` ad-hoc), refactor so callers from already-locked
  scopes pass a flag (`_already_locked=True`) or simply don't acquire when
  the lock file's holder PID matches `os.getpid()`.
- **Always set `timeout_s`** on `agent_lock` calls — at least a sane upper
  bound (e.g. 180-900s). The default `timeout_s=None` means "wait forever"
  which converts any future bug like the above into a multi-hour wedge.
- **Add a wall-clock watchdog** at the daemon entry point (SIGALRM-based
  contextmanager pattern in `trading/daemon.py:_watchdog`) so even if
  something inside the lock body hangs (HTTP timeout, infinite loop,
  future re-entrancy bug), the process self-aborts at a known budget
  instead of wedging indefinitely.

**Detection signature:**

- `~/Library/Logs/<daemon>.stderr` shows
  `[<name>] waiting for agent lock — held by <PID> <other-name>`
  where the holder PID is the SAME process as the waiter. The "other-name"
  in the lock file is the OUTER acquisition's name; the waiter's name is
  the INNER call. Easy to miss in logs because both names look like
  legitimate daemons.
- `ps -p <PID> -o etime,pcpu` shows long elapsed time at ~0% CPU.

**Concrete recurrence today (2026-05-27):**

- `trading.daemon.run_scan` held `agent_lock("trading-scan")` at line 178.
- `trading.analyze.analyze_signal` tried `agent_lock("trading-analyze")` at line 263.
- Deadlocked at 08:04 immediately after `signals.collect_and_gate` finished
  (730 fresh candidates returned). Process stayed wedged until manually
  killed at 13:25.
- Fix: removed the inner `agent_lock` entirely; outer lock is sufficient.
  Docstring on `analyze_signal` documents the new caller contract.

Related: [[project-trading]] (today's hardening pass).
