# What's new in Autopilot

The skill's version is the `metadata.version` field in `skills/autopilot/SKILL.md`. At the start of every build, Autopilot checks it against this repository and tells you in one line if a newer version is out. To update:

```bash
npx skills update autopilot -g
```

## Unreleased

- README, this changelog and `docs/` are now in English; the Russian README lives on as [README.ru.md](README.ru.md). Screenshots show the English dashboard, each one a different part of it: the header, build progress, a finished build's report, waiting for you, your words.
- Dashboard: in English, ticket titles in the header are quoted “like this” instead of «like this».

## 2.1.0 — 2026-10-10

**New dashboard**
- The header color tells you what is going on: blue means the build is running, orange means it is waiting for you, green means done, red means a stage failed. Below it are one plain sentence ("2 of 6 tickets done, 3 more in progress") and the eight stages as a strip.
- **"Waiting for you".** When the agent asks you something — the briefing, an "ok" on the spec or the plan in manual mode, a decision before an irreversible step — the header turns orange, the browser tab reads "● Waiting for you", and the build clock stops: the time you take to answer no longer counts as work.
- Tasks are back as bright bars, with a color for each state: done, writing, in review, in repair, failed, waiting. Under a repair bar you see exactly what the review sent back; under a review bar, why the task is being reviewed on its own.
- **"Your words".** Every requirement from the brief, quoted in your words, with what became of it; deferred ones and placeholders show the reason. Filter by status.
- **Checks G1–G4** in plain sight: the brief is fully read, the spec lost nothing, the plan covers every requirement, blind acceptance.
- **Feed** — what happened, in words: task done, sent to review, back for repair.
- The "left" estimate now includes code review and acceptance. On real runs that is almost half the time, yet the estimate used to end along with development.
- A finished build is a report: what came out, what you need to provide before running it (keys, placeholders), blind acceptance disagreements, open questions and review notes. Review notes and report items used to be collected but never reached the screen.
- In a narrow panel next to the chat, the dashboard is denser: the same blocks without the huge numbers.
- The real logo as a single image — the page is ~25 KB lighter.
- Removed from the main screen: the duplicate task table, four different percentages, the ring, "tier" and "critical path".

**Under the hood**
- `ap.py ask <briefing|spec|plan|action|stop> --note "…"` and `ap.py answered` — the build is waiting for the user. A forgotten `answered` is closed by the next stage transition or task event.
- `ap.py` now keeps the event feed itself (`log`), marks gates (`gates`) from the events that decide them, and puts the list of quoted requirements (`reqList`) and the build outcome (`result`) into the state.
- The page re-renders on the `rev` counter, so a sync without an event still reaches the screen.

## 2.0.1 — 2026-10-02

**Dashboard**
- Working time is no longer lost when the computer sleeps or the dashboard server crashes: every build event is now written to the state, not only to wherever an open page happened to see it.
- A crashed server comes back on the same address, and an open tab comes back to life by itself — no reload needed. It used to be able to move to another port, leaving the tab dead.
- If the connection drops, the dashboard says so at the top and suggests what to tell the agent.
- The circular arrow at the top now refreshes the dashboard right away. It used to be an auto-refresh toggle: clicking it silently stopped updates.

## 2.0.0 — 2026-10-01

The first numbered version. Everything before it counts as 1.x. For its first hour after release it was called 1.1.0 — that is the same version.

**Launch**
- Autopilot starts only with the `/autopilot` command. It used to be able to start on its own on phrases like "build it turnkey" — and immediately create a repository and commit. To resume an interrupted build, run `/autopilot` with no arguments.

**Cheaper and faster — the main point of this version**
- Review no longer runs after every task. Before commit, only the foundation (scaffold, schema, shared modules) and risky tasks are reviewed — login and permissions, money, data deletion, secrets, messages to third parties. At the end there is one review of the whole build, security included; everything worth fixing is done by a single executor. On real runs, reviews and repairs ate 13–47% of the spend.
- Only what the reviewer marked as blocking goes to repair — verbatim.
- Routine tasks are built by a lighter model; the foundation and risky tasks by a strong one.
- Briefing questions come in rounds: several at once, each with a recommended answer, and "ok" accepts them all.
- `ap.py` keeps the build's books: one command per event instead of hand-editing `state.js`. The script sets the times, and it also handles requirement statuses in the manifest and gate G3.
- The instructions the agent keeps in mind before development starts shrank from 188 to 75 KB: the rationale moved to `docs/autopilot/design.md`, rare branches to separate files.

**Quality**
- Acceptance installs the project from scratch in a clean copy of the repository, which catches unrecorded dependencies and uncommitted files.
- The executor proves the new test failed before the code was written (the `REDPROOF` line), and verification is a single command: tests, types, linter.
- Every acceptance criterion is false at the start of the task.
- In an existing codebase, one agent first writes notes on the project — the exact names of entities for every executor.
- Subagents do not call other skills or launch their own agents.
- Your references (example sites) go to the executors of UI tasks right away.
- Even the smallest project is built by a separate executor — the orchestrator never writes code.
- Autopilot commits only its own files: code by the task's zone, everything else by an explicit list. Your uncommitted work in the tree does not slip into its commits.
- The executor and acceptance never do anything external: no `git push`, no deploys, no real emails or payments.
- Names in code are in English, no transliteration: `order`, not `zakaz`. In someone else's code, that code's convention applies.
- A wide change in existing code — renaming a field, changing the money type, replacing a library — is split into three steps: the new alongside the old, migration zone by zone, removal of the old. Such a task used to be unable to go green.

**Removed**
- `polish` mode (refining to match a reference) — in eight real runs it was never turned on.
- Handing a task over between contexts based on a call counter — in ~60 tasks it fired once.

**Project memory**
- A new project gets `AGENTS.md` and a one-line `CLAUDE.md` that points to it, so Claude Code, Codex and Cursor all read the file.
- If the project already has its own `CLAUDE.md` or `AGENTS.md`, Autopilot leaves it alone and at the end suggests what to add and asks you.
- Memory holds pointers and pitfalls, not a retelling of the code; the architecture of large projects goes in `docs/architecture.md`.
- In large projects, a `CONTEXT.md` glossary appears at the end of the build: the project's own words ("request", "shift", "booking"), what each one means and which synonyms not to use for it. The next build in the same repository reads it first and writes the spec in the same words. Terms used to land in a separate decision record (ADR), and the exact names from the code notes were lost along with the build. The format follows Matt Pocock's `domain-modeling`, only without questions to the user: the glossary is written by the same agent that writes the ADR, at no extra cost.
- Autopilot keeps its own glossary up to date rather than only appending to it: if a build changed the meaning of a word, the definition is rewritten; if it removed a concept, the term is deleted. Words the build did not touch are left alone. Autopilot recognizes its own glossary by a marker in the first line.
- Your own glossary (`CONTEXT.md` or `GLOSSARY.md` without the marker) Autopilot only appends to: it does not change existing definitions and raises conflicts as questions in the report. In a repository with several areas (`CONTEXT-MAP.md`), no glossary is written.
- When the glossary has more than forty terms, the report suggests splitting it into areas. Autopilot does not split it itself.

**Update notifications**
- At the start of a build, a line if a new version is out. Without internet the check stays silent; turn it off with the `AUTOPILOT_NO_UPDATE_CHECK=1` variable.

**Windows and the dashboard**
- The dashboard no longer spawns multiple servers on Windows and shuts its server down at the end on every OS.
- A stage whose file is already on disk is marked as passed automatically.
- Requirements in the task table are color-coded: done / partial.
