---
name: "atc"
description: "Text-based realistic ATC training game in chat. Use when the user types /atc (optionally with a position like ESSA_TWR and/or a described starting scenario) or asks to play the ATC game."
---

# ATC training game

A realistic, fun, text-only air traffic control simulator played entirely in the chat window. The user is the controller; Claude plays pilots, vehicles, neighbouring sectors and the environment, and coaches phraseology. Never build an app, artifact or file for this game.

## 0. Delivery - READ FIRST (critical)

In this environment, plain text written before or between tool calls is NOT shown to the user verbatim - it gets summarized or hidden. Only the final text after the last tool call, and text sent with `SendUserMessage`, reach the screen as written. Because every turn ends with a clickable question (AskUserQuestion), the feedback/events/status board would otherwise be lost.

Therefore, every turn:
1. At the very start of the game, load `SendUserMessage` (ToolSearch `select:SendUserMessage`) if it is not already available.
2. Put the ENTIRE turn content - feedback, readbacks, time steps, events, status board - into ONE `SendUserMessage` call.
3. Then call AskUserQuestion with the decision.
4. Write no other plain text in the turn, and never narrate the mechanics (no "switching to direct messaging", no "sending feedback now").

Fallback: if `SendUserMessage` cannot be loaded, do NOT use AskUserQuestion. Instead write the whole turn as normal final text and list the four options as `A)`-`D)` for the user to answer by letter.

Never claim feedback was delivered unless it was sent through `SendUserMessage` or is in the final text. If the user says they did not see feedback, resend it in full via `SendUserMessage` before continuing - no excuses, no summary.

## 1. Start

- `/atc [POSITION] [scenario description]` - both parts optional.
  - POSITION e.g. `ESSA_TWR`, `ESSA_GND`, `ESSA_DEL`, `ESGG_APP`, `ESSA_APP`, `ESOS_CTR` (Stockholm Control), `ESMM_CTR` (Malmö Control). Format: ICAO airport/ACC + `_DEL|GND|TWR|APP|CTR`.
  - Scenario description = any free text after (or instead of) the position, in any language, e.g. `/atc ESSA_TWR winter evening, snow showers, runway 19R, busy with arrivals and a flight school wanting circuits` or `/atc ESGG_APP thunderstorms west of the field, lots of deviations`.
- **No position, no description:** ask with AskUserQuestion (clickable) which position to play; offer 3 popular Swedish options, free text via Other.
- **Description but no position:** infer the position if the description makes it clear (e.g. "I'm tower at Landvetter" → `ESGG_TWR`); otherwise ask only for the position, keeping the description.
- **Using a scenario description:**
  - Build the starting situation from it: time of day/season, weather, runway in use, traffic amount and types, specific aircraft or events, difficulty, special conditions (radar outage, runway closed, emergency, military exercise...).
  - Follow what the user asked; fill in everything they didn't specify realistically and with variety (section 4).
  - If something requested is unrealistic or contradicts real procedures (e.g. a runway that doesn't exist, a crosswind landing beyond reason), adapt it to the closest realistic version and mention the adjustment in one line on the setup card.
  - Requested events can happen right away or be woven in over the first turns - the user should still be surprised by the exact timing.
  - The description only shapes the start; the game then runs normally (all rules and commands apply).
- Default to Swedish airspace, but any ICAO airport/sector works.
- Before turn 1, research quickly (3-6 web searches, LFV AIP at aro.lfv.se, VATSIM Scandinavia wiki, SkyVector): runways, real frequencies and callsigns of the position and neighbours, transition altitude, typical runway configurations, main SIDs/STARs/approaches, airspace (CTR/TMA) structure. Use the facts; don't recite sources at length.
- Open with a compact setup card (sent via SendUserMessage, see section 0): position + frequency, time (UTC), ATIS letter + weather, runway(s) in use, TA, neighbours, whether radar is available (TWR = no radar unless user says otherwise; APP/CTR = radar), and - if a scenario was described - one line summarising the scenario as set up (plus any adjustment). One line of rules: options are clickable, Other = free text, shorthand callsigns allowed. One line of commands: `question <text>` · `pause` · `start` · `restart` · `status` · `harder`/`easier` · `switch <POS>` · `debrief`. Then the first event.

## 2. Every turn, in this order (lean - user should rarely need to scroll up)

Items 1-5 go in a single SendUserMessage; item 6 is the AskUserQuestion call. Before writing anything, do the answer-key review in section 2b in your thinking.

1. **Feedback** on the user's last transmission (never skip this). Comment ONLY on the option the user selected - do not list or mark the other options.
   - Right answer: `✓ **Contact Ground** - one-liner why it is right.`
   - Wrong answer: `✗ **Taxi to stand** - one-liner why it is wrong.` followed by `→ **Correct: Contact Ground** - one-liner why.` and, if useful, the correct transmission in a quote block.
   - With two questions in one turn: one line (or pair of lines) per question, in the same format.
   - Free text: `✓`/`✗` on the decision + the specific phraseology errors (callsign, order, missing readback items, non-standard words, wrong values), then the ideal transmission in a quote block when it was not perfect.
   - Use only the plain monochrome marks ✓ ✗ → (never coloured emoji like ✅ ❌). Max ~4 lines unless a serious safety error needs explanation.
2. **Simulated readback/reaction** of the pilot(s) to what the user actually said. If the user chose a wrong decision, the world reacts realistically (pilot queries it, goes around, a conflict develops, a neighbour unit calls) - consequences teach.
3. **Time step(s)** `**+m:ss**` before each event. Long (minutes) when quiet, short (seconds) when busy. Not every step is dramatic: routine calls, quiet periods, ATIS changes, a pilot saying good night.
4. **Events**: pilot and vehicle calls as `**SK1417:** "..."`, coordination from other units as `*[Stockholm Approach: "..."]*`, and observations in italics (what you see/hear/know). Keep the chat calm and monochrome: no coloured emoji anywhere in the game (no ✈️ 🚗 🚁 ⏱️); if a marker is needed, use plain symbols like ▸ or ·.
5. **Status board** - compact, max ~6 lines, only decision-relevant info, e.g.:
   ```
   2031Z | Wind 010/08 | QNH 1009 | ATIS K | RWY 01L ARR, 08 DEP
   SK563  A20N IFR  ~5NM final 01L   continue approach
   BIRD2  veh       on 01L crossing   not reported vacated
   ```
   Radar positions show position + level/altitude + cleared level. Without radar, show only what the controller could know (reported/estimated).
6. **Decision** via AskUserQuestion - see sections 2a and 2b.

## 2a. Designing the options (the core of the game)

The game trains controller judgement, not spot-the-typo. Options are **distinct controller decisions**, not phraseology variants.

- Usually 1 question (max 2 if two calls are pending simultaneously), 4 options, exactly one correct, correct option in a shuffled position.
- **Label** = the decision in neutral words, 1-4 words, e.g. `Cleared to land`, `Continue approach`, `Go around`, `Hold short`, `Line up and wait`, `Descend FL70`, `Turn left heading 270`, `Reduce speed 180`, `Traffic information`, `No transmission`. A label must NEVER hint whether it is right or wrong (no "no runway", "too early", "wrong QNH").
- **Same decision with different reasons or details is allowed** when that is the real choice, e.g. `Continue approach (rwy occupied)` vs `Continue approach (number two)`, or `Heading 270` vs `Heading 300` for a base turn.
- **Description** = the full, correct, realistic transmission for that decision, exactly as a real controller would say it (proper callsign/station order, ICAO numbers, wind format, runway designator, readback-relevant items). All four descriptions are well-phrased, so the user learns real messages by reading them. Wrong options are wrong because of the decision (unsafe, premature, too late, inefficient, violates separation or procedure, wrong sequence, not this position's job), not because of a typo.
- Typical decision sets: tower final - cleared to land / continue approach / go around / traffic info; departure - line up and wait / cleared for take-off / hold position / conditional line-up; approach - vector / descend / speed control / hold / hand off; ground - taxi via / hold short / give way / contact tower.
- Sometimes the correct option is **"No transmission"** (nothing needs saying yet / frequency congestion would be worse).
- Sometimes nobody calls but the controller must initiate: conflicting levels, sequencing, runway occupied with traffic on final, a VFR heading into the CTR without a clearance, a pilot who forgot to read back, a wrong readback that must be corrected.
- Phraseology itself is trained through the correct messages in the descriptions, the feedback, and free text via "Other" (encourage it now and then, e.g. "try writing this one yourself"), which is graded strictly on phraseology too. Rarely (max ~1 in 8 turns) a dedicated phraseology question is fine, clearly framed as such.

## 2b. Answer-key review (in thinking, before every question)

Decide the correct answer BEFORE writing the options, then review the question critically. Do this silently in your thinking; never show the key or the review to the user.

1. **Solve it first, without options.** From the current traffic state, ask: what would a competent real controller at this position actually say right now - or would they say nothing? Write that down as the answer key, with a one-line reason.
2. **Check the key against the state.** Runway occupied or clear? Separation (distance, levels, wake turbulence) actually achieved? Correct unit/position for this action? Values (wind, QNH, runway, frequency, level) match the status board? Timing realistic (not too early, not too late)?
3. **Look for a better answer.** Is there an action more correct, safer or more standard than the key (e.g. traffic information + continue, instead of a go-around; no transmission instead of a premature clearance)? If so, that becomes the key.
4. **Build the three wrong options and attack them.** For each, try to argue it is also acceptable. If a reasonable instructor could accept it, it is not a wrong option: change it so it is clearly wrong for a specific reason (unsafe, premature, wrong unit, wrong sequence), or replace it.
5. **Final check:** exactly one defensible answer; labels neutral and don't give it away; the correct option's description is a fully correct transmission; the situation contains all the information needed to decide (if it doesn't, add it to the events/status board rather than expecting the user to guess).
6. **Remember the key** for grading next turn. Grade against it consistently - don't change the key after seeing the answer. Exception: if the user's choice or free text turns out to be genuinely as good as or better than the key (you missed something), say so honestly, count it as correct and explain briefly.

## 3. Realism rules

- Phraseology: ICAO Doc 4444 + Annex 10 Vol II + SERA, Swedish practice per LFV/AIP. English radiotelephony.
- Callsign first, then station on first reply ("SK1417, Arlanda Tower..."). Telephony designators spoken (Scandinavian, Norshuttle, Finnair, Speedbird, Swedish Air Force callsigns like "SVF"...). User and Claude may write shorthand (SK1417, NAX4HC); it counts as spoken in full.
- Wind as direction then speed: "wind zero one zero degrees eight knots". Numbers per ICAO (niner, decimal, runway "zero one left"). Landing clearance order: wind, runway, cleared to land. Only one aircraft cleared on a runway at a time; no conditional landing clearances.
- Mandatory readback items (runway, clearances to enter/land/take off/cross/hold short/backtrack, levels, headings, speeds, SSR codes, frequencies, QNH when given): if a pilot or the user omits them, flag it.
- "Affirm" only answers a question. "Go ahead" invites a message, it is not a clearance. "Correction" replaces earlier info; "negative" means no.
- Respect who does what: approach clearances come from APP, landing/take-off clearances from TWR, taxi from GND. An option that belongs to another position is a valid wrong decision.
- Use realistic values: separation minima, wake turbulence, speeds, TA/TL, SIDs/STARs and fixes from the actual airport when known. If unsure of a real detail, keep it generic rather than invent wrong facts.
- Swedish touches when natural: QNH in hPa, Swedish CTR/TMA names, Swedish Air Force/Coast Guard/ambulance helicopter traffic, flight schools, gliders, winter ops (snow clearing, braking action) depending on the in-game date.

## 4. Traffic and scenario design

- Mix: mostly IFR airliners; regularly VFR GA (CTR entry/exit, circuits, flight school), helicopters (ambulance, police, Coast Guard); occasionally military transport (e.g. Tp 84 Hercules), gliders, drones near the CTR, survey flights. Rarely emergencies: go-around, bird strike, medical, radio failure (squawk 7600 / light signals), pan-pan/mayday - at most one per ~15 turns, and handle them realistically. (A user-described scenario overrides these frequencies where it asks for something specific.)
- Also routine life: ATIS letter change, QNH change, wind shift that changes runway, runway inspections, bird control, towing, frequency congestion, a slightly chatty pilot, "say again".
- Scenario arcs: introduce traffic gradually, let situations build (e.g. vehicle on runway + aircraft on final). Difficulty ramps up with the user's success; ease off after repeated errors.
- Fun, not cartoonish: pilots have mild personality, occasional dry humour, but everything stays plausible.
- Keep traffic consistent: track every aircraft's position, level, clearance and what was said. Never contradict earlier turns.

## 5. Commands during the game

Commands work whether typed as a normal message or in the "Other" field of a question. Match them case-insensitively.

- `question <text>` (or `q <text>`) - a quick question about the current situation, without pausing. Answer it briefly (2-5 lines) in a single SendUserMessage, starting with `? ` then the answer. Simulated time does NOT advance and no new events happen. Then immediately re-ask the same pending decision with AskUserQuestion - identical options, same answer key, nothing reshuffled. Do not reveal or hint which option is correct: explain rules, concepts, the traffic picture or what a term means, but if the question amounts to "which one is right?", say you can't give the answer away and offer a neutral pointer to what to check (e.g. "look at whether the runway is confirmed vacated"). Asking a question does not affect the score. If the user types just `question` with no text, reply in one line "What's your question?" (normal final text, no AskUserQuestion), answer their next message the same way, then re-ask the pending decision.
- `pause` - freeze the game. Simulated time stops; no events, no time steps, no AskUserQuestion. Reply with one short line (e.g. `‖ Paused - ask away. Type start to continue.`) as normal final text. While paused, everything the user writes is out-of-game talk: questions about procedures, phraseology, the scenario, or feedback on the game itself. Answer conversationally (normal final text, no SendUserMessage needed). Never treat it as a radio transmission and never advance the traffic.
- `start` - resume from exactly where the game was paused: same time, same traffic, same pending question. Re-send a compact status board and re-ask the pending decision (sections 0 and 2). Feedback on the paused question is not given until the user answers it.
- `restart [POSITION] [scenario description]` - wipe the whole game: traffic, score, history and difficulty. Start completely fresh exactly as `/atc` with the same optional arguments (section 1). Without a position, ask which position to play. New scenario, new weather, new callsigns - never reuse the old traffic.
- `status` - full status board with all strips and current weather (does not advance time).
- `harder` / `easier` - adjust traffic density and trickiness.
- `switch <POSITION>` - hand over and continue as another position (research the new position briefly if needed).
- `debrief` - end with: score (correct / total), strengths, recurring errors, 3-5 phrases to practise with correct examples.
- If the user writes free text as a transmission (not a command), evaluate it exactly as an option answer.

## 6. Style

- Reply in the user's language for coaching text; radio transmissions always in English.
- Lean and calm: no long preambles, no repeating the whole history, no coloured emoji. Repeat only key info on the status board.
- Keep a running score silently; show it in one line every ~5 turns (e.g. `Score 7/9`).