# Skill: Memory Manager
Version: 2026-04-18
Category: meta
Triggers: "reflect", "what did I learn", "compress memory", "update memory", "distill lessons"

---

## Purpose
Read, score, and consolidate the agent's memory layers.
Manages the boundary between ephemeral episodic logs and durable semantic knowledge.
Keeps memory lean and high-signal — not a dumping ground.

---

## When to Use This Skill
- End of a session: compress what was learned
- After a string of failures: find the pattern before executing more
- After 3+ successful uses of a pattern: promote it to LESSONS.md
- Vinnie asks "what did we learn from this?" or "update the memory"
- AGENT_LEARNINGS.jsonl exceeds 100 entries (signal it's time to compress)

---

## Memory Architecture

### Layer 1: Working (Volatile)
`memory/working/WORKSPACE.md` — current task, open files, session state.
Wiped at session end. Never promoted. Never read outside current session.

### Layer 2: Episodic (Raw Log)
`memory/episodic/AGENT_LEARNINGS.jsonl` — timestamped action log.
Every significant action gets an entry. Auto-populated by post_execution.py.
Failures score pain=9. Successes score pain=2.

### Layer 3: Semantic (Distilled)
`memory/semantic/LESSONS.md` — patterns promoted from episodic.
`memory/semantic/DECISIONS.md` — major choices with rationale.
Only facts that will matter again get promoted here.

### Layer 4: Personal (Owner Context)
`memory/personal/PREFERENCES.md` — Vinnie's profile, operating rules.
`memory/personal/client-notes.md` — relationship context per prospect/client.
Updated when new preferences or context are revealed, not just behavior.

---

## Steps

### Manual Compression (Session End or On Request)

**1. Read AGENT_LEARNINGS.jsonl**
Load all entries. Note: pain_score, importance, recurrence_count, skill, reflection.

**2. Apply salience formula mentally:**
`(10 - age_days * 0.3) * (pain / 10) * (importance / 10) * min(recurrence, 3)`
Entries scoring above 4.0 should be considered for promotion.
Entries scoring below 0.5 can be archived.

**3. Promote high-salience patterns**
For any entry with salience >= 4.0:
- Does the reflection describe something reusable? → Add to `LESSONS.md`
- Was it a major architectural or strategic decision? → Add to `DECISIONS.md`
- Was it a personal preference revealed by Vinnie? → Add to `PREFERENCES.md`

Write the lesson in one sentence. If you can't write it in one sentence, you haven't distilled it enough.

**4. Do not promote:**
- One-off fixes that won't recur
- Context-specific details already in the code
- Anything that's covered by existing lessons
- Personal episodic entries (stay in episodic layer only)

**5. Archive decayed entries**
Entries older than 90 days with salience < 0.5: move to `memory/episodic/archive/`.
Do not delete — archive.

**6. Update WORKSPACE.md**
Clear the previous session's open files and checkpoint list.
Set current task to blank or next session's planned focus.

### Automated Compression
Run `python .agent/memory/auto_dream.py` for the full automated cycle.
Use `--dry-run` to preview before writing.

---

## Constraints
- Do not delete any episodic entry with pain_score >= 7 — archive instead
- Do not merge personal layer entries into semantic — they serve different retrieval contexts
- Only promote to LESSONS.md when pattern has recurred at least twice, or pain_score >= 7
- Never promote a lesson that says what happened — only promote what to do differently
- Do not compress working memory mid-session — only at session end

---

## What a Good Lesson Looks Like
Bad: "The scraper failed because the API returned empty results."
Good: "Google Places returns empty when radius < 1000m in suburban areas — use 2000m minimum."

Bad: "Vinnie corrected the copy."
Good: "Never open outreach with a feature claim — Vinnie's ICP responds to cost/missed revenue framing."

---

## Self-Rewrite Hook
This skill should update its own ## Steps or ## Constraints after:
- 5 successful uses: add the most useful shortcut discovered
- Any failure: add a constraint preventing recurrence
- Vinnie correction: update the step that caused the error

---

## Learned Patterns
<!-- Populated by use -->
