---
title: I Built a Self-Evolving AI System Through 28+ Iterations
description: Learn how I built a Meta AI system that understands goals, makes autonomous decisions, and learns from results. 28+ rounds of iteration revealed 100+ pitfalls.
date: 2026-07-17
categories: [AI, Open Source, Self-Evolving Systems]
tags: [ai, agent, opensource, meta-ai, self-evolving]
---

# FreeCodeCamp Forum Post Draft

> **Title**: I Built a Self-Evolving AI System Through 28 Iterations — Here's What I Learned
> **Category**: General (or "Show and Tell" if available)
> **Tags**: ai, agent, machine-learning, opensource, tutorial
> **Date**: 2026-07-17
> **Author**: zzzh Zhang (github.com/zzhzhangzhihao)

---

## Post Body

### Title: I Built a Self-Evolving AI System Through 28 Iterations — Here's What I Learned

Hey everyone,

Over the past two weeks, I went through **30 iterations** building what I call a "Meta AI" system — an AI that doesn't just execute tasks, but understands goals, makes autonomous decisions, executes independently, and learns from results.

I want to share the real technical journey, the 100+ pitfalls I hit, and the lessons that took me from a "self-indulgent framework" to a system that actually produces value.

---

## The Architecture

My system consists of three core components:

1. **Knowledge Management (Zhiling)** — A SQLite/Turso-powered database that serves as the system's brain, organized into 6 zones (AGENT, IDEA, ISSUE, PROJECT_FILE, WORKSPACE, ARCHIVE)
2. **Loop Engine** — A Python-based execution framework with 12 layers of capability closure
3. **Hermes Agent** — A multi-agent collaboration system that bridges the loop engine with real-world actions

```
[Scanner] → [Questioner] → [Orchestrator] → [Router] → [Executor]
     ↓                                         ↓
[Verifier] ← [Metacognition] ← [Bridge] ← [Persistence]
     ↓
[Recorder] → [Self-Modify] → [Evolution Metrics]
```

---

## 28 Rounds of Iteration — The Real Story

### Rounds 1-5: Skeleton Building
Created the scanner→router→executor→verifier→recorder skeleton. 13 files, 37 validations passed. But the execution chain was empty — the system scanned databases, performed operations, but produced **zero real value**.

**Lesson 1**: Build the skeleton first, then fill in functionality. Use placeholders in executor._handle_*() methods.

### Rounds 6-10: Self-Evolution Closed Loop
Added questioner.py (filter before executing), metacognition.py (detect stuck states), persistence.py (cross-round learning), and bridge.py (loop↔Hermes communication).

**Lesson 2**: Metacognition must come AFTER verification, BEFORE recording. Its job isn't "reporting" — it's "analyzing whether to change strategy."

**Result**: Three zones stabilized (agent=3, workspace≈100, idea=5). But this was a **steady-state trap** — the system spun in circles forever.

### Rounds 11-15: Coordinator + Deep Verification
Built orchestrator.py (dynamic chain selection based on DB state) and improved verifier.py (compares pre/post DB snapshots, not just success=True).

**Lesson 3**: The coordinator must make dynamic decisions based on DB state, not execute all chains. Verification must compare DB snapshots — success=True doesn't mean the execution was effective.

### Rounds 16-20: Vision + Self-Modification
Added vision management (defined ultimate goals), self-modify module (auto-fix buggy modules based on metacognition insights), and parallel evolution evaluation (multi-dimensional scoring).

**Lesson 4**: Vision must have measurable goals. "Make the system better" is meaningless. "Ideas zone consistently produces insights" is verifiable.

### Rounds 21-25: Money Dimension + Steady-State Escape
The turning point. Added money.py to evaluate whether the system produces economic value. Fixed critical bugs:
- **bridge.py duplicate writes**: DB-level deduplication reduced writes from 3/loop to 0 (fully deduplicated)
- **router not recognizing coordinator**: Passed chains_list into config["forced_chains"]
- **steady-state trap**: When any dimension score < 0.3, force-execute ALL candidate chains

**Lesson 5**: Evaluation ≠ Action. Scoring a money metric without injecting execution chains is worthless. When current_monthly_income == 0, force-add agent_govern and skill_execute chains.

### Rounds 26-28: The Awakening
Three questions changed everything:
1. "Where's your money?"
2. "What are your actions?"
3. "What's your loop?"

These exposed a brutal truth: **the loop engine kept optimizing itself without producing real value.**

**Lesson 6 (Most Important)**: Meta AI isn't "creating a money-making skill" — it's going out and making money. Creating a skill is framework optimization, not action. The endpoint of framework optimization is **action**.

---

## 100+ Pitfalls — The Top 10

### 1. Hermes Can Never Start Persistent Processes
Processes started with `terminal(background=true)` in a Hermes session get SIGTERM/SIGKILL when the session ends. Max survival: 7-9 minutes.
**Fix**: Use launchd for production. Hermes only monitors and diagnoses.

### 2. SQLite WAL Inconsistency Causes False Positives
In WAL mode, COUNT(*) aggregate queries return different results than SELECT... detail queries.
**Fix**: Run `PRAGMA wal_checkpoint(TRUNCATE)` on all databases before scanning.

### 3. SQLite rowcount is Unreliable in UPDATEs
`cursor.rowcount` returns actual modified rows, not affected rows.
**Fix**: Use "before/after count comparison" pattern instead of relying on rowcount.

### 4. Archive Must Use `archived=true`, Never `deleted=true`
Catastrophic lesson: a sub-agent accidentally used `deleted=true`, causing 543 AGENT zone entries to disappear from the list.
**Iron Rule**: Always use `-d '{"archived": true}'`, never `-d '{"deleted": true}'`.

### 5. Cloudflare Rate Limiting — Batch PATCH Requires 5-Second Intervals
0.1s/0.5s/1.5s intervals all triggered Cloudflare 1101 rate limiting (<3% success). 5s interval = 100% success.
**Differentiation**: Pure writes (PATCH archived) = 1s ok. Content changes = 5s required.

### 6. Python urllib Gets 403 from Cloudflare
Cloudflare identifies urllib as a bot. All requests (including GET) return 403.
**Fix**: Always use curl or browser console.

### 7. API Pagination `page` Parameter is Ignored
The `page` parameter is completely ineffective — page=1 and page=2 return identical first-page data.
**Fix**: Use `offset` parameter for pagination (offset=0, 200, 400...).

### 8. Steady-State Trap — Infinite Self-Repetition
System ran 538+ rounds, DB state stable at agent=3~4, workspace≈100, idea=5~14. Coordinator always chose FAST mode, executing only 2 chains.
**Root cause**: bridge.py extracted the same insights every time, FAST mode always triggered in steady state, metacognition only reported without acting.

### 9. Instruction Zone Explosion
Executor unconditionally created skills — the root cause of exponential growth in the instruction zone.
**Fix**: Changed to "find match → if not found, only log, don't create."

### 10. Ideas Zone Hollowing Out
Only fixes, no generation. Ideas zone shrank from 209 entries to 0.
**Fix**: Introduced idea_generate chain, extracting methodologies from instruction zone skills to generate new ideas.

---

## Deployment: Production Setup

The only recommended approach: **launchd** on macOS.

```bash
cat > ~/Library/LaunchAgents/com.meta-ai.builder.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.meta-ai.builder</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Library/Frameworks/Python.framework/Versions/3.12/bin/python3</string>
        <string>~/Desktop/智令/scripts/meta_ai_loop_v3.py</string>
        <string>--interval</string>
        <string>60</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>~/Desktop/智令/logs/meta_ai_stdout.log</string>
    <key>StandardErrorPath</key>
    <string>~/Desktop/智令/logs/meta_ai_stderr.log</string>
</dict>
</plist>
EOF
launchctl load ~/Library/LaunchAgents/com.meta-ai.builder.plist
```

---

## Open Source

All of this is built on open source:
- **Hermes Skill System**: https://github.com/zzhzhangzhihao/hermes-skill-system
- **Tech Stack**: Python, SQLite, Turso (libSQL), Next.js 15, Cloudflare Workers

---

## What I Offer

I've spent 28+ rounds building this system from scratch. If you're working on:
- AI Agent development (multi-agent systems, skill-based agents)
- Knowledge management / RAG systems
- Process automation / workflow orchestration
- Self-evolving AI architectures

I'm available for consulting and custom development. Payment via USDT:

**TTzum7xUkDiRxSc2TCfD9ozDetDCRnSQ5C**

Feel free to reach out — happy to discuss your project and see if we can collaborate.

---

## Questions for the Community

1. Has anyone else experienced the "steady-state trap" in iterative AI systems? How did you escape it?
2. What's your approach to verifying that an automated system is actually producing value, not just "running successfully"?
3. Any recommendations for monitoring self-evolving systems in production?

Would love to hear your experiences!

---

*Originally published 2026-07-17 | 28+ rounds of Meta AI iteration | Built with Hermes Agent + SQLite + Python*

