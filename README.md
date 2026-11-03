# Building ORION

Notes on how I'm designing [ORION](https://impacts-ai.com/orion), a permission-based AI agent that operates a
computer for you, and the decisions behind it. This repo holds no ORION source code (that's private).
It covers the *why* and the *how*, and is honest about what isn't done.

## The problem I'm working on

Chat assistants are good at *explaining* how to do something on a computer. They're much worse at actually
*doing* it, and when they act, they often report success without checking. My first versions had exactly
this bug: I asked ORION to change the screen brightness, the model said "Done", and nothing changed.

That bug shaped the whole design: **what the model says is not evidence.**

## How it got here

| Stage | What it was | What I learned |
| :--- | :--- | :--- |
| **Project ARVIS** | My first Jarvis-style assistant experiment | The idea works. Talking to your computer feels natural. |
| **ORION desktop assistant** | Renamed and rebuilt around real computer actions: opening apps, typing, searching | Tools existing is not the same as tools working end to end. |
| **Electron + HUD** | A desktop app with a 3D HUD, screen capture and multi-provider model routing | It looked impressive, but it was heavy and laggy, and the visuals didn't make the agent more capable. |
| **CLI-first agent (today)** | A fast terminal app (TypeScript, Node.js, Ink) running on local Ollama models | Less chrome, more reliability. Effort goes into execution, permissions and verification. |

## Architecture (high level)

```text
            ┌──────────── fast paths (no model) ────────────┐
request ──► router ──► exact maths · app/site launch · known commands ──► execute
               │
               └──► planner (local model) ──► structured tool calls
                                                   │
                                        permission service (LOW / MEDIUM / HIGH / CRITICAL)
                                                   │  human approval for anything important
                                                executor ──► observe ──► verify ──► report
```

- **Router fast paths.** Common, unambiguous requests (maths, "open youtube") are handled by deterministic
  code. That makes them instant and exact, and keeps small local models from getting them wrong.
- **Planner / executor split.** The model *plans* by emitting structured tool calls. Separate code *executes*
  them, so planning never gets direct access to the machine.
- **One permission service.** Every action passes through a single authority with four risk levels.
  Every action goes into a tamper-evident log. An emergency stop is built but not yet verified live.
- **Verification.** Each action type has a check (the window exists, the file was written) that has to pass before ORION reports success.
- **Local models.** Runs on Ollama models on the user's own machine. No cloud account is required for the core loop.

## The pipeline

Every request goes through the same stages:

```text
request → understand → plan → authorise → execute → observe → verify → report
```

- **Understand.** Messy, casual wording ("yo pull up youtube") is normalised first. Some things never go to
  a language model: exact arithmetic is handled by a deterministic parser ([orion-math](https://github.com/SaqibsAura/orion-math)).
- **Plan.** The model proposes actions as structured tool calls, not free text.
- **Authorise.** Each action has a risk level. Low-risk actions (opening an app) run straight away.
  Anything important (deleting, sending, paying) waits for a human yes. An earlier approval doesn't carry over to a new action.
- **Execute → observe → verify.** After acting, ORION checks the real result (did the window appear, does
  the file exist) before it reports success.
- **Report.** If verification fails, ORION says so. No fake success messages.

## Design decisions

**CLI first.** ORION started as an Electron desktop app with a 3D HUD. It looked impressive but was heavy and
slow, so I moved to a terminal-first app (TypeScript, Node.js, Ink). The interface should be fast and stay out of the way.

**Local models first.** ORION can run on local models through Ollama, so personal data doesn't have to leave the machine.

**Deterministic code governs, models propose.** Permission checks, the action log and the emergency stop are plain,
testable code. The model never decides whether its own action is allowed.

**Reuse proven patterns.** Instead of reinventing everything, I study how mature agent harnesses handle tool execution,
sessions, permissions and recovery, then adapt the ideas that fit ORION.

## Status (honest)

Measured live on a real Windows machine, November 2026:

| Works today | Not yet |
| :--- | :--- |
| Opening apps and websites | Brightness and volume control |
| Web search, playing a song or video | Clicking buttons inside any app |
| Typing into an app | Reading what's on screen |
| Creating Word, PDF and Excel documents | Multi-app tasks, filling in forms |
| Approval before risky actions | Voice, Arabic |
| Checking work before saying "done" | Desktop control on macOS / Linux |

The full, regularly updated breakdown is on the [project page](https://impacts-ai.com/orion).

## Where it's going

**Zero-bureaucracy:** turning long, confusing digital processes, like government services, into one guided
conversation. ORION does the repetitive steps, and a person approves anything that matters. That's the focus of my work at the
UAE Hackathon 2026 (Responsible AI / Smart Government).

---

Built by Sheikh Mohammed Saqib, founder of [iMpact AI](https://impacts-ai.com) · United Arab Emirates
