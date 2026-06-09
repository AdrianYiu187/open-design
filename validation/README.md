# validation — main-chain validation suite

`main-chain-suite.yaml` is open-design's **"must work every release" regression contract /
acceptance criteria**. It lives alongside the product code, is owned by the product/QA team,
and is updated in the same PR whenever a feature changes.

## Who uses it

- **nexu-xray autonomous validation**: when validating open-design, the agent reads this
  contract from the repo and runs each case autonomously (`per_model` cases run once per AMR model).
- **Autonomous-validation deviation**: the agent's results are aligned with the test team's
  human acceptance by case `id` to compute a deviation rate (target → 0 over releases).
  Dashboard: the "Agent autonomous validation vs human acceptance" card on the R&D page
  (`/daily/?view=yang`).

> Design principle: **each product ships its own validation contract; the tool (nexu-xray) is a
> generic runner.** So the contract lives in the product repo, not the tool repo.

## How it grows (sedimentation)

- Every **human acceptance doc**: its P0/P1(/P2) issues → match an existing case (append to
  `sources`) or add a new case.
- A feature change → update the affected cases in the same PR.
- Append-only: every issue humans ever found becomes a reproducible case.

## Case shape

```yaml
- id: mc-exec-completes-no-error   # stable kebab id (the join key across both sides)
  chain: generate                  # generate|preview|comment|edit|chat|mark|session|home|ui|export|settings|amr|...
  scope: per_model                 # per_model = run once per AMR model; once = run once
  severity: P0                     # P0 main-chain blocker / P1 high / P2 medium / P3 low
  exec_dependent: true             # (optional) needs a real model run → exercise on a real backend
  error_path: true                 # (optional) tests an error/edge path (transient-error recovery, timeout, ...)
  title: 每个 AMR 模型执行生成都不报错        # case content is in Chinese (matches the Chinese UI + acceptance docs)
  steps: [新建项目并输入 prompt, 选定该 AMR 模型, 运行生成]
  expected: 执行完成、无 connection reset / socket / opencode event stream 等后端报错
  sources: [0.10.0-nightly P0-2, P0-5/6 gpt-5.5 socket]   # provenance: acceptance item or test file
```

> Note: docs/structure are in English; **case content (`title`/`steps`/`expected`) stays in
> Chinese** — it mirrors the Chinese UI and is transcribed from the team's Chinese acceptance docs.

## AMR all-models policy

`amr_regression`: AMR (the cloud model runtime) — **all models are CORE; every release must
regress the main chain under every model in the live AMR catalog**. The 0.10.0 acceptance proved
most P0s are model-backend-specific execution errors, which a single-backend run structurally
misses. The model list is **enumerated at run time** from `apps/daemon/src/runtimes/registry.ts`
`BASE_AGENT_DEFS` plus each agent's `listModels` discovery — never hardcoded.

---

Format is YAML (readable by humans and agents, diff-friendly, comment-capable, lighter than JSON).
Migrated from the nexu-xray tool repo's `corpus/main-chain-suite.json` (2026-06-08); this is now
the single source of truth.
