# ALDEN.md — Your Guide to This Fork

## Why this fork exists

Daytona is excellent infrastructure. Sub-90ms sandbox provisioning, 2M+ sandboxes per day, SDKs in every language. But it has two gaps that matter for AI agents:

1. **Sandboxes can exfiltrate secrets.** If an agent has access to `api.github.com`, nothing stops it reading env vars and POSTing them anywhere. The current network policy is 440 lines of Go wrapping iptables — CIDR allowlists only. No HTTP inspection, no secret hiding. A misbehaving agent can drain credentials and Daytona can't see it.

2. **Multi-step agent workflows don't survive crashes.** Daytona's runner is a poll-execute loop. There's no checkpoint, no resumption. If a task (generate → test → human review → deploy) crashes at step 3, everything reruns from scratch.

This fork fixes both by integrating two open-source tools by Armin Ronacher (creator of Flask):

| Problem | Current | This Fork | Key gain |
|---|---|---|---|
| Network security | iptables CIDR rules | **Gondolin** VM-level MITM | Secrets never leave the host. Bypass-proof. |
| Workflow durability | Stateless poll-execute loop | **Absurd** on Postgres | Crash → resume from last step. No reruns. |

**After both phases:**
- Sandbox credentials are injected at the VM proxy level — agent code gets a placeholder, real token stays on host
- Agent workflows checkpoint every step to the same Postgres Daytona already runs
- `apps/runner/pkg/netrules/` (440 lines of iptables Go) deleted entirely — replaced by Gondolin
- `apps/api/src/sandbox/services/job.service.ts` Redis queue replaced by Absurd — no new infrastructure
- Everything open source, no new services

---

## Hi Alden

You're going to build this. Each phase is a self-contained PR that makes the system meaningfully better. As a side effect, each phase teaches you a concrete layer of how production systems are built — by reading and modifying real code, not tutorials.

By the end, you'll have:
- Real PRs on a 72K-star public GitHub repo
- A public reference implementation others can run and learn from
- Deep working knowledge of how NestJS services, job queues, container networking, and durable workflows actually work
- A blog post per phase with a demo recording

---

## Use Daytona for your own projects

Don't just study this as an isolated project. **Use it.**

Pick one of your own repos and run its code inside a Daytona sandbox using the Python or TypeScript SDK. That's the real test — does the thing work on something you actually care about?

Every time you use Daytona on your own project, you experience the system from both sides: as someone building it, and as someone depending on it. That dual perspective is what makes the blog post worth reading. The screen recording of the agent working on your actual code is the demo.

---

## Writing About What You Learn

After each phase, write about it. Not a tutorial — a reflection. What did you expect? What surprised you? What broke?

Model your writing on [Maggie Appleton's digital garden](https://maggieappleton.com) — personal, well-researched, specific, readable. Her posts start from a lived experience, explain the mechanism, and land on a clear argument. Before writing, read her talk *"One Developer, Two Dozen Agents, Zero Alignment"* for the framing: coordination > individual productivity, craft > volume.

Also look at [ColeMurray/background-agents](https://github.com/ColeMurray/background-agents) for how a public learning repo can become a reference others build on. And read about the "paperclip maximizer" thought experiment — the abstract version of Gap 1. An agent optimising for a goal with no network constraints is the AI safety version of what Gondolin is solving architecturally.

### Post format

```
Title: [Something specific]
  Good: "The 440 lines of Go I deleted — and what replaced them"
  Bad:  "What I learned about container networking"

Planted: [date]
Tags:    [2-3 topic tags]

---

[Opening: one concrete thing that happened — not "in this post I will..."]
[The mechanism: what actually changed in the code, with snippets]
[The surprise: what you didn't expect to find]
[The argument: what this teaches about the broader system]
[The demo: embed your screen recording here]
```

**Suggested title angles per phase:**

| Phase | Title angle |
|---|---|
| Phase 1 | "I built a sandbox that can't leak secrets — here's the architecture" |
| Phase 2 | "Replacing a Redis queue with Postgres-native durable execution (in a 72K-star repo)" |
| Phase 3 | "Deleting 440 lines of iptables Go and what we put in its place" |

### The demo recording (required)

Under 60 seconds. Structure:

```
0:00–0:01  HOOK     Bold text on screen: what you built. No talking yet.
0:01–0:05  SHOW     The thing running — before you explain anything
0:05–0:10  SETUP    One sentence: what problem this solves
0:10–0:50  DEMO     One complete workflow on one of YOUR OWN projects
0:50–1:00  CLOSE    Repo URL. Nothing else.
```

Don't narrate. Show it. Every second of explanation is a second someone stops watching.

Tools: QuickTime → upload YouTube unlisted → embed in post.

### Promoting each post (X/Twitter)

Each post gets a companion thread. Hook is 95% of whether anyone reads it.

**Hook formula — 3 lines:**
- Line 1: Why this matters
- Line 2: The challenge or surprise
- Line 3: Tease what's coming

**Phase 1 hook example:**
```
Your AI agent has access to api.github.com.
Nothing stops it reading your env vars and POSTing them anywhere.
Here's the architecture that makes that impossible:
```

**Phase 2 hook example:**
```
I just replaced a Redis queue in a 72K-star repo with pure Postgres.
The Redis queue dispatched sandbox jobs. Absurd does it durably.
Here's everything I changed (and what I broke first):
```

End every thread with: `1. Read the full post: [link]  2. Retweet tweet 1`

---

## What ends up publicly visible

| Phase | Public artifact |
|-------|----------------|
| 1 (PoC) | New public repo `[you]/daytona-gondolin-absurd` — runnable reference implementation |
| 2 (Absurd) | Fork of `daytonaio/daytona` + draft PR `feat/absurd-workflow-engine` |
| 3 (Gondolin) | Second PR on same fork `feat/gondolin-sandbox-runtime` |

A merged PR on a 72K-star repo is a meaningful portfolio item. Phase 1 ships first and stands alone — you can link it before Phase 2 is started.

---

## What You're Going to Build — Three Phases

### Phase 1: Standalone PoC (~2 weeks)
**Goal:** Prove that Gondolin and Absurd compose. Build a public reference implementation.
**Public artifact:** New repo `[you]/daytona-gondolin-absurd`

**Read first — in this order:**
```
tmp/gondolin/host/src/index.ts               — VM.create() API surface
tmp/gondolin/host/src/http/hooks.ts          — createHttpHooks() and onRequest shape
tmp/gondolin/host/src/qemu/contracts.ts      — HttpHooks type definition
tmp/gondolin/host/examples/docker.json       — Docker-inside-VM pattern
tmp/gondolin/host/examples/docker-init-extra.sh — how MITM CA is injected into containers
tmp/absurd/sdks/typescript/examples/agent-loop/agent-loop.ts — ctx.step() pattern
tmp/absurd/sql/absurd.sql                    — skim the header comments
```

**What you'll build:**
```
daytona-gondolin-absurd/
  README.md                   — the insight + how to run it
  docker-compose.yml          — Gondolin process + Absurd worker + Postgres
  src/
    gondolin-vm.ts            — VM.create() with onRequest hook
    absurd-worker.ts          — registerTask, ctx.step(), Postgres poll for policy
    policy-resolver.ts        — CLI: approve/deny intercepted requests from terminal
  schema.sql                  — absurd.sql + network_events table
```

**Key architectural constraint:** Gondolin's `onRequest` hook is async and can `await` a Promise — but Absurd's `ctx.awaitEvent()` works by throwing a `SuspendTask` exception to exit the task handler. Calling `ctx.awaitEvent()` from inside an `onRequest` callback crashes the VM host. The correct pattern: the hook polls a Postgres `network_events` table directly using a lightweight sleep loop. The `policy-resolver.ts` CLI writes the decision there. See `docs/design.md` for the full working pattern.

**What you'll learn:**
- How a VM-level HTTPS proxy intercepts traffic without the guest knowing
- What durable execution actually means — step checkpointing, suspension, replay
- How to wire two systems that weren't designed for each other

**Verify before writing code:**
```bash
brew install qemu node
qemu-system-aarch64 --version          # should print a version
cd tmp/gondolin && npm install
npx tsx host/examples/basic-usage.ts   # a VM should boot (no 'bash' CLI subcommand)
cd ../absurd && head -60 sql/absurd.sql # read the schema header
```

---

### Phase 2: Absurd into Daytona's job queue (~1 month)
**Goal:** Replace Redis BRPOP in `job.service.ts` with Absurd. First real Daytona PR.
**Public artifact:** Fork of `daytonaio/daytona` + draft PR `feat/absurd-workflow-engine`

**Read first — in this order:**
```
apps/api/src/sandbox/services/job.service.ts          — Redis BRPOP queue (what you're replacing)
apps/api/src/sandbox/services/job-state-handler.service.ts — job state transitions
apps/api/src/sandbox/entities/job.entity.ts           — Job schema
apps/api/src/webhook/                                  — where lifecycle events are emitted
apps/runner/pkg/runner/v2/poller/                      — Go runner polls HTTP API (NOT Redis)
tmp/absurd/sdks/typescript/src/index.ts                — Absurd SDK: spawn(), startWorker(), awaitEvent()
tmp/absurd/sql/absurd.sql                              — full schema (~1400 lines PL/pgSQL)
```

**Important before touching anything:**
- The Go runner never touches Redis. It polls the Daytona API over HTTP (`PollJobs`). Only `job.service.ts` in the NestJS API uses Redis BRPOP. **The Go runner does not change in Phase 2.**
- `absurd.sql` is 1,400 lines of PL/pgSQL with a custom `absurd` schema. Apply it as a raw TypeORM migration using `queryRunner.query(fs.readFileSync('migrations/absurd.sql', 'utf8'))` — not auto-generated entity DDL.

**What changes:**
```
apps/api/src/sandbox/services/job.service.ts
  1. Remove @InjectRedis() constructor param + InjectRedis import
  2. Remove BullModule/RedisModule from the NestJS module imports
  3. Add AbsurdModule + Absurd instance — replace BRPOP with absurd.spawn() / startWorker()

apps/api/src/webhook/services/
  → calls absurdApp.emitEvent() on sandbox.created, sandbox.state.updated

migrations/
  → new file: absurd.sql (raw migration)
```

**Durable sandbox lifecycle:**
```typescript
absurd.registerTask({ name: "sandbox-lifecycle" }, async (params, ctx) => {
  await ctx.step("provision", () => sandboxService.create(params.id));
  await ctx.awaitEvent(`sandbox.started.${params.id}`);   // webhook emits this
  await ctx.step("execute", () => runAgentCode(params));
  await ctx.awaitEvent(`sandbox.stopped.${params.id}`);
  await ctx.step("destroy", () => sandboxService.destroy(params.id));
});
```

**Latency tradeoff to call out in the PR:** Redis BRPOP dispatch ~1ms. Absurd's pull model ~100ms configurable poll interval. Flag this explicitly; the Daytona team will ask.

**What you'll learn:**
- How a real NestJS service handles job dispatch with blocking Redis queues
- How to replace a queue backend without changing the interface above it
- How to apply a large raw SQL schema inside a TypeORM migration project

---

### Phase 3: Gondolin replaces Docker sandbox runtime (~2–3 months)
**Goal:** Daytona sandboxes become Gondolin VMs. Delete the iptables code.
**Public artifact:** Second PR on same fork `feat/gondolin-sandbox-runtime`

**Read first — in this order:**
```
apps/runner/pkg/docker/create.go              — DockerClient.Create() (what you're replacing)
apps/runner/pkg/docker/network.go             — UpdateNetworkSettings() (iptables calls)
apps/runner/pkg/netrules/                     — entire package (440 lines, 7 files) to delete
tmp/gondolin/host/src/vm/                     — VM lifecycle API
tmp/gondolin/host/src/http/                   — HTTP hook utilities
tmp/gondolin/host/examples/docker.json        — Docker-inside-VM config
tmp/gondolin/host/examples/docker-init-extra.sh — how dockerd runs inside the VM
```

**What changes:**
```
apps/runner/pkg/
  gondolin/               ← new package (Go → Gondolin Node.js SDK via subprocess)
    client.go             — GondolinClient wrapping VM.create() / VM.close()
    config.go             — maps SandboxDTO to Gondolin VM options
  docker/create.go        — DockerClient.Create() → GondolinClient.Create()
  docker/network.go       — UpdateNetworkSettings() → httpHooks config on VM
  netrules/               ← deleted entirely
```

**Open question to resolve before starting:** Does the Daytona runner host expose `/dev/kvm`? QEMU without KVM acceleration is significantly slower than Docker containers. Verify on the target Linux host before committing to this approach.

**What you'll learn:**
- How Daytona's Go runner provisions and manages Docker containers
- How iptables per-sandbox chains work and why they're bypassable
- How to call a Node.js SDK from a Go service (subprocess with JSON IPC)
- How to write a PR that makes an architectural argument, not just ships code

---

## What You'll Learn — Architecture Layer Map

| Layer | Concept | When You Learn It |
|---|---|---|
| Systems thinking | What a MITM proxy is; what durable execution means | Phase 1 — read Gondolin + Absurd source |
| Async messaging | Event-driven arch, job queues, BRPOP vs poll | Phase 2 — read `job.service.ts` |
| Service interfaces | How NestJS modules + DI work; replacing backends | Phase 2 — the job.service rewrite |
| Data layer | Postgres-native workflows; raw SQL migrations | Phase 2 — applying `absurd.sql` |
| Container networking | iptables chains; Docker sandbox lifecycle | Phase 3 — read `netrules/` |
| Cross-language FFI | Go calling Node.js SDK via subprocess | Phase 3 — GondolinClient |

---

## How to Navigate the Codebase

```
apps/
  api/                    NestJS — the brain (TypeScript)
    src/sandbox/          Sandbox service, job dispatch, entities
    src/webhook/          Lifecycle event emission
  runner/                 Go — the compute plane
    pkg/docker/           Docker container management
    pkg/netrules/         iptables network policy (deleted in Phase 3)
    pkg/runner/v2/        Poller + executor
  daemon/                 Go — runs inside each sandbox
  dashboard/              Next.js — web UI
  cli/                    Go — CLI

tmp/                      Reference repos (read-only, don't edit)
  gondolin/               Gondolin micro-VM + network proxy
  absurd/                 Absurd durable execution engine

docs/
  design.md               Full technical design doc — read this before Phase 2+
```

**Key files to read first, in order:**
1. `README.md` — what Daytona is
2. `docs/design.md` — full design: code patterns, open questions, PR descriptions
3. `apps/api/src/sandbox/services/job.service.ts` — before Phase 2
4. `apps/runner/pkg/docker/create.go` — before Phase 3

---

## How to Work on This (Using gstack)

We use [gstack](https://github.com/garrytan/gstack) for structured work sessions. It's already installed.

| Command | When to use |
|---|---|
| `/office-hours` | Design a phase, challenge assumptions, produce a design doc |
| `/plan-eng-review` | Break a phase into concrete implementation tasks |
| `/investigate` | Something is broken and you don't know why |
| `/review` | Before opening a PR |
| `/ship` | Open the PR |

Start with `/office-hours` when approaching a new phase. It will read `docs/design.md` automatically and pick up where the design left off.

---

## Reference: Current Architecture

```
Daytona Runner (Go)
  └─ DockerClient.Create(sandboxDto)
       └─ Docker container
            ├─ Agent code runs here
            └─ Network: iptables per-sandbox CIDR chains
                 → DROP or ACCEPT based on destination IP
                 → No HTTP inspection. No secret handling.

Daytona API (NestJS)
  └─ JobService
       ├─ Redis BRPOP — blocks waiting for job from runner
       └─ Postgres — stores job state, sandbox metadata
```

---

## Reference: Target Architecture (After Both Phases)

```
Daytona Runner (Go)
  └─ GondolinClient.Create(sandboxDto)        ← Phase 3
       └─ VM.create({ httpHooks, secrets })
            └─ QEMU micro-VM
                 ├─ Docker daemon (inside VM)
                 ├─ Agent code (Docker container inside VM)
                 └─ Gondolin network stack
                      ├─ HTTP/TLS MITM — agent never sees real secrets
                      ├─ onRequest hook → Postgres network_events
                      └─ Per-host allowlists + DNS policy

Daytona API (NestJS)
  └─ Absurd on Postgres                        ← Phase 2
       ├─ absurd.spawn() — durable task queue
       ├─ ctx.step() — checkpointed execution
       ├─ ctx.awaitEvent() — suspend on lifecycle events
       └─ No Redis for job dispatch
```

---

## Reference: Component Mapping

| Component | Current | This Fork | Change |
|---|---|---|---|
| Sandbox networking | `apps/runner/pkg/netrules/` iptables | Gondolin `httpHooks` at VM level | Phase 3 |
| Sandbox provisioning | `DockerClient.Create()` in Go | `GondolinClient.Create()` in Go | Phase 3 |
| Job dispatch | Redis BRPOP in `job.service.ts` | Absurd `spawn()` / `startWorker()` | Phase 2 |
| Workflow durability | None — stateless loop | Absurd steps + checkpoints | Phase 2 |
| Secret handling | Env vars in container | Gondolin placeholder injection | Phase 3 |

---

## Reference: The Three Projects

**Daytona** — [daytonaio/daytona](https://github.com/daytonaio/daytona) — 72,360 stars
Stack: NestJS API, Go runner/daemon/CLI, Docker, Postgres, Redis. v0.168.0.

**Gondolin** — [earendil-works/gondolin](https://github.com/earendil-works/gondolin) — 962 stars
Author: Armin Ronacher (creator of Flask). TypeScript + QEMU. VM-level HTTP/TLS MITM proxy with JS hooks and secret placeholder injection. v0.7.0.

**Absurd** — [earendil-works/absurd](https://github.com/earendil-works/absurd) — 1,568 stars
Author: Armin Ronacher. Single `absurd.sql` + thin SDKs. Durable execution on Postgres — no extra services. Checkpointed steps, event-driven suspension. v0.3.0.

---

## The Core Insight

> Gondolin's `onRequest` JS hook and Absurd's `awaitEvent()` are the same abstraction — a synchronous checkpoint in an otherwise async execution.

Neither project was designed with the other in mind. That parallel is what makes this integration worth building.
