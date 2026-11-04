<!-- Exported from the Claude Doc "Internalize Requirements" (https://claude.ai/artifact/9Z7J45QBDSnWxqCmwbhrXF) on 2026-11-04. The doc is the editable original; this copy sits beside the prototype it describes (index.html). -->

# Internalize Requirements

Nov 2, 2026 · @Thomas Patella

## Purpose and scope

This document specifies two things from the Letterpress prototype for internalize: how typed and spoken input is read into words, and how the input text (the tray) and the destination text (the words being set) are drawn and animated at each stage. It is written to be pasted into the internalize development chat as its brief.

The reference implementation is `index.html` in [iambilliam/hello-world](https://github.com/iambilliam/hello-world) on branch `claude/busy-ritchie-yetk06` (one file, no build step). Attach that repo to the chat and treat the code as the tie-breaker wherever a sentence here is ambiguous.

Out of scope, on purpose: the choice of texts, the settings screen, page chrome and copy, and how any of this scores or schedules practice. Where a requirement meets an internalize concept it is noted in section 8; the pedagogy is the app's own.

Sections 2 to 7 are the requirements. "Must" marks behaviour arrived at by fixing a real problem on a phone and not to be re-derived; "should" leaves room for a better idiom in the internalize stack. Section 9 lists the scenarios the prototype was tested against; they are the definition of done. Section 10 lists what is unresolved; the appendix holds the constants and the alias table.

## The two texts and their states

Two texts share the screen: the destination text, set in type one word at a time, and the input text in the tray beneath it. Both are drawn from the same tokens in the same letterforms (Cormorant Garamond 400), so a letter that leaves the tray lands as the same letter in the text.

**Words, slots and punctuation.** Split the text on whitespace. Each token's letters (case-insensitive) form its target, lowercased; every other character is punctuation, kept for display and never typed; tokens with no letters are dropped. A word is drawn as a row of letter slots, one per letter, each with a thin rule under it, plus its punctuation as plain glyphs, and never breaks across lines. What lands in a slot is always the text's own character: "LORD" is typed l-o-r-d and sets as LORD; "in't." has the target int and shows in't.; "green-eyed" is one word.

**The three stages.** A text is set three times with less shown each time:

1. Full: every letter is an outline.
2. Initials: the first letter of each word is an outline; the other letters are hidden, each with a dash.
3. From memory: every letter is hidden; only the dashes remain.

Between stages the words and their layout do not move; only how each letter is drawn changes. The stage name and three ticks, filled up to the current stage, sit above the text.

**What a letter can look like.**

| State | Letter | Rule under it |
| --- | --- | --- |
| hidden | invisible: transparent fill, no stroke | 1 px neutral-300 dash per letter, so a hidden word still shows its length |
| outlined (shown, not yet set) | transparent fill with a 1 px neutral-400 stroke | transparent on Full; neutral-300 on the other stages |
| up next (the one word that can be set now) | as hidden or outlined | accent dashes |
| fuse burning (hidden word up next, clock running) | as hidden | dashes dim to neutral-300 under a 1.5 px accent line that shrinks from full width to nothing (section 5) |
| revealed (shown by hesitation) | outlined again for the rest of the stage | accent while it is up next |
| in flight (reserved) | slots fill one by one as the letters land; the following word is already up next | accent until the last letter lands, then a thin accent rule grows and fades |
| set | solid text colour | transparent |

Punctuation is neutral-400 until its word is set, then text colour.

**Strict order.** The word up next is the first that is neither set nor in flight. Only it can be set; any other word offered is a miss (section 5). While a word's letters are flying it counts as reserved, so the following word is up next at once and a fast typist runs ahead of the animation. A word is set when all its slots are filled; every slot is filled at the end even when the word sent was shorter, which voice leniency allows. When exactly one word remains open and the tray spells it, it is sent without a space or a pause.

**Stage change.** When every word is set, the letters bounce once in sequence; 1.9 s later the hiding animation runs on the letters that will be hidden next (section 6), the next stage begins in place, and the newly hidden slots fade in as dashes. After the third stage a new text is laid out fresh and rises in.

**The tray.** Input text sits under a hairline rule in Cormorant 34 px, beside a cursor: a 1 px accent caret blinking in steps (1.1 s) when typing, a small outlined mic glyph pulsing (1.6 s) while listening. Letters are accent while they are a prefix of the word up next and neutral-500 once they are not. A new letter rises 0.18 em and fades in over 200 ms; letters already in the tray keep their identity and do not re-animate. In voice mode the tray holds up to three heard words as separate groups 0.35 em apart; group 0 is the word being offered. On a miss the tray's letters lift out as ghosts (section 5) and the tray empties at once. Letters only: digits and punctuation, typed or heard, never appear in it.

**Colours.** text #201f1d; accent #b68235, the live ink (the next word's dashes, the fuse, flying letters, the caret, on-track tray letters); accent-200 #ffe3bf, 300 #facb8d, 700 #7d5411, 800 #5a3b0a and 900 #3a270d in landing effects; neutral-300 #d7d3d3 (hidden dashes), 400 #bab6b6 (outlines, punctuation), 500 #9b9797 (tray letters off track). Light paper only.

## Typing input

The keyboard owns a hidden text field, and the app only ever reads it; this is the one design that made tap typing, swipe typing and suggestion taps all work on Android and iOS, and it must be kept exactly.

**The field.** A real text input, visually hidden (2 px square, opacity 0.01, transparent caret, no pointer events) but focusable, with `autocomplete=off autocorrect=off autocapitalize=none spellcheck=false inputmode=text enterkeyhint=done`. Tapping anywhere on the screen focuses it, which opens the keyboard. On touch devices, until it is focused, the prompt reads "Tap anywhere to begin" and the caret is dim; on a desktop it is focused on load. The screen resizes to the visual viewport so the text stays above the keys (section 7).

**The buffer rule.** The field's value is a running buffer. The app must never write to it while a text is in progress. Every programmatic write resets the keyboard's composition state: suggestion taps stop replacing the current word and append instead, and swipe keyboards stop inserting their automatic space. The value is cleared only when a pass or text is reset, or the input mode changes.

**Reading the buffer.** Keep two things: `consumed`, the number of characters already turned into sent words, and `consumedText`, what those characters were. On every input event:

1. Compare `consumedText` with the start of the current value and set `consumed` to the length of their common prefix. This absorbs a keyboard editing inside text already sent, and a user backspacing into it (the tray simply empties).
2. Split the rest of the value on whitespace. Every segment before the last whitespace is a word to send, in order (letters only, lowercased; empty segments skipped). The segment after the last whitespace is the tray.
3. Set `consumed` to the value's length minus the tray segment's length, and remember that prefix as `consumedText`.

**Why it is shaped this way.** Swipe keyboards insert the space before the next word, not after the last one: after swiping "sinned" the next swipe arrives as "sinned and" in a single event, so the earlier word must be sent from that event. A suggestion tap replaces the word in progress and appends a space. Gboard's backspace after a swiped word removes the whole word. None of these need special handling under the rule above; all of them break under a model that treats each event as appended characters, or that clears the field after each word.

**Sending.** Sending takes the tray's letters and compares them to the next word's target. Equal: the word is reserved and its letters fly (section 6). Different: a miss (section 5). If no word is open, the tray is dropped silently. The last open word auto-sends when spelled (section 2).

**Keys outside the field.** With a hardware keyboard and the field unfocused (desktop), letters append to the buffer, Backspace removes one character, and Space or Enter send the tray; a send from a key sets `consumed` to the end of the buffer so the word is not read again. With the field focused, letters arrive as input events; Enter is intercepted and sends.

**Tray while typing.** Letters show as typed, accent while they are a prefix of the next word. Only letters count; digits and punctuation typed are ignored. Every input event re-arms the hesitation clock (section 5).

**Known trade-off.** Because the buffer holds the text so far, the keyboard's prediction bar gets the sentence as context and may offer the next word. Clearing the buffer at safe points (right after a trailing space, with nothing being composed) would remove that context and should keep swiping intact, but was not tested; treat it as an experiment, not a requirement.

## Voice input

Speaking must feel like typing with the hands free: heard words appear in the same tray and fly into place the same way, within a few hundred milliseconds of being said. The rules below are written against any recognizer that streams results with a provisional/final distinction; the prototype used the browser's Web Speech API, internalize has its own recognizer (section 8).

**Mode.** "Speak" in the header. A tap anywhere starts listening (the tap is the user gesture the microphone permission needs). The prompt reads "Listening" while listening is wanted, even across the recognizer's own restarts, and "Tap anywhere to listen" otherwise; the mic glyph pulses while listening. The keyboard is closed in this mode and hardware typing is ignored. The chosen mode is remembered.

**The transcript is the buffer.** Flatten the recognizer's ordered results into a list of words, each flagged final or provisional. Normalise each word: lowercase, digits 0 to 10 become words ("3" to "three"), then letters only; drop empties. Keep a cursor, the count of words already consumed. On every update:

1. The words from the cursor on are pending. Show up to three of them in the tray, each word its own group with a 0.35em gap.
2. Walk the pending words in order. If the first one(s) match the next target by the rules below, replace that tray group with the target's own letters, send it (it flies), and advance the cursor by the words used. Otherwise, if the word is final, send it as it is (it becomes a miss) and advance. Otherwise stop and wait for the recognizer.
3. Any change in the transcript text re-arms the hesitation clock. If the list is shorter than the cursor, clamp.

A new recognizer session starts a fresh transcript, so the cursor resets with it. Consumed words are never re-read, even if the recognizer later revises them; counting by position tolerates revisions well enough in practice.

**Why provisional words are consumed.** Waiting for final results costs one to two seconds per word and kills the feel. Matching on provisional words gives credit the moment the word is heard, while never counting a provisional word as wrong; a mishearing only costs something once the recognizer has settled on it.

**Matching.** A heard word matches a target when it is identical; when both belong to the same alias class (appendix); or when the target has seven or more letters and the two are one edit apart (insert, delete or substitute one letter). Short words must be exact, so "those" is not "these" and "shalt" is not "shall". Three shapes are tried in this order: one heard word to one target; two or three heard words joined to one target ("whither so ever"); one heard word to the next two targets joined ("today" for "to day", "forever" for "for ever").

**Misses.** A wrong final word drops out of the tray without the shake typing gets (it may be the recognizer's fault), the target word blinks, and the hint reads Heard “word”. Not quite. so a mishearing can be told from a slip.

**Session lifecycle.** Continuous recognition with provisional results, language en-US. When a session ends on its own, start another after 300 ms as long as listening is still wanted and the page is visible. Three sessions in a row that end within a second without a result mean something is wrong: stop and show "Listening keeps stopping. Tap to try again." Map errors to hints: permission denied, "The microphone is blocked here. Allow it in the browser, or open the page in a browser tab."; no microphone, "No microphone was found."; network, "Voice recognition needs a connection."; recognizer unavailable, "Voice input isn't available in this browser. Typing still works." (Speak stays selectable so the message can be read.) The prompt reads "Voice unavailable" while an error stands. When the page is hidden, stop and remember to resume on return. When a session restarts while the hesitation clock is already running for the current word, leave the clock alone.

**Known constraints.** The microphone needs HTTPS and a permission grant, and is refused inside an embedded frame that does not allow it. Chrome on Android plays its start sound each time a session restarts; nothing in the page can suppress it.

## Misses and hesitation

A wrong word costs nothing but a moment; a long silence on a hidden word shows it. Mistakes never reveal anything, and the prototype keeps no strike count.

**A miss.** When the word sent is not the word up next:

1. The tray's letters lift into the overlay as ghosts, fixed copies at their exact positions, and the tray empties at once so the next word can begin.
2. In typing mode the ghosts shake (5 px left, 5 right, 3 left, 320 ms) and after 420 ms drop (14 px down while fading, 260 ms, 25 ms apart). In voice mode they only drop; a mishearing is not scolded.
3. If new letters arrive while ghosts are still up, they drop immediately (180 ms) so the next word is never covered. This matters with swiping, where the miss and the next word arrive in the same moment.
4. The word up next blinks (opacity 1, .35, 1, .35, 1 over 700 ms) so the reader sees which word was wanted.
5. The hint reads "Not quite." (typing) or Heard “word”. Not quite. (voice) until that word is set or revealed.
6. The input event that carried the wrong word re-arms the hesitation clock like any other; the miss itself neither resets nor shortens it.

**Hesitation.** The clock runs for the word up next whenever that word is hidden in the current stage (not shown and not already revealed) and the reader is engaged: keyboard open (or a desktop), or listening wanted; never while the page is hidden. It never runs on the Full stage.

- Arm when the word becomes current: at the start of a stage and whenever a word is sent. Re-arm (restart from full) on every input event and every transcript change. Leave it alone when the recognizer merely restarts a session.
- The fuse is the clock made visible: a 1.5 px accent line spanning the word just under its baseline, over the dashes, which turn dim (neutral-300) while it burns. It scales from full width to nothing, from the left, linearly over the allowance.
- When it burns out: the word is marked revealed, its outlines appear with the chosen revealing animation (section 6), its dashes return to accent because it is still the word up next, and the hint says the word is shown. The reader still has to set it; the clock does not restart for that word. A revealed word stays shown for the rest of the stage and is hidden again on the next.
- Disarm on send (then arm for the next word), on completion, when the keyboard closes or listening stops, when the page is hidden (re-arm on return), and on reset.

**The allowance.** The prototype uses a flat 3.5 s. Internalize already has a long-pause allowance that depends on position (about 3.5 s plus a little per letter mid-line, 5.5 s after a line break or stop, 7 s at a chunk's first word); use that function for the fuse length instead of the constant, so the fuse and the engine's shown-word rule agree (section 8).

**Reduced motion.** The clock still runs and the word is still revealed; the fuse does not animate, ghosts are removed at once, and the reveal plays no animation.

## Animations

Three moments are styled, four ways each, and every style is one function that takes a style name and works on any word built the same way, so a preview and the live text can run identical code. Effects that leave the word (flying letters, rings, bars, ghosts) are drawn in a fixed overlay above everything, with pointer events off. Timings below are the prototype's and are tuned; keep them unless a device proves otherwise.

**Entering** (flight 700 ms; letters start min(70, 400 / n) ms apart):

- Arc (default): a box the size of the target slot starts at the tray letter's position and scale and travels to the slot over 700 ms, cubic-bezier(.45, 0, .15, 1); inside it the letter lobs up by min(140 px, 30% of the distance) with a -6° to 2° to 0° tilt, accent to accent-700 with a 0.6 px blur at the apex. Landing: the slot scales 1.12 to 0.97 to 1 while going accent to accent-800 to text (620 ms); a 1 px accent ring grows from 0.35 to 1.6 of the letter's size and fades (700 ms); on the word's last letter seven accent dots burst outward (560 to 760 ms).
- Stamp: no stagger; the box arrives on cubic-bezier(.3, .9, .25, 1) while the letter swells to 1.18 at 80% and blurs 2 px at the midpoint, accent to accent-800. Landing: the slot presses from 1.22 (blur 3 px, half opacity, accent-800) through 0.97 (accent-900) to 1 (text) over 560 ms; a radial accent-200 bloom grows and fades (900 ms); on the last letter a horizontal accent line under the word draws and fades (800 ms).
- Ink: nine 2 to 5 px accent droplets per letter start scattered over the tray letter and fly, each on its own bend (up to 80 px sideways), to scattered points on the slot, swelling to 1.2 then shrinking to 0.3, over 0.85 to 1.15 of the flight time, 18 ms apart; the letter lands when its last drop arrives. Landing: the slot fills from the baseline up (clip-path inset 100% to 0) with blur 2 to 1 to 0 and accent-700 to accent-800 to text (820 ms); on the last letter an elliptical accent-300 pool under the baseline swells and fades (1 s).
- Strike: a straight 240 ms flight on cubic-bezier(.7, 0, 1, .6), letters 55 ms apart, accent to accent-900. Landing: the slot jolts down 0.07em and to 1.05 scale at 25% then settles to text (340 ms); a 2 px accent-800 impression under the letter flashes and narrows (320 ms); on the last letter the whole word dips 0.04em and back (300 ms).
- After any style's last letter lands: a 1 px accent rule grows under the word (full width by 50%, then fades) over 1300 ms, cubic-bezier(.6, 0, .2, 1).

**Revealing** (played on a word's letter slots once they are painted as outlines):

- Unfold (default): each letter from invisible, 0.22em low, -4° and 5 px blur, overshooting 0.03em high at 70%, to rest; 560 ms, 70 ms apart, cubic-bezier(.2, .8, .2, 1), fill backwards.
- Develop: the whole word sharpens from 14 px blur at 20% opacity, through 3 px at 70%, to clear over 1100 ms, cubic-bezier(.3, .6, .2, 1), while a 1 px accent light (glow 14 px of accent-200) sweeps across the word's width over 1000 ms, in by 10% and out by 90%.
- Bleed: each letter fills from the baseline up (clip-path inset 100% to 0), blur 3 to 0, opacity .4 to 1; 720 ms, 60 ms apart.
- Turn: each letter turns face up from the baseline, perspective 300 px, rotateX -90° to 12° at 70% to 0°, invisible to visible; 560 ms, 65 ms apart.

**Hiding** (played on the letters that will be hidden next: every non-initial letter before Initials, every letter before From memory, every letter and punctuation mark before a new text; animations hold their end state until the pass resets; done when the longest finishes plus 40 ms):

- Fade (default): opacity to 0 over 420 ms ease-in, each word 45 ms after the one before.
- Erase: a 2 px neutral-400 bar with an 18 px halo of paper colour sweeps left to right across the letters' bounding box over about 1 s; each letter fades with a 2 px blur (200 ms) at a delay proportional to its horizontal centre.
- Scatter: each letter slides up to 0.4em sideways and 0.5 to 1.2em down, turning up to 30°, while fading; 520 to 740 ms on cubic-bezier(.4, 0, .8, .4), starting at random within 260 ms.
- Sink: each letter shrinks to 0.7, settles 0.08em, blurs 3 px and fades over 480 ms ease-in, in a wave of min(14 ms, 600 / n) per letter.
- Afterwards the pass changes with no re-layout: the slots that were just hidden fade in as dashes (360 ms); a new text rises in (opacity 0 to 1 while moving up 12 px, 480 ms).

**Other motion.** Completion: each letter lifts 0.07em and back (520 ms), starting 200 ms + 25 ms per letter. New tray letters rise 0.18em and fade in (200 ms). Caret blink: 1.1 s in steps; mic pulse: 1.6 s. Ghosts and the fuse are in section 5.

**Reduced motion.** With `prefers-reduced-motion`: letters land at once without flying, ghosts vanish at once, the fuse does not draw its burn, revealing and hiding apply their end state immediately, and caret, mic and tray-in animations are off. State and timing are unchanged.

## Sizing, viewport and the overlay

The destination text is sized to fill the space left for it and stays above the keyboard; everything that moves between the two texts lives in one overlay.

**Sizing the text.** The text is a centred flex-wrap of word elements (column gap 0.32 em, row gap 0.12 em, line-height 1.08). Its font size is found by binary search, nine steps between 14 and 72 px, for the largest size at which the block fits both the height left between the stage label and the hint and the available width; take the floor. Re-run on resize, on visual viewport changes, and once the web fonts have loaded. The tray's size never changes.

**Keyboard and the visual viewport.** On `visualViewport` resize and scroll events, set the frame's top to the viewport's offsetTop and its height to the viewport's height, scroll the window to 0, and re-fit. This is what keeps the text above the on-screen keyboard on iOS and Android; without it the text is set under the keys.

**The overlay.** A fixed layer over everything, pointer events off, overflow hidden, holding every transient effect: flying letters, rings, bars, ghosts, the finished-word rule. Effects take their positions from `getBoundingClientRect()` when they start, so a re-fit mid-flight is tolerated but not pretty; avoid re-fitting while letters fly.

**Focus and taps.** A tap anywhere focuses the hidden input without scrolling (typing) or starts listening (voice). Any control elsewhere on the screen prevents default on pointerdown and stops its click, so it never steals focus from the input or disturbs the keyboard.

**Visibility.** When the page is hidden: disarm the hesitation clock and stop listening, remembering to resume. On return: resume listening in voice mode and re-arm the clock.

## Where this meets the internalize code

Internalize already owns the matching, the pointer and the fuse; the prototype supplies the input reading, the slot rendering and the motion. Build on these hooks and nothing else.

- `src/engine/recite.ts` is the pass state machine: `result` events with `final`, `text` and `alts`, plus `session-start`/`end`, `onset`, `silence` and `shown`; `displayStates()` gives each word as said, tentative, given, waiting or upcoming; `fuseAt(state, t)` gives the fuse from 0 to 1 (nothing for the first 30% of the allowance, then filling); the allowances live in `DEFAULT_CONFIG` (3.5 s mid-line, 1.6x after a line break, 2x after a stop, plus 60 ms a letter). Drive the slot states and the fuse from these; the prototype's own clock and its `similar()` matcher are not needed.
- `src/engine/keys.ts` and `useKeySession.ts` already treat the keyboard as a recognizer emitting the same events: onset on a segment's first letter, interim results while a word is typed, a final result when whitespace or Enter ends it, silence after 400 ms idle. Rewrite them on the buffer rule of section 3: `KeyState` gains `consumed` and `consumedText`, and the field is never cleared mid-pass. Today `keyInput` returns `clear: true` after every committed word and the hook empties the field, which is exactly what breaks swipe typing and suggestion taps. `boundary` resets the counters (a safe moment to clear). Keep the composition-event handling. The typing scenarios of section 9 become `keyInput` sequence tests.
- `src/voice/recognizer.ts` delivers the whole session transcript as revisions, final once at session end, with `boundary()` marking how many words belong to the chunk that ended; the stream through the Worker (`src/voice/stream.ts`, Deepgram) presents the same handle and never ends on a pause. The cursor of section 4 is the count of transcript words already consumed since the boundary; a word is drawn tentative on its first match and settles to said.
- `docs/design/word-states.html` is the agreed drawing of word states: the fuse (gold filling left to right, invisible for the first third), verge (past 55%: fidget, thicker glow), brink (past 85%: the word bleeds through gold and blurred) and given (gold italic, letters hopping in; it stays gold, the one state that must never look like the reader's own). The slot states of section 2 extend these; given replaces the prototype's revealed look.
- `MaskedText.tsx` draws words today from `displayFor()` (hidden positions, first-letter positions, bare passes); a `TypeText` component replaces it with letter slots. Keep line structure: verse and scripture forms have lines and verse numbers (`segmentBody` yields the newlines), so only prose may flow as the prototype's flex-wrap does, and the fit-to-space sizing measures the lined block.
- Letters are `\p{L}`, not `[a-z]`: the app is Spanish and English and `norm` in `align.ts` handles diacritics; the prototype's parser is English-only. The prototype's leniencies (alias classes, digits to number words, a heard word split in two or three, one heard word for two tokens) belong in `align.ts` as language-specific normalisation under test, never in the view.
- The engine stays pure: the animation module is plain DOM and the Web Animations API, takes a style name, reads `fuseAt()` with `t` from the frame, and writes nothing to state. No new dependencies; Playwright is never saved; Vitest covers the pure parts; `npx tsc -b --force` and `npm run test:run` stay clean.

## Acceptance scenarios

These are the flows the prototype was tested against in a headless phone browser (iPhone 13 viewport, touch) with a stubbed recognizer for voice; the build is done when each holds. Internalize never takes Playwright as a dependency, so the pure parts (buffer reading, cursor, fuse rules, matching) are Vitest tests and the rest is a device checklist.

**Typing**

1. Tap typing, whole text: each word followed by a space flies into place; the last word is sent as soon as it is spelled, with no space; three stages complete and a new text loads.
2. Swiping with no space bar: the field reads "for", then "for all", then "for all have"; each arrival sends the previous word and shows the newest in the tray.
3. Suggestion tap: with "subbed" in the tray the field becomes "sinned "; "sinned" is sent and the tray is empty.
4. A wrong swipe followed at once by the next: "... wrong" then "... wrong come"; "wrong" misses (its ghosts drop immediately), "come" is in the tray.
5. Gboard word backspace: the field loses its last word; the tray empties and nothing is re-sent.
6. Backspace into sent text, then type: the tray shows only the newly typed letters.
7. Tapping a letter after a swipe arrives as a space plus the letter; the space sends the swiped word.
8. Hardware keys with the field unfocused: letters, Backspace, Space and Enter behave as in section 3; with it focused, Enter sends.
9. The app never writes the field's value between resets (assert the value is untouched after every event).

**Hesitation**

10. From memory, keyboard open: 3.5 s of quiet reveals the first word; a keystroke within that time restarts the clock; a wrong word reveals nothing; closing the keyboard disarms; Full never arms; Initials arms.
11. While the fuse burns the word's dashes are dim; after the reveal they are accent again and the clock does not restart for that word.

**Voice** (stubbed results)

12. Provisional "the law": "the" flies, "law" waits in the tray. Revised to "the lord is my": three more fly. Final "the lord is my" plus provisional "shepherd eye": shepherd flies, eye waits. "eye" turns final: it drops, with the heard word named in the hint. The rest completes the stage.
13. Leniency: "whither so ever" sets whithersoever; "today" sets to and day; "forever" sets for and ever; "the" sets thee; "thigh" sets thy; "these 3" sets these three; "showeth" and "handiwork" set sheweth and handywork; "those" does not set these.
14. A session ending on its own restarts within 300 ms; three sessions ending within a second with no result stop with the stalled message; a permission error shows the blocked message.
15. 3.7 s of silence on a hidden word reveals it; a session restart during the burn does not reset the clock.

**Rendering**

16. Every one of the twelve styles plays in the text, entering on a send, revealing on a hesitation, hiding at a stage change, with no console errors.
17. At phone width nothing overflows horizontally; a 38-word text fits above the tray; opening the keyboard re-fits the text and the tray stays in view.
18. The tray never re-animates letters already in it; a new letter rises in once; on a miss the tray is empty before the ghosts have finished dropping.
19. With reduced motion on, every flow above completes with no animation.

## Known limitations and open decisions

Three decisions touch the mechanics and should be made before coding; the rest are limits of the prototype to carry forward knowingly.

**Decide first.**

1. Fuse direction. The prototype drains an accent line from full to nothing; internalize's agreed design fills it gold from empty, invisible for the first third, with verge and brink past 55% and 85%. Recommendation: the app's convention, since `fuseAt()` already returns a fill value and those states are protected in the design review.
2. Order of words. The prototype only ever accepts the word up next; the engine aligns with skips, strays, implied and confirmed words. Recommendation: the view follows the engine; a skipped word keeps its dashes, the fuse moves on, and a stray drops as a ghost.
3. Timings. The 700 ms flight, the 1.9 s pause before hiding and the stagger constants are tuned for the prototype's feel on a phone; keep them unless a device says otherwise, and take the hesitation allowance from the engine rather than the prototype's flat 3.5 s.

**Limits carried from the prototype.**

- The keyboard's prediction bar sees the text so far and may offer the next word (section 3). The possible mitigation is untested.
- Chrome on Android plays its start sound at every recognizer restart; this affects only the browser fallback, since the stream never ends on a pause.
- Voice was tested against a stubbed recognizer only, and never on iOS; the tray-and-flight path on a real phone is the first thing to check.
- The microphone needs HTTPS and a permission grant, and is refused inside embedded frames without a permission policy.
- The prototype parses English letters only and flows every text as prose (section 8).
- Ink creates nine droplets per letter, so a long word lands a few hundred elements in sequence. Fine in testing; watch long chunks on older phones.
- Accessibility is minimal: a labelled hidden input and `aria-live` on the text. `MaskedText` already labels hidden words; keep that in `TypeText`.

## Appendix

**Constants**

| Name | Value | Where |
| --- | --- | --- |
| flight | 700 ms (Strike 240 ms) | entering |
| letter stagger | min(70, 400 / letters) ms; Stamp 0; Strike 55 | entering |
| hesitation | 3500 ms flat in the prototype; the engine's allowance in internalize | fuse |
| hide delay | 1900 ms after the last word lands | stage change |
| tray-in | 200 ms | new tray letter |
| ghost shake / drop | 320 ms; drop after 420 ms over 260 ms, 25 ms apart; fast drop 180 ms | miss |
| word rule | 1300 ms | after a word is set |
| recognizer restart | 300 ms; stalled after 3 ends under 1 s with no result | voice |
| leniency | one edit on targets of 7+ letters; 2 or 3 heard words joined; one heard word for 2 targets | voice |
| text size search | 9 steps between 14 and 72 px | sizing |
| key idle | 400 ms (internalize's `KEY_IDLE_MS`) | typing silence |

**Alias classes** (a heard word matches a target in the same class): the, thee · thy, thigh · o, oh · handywork, handiwork · saith, sayeth · shewed, showed · sheweth, showeth · labour, labor · neer, never · oer, over · em, them · int, init ("in't" heard as "in it") · valour, valor · glisters, glitters. Digits 0 to 10 become zero to ten before matching.
