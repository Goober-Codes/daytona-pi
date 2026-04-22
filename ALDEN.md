# ALDEN.md

A personal onboarding guide for working on this project. The official project README is at [`README.md`](README.md) — read that first for a full picture of what Daytona is.

---

## What is this project?

**Daytona** is open source infrastructure for running AI agent code in sandboxes. Think of it as a fast, secure, isolated computer you can spin up in under 90ms, run code inside, and throw away. Over 2 million sandboxes per day. It's used by teams building AI coding agents, automated testing pipelines, and anything else that needs to run untrusted code safely.

The codebase is a NestJS API (TypeScript), a Go runner/daemon, Docker containers, Postgres, and Redis.

---

## What are we actually building?

Daytona has two gaps identified in [`COMPANY.md`](COMPANY.md):

**Gap 1 — Network security is shallow.** Right now, if a sandbox has permission to reach `api.github.com`, code running inside can read environment variables and silently POST them to the internet. Daytona can only block by IP address range — no HTTP-level inspection, no secret hiding, nothing. A project called [**Gondolin**](https://github.com/earendil-works/gondolin) (by Armin Ronacher, who also created Flask) solves this: it runs a micro-VM with a full HTTP/TLS interceptor built in. Secrets are never exposed to the sandboxed code — they're injected at the proxy level only for allowed destinations.

**Gap 2 — Workflows don't survive crashes.** Daytona's runner is a simple loop: poll for a job, execute it, done. If a multi-step agent task (generate code → run tests → wait for human review → deploy) crashes halfway through, everything reruns from scratch. A project called [**Absurd**](https://github.com/earendil-works/absurd) (also by Armin) solves this: durable, checkpointed workflows built entirely on Postgres — no extra services, no queue infrastructure, just SQL.

Both repos are in `tmp/`:

```
tmp/gondolin/   — the VM + network proxy
tmp/absurd/     — the durable workflow engine
```

---

## Why would the world find this interesting?

- **AI agent security is an unsolved problem right now.** Every team running LLM-generated code in a sandbox is one misconfiguration away from credential exfiltration. The solution described here (VM-level HTTPS interception, not iptables rules) is architecturally stronger than what anyone ships today.
- **Armin Ronacher** is the intended first audience. He built Flask (you've probably used it), Gondolin, and Absurd. Showing him a real integration of his two tools — especially one that discovers a composability he didn't explicitly design — is a credible way to get on his radar.
- **The Daytona team** has 72,000 GitHub stars and is actively building. A real upstream PR demonstrating durable agent workflows is something they'd look at.

---

## What you'll learn

This project is designed as a learning sequence through production code. No tutorials. You read real source files, understand what they do, then build something small that proves you understood it.

By the end of each phase you will have:

**Phase 1 — TypeScript + systems thinking**
- How a VM-level HTTPS proxy works (Gondolin's network stack)
- What durable execution means and why it matters (Absurd's checkpoint model)
- How to wire two systems together that weren't designed for each other
- How to write a public README that makes a technical insight legible to someone smart

**Phase 2 — NestJS + database migrations**
- How a real NestJS service handles job dispatch (reading `apps/api/src/sandbox/services/job.service.ts`)
- How Redis BRPOP works and why it's used for job queues
- How to replace a queue backend without changing the interface
- How to write a raw SQL migration inside a TypeORM project

**Phase 3 — Go + container networking**
- How Daytona's Go runner provisions Docker containers (`apps/runner/pkg/docker/create.go`)
- How iptables network rules work and why VM-level interception is harder to bypass
- How to call a Node.js SDK from a Go service (subprocess or FFI)
- How to write a PR that makes an architectural argument, not just ships code

---

## Use Daytona for your own projects

Don't just study Daytona — use it. Every project you're already working on should run its code inside a Daytona sandbox. This is the fastest way to understand what it actually does, find the gaps, and have something real to write about.

Practical starting point: pick one of your existing projects and replace wherever you currently run code locally with a Daytona sandbox. Use the Python or TypeScript SDK. When something feels awkward or breaks, that's a bug report or a PR.

---

## Write about it

The goal is one public blog post, written in the style of [Maggie Appleton's digital garden](https://maggieappleton.com) — evergreen notes, a "planted on" date, honest and personal, essay-length. Not a tutorial. A point of view.

**Inspiration to read first:**
- Maggie's talk *"One Developer, Two Dozen Agents, Zero Alignment"* — the argument that all coding agents are single-player and that's wrong. The framing here (coordination over individual productivity, craft over volume) is a good lens for writing about what you're learning with Daytona.
- [ColeMurray/background-agents](https://github.com/ColeMurray/background-agents) — a public repo showing background agent patterns. Look at how it's structured as a learning resource and how people engage with it.
- The "paperclip maximizer" thought experiment — relevant if you write about AI agents that can exfiltrate secrets (Gap 1 in this project). An agent optimising for a goal with no constraints is the abstract version of what Gondolin is preventing architecturally.

**What the post could be about:**
A junior developer learning to read production codebases by building real integrations — not tutorials, not toy projects. What that actually feels like. What you found in the code that surprised you. Why Gondolin's `docker.json` example changed how you thought about VM isolation. What it means to make a real PR to a 72K-star repo as your first open source contribution.

**It must include a demo recording.** Not a screen recording of typing — a demo of the thing working. Gondolin intercepting an HTTPS call. Absurd resuming a workflow after a crash. Something that makes someone watching say "oh, I get it now." Tools: Loom, Asciinema, or a short MP4 embedded in the post.

**Format (Maggie Appleton style):**
- Title that says something, not just describes the topic
- "Planted [date]" at the top
- Personal, first-person, specific — not generic developer content
- One clear argument the reader can disagree with
- Ends with an open question, not a conclusion

---

## Building in public

The work you do on this project is the content. Every file you read, every thing that surprised you, every PR you open — these are posts. You don't need to manufacture anything.

**Your ICP (who you're writing for):** Junior-to-mid developers curious about AI agent infrastructure, open source contribution, and learning by doing. People who want to see someone actually do the thing, not just explain it.

**Platform strategy:**
- **X/Twitter** — visibility. Short posts, milestones, demo clips. Developer community is here.
- **Blog (Maggie Appleton style)** — connection. The deep post that converts a viewer into a reader.
- Pick one and be consistent. Don't try both from day one.

---

### What to post as you work through the phases

The audience-growth playbook says: share what you do in real time. The cooler the thing, the more follows. Here's what "real time" looks like for this project:

**Phase 1 posts (highest shareability):**

| Moment | Post format |
|--------|------------|
| You boot a Gondolin VM for the first time | Screenshot of the terminal. "Just ran a QEMU micro-VM with a full HTTP/TLS interceptor from a TypeScript script. Armin Ronacher built this. It's wild." |
| You intercept your first HTTPS request | Short screen recording (15 sec). The request appearing in your terminal. No explanation needed. |
| Absurd workflow survives a crash | GIF or video. Kill the process, restart, watch it resume from step 2. "Crashed it. Restarted. It remembered." |
| You publish the public repo | "Built a PoC wiring Gondolin's `onRequest` hook with Absurd's `awaitEvent()`. Two tools by the same author that were never designed to talk to each other. They do now." + link |

**Phase 2 posts:**
- "Reading `job.service.ts` in the Daytona codebase. This is how a real NestJS service handles job dispatch with Redis BRPOP. Thread on what I found." (thread opportunity)
- "Opened my first draft PR to a 72K-star repo." Screenshot of the PR. Nothing else needed.

**Phase 3 posts:**
- "Learning Go to delete 440 lines of it." One-liner. High engagement.
- "The iptables approach to sandbox network security can be bypassed by raw sockets. The VM-level approach cannot. Here's why." Thread.

---

### Demo video structure (for the blog post and X)

1 minute max. Follow this structure:

```
0:00–0:01   Hook        — Bold text: "AI agent sandbox. Zero credential leakage."
0:01–0:05   Show it     — Gondolin intercepting a request (no talking)
0:05–0:10   Setup       — "I'm running code in a sandbox that has my GitHub token."
0:10–0:50   Demo        — Full workflow: request intercepted → policy prompt → approved/denied → result
0:50–1:00   Close       — "The sandbox never saw the real token. github.com/[you]/daytona-pi"
```

Don't explain it. Show it. Every second of explanation you add is a second someone stops watching.

---

### The blog post hook

The Maggie Appleton style rewards a specific kind of post: personal, opinionated, one clear argument. Here's a strong angle:

> **"I learned to read production code by trying to impress someone I've never met"**

The argument: the best way to learn a codebase isn't tutorials — it's picking a real target audience (one person, one repo, one PR) and building toward it. The target makes the learning specific. Generic learning produces generic results.

That's a post developers will share. It's also exactly what you're doing.

**Thread version for X (once the repo is live):**
```
Hook:   I'm a junior dev. I've never contributed to open source.
        So I decided to try to get Armin Ronacher's attention.
        Here's what I built (and what I learned reading his code):

Body:   3–4 chapters — the two tools, the composability insight,
        the one thing that surprised me in the source
CTA:    "Full PoC at [link]. The deep write-up: [blog link]"
```

---

## What ends up publicly visible

Each phase produces something on your GitHub profile:

| Phase | Public artifact |
|-------|----------------|
| 1 | New repo: `[you]/daytona-gondolin-absurd` — a reference implementation others can run |
| 2 | Fork of `daytonaio/daytona` + draft PR: `feat/absurd-workflow-engine` |
| 3 | Second PR on the same fork: `feat/gondolin-sandbox-runtime` |

A merged PR on a 72K-star repo is a meaningful portfolio item.

---

## The design doc

The full technical design (architecture diagrams, exact files to read, code patterns for each phase) lives here:

```
docs/design.md
```

Or read it in your editor. It has:
- Exact files to read before writing any code for each phase
- The correct code patterns (including known architectural constraints)
- What to say in each PR description
- Open questions to verify before starting Phase 2

---

## How to use gstack (the AI assistant)

This project uses [gstack](https://github.com/garrytan/gstack), a set of AI workflow skills layered on top of Claude Code. It's already installed.

**To continue this design session:**
```
/office-hours
```
This picks up where we left off — the design doc is loaded automatically.

**To plan the engineering implementation of a phase:**
```
/plan-eng-review
```
The skill reads the design doc and helps you break it into concrete tasks.

**To get a code review before opening a PR:**
```
/review
```

**To ship a phase (commit + PR):**
```
/ship
```

You don't need to know all of these upfront. Start with `/office-hours` if you have questions about the design, and `/plan-eng-review` when you're ready to start a phase.

---

## Where to start

**Right now:**

```bash
# 1. Verify QEMU is installed
brew install qemu node
qemu-system-aarch64 --version

# 2. Verify Gondolin builds
cd tmp/gondolin && npm install
npx tsx host/examples/basic-usage.ts   # a VM should boot

# 3. Verify Absurd's schema looks sane
head -60 tmp/absurd/sql/absurd.sql

# 4. Read these files (in order, before writing any code):
#    tmp/gondolin/host/src/index.ts
#    tmp/gondolin/host/src/http/hooks.ts
#    tmp/gondolin/host/examples/docker.json
#    tmp/absurd/sdks/typescript/examples/agent-loop/agent-loop.ts
```

**Then:** Create a new public GitHub repo called `daytona-gondolin-absurd` and start with `src/gondolin-vm.ts` — a single `VM.create()` call with an `onRequest` hook that logs intercepted URLs to stdout.

The full step-by-step is in the design doc.

---

## Technical reference

### The gaps in detail

**Gap 1 — Network security**

Daytona's entire network policy today:

```
apps/runner/pkg/netrules/ — 440 lines of Go wrapping iptables
```

It creates per-sandbox chains in `DOCKER-USER` that allow/drop based on destination CIDR ranges. That's it. No request inspection, no secret mediation, no HTTP-level hooks.

Gondolin's network stack is 4,656 lines of TypeScript that implements:
- Full HTTP/TLS MITM proxy (`mitm.ts` + `qemu/http.ts`)
- Per-request JS hooks (`onRequest` / `onResponse`)
- Secret placeholder substitution in headers
- Host pattern matching with wildcards
- WebSocket mediation
- DNS modes (synthetic / trusted / open)

**Gap 2 — Workflow durability**

Daytona's runner today:

```
apps/runner/pkg/runner/v2/poller/   — polls API for jobs
apps/runner/pkg/runner/v2/executor/ — executes jobs (create/start/stop/destroy)
```

The API has a `JobService` backed by Postgres + Redis, but only for infrastructure operations. There is no concept of multi-step workflows, crash recovery, event-driven suspension, or step-level retry.

---

### Integration points

**Gondolin → Daytona Runner**

```
Current:
  Runner → Docker container → iptables CIDR rules → internet

With Gondolin:
  Runner → Gondolin VM → Docker container inside VM → Gondolin network stack → internet
                                  ↓
                            secret substitution
                            request/response inspection
                            per-host allowlists with JS hooks
```

Files that change:
- `apps/runner/pkg/netrules/` — deleted entirely (Gondolin replaces this)
- `apps/runner/pkg/docker/create.go` — `DockerClient.Create()` → `GondolinClient.Create()`
- `apps/runner/pkg/docker/network.go` — `UpdateNetworkSettings()` → Gondolin `httpHooks` config

**Absurd → Daytona API**

```
Current:
  SDK → API → create sandbox → run code → done (stateless)

With Absurd:
  SDK → API → Absurd task spawned →
    step("provision")              → create sandbox
    step("execute")                → run agent code
    step("validate")               → check results
    awaitEvent("human.approved")   → suspend until review
    step("deploy")                 → push to production

  (crash at any point → resumes from last checkpoint)
```

Files that change:
- `apps/api/src/sandbox/services/job.service.ts` — Redis BRPOP queue → Absurd `spawn()` / `startWorker()`
- `apps/api/src/webhook/` — lifecycle events → `absurd.emitEvent()`

---

### What it all looks like together

```
Daytona (infra + orchestration)
├── Absurd (durable workflow engine on Postgres)
│   ├── Multi-step agent tasks with checkpointing
│   ├── Event-driven suspension (human review, external signals)
│   └── Retry without re-execution of completed steps
│
├── Gondolin network stack (per-sandbox VM)
│   ├── HTTP/TLS request interception
│   ├── Secret placeholder injection
│   ├── Per-host allowlists with JS hooks
│   └── DNS policy control
│
└── Existing Daytona (unchanged)
    ├── Sub-90ms sandbox provisioning
    ├── Docker/OCI container runtime
    ├── SDKs (Python/TS/Go/Ruby)
    ├── Snapshots, volumes, lifecycle
    └── Dashboard, billing, multi-region
```

Daytona provides the compute. Gondolin provides the security. Absurd provides the durability.

---

### Project backgrounds

**Daytona**
- Repo: [daytonaio/daytona](https://github.com/daytonaio/daytona) — 72,360 stars
- Stack: NestJS API, Go runner/daemon/CLI, Docker containers, Postgres, Redis
- Latest: v0.168.0 (April 21, 2026)
- Scale: 2M+ sandboxes/day, sub-90ms provisioning

**Gondolin**
- Repo: [earendil-works/gondolin](https://github.com/earendil-works/gondolin) — 962 stars
- Author: Armin Ronacher (mitsuhiko) — creator of Flask
- Stack: TypeScript control plane, QEMU/libkrun backends, JS-implemented network stack
- Latest: v0.7.0 (March 2026)
- Key feature: guest code never sees real credentials — secrets are injected at the proxy level only for allowed destinations

**Absurd**
- Repo: [earendil-works/absurd](https://github.com/earendil-works/absurd) — 1,568 stars
- Author: Armin Ronacher (mitsuhiko)
- Stack: single `absurd.sql` file + thin SDKs for TypeScript, Python, Go
- Latest: v0.3.0 (April 2, 2026)
- Key feature: no extra services beyond Postgres — drops into any existing database

---

## One thing worth knowing

The core insight that makes this project worth building:

> Gondolin's `onRequest` JS hook and Absurd's `awaitEvent()` are the same abstraction — a synchronous checkpoint in an otherwise async execution.

Neither project was designed with the other in mind. Seeing that parallel, and building something that demonstrates it, is the point. That's the thing Armin would actually read.
