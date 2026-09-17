# Design Notes — Buy Flow Prototype

Running log of notable design decisions and the reasoning behind them. Newest entries at the top.

---

## 2026-09-17

**Thesis/Kill Criteria's arrow attaches right after the "…" instead of dropping to its own line.**
Switched `.memo-case-text` from a flex layout to plain inline text flow — a flex item can't merge into a sibling's wrapped last line, so the arrow was always forced onto a new third line. Now it flows like the next word in the sentence and lands wherever the text actually ends.

**Review/Order Filled memo block: Thesis and Kill Criteria's "continue" arrow only shows when there's actually more to see.**
The arrow next to a truncated Thesis/Kill Criteria answer now only renders once the underlying text is long enough to have been truncated (past ~2 lines). A short answer, or an empty "—", no longer shows a dangling arrow that implies there's more hiding behind it.

**Account Select is the one screen in this flow that is NOT skippable.**
Continue on Account Select now starts grayed out (`pointer-events:none`, dimmed — same disabled look already used elsewhere) and only lights up once an account is actually chosen.
- *Why:* Every other step (Classification, Role, Thesis, Kill Criteria, Prediction, Source) is part of the optional reflection memo — the user can skip it and still buy. Which account to buy from isn't part of that memo, it's a required trade detail, so it doesn't get the same skip-friendliness as the memo fields. This is a deliberate exception to the "always skippable" rule noted below, not a contradiction of it — the rule is about the memo content, not every screen in the flow.

**"Included in this memo" chips: outline instead of solid fill.**
Changed the "included" chip state from solid white background + black text/icon to a white outline with white text/icon on a transparent background (the "hidden" chips stay dim gray, unchanged). Brighter/more legible against the dark background than a big white block, while still reading clearly as "on" versus the dim "off" chips.

**No rounded corners on the "Memo Fields" card or its chips.**
Squared off `.memo-chip-block` (was `border-radius:16px`) and every `.memo-chip` pill (was `border-radius:14px`) to `0`.
- *Why:* Flagged as inconsistent — the app's visual language doesn't use capsule/pill rounding anywhere else. The few places that do round corners (bidask bar, cards, keyboard keys) use small 5–8px radii on functional chrome, not fully-rounded pill shapes. The Memo Fields redesign had drifted from that.
- *Rule of thumb going forward:* if a new component needs rounding, keep it in the 5–8px range that matches existing chrome — don't introduce pill/capsule shapes.

**No blinking/flashing feedback, anywhere.**
Built, then removed, a "breathe" pulse animation on the Prediction screen meant to draw attention to an unanswered slider when the user tapped Continue.
- *Why:* Explicitly ruled out as a feedback pattern for this prototype. Also: the app-wide rule is that every step (including Prediction) stays skippable — a user can tap Continue with nothing entered and it just proceeds, same as every other memo step. Since skipping is always allowed anyway, gating the first tap behind an animation added friction without a real payoff, and any pulsing/blinking motion is off the table regardless of purpose.
- *Rule of thumb going forward:* skip-friendliness is a hard invariant of this flow. Don't build "soft blocks" that intercept a first tap before letting the second one through — Continue should always act on the first tap.

**Prediction screen: header + forecast sentence are pinned while the sliders scroll underneath.**
Kept from the "Pinned & Guided" redesign brief — only the "Guided" (blocking) half was reverted, the "Pinned" half stayed since it's pure layout with no motion/blocking implications.

**Prediction sliders default to an empty state, not a pre-filled number.**
Probability and Return Target show "—" until the user actually drags them (slider itself still needs a resting position, so it sits at its min end); Horizon starts with no option selected. The forecast sentence lights up one blank at a time in order (Probability → Return Target → Horizon) so it's clear which one you're currently being asked to answer.
- *Why:* A pre-filled 50%/20% default looked like an answer the user never gave. Nothing about this blocks proceeding — see the skip-friendliness rule above.
- Also: the forecast sentence intentionally does **not** use fixed-width blanks for the numbers — tried that to stop the sentence from reflowing while dragging a slider, but it left ugly gaps once real values were filled in. Natural/tight text flow won out over reflow-stability.

**Probability slider snaps in increments of 10.**

**"Memo Fields" (show/hide toggle) redesigned as its own elevated card** with an eye/eye-off icon per field reflecting included vs. hidden state, replacing plain unlabeled pills. Intent: make it obvious these are persistent toggles the user controls (what counts toward future evaluation of the position), not a one-off filter.

**Hidden fields now stay hidden everywhere downstream, not just on Memo Review.** Previously, dismissing a field (e.g. Classification, Source) only hid it from the Memo Review ledger — the Review and Order Filled screens still showed every field. Fixed by giving the Review/Order Filled memo rows the same `data-memo-field` + hidden-check wiring as the Memo Review ledger. Any new screen that reads memo data in the future needs the same wiring — treat it as a checklist item, not something to eyeball.

**Keyboard-open layout on Select Shares (CAD):** Bid/Ask must always stay visible along with Limit Price/Shares/Duration/Estimated Cost & Weight; Positions/Orders/History is the part that's allowed to end up fully covered by the keyboard. Spacing in that "kb-compact" state was tightened at first, then loosened back up once it was confirmed there's plenty of slack — no need to cram when the tabs section is fine being invisible.

**Quote block layout (CAD only):** swapped so price + % change sit on the left, ticker/classification/account info on the right (previously the reverse). USD and "Page Update" (v2) iterations were left untouched.

**Keyboard redesign (CAD only):** removed the ABC/DEF letter subscripts, gave only the digit keys a solid dark key background, dropped the background off "." and backspace, replaced the pill-shaped backspace icon with a thin arrow glyph, and shortened the keyboard overall. USD/v2 keep the original keyboard untouched.

**"Hide vs. skip" overhaul — the foundational change everything else this session builds on:**
- The show/hide chips moved off the Limit Price screen entirely and onto the bottom of Memo Review — the one screen guaranteed to be seen no matter what gets skipped.
- A hidden field disappears from the Memo Review ledger completely (not a "—" placeholder) — it only exists as a chip.
- Tapping a chip only toggles hidden/shown; it no longer auto-navigates into that field's screen. (Two-tap flow: un-hide the chip, then tap the reappeared ledger row yourself if you want to edit it.)
- The step counter ("n / 6") is dynamic — it only counts currently-active (non-hidden) steps.
