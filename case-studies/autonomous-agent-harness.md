# Case Study: Autonomous Agent Harness (Level 3 practice)

A worked LLD problem following the seven steps in the [README](../README.md#how-to-approach-an-lld-problem).
The reference implementation is [rustAgent](https://github.com/davialos/rustAgent), written in Rust.

## 1. Requirements

**Functional**
- Take a task in natural language and work on it autonomously on the local machine.
- Call a language model with tools: shell, files, search, web, plan, memory.
- Run independent tool calls in parallel; delegate sub-tasks to sub-agents.
- Finish only when verification commands pass (for example `cargo test`).
- Persist transcripts and resume them; queue tasks for unattended runs.
- Keep the computer awake while it works.

**Non-functional**
- Safe by default: no destructive commands or writes outside the workspace without approval.
- Unattended runs never block on a prompt.
- Survives long runs: bounded memory, context compaction, retries, budgets.
- Small footprint: one static binary, no async runtime.

## 2. Entities and actions

| Noun | Responsibility |
|---|---|
| `Agent` | Owns the conversation; runs the loop |
| `Llm` | Interface to a model backend (complete a request) |
| `Tool` / `Registry` | Named capability with a JSON schema; lookup and dispatch |
| `Policy` | Decides Allow or Deny for an `Action` (Shell, Write, Spawn, Network) |
| `Ctx` | What a tool may touch: cwd, policy, events, shared state, cancel flag |
| `State` | Todos, background processes, sub-agent pool, memory file |
| `SessionLog` | Append-only transcript (JSONL) |
| `KeepAwake` | RAII guard holding an OS sleep inhibitor |
| `Event` / `Sink` | Observer stream rendered for humans or as NDJSON |

Verbs: *complete*, *run tool*, *check*, *compact*, *delegate*, *approve*, *enqueue*.

## 3. Classes and relationships

```
Runtime ──creates──▶ Agent ──uses──▶ Llm            (Strategy: anthropic | openai)
                       │
                       ├──uses──▶ Registry ──owns──▶ Tool*   (Command pattern)
                       │                 └── each Tool ──asks──▶ Policy  (Proxy / guard)
                       ├──writes─▶ SessionLog                (Memento for resume)
                       └──emits──▶ Events ──▶ Sink           (Observer)
Delegate tool ──creates──▶ child Agent (same Registry type, fresh history, depth+1)  (Composite)
```

- **Dependency inversion**: `Agent` depends on `trait Llm` and `trait Tool`, never on a vendor.
  The tests replace `Llm` with a scripted fake that replays canned turns.
- **Interface segregation**: a tool sees only `Ctx`, not the agent.
- **Open/closed**: a new tool is a new `impl Tool` plus one registration line.

## 4. Key flow: one step

```
Agent            Llm             Registry/Tool        Policy
  │ complete(history)──▶│               │                 │
  │◀── assistant turn (n tool_use) ─────│                 │
  │ spawn n scoped threads ────────────▶│ check(Action) ──▶│
  │                                     │◀── Allow/Deny ───│
  │◀── n results, call order kept ──────│                 │
  │ append ONE user message with n tool_results           │
  │ (no tool_use) → run checks → fail? feed back : done   │
```

## 5. Concurrency

- **Parallel tools**: `std::thread::scope` so tools borrow the agent safely; results are
  joined in call order.
- **Shared state**: `Mutex` per structure (todos, process registry, sub-agent pool) with
  short critical sections. Poisoned locks are recovered rather than propagated.
- **Approval prompts** are serialized by their own mutex so two threads never interleave
  on the terminal.
- **Cancellation**: an `AtomicBool`, polled in the loop and in every child-process wait.
- **Producer/consumer**: the task queue is a directory. `queue add` produces;
  `daemon` consumes one task at a time and moves files `pending → running → done|failed`.
  Moving the file with `rename` (atomic on POSIX filesystems) is the claim.

## 6. Resilience patterns (Level 3, item 30)

| Pattern | Where |
|---|---|
| Retry with exponential backoff, `retry-after` | model HTTP client |
| Bulkhead | each command in its own process group, killed as a tree on timeout |
| Bounded buffers | output captured into a capped buffer; head and tail kept when clipped |
| Timeouts / budgets | per command, per task (`max_iterations`, `max_minutes`) |
| Fail-safe defaults | unattended plus needs approval means Deny, reported to the model |
| RAII resource release | `KeepAwake`, background processes killed at task end |

## 7. Edge cases worth testing

- A turn with several tool calls, one of them failing: every call still gets a result.
- Model says "done" but the check fails: the agent must continue.
- A runaway loop: the iteration budget stops it.
- Writes via `..` path traversal or into `~/.ssh`: denied.
- Compaction must keep every `tool_use` paired with its `tool_result`.
- A sub-agent sees only its brief, not the parent's history.

All of these are covered in `rustAgent/tests/agent_loop.rs`.

## Extensions to try

- A circuit breaker around the model client (open after N consecutive failures).
- A token-bucket rate limiter shared by parallel sub-agents.
- Running each task in its own git worktree (see MHarness Board) so several queue workers
  can run at once.
