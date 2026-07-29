# The Cant of Sietro — a thieves' cant, one term a day

An automated daily job coins exactly **one new cant term per day** for the underworld of
**Sietro** (a Venice-with-dragons trade republic on the Navinian Coast). Over time the lexicon
grows while the cant stays coherent, because every term must obey the same style and extend the
same core images. It's the same accretion trick you'd use to grow a constructed language —
pointed at a secret criminal argot instead.

## The files
- [SETTING.md](SETTING.md) — the Sietro canon the cant is grounded in (the law, the crime
  families, the districts). The anchor that keeps terms feeling like *this* city. Rarely changes.
- [CANT_STYLE.md](CANT_STYLE.md) — how a term is coined. The coherence anchor (the cant's
  "phonology"): rules, tone, and what a lexicon entry looks like.
- [CORE_IMAGERY.md](CORE_IMAGERY.md) — the ~20 source images (the "roots") that new terms
  extend: water = law, salt = money, crows = death, and so on.
- [CONVENTIONS.md](CONVENTIONS.md) — rules of use (the cant's "grammar"). The job appends one
  new convention every 10 days.
- [CONCEPTS.md](CONCEPTS.md) — the queue: ~150 underworld concepts as checkboxes, coined in
  order, checked off as done.
- [LEXICON.md](LEXICON.md) — append-only output. One line per term. **Never read by the daily job.**
- [STATE.md](STATE.md) — day counter plus the 7 most recent terms, so consecutive days stay
  stylistically continuous.

## How the daily accretion works
Each day the job reads only the small anchor files (setting, style, imagery, conventions, state,
and just the first few unchecked concepts), coins one term for the next unchecked concept,
**appends** it to LEXICON.md via a shell append, checks off the concept, and updates STATE.md.
Every 10th day it also adds one convention.

## Cost design
The job's daily reading cost is **flat** by construction: the anchor files are small and
fixed-size, the recent-terms window in STATE.md is capped at 7 entries, the concept queue is
consulted with a single `grep`, and the ever-growing LEXICON.md is written to with a shell
append but **never read**. No file the job reads grows with the age of the project. Run it on
**Claude Haiku**, the cheapest model — the task is deliberately small enough that it doesn't
need more.

## Pausing
Pause or delete the scheduled routine itself (in Claude's routines settings). Resume it the same
way — the queue picks up exactly where it left off, since all state lives in STATE.md and
CONCEPTS.md.

## Daily routine prompt
Run on Claude Haiku. This is the text that goes in the routine's **Instructions** field,
verbatim. It works entirely on a fixed branch called `claude/cant` so each day accumulates on
the last (see the article for why this matters):

```
You are the daily keeper of the thieves' cant of Sietro, stored in this repo. Do exactly one
small increment, cheaply, then stop. Work entirely on a fixed branch called claude/cant.

1. Put yourself on the accumulation branch, which holds every prior day. Run:
   git fetch origin
   git checkout -B claude/cant origin/claude/cant || git checkout -b claude/cant
   Do ALL work on this branch. Do NOT create any other branch.
2. Read ONLY: SETTING.md, CANT_STYLE.md, CORE_IMAGERY.md, CONVENTIONS.md, STATE.md.
   Do NOT read LEXICON.md — it may be huge.
3. Let N = the Day number in STATE.md, plus 1.
4. Find today's concept: run  grep -n -m1 '^- \[ \]' CONCEPTS.md . That line is the concept.
   If none is found, append one fitting "- [ ] concept" line to CONCEPTS.md and use that.
5. Coin ONE new cant term for that concept, strictly obeying CANT_STYLE.md. It MUST extend one
   of the image families in CORE_IMAGERY.md — never invent an unrelated picture.
6. Check it's free: run  grep -i '^- \*\*<term>\*\*' LEXICON.md . If it matches, tweak until unique.
7. Append to the lexicon WITHOUT reading it:
   printf '%s\n' '- **<term>** — <hidden meaning> — *extends: <image>* — "<one line in context>" (Day N)' >> LEXICON.md
8. Mark the concept done: change its one line in CONCEPTS.md from "- [ ]" to "- [x]".
9. Update STATE.md: set Day to N, and keep only the 7 most recent terms under "Recent terms:".
10. If N is a multiple of 10, add exactly ONE new convention to CONVENTIONS.md that fits the
    cant so far.
11. Commit and push to the same branch:
    git add -A
    git commit -m "Day N: <term>"
    git push origin claude/cant
    Nothing else — no extra files, no pull request, no commentary.
```
