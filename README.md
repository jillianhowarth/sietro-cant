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
- [CORE_IMAGERY.md](CORE_IMAGERY.md) — the seven image families (the "roots") that new terms
  extend: water = law, salt = money, crows = death, and so on. Each family has a stated domain
  of meanings it may carry and a pool of ordinary things its terms are drawn from.
- [CONVENTIONS.md](CONVENTIONS.md) — rules of use (the cant's "grammar"). The job appends one
  new convention every 10 days.
- [CONCEPTS.md](CONCEPTS.md) — the queue: ~150 underworld concepts as checkboxes, interleaved
  across kinds so consecutive days vary, coined in order, checked off as done.
- [LEXICON.md](LEXICON.md) — append-only output. One line per term. **Never read by the daily job.**
- [STATE.md](STATE.md) — day counter, a running count of terms per image family (the job may
  not use the fullest family, which keeps the cant balanced without ever reading the lexicon),
  and the 7 most recent terms as a list of nouns to avoid echoing.

## How the daily accretion works
Each day the job reads only the small anchor files (setting, style, imagery, conventions, state,
and just the first few unchecked concepts), picks the least-used image family, coins one term
for the next unchecked concept as a real ordinary thing from that family's world,
**appends** it to LEXICON.md via a shell append, checks off the concept, and updates STATE.md.
Every 10th day it also adds one convention.

## Cost design
The job's daily reading cost is **flat** by construction: the anchor files are small and
fixed-size, the recent-terms window in STATE.md is capped at 7 entries, the concept queue is
consulted with a single `grep`, and the ever-growing LEXICON.md is written to with a shell
append but **never read**. Balance across image families comes from the per-family tally in
STATE.md, not from reading back what was coined. No file the job reads grows with the age of the project. Run it on
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
5. Choose the image family FIRST. STATE.md lists a count per family. You may NOT use the
   family with the highest count today. Pick the least-used family that can plausibly carry
   the concept. The family must be one of the exact names listed in CORE_IMAGERY.md.
6. Coin the term by this procedure, strictly obeying CANT_STYLE.md:
   a. Name a real, ordinary THING from that family's world that Sietrans already say with its
      plain meaning: a bird, a tool, a knot, a fish, a plant, a job, a weather, an hour of the
      day. That thing is the term. Write down its plain meaning; it goes in the entry.
   b. The term's kind must match the concept's kind: a person-concept gets a noun for a
      person, an act gets a verb phrase, an object gets a noun for an object.
   c. It must not describe the hidden meaning. If the words themselves tell a stranger who
      or what it is (a "quiet hand" for a cheat, a "blind captain" for a bribed official),
      it is a description, not cant. Throw it out and pick a different thing.
   d. Do not use any word on the "Worn words" list in CANT_STYLE.md, and do not echo any
      noun from the "Recent terms" list in STATE.md. Those lists are what to avoid.
7. Write the context line, then check it against every convention in CONVENTIONS.md and
   against SETTING.md. It must not name the Collective, must not name the Watch, must not
   contain any person's name, and must read as complete, innocent talk about the plain
   thing. If it fails any check, rewrite the line or recoin the term.
8. Check it's free: run  grep -i '^- \*\*<term>\*\*' LEXICON.md . If it matches, tweak until unique.
9. Append to the lexicon WITHOUT reading it:
   printf '%s\n' '- **<term>** — <hidden meaning> — *literally: <plain meaning>* — *extends: <exact family name>* — "<one line in context>" (Day N)' >> LEXICON.md
10. Mark the concept done: change its one line in CONCEPTS.md from "- [ ]" to "- [x]".
11. Update STATE.md: set Day to N, add 1 to the count for the family you used, and keep
    only the 7 most recent terms under "Recent terms:".
12. If N is a multiple of 10, add exactly ONE new convention to CONVENTIONS.md that fits the
    cant so far.
13. Commit and push to the same branch:
    git add -A
    git commit -m "Day N: <term>"
    git push origin claude/cant
    Nothing else — no extra files, no pull request, no commentary.
```
