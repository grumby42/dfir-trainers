# _template — starting a new trainer

There is no separate boilerplate file to copy yet: the **`squidoor/` trainer is
the template.** It's built as *architecture, not content* — vanilla JS data
arrays feeding render functions — so a new case reuses the whole shell and only
changes the data.

## How to start a new trainer

1. **Read [`../TRAINER_SPEC.md`](../TRAINER_SPEC.md) first.** It's the contract —
   especially the fact-check bar in section B. Do the sourcing before you touch
   the HTML.
2. **Copy the shell.** `cp ../squidoor/index.html ../<slug>/index.html` and strip
   the case-specific data arrays, keeping the structure, styles, render
   functions, meters, bias thread, case log, and build-stamping.
3. **Reuses as-is** (don't rebuild these):
   - the three modes (walkthrough / exercise generator / case log) and their
     render logic;
   - the Observe → Orient → Relevance → Sufficiency reasoning layer;
   - the framework meters (Kill Chain, NIST IR, PEAK, Pyramid of Pain, Diamond);
   - the confirmation-bias thread + ACH gate + watchlist;
   - the Defender ⇄ Attacker toggle mechanism;
   - the Hunt Lead persona / SITREP scaffolding;
   - `BUILD_ID` / `STAMP_UTC` stamping and keyboard nav.
4. **Rebuild per case** (this is the real work):
   - the walkthrough **step array** (mission brief → eradication);
   - the **framework-meter mapping** for each step;
   - the **Defender ⇄ Attacker** reasoning for each phase (as inference);
   - the **case-specific bias traps** (where would *this* case mislead you?);
   - the **`DETECTIONS.md`** cards for this case;
   - the **credits** (the researchers/teams who reported this case).
5. **Add the case to the exercise generator pool** if it isn't already there.
6. **Run the pre-ship checklist** in `TRAINER_SPEC.md` section B before committing.

## Promotion to a shared shell

When trainer #2 is done, factor the common shell out of the two `index.html`
files into a shared asset here (e.g. `_template/shell.js` + `shell.css`) so
future trainers import rather than copy. Until then, Squidoor is the source of
truth and copy-from-reference is the workflow.

---

*Generated 2026-10-16T01:36Z.*
