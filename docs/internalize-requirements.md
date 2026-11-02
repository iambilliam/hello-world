<!-- Exported from the Claude Doc "Internalize Requirements" (https://claude.ai/artifact/9Z7J45QBDSnWxqCmwbhrXF) on 2026-11-02. The doc is the editable original; this copy sits beside the prototype it describes (index.html). -->

# Internalize Requirements

Nov 2, 2026 · @Thomas Patella

## Purpose and scope

Internalize should reproduce the Letterpress memorization loop of the prototype: a short text is set in type one word at a time, in order, over three passes that hide more each time, by typing or by voice, with hesitation (not mistakes) revealing a word. This document is the complete requirements for that, written to be pasted into the internalize development chat as its brief.

The reference implementation is a single file, `index.html` in [iambilliam/hello-world](https://github.com/iambilliam/hello-world) on branch `claude/busy-ritchie-yetk06` (about 1,000 lines, no build step, no dependencies beyond two Google Fonts). Its `README.md` is the short user-facing description. Attach that repo to the development chat alongside this document: when a requirement below says "as in the prototype", the code is the tie-breaker.

How to read this document:

- Sections 2 to 9 are the requirements. "Must" means the behaviour was arrived at by fixing a real problem on a phone and should not be re-derived; "should" leaves room where the internalize stack has a better idiom.
- Section 10 maps the prototype onto the internalize codebase and says what to reuse.
- Section 11 lists the acceptance scenarios the prototype was tested against. They are the definition of done.
- Section 12 lists what is deliberately unresolved. The appendix holds every constant, copy string and built-in text.

In scope: the full loop (texts, passes, strict order), both input modes, misses, the hesitation reveal, the four-by-three animation styles with landing effects, the settings sheet with its live preview, and the two built-in text sets. Out of scope here: accounts, progress tracking, scheduling or spaced repetition, and a full-Bible source; those are open decisions in section 12.

## The experience

The whole thing is one phone screen that never scrolls: a header, the text being set in the middle, and a tray of letters at the bottom, drawn like type being set on paper. The screen is a three-row grid (header auto, main 1fr, footer auto) that fills the visible viewport and shrinks when the keyboard opens, so the text is always in view above the keys.

**Palette.** Light only, by design (paper); no dark mode. Every colour is a token; nothing is hard-coded in components.

| Token | Value | Used for |
| --- | --- | --- |
| bg | #f3f2f2 | the page |
| surface | #eae9e9 | the settings sheet |
| text | #201f1d | set letters, headings |
| accent | #b68235 | live ink: the next word's dashes, the fuse, flying letters, the caret, selected controls |
| accent-700 | #7d5411 | small accent text (reference, New verse button) |
| accent-200 / 300 / 800 / 900 | #ffe3bf / #facb8d / #5a3b0a / #3a270d | landing effects (bloom, pool, pressed ink) |
| neutral-300 / 400 / 500 / 600 | #d7d3d3 / #bab6b6 / #9b9797 / #7d7979 | hidden dashes / outlines and punctuation / tray letters off track / labels and hints |
| divider | text at 16% | hairlines |

**Type.** Cormorant Garamond (400 for the text being set and the tray, 600 for headings and control labels) and Lora (body: hints, labels). Labels are 10px uppercase with 0.16em letter-spacing (header, meta row, prompt). Hints are Lora italic 13px in neutral-600, centred, at most 34 characters wide. The text being set is Cormorant 400, line-height 1.08, sized to fit the space (section 9). Spacing scale 4.6 / 9.2 / 13.8 / 18.4 / 27.6 px; radii 4 and 7 px.

**How state reads.** A letter that has been set is solid text colour. A letter that is shown but not yet set is a transparent fill with a 1px neutral-400 stroke (an outline). A hidden letter is invisible, with a 1px dash under it (neutral-300); dashes are per letter with gaps, so a hidden word still shows its length. The word up next has accent dashes. Punctuation is always visible: neutral-400 until its word is complete, then text colour, and it is never typed. The three passes are shown as three short ticks beside the reference and pass name.

**Tray.** Letters appear under a hairline rule in Cormorant 34px. In typing mode a 1px accent caret blinks (steps, 1.1 s); in voice mode a small outlined mic glyph pulses while listening. Tray letters are accent while they are a prefix of the next word and neutral-500 once they are not. Each new letter rises and fades in over 200 ms; letters already in the tray do not re-animate.

**Motion vocabulary.** The accent is live ink: letters fly from the tray in accent and settle to text colour; a finished word gets a thin accent rule that grows under it and fades; a finished text bounces its letters once in sequence. Nothing bounces, pops or slides except these.

**Header.** Brand label on the left (the prototype says "Letterpress"; internalize uses its own), a Type / Speak picker in the centre (selected: text colour with an accent underline; the other: neutral-500), and on the right "New verse" (or "New line" for Shakespeare) in accent-700 with an accent underline, plus a sliders icon that opens settings.

**Tone of copy.** Short, plain, typographic; sentence case; no exclamation marks, no emoji. Examples: "Type a word, then space", "Tap anywhere to begin", "Not quite.", "Once more, with less to go on.", "Well set. A new verse is coming." The full list is in the appendix.

## Content and passes

A text is set three times, in strict word order, with less shown each time; the loop below is the core of the product and must survive any re-platforming unchanged.

**Texts.** Two built-in sets, each item a pair of reference and text: Scripture (41 KJV verses, referenced "Book chapter:verse", e.g. John 3:16) and Shakespeare (48 lines, referenced "Play act.scene" or "Sonnet n", e.g. Hamlet 3.1). Items come up in a shuffle-bag order: every item once before any repeats, and never the same item twice in a row. Switching set reshuffles. Both lists are in the appendix and were typed from memory; proofread them before shipping.

**Parsing a text into words.** Split on whitespace. For each token, the letters (A to Z, case-insensitive) form the target in lowercase; every other character is kept for display as punctuation and is never typed. Tokens with no letters are dropped. Examples: "LORD" sets as l-o-r-d but displays LORD; "in't." has target "int" and displays in't.; "green-eyed" is one word, "greeneyed"; "'em" is "em". What lands in a slot is always the text's own character, so capitals come from the text, never from what was typed.

**The three passes.**

1. Full: every letter of every word is shown as an outline.
2. Initials: the first letter of each word is an outline; the rest are hidden, with a dash under each hidden letter.
3. From memory: every letter is hidden; only the dashes remain.

The pass name (Full / Initials / From memory) and three ticks, filled up to the current pass, sit in the meta row with the reference. A word revealed by hesitation (section 6) is shown as outlines for the rest of that pass only.

**Strict order.** The word up next is the first word that is neither set nor in flight. Only that word can be set; any other word offered is a miss (section 6). Its dashes are accent so it is always findable. While a word's letters are flying it counts as reserved, so the following word becomes current at once and a fast typist can run ahead of the animation.

**Setting a word.** The tray's letters fly into the word's slots in order (section 7). A word is set when all its slots are filled; its punctuation turns to text colour and a thin accent rule grows under it. If the word that was sent has a different letter count from the target (possible in voice mode through leniency), every slot is still filled at the end.

**Auto-send.** When exactly one word remains open and the tray spells it, it is sent without waiting for a space or a pause.

**Completion.** When every word is set: the hint reads "Once more, with less to go on." after passes one and two, or "Well set. A new verse is coming." after the third; the letters bounce once in sequence; after 1.9 s the hiding animation runs (section 7) and the next pass of the same text begins, or a new text at pass one. Between passes the words and their layout do not move; only what is shown changes. A new text is laid out fresh and rises in.

**New verse / New line.** Available at any time; swaps the text immediately with a short rise-in (no hiding animation), clears any in-flight letters, and puts the input back where it was (keyboard open, or listening).

## Typing input

The keyboard owns a hidden text field, and the app only ever reads it; this is the one design that made tap typing, swipe typing and suggestion taps all work on Android and iOS, and it must be kept exactly.

**The field.** A real text input, visually hidden (2 px square, opacity 0.01, transparent caret, no pointer events) but focusable, with `autocomplete=off autocorrect=off autocapitalize=none spellcheck=false inputmode=text enterkeyhint=done`. Tapping anywhere on the screen focuses it, which opens the keyboard. On touch devices, until it is focused, the prompt reads "Tap anywhere to begin" and the caret is dim; on a desktop it is focused on load. The screen resizes to the visual viewport so the text stays above the keys (section 9).

**The buffer rule.** The field's value is a running buffer. The app must never write to it while a text is in progress. Every programmatic write resets the keyboard's composition state: suggestion taps stop replacing the current word and append instead, and swipe keyboards stop inserting their automatic space. The value is cleared only when a pass or text is reset, or the input mode changes.

**Reading the buffer.** Keep two things: `consumed`, the number of characters already turned into sent words, and `consumedText`, what those characters were. On every input event:

1. Compare `consumedText` with the start of the current value and set `consumed` to the length of their common prefix. This absorbs a keyboard editing inside text already sent, and a user backspacing into it (the tray simply empties).
2. Split the rest of the value on whitespace. Every segment before the last whitespace is a word to send, in order (letters only, lowercased; empty segments skipped). The segment after the last whitespace is the tray.
3. Set `consumed` to the value's length minus the tray segment's length, and remember that prefix as `consumedText`.

**Why it is shaped this way.** Swipe keyboards insert the space before the next word, not after the last one: after swiping "sinned" the next swipe arrives as "sinned and" in a single event, so the earlier word must be sent from that event. A suggestion tap replaces the word in progress and appends a space. Gboard's backspace after a swiped word removes the whole word. None of these need special handling under the rule above; all of them break under a model that treats each event as appended characters, or that clears the field after each word.

**Sending.** Sending takes the tray's letters and compares them to the next word's target. Equal: the word is reserved and its letters fly (section 7). Different: a miss (section 6). If no word is open, the tray is dropped silently. The last open word auto-sends when spelled (section 3).

**Keys outside the field.** With a hardware keyboard and the field unfocused (desktop), letters append to the buffer, Backspace removes one character, and Space or Enter send the tray; a send from a key sets `consumed` to the end of the buffer so the word is not read again. With the field focused, letters arrive as input events; Enter is intercepted and sends.

**Tray while typing.** Letters show as typed, accent while they are a prefix of the next word. Only letters count; digits and punctuation typed are ignored. Every input event re-arms the hesitation clock (section 6).

**Known trade-off.** Because the buffer holds the text so far, the keyboard's prediction bar gets the sentence as context and may offer the next word. Clearing the buffer at safe points (right after a trailing space, with nothing being composed) would remove that context and should keep swiping intact, but was not tested; treat it as an experiment, not a requirement.

## Voice input

Speaking must feel like typing with the hands free: heard words appear in the same tray and fly into place the same way, within a few hundred milliseconds of being said. The rules below are written against any recognizer that streams results with a provisional/final distinction; the prototype used the browser's Web Speech API, internalize has its own recognizer (section 10).

**Mode.** "Speak" in the header. A tap anywhere starts listening (the tap is the user gesture the microphone permission needs). The prompt reads "Listening" while listening is wanted, even across the recognizer's own restarts, and "Tap anywhere to listen" otherwise; the mic glyph pulses while listening. The keyboard is closed in this mode and hardware typing is ignored. The chosen mode is remembered.

**The transcript is the buffer.** Flatten the recognizer's ordered results into a list of words, each flagged final or provisional. Normalise each word: lowercase, digits 0 to 10 become words ("3" to "three"), then letters only; drop empties. Keep a cursor, the count of words already consumed. On every update:

1. The words from the cursor on are pending. Show up to three of them in the tray, each word its own group with a 0.35em gap.
2. Walk the pending words in order. If the first one(s) match the next target by the rules below, replace that tray group with the target's own letters, send it (it flies), and advance the cursor by the words used. Otherwise, if the word is final, send it as it is (it becomes a miss) and advance. Otherwise stop and wait for the recognizer.
3. Any change in the transcript text re-arms the hesitation clock. If the list is shorter than the cursor, clamp.

A new recognizer session starts a fresh transcript, so the cursor resets with it. Consumed words are never re-read, even if the recognizer later revises them; counting by position tolerates revisions well enough in practice.

**Why provisional words are consumed.** Waiting for final results costs one to two seconds per word and kills the feel. Matching on provisional words gives credit the moment the word is heard, while never counting a provisional word as wrong; a mishearing only costs something once the recognizer has settled on it.

**Matching.** A heard word matches a target when it is identical; when both belong to the same alias class (appendix); or when the target has seven or more letters and the two are one edit apart (insert, delete or substitute one letter). Short words must be exact, so "those" is not "these" and "shalt" is not "shall". Three shapes are tried in this order: one heard word to one target; two or three heard words joined to one target ("whither so ever"); one heard word to the next two targets joined ("today" for "to day", "forever" for "for ever").

**Misses.** A wrong final word drops out of the tray without the shake typing gets (it may be the recognizer's fault), the target word blinks, and the hint reads Heard “word”. Not quite. so a mishearing can be told from a slip.

**Session lifecycle.** Continuous recognition with provisional results, language en-US. When a session ends on its own, start another after 300 ms as long as listening is still wanted and the page is visible. Three sessions in a row that end within a second without a result mean something is wrong: stop and show "Listening keeps stopping. Tap to try again." Map errors to hints: permission denied, "The microphone is blocked here. Allow it in the browser, or open the page in a browser tab."; no microphone, "No microphone was found."; network, "Voice recognition needs a connection."; recognizer unavailable, "Voice input isn't available in this browser. Typing still works." (Speak stays selectable so the message can be read.) The prompt reads "Voice unavailable" while an error stands. When the page is hidden, stop and remember to resume on return; the same when the settings sheet opens and closes. When a session restarts while the hesitation clock is already running for the current word, leave the clock alone.

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

**Hesitation.** The clock runs for the word up next whenever that word is hidden in the current pass (not shown and not already revealed) and the reader is engaged: keyboard open (or a desktop), or listening wanted; never while the settings sheet is open or the page is hidden. It never runs on the Full pass.

- Arm when the word becomes current: at the start of a pass and whenever a word is sent. Re-arm (restart from full) on every input event and every transcript change. Leave it alone when the recognizer merely restarts a session.
- The fuse is the clock made visible: a 1.5 px accent line spanning the word just under its baseline, over the dashes, which turn dim (neutral-300) while it burns. It scales from full width to nothing, from the left, linearly over the allowance.
- When it burns out: the word is marked revealed, its outlines appear with the chosen revealing animation (section 7), its dashes return to accent because it is still the word up next, and the hint reads "Revealed. Set it in place." The reader still has to set it; the clock does not restart for that word. A revealed word stays shown for the rest of the pass and is hidden again on the next.
- Disarm on send (then arm for the next word), on completion, when the keyboard closes or listening stops, when the sheet opens, when the page is hidden (re-arm on return), and on reset.

**The allowance.** The prototype uses a flat 3.5 s. Internalize already has a long-pause allowance that depends on position (about 3.5 s plus a little per letter mid-line, 5.5 s after a line break or stop, 7 s at a chunk's first word); use that function for the fuse length instead of the constant, so the fuse and the engine's shown-word rule agree (section 10).

**Reduced motion.** The clock still runs and the word is still revealed; the fuse does not animate, ghosts are removed at once, and the reveal plays no animation.

## Animations

Three moments are styled, four ways each, and every style is one function that takes a style name and works on any word built the same way, so the settings preview and the live text run identical code. Effects that leave the word (flying letters, rings, bars, ghosts) are drawn in a fixed overlay above everything, with pointer events off. Timings below are the prototype's and are tuned; keep them unless a device proves otherwise.

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

**Other motion.** Completion: each letter lifts 0.07em and back (520 ms), starting 200 ms + 25 ms per letter. New tray letters rise 0.18em and fade in (200 ms). Caret blink: 1.1 s in steps; mic pulse: 1.6 s. Ghosts and the fuse are in section 6. The settings sheet slides up over 300 ms on cubic-bezier(.2, .8, .2, 1) behind a 200 ms backdrop fade, and closes in 220 ms ease-in.

**Reduced motion.** With `prefers-reduced-motion`: letters land at once without flying, ghosts vanish at once, the fuse does not draw its burn, revealing and hiding apply their end state immediately, caret, mic and tray-in animations are off, and the sheet appears in place. State and timing are unchanged.

## Settings sheet

One sheet holds the text source and the three animation choices, and every choice is shown on a sample word the moment it is picked, so nobody has to go back to the text to see what they chose.

**Opening and closing.** The sliders icon at the right of the header opens it. On phones it is a bottom sheet: full width, at most 88% of the height, scrolling inside, surface colour, 7 px top corners, the large shadow, a hairline border, and padding that respects the safe-area insets; from 600 px wide it is a centred 440 px dialog. The backdrop is neutral-900 at 50%. It slides up over 300 ms and closes on Done (top right, Cormorant 600 14 px, accent), on a backdrop tap, or on Escape. Focus moves to Done when it opens.

**Contents, in order.**

1. Title: Settings.
2. Text: a two-option row, Scripture or Shakespeare, with a note under it ("Well-known verses from the King James Bible." / "Famous lines from the plays and sonnets."). Choosing loads a new text at once behind the sheet, with the rise-in; the header button and hints switch between verse and line.
3. A hairline, then the preview stage.
4. Entering, Revealing, Hiding: each a group with a small-caps legend in accent (10 px, 0.1em tracking) followed by a muted sublabel ("a word lands in the verse", "a word you hesitated on is shown", "the words clear for the next pass"), a segmented row of four equal options (Cormorant 600 14 px; the selected one in accent with a 1 px inset accent ring; others tint on hover), and an italic note describing the selected option (appendix).

**The preview stage.** A 132 px panel in paper colour with the sample word "letter" set in Cormorant 44 px, built exactly as a word of the text is (letter slots with rules), and a small tray row (24 px) beneath it; "Tap to replay" sits bottom right. Picking an option plays its moment after a short beat (350 to 450 ms): for Entering, the word starts outlined, the six letters sit in the mini tray, then fly and land with the chosen style; for Revealing, the word starts hidden (dashes), then its outlines appear with the chosen style; for Hiding, the word starts solid, hides with the chosen style, and is restored 700 ms later. Tapping the stage replays the last moment. A new preview cancels the previous one and clears the overlay, so nothing from an earlier style lands late.

**Persistence.** The prototype keeps three keys in browser storage: the input mode (typing or voice), the three animation choices (validated against the known names; defaults Arc, Unfold, Fade) and the text source (default Scripture). In internalize these become user settings stored with the account (section 10).

**While the sheet is open.** The hesitation clock is disarmed, the keyboard is closed, and listening stops (remembering to resume). Typing and speech are ignored. Closing reopens the keyboard or resumes listening, and re-arms the clock.

## Layout and platform details

The screen is a fixed, non-scrolling grid that fits the visible viewport, with the text sized to fill whatever is left between the meta row and the tray.

**Frame.** A fixed element filling the viewport, `display: grid; grid-template-rows: auto 1fr auto` (header, main, footer); `html` and `body` with overflow hidden; user selection off, `touch-action: manipulation`, tap highlight transparent. Side gutters are `max(16px, env(safe-area-inset-left/right))`; the header adds the top inset to its padding and the footer the bottom inset; the viewport meta carries `viewport-fit=cover`. Nothing scrolls except the inside of the settings sheet.

**Main.** A centred column with an 18.4 px gap: the meta row (reference, hairline separator, pass name, three ticks; it may wrap and is centred), the text, and the hint, which keeps a minimum height of 1.6em so the layout never jumps when it empties.

**Sizing the text.** The text is a centred flex-wrap of word elements (column gap 0.32em, row gap 0.12em, line-height 1.08); each word is a flex row of letter slots and punctuation that never breaks inside. Its font size is found by binary search, nine steps between 14 and 72 px, for the largest size at which the block fits both the available height (main's height minus the meta row and hint heights and the gaps) and the available width; take the floor. Re-run on window resize, on visual viewport changes, and once the web fonts have loaded.

**Keyboard and the visual viewport.** On `visualViewport` resize and scroll events, set the frame's top to the viewport's offsetTop and its height to the viewport's height, scroll the window to 0, and re-fit. This is what keeps the text above the on-screen keyboard on iOS and Android.

**The overlay.** A fixed layer over everything (above the settings sheet), pointer events off, overflow hidden, holding every transient effect: flying letters, rings, bars, ghosts, the finished-word rule. It is cleared on New verse, on a source switch, and when the sheet closes.

**Focus and taps.** A tap anywhere on the frame focuses the hidden input without scrolling (typing mode) or starts listening (voice mode). The mode buttons, New verse and the settings icon prevent default on pointerdown so they do not steal focus from the input, and stop their click from reaching the frame. Hardware keys work with the input unfocused (section 4); Escape closes the sheet.

**Visibility.** When the page is hidden: disarm the clock and stop listening, remembering to resume. When it returns: resume listening if in voice mode and re-arm the clock.

**Fonts.** Cormorant Garamond 400 and 600 and Lora 400 regular and italic from Google Fonts with preconnect hints, falling back to Georgia and the system serif.

**Desktop.** The same layout, centred, with the sheet as a dialog, the input focused on load, and the hardware keyboard path; the mic glyph and caret behave the same.

**Reduced motion** is honoured everywhere (section 7).

## Fit with the internalize codebase

Build this as a new presentation of a pass over the recite engine that already exists, not as a second engine: internalize already has the pointer, the alignment, the fuse and the records, and the prototype supplies the rendering, the input handling and the motion. The tokens are already the same.

**What is already there (do not rebuild).**

- `src/engine/recite.ts`: the pure pass state machine. Events in (`result` with `final`, `text`, `alts`; `session-start`/`end`; `onset`; `silence`; `shown`; `tick`), `displayStates()` per word (said, tentative, given, waiting, upcoming), `fuseAt(state, t)` from 0 to 1 (nothing for the first 30% of the allowance, then filling), the allowances in `DEFAULT_CONFIG` (3.5 s mid-line, 1.6x after a line break, 2x after a stop, plus 60 ms a letter, decision #132), given words, strays, the derived grade, and the `RecitedWord` rows that become `pass_words`.
- `src/engine/align.ts`: matching heard words to tokens with edit-distance limits, by language (es, en).
- `src/engine/display.ts` `displayFor()`: which positions are hidden, which keep a first letter, which passes are bare. `MaskedText.tsx` draws that today.
- `src/engine/keys.ts` and `useKeySession.ts`: the keyboard as a second recognizer emitting the same events (#108), with a 400 ms idle giving `silence`.
- Voice: `useVoiceSession` (the stream through the Worker to Deepgram, which never ends on a pause, with the browser recognizer as fallback), `startRecognizer` (delivers the whole session transcript as revisions, final once at session end, with `boundary()`), capability checks and their messages, and the status line (Microphone off / Starting / Listening / Hearing you / Finished).
- `src/index.css` carries the Classical tokens (#43), identical to the prototype's, with Cormorant Garamond and Lora already loaded.
- `docs/design/word-states.html`: the agreed word states, including the fuse (gold fills left to right, invisible for the first third), verge (past 55%: fidget, thicker glow), brink (past 85%: the word bleeds through gold and blurred) and given (gold italic, letters hopping in; it stays gold, the one state that must never look like the reader's own).
- The `settings` table (key and value per user), the settings page at `/me`, `src/release-notes.ts`, and the glossary rule for every line on screen.

**Concept by concept.**

| Prototype | Internalize |
| --- | --- |
| verse or line | chunk; the reference row shows the chunk's name and the exercise label |
| built-in Scripture and Shakespeare sets | the reader's own texts; the two lists can ship as sample texts to import (section 12) |
| Full / Initials / From memory | display states full / first\_letter / blank, but per position: `displayFor()` hides some words and not others while a chunk erodes, so the renderer takes a state per word, never a rule per pass |
| three passes of one text | the three-attempt shape already in Join, Title join and Word drill (read aloud, first letters, from memory) and the two New boxes |
| the word up next, strict order | the engine's frontier; skips, strays, implied and confirmed words are the engine's decisions and the view only draws them |
| sending a word (letters fly) | a position turning tentative or said; fly on the first match for latency and settle to text colour on said |
| a miss (ghosts) | heard words the engine did not place (extras) and substituted words; ghosts drop as in section 6 |
| hesitation reveal (fuse) | the given word: the fuse from `fuseAt()`, the allowance from the config, the word drawn gold italic with the chosen revealing animation and kept gold for the pass; its return next pass and the halt to erosion are already the engine's |
| completion | the end of a pass: rows written as any pass's, grade derived, nothing set from the view |
| tray | the heard line (the provisional transcript echoed under the text) and the typed letters, in one place |
| hint | the guidance line: one line, glossary words, never "I" |
| prompt | the microphone or keyboard status line |
| New verse | not needed; the session plans the next pass |
| settings persistence | `settings` keys per user (proposed `letterpress.entry`, `.reveal`, `.hide`, `.input`), or device storage if animation taste is per device (section 12) |

**What to build.**

1. A letterpress pass view composed as the practice screen is (#43): title row, tube, the text, one guidance line, the control slot holding the tray, the status line and the microphone and keyboard buttons. The app's nav stays as the header's brand slot.
2. A `TypeText` component drawing each word as letter slots with rules, from `displayFor()` and `displayStates()`: solid, outlined, first-letter, hidden, tentative, said, given. Punctuation always shown and never typed. Keep line structure: verse and scripture forms have lines and verse numbers (`segmentBody` yields the newlines), so only prose may flow as the prototype's flex-wrap does; the fit-to-space sizing (section 9) then measures the lined block.
3. The keys adapter rewritten on the buffer rule: `KeyState` gains `consumed` and `consumedText`, the field is never cleared mid-pass (today `keyInput` asks the hook to clear it after every word, which is exactly what breaks swiping and suggestion taps), the same events still come out (onset on a segment's first letter, interim results while a word is typed, a final result when whitespace or Enter ends it), and `boundary` resets the counters. Keep the composition-event handling. Scenarios 2 to 9 become `keyInput` sequence tests.
4. Letters use `\p{L}`, not `[a-z]`: the app is Spanish and English, and `norm` in align.ts already handles diacritics; the prototype's parser is English-only.
5. Leniency: check align.ts for what the prototype added by hand (alias classes, digits to number words, a heard word split in two or three, one heard word for two tokens) and add what is missing as language-specific normalisation under test, never as view logic.
6. An animations module (plain DOM and the Web Animations API, no library) exporting the entering, revealing and hiding functions by style name, the landing effects, and the overlay; the preview stage; the settings UI as a sheet on the practice screen or a section of `/me`.
7. The fuse drawn from `fuseAt()` each animation frame, with verge and brink as the design has them; see the direction question in section 12.
8. Every line through the glossary (appendix) and one line in `src/release-notes.ts` in the same commit.

**Constraints that bind.** The engine stays pure (no DOM, no clock; the fuse reads `t` from the frame). Passes remain the only source of truth; the view writes nothing but a pass. No new dependencies without asking; Playwright never saved. Vitest for the pure parts, 801 tests must stay green, `npx tsc -b --force` and `npm run test:run` clean before anything ships. Typing counts as a silent attempt under the existing rules (past New only spoken attempts count), unless that rule is changed.

## Acceptance scenarios

These are the flows the prototype was tested against in a headless phone browser (iPhone 13 viewport, touch), with a stubbed recognizer for voice; the build is done when each holds. Internalize never takes Playwright as a dependency, so write the pure parts (buffer reading, cursor, fuse rules, matching) as Vitest tests and keep the rest as a device checklist.

**Typing**

1. Tap typing, whole text: each word followed by a space flies into place; the last word is sent as soon as it is spelled, with no space; three passes complete and a new text loads.
2. Swiping with no space bar: the field reads "for", then "for all", then "for all have"; each arrival sends the previous word and shows the newest in the tray.
3. Suggestion tap: with "subbed" in the tray the field becomes "sinned "; "sinned" is sent and the tray is empty.
4. A wrong swipe followed at once by the next: "... wrong" then "... wrong come"; "wrong" misses (its ghosts drop immediately), "come" is in the tray.
5. Gboard word backspace: the field loses its last word; the tray empties and nothing is re-sent.
6. Backspace into sent text, then type: the tray shows only the newly typed letters.
7. Tapping a letter after a swipe arrives as a space plus the letter; the space sends the swiped word.
8. Hardware keys with the field unfocused: letters, Backspace, Space and Enter behave as in section 4; with it focused, Enter sends.
9. The app never writes the field's value between resets (assert the value is untouched after every event).

**Hesitation**

10. From memory, keyboard open: 3.5 s of quiet reveals the first word with "Revealed. Set it in place."; a keystroke within that time restarts the clock; a wrong word reveals nothing; closing the keyboard disarms; Full never arms; Initials arms.
11. While the fuse burns the word's dashes are dim; after the reveal they are accent again and the clock does not restart for that word.

**Voice** (stubbed results)

12. Provisional "the law": "the" flies, "law" waits in the tray. Revised to "the lord is my": three more fly. Final "the lord is my" plus provisional "shepherd eye": shepherd flies, eye waits. "eye" turns final: it drops, hint Heard “eye”. Not quite. The rest completes the pass.
13. Leniency: "whither so ever" sets whithersoever; "today" sets to and day; "forever" sets for and ever; "the" sets thee; "thigh" sets thy; "these 3" sets these three; "showeth" and "handiwork" set sheweth and handywork; "those" does not set these.
14. A session ending on its own restarts within 300 ms; three sessions ending within a second with no result stop with the stalled message; a permission error shows the blocked message and "Voice unavailable".
15. 3.7 s of silence on a hidden word reveals it; a session restart during the burn does not reset the clock.

**Settings**

16. Opening the sheet disarms the fuse, closes the keyboard and stops listening; closing resumes all three. Picking an option plays its preview and updates its note; choices survive a reload; Escape, Done and a backdrop tap all close it.
17. Every one of the twelve styles plays both on the stage and in the text (entering on a send, revealing on a hesitation, hiding at a pass change), with no console errors.
18. Switching Text loads a new item immediately and relabels the button and hints (verse / line).

**Layout**

19. At phone width nothing overflows horizontally; a 38-word verse fits above the tray; opening the keyboard re-fits the text; a long reference wraps the meta row.
20. With reduced motion on, every flow above completes with no animation.

## Known limitations and open decisions

Seven decisions should be made before coding; the rest are limits of the prototype to carry forward knowingly.

**Decide first.**

1. Fuse direction. The prototype drains an accent line from full to nothing; internalize's agreed design fills it gold from empty, invisible for the first third, with verge and brink past 55% and 85%. Recommendation: the app's convention, since `fuseAt()` already returns a fill value and the states are protected in the design review.
2. Order of words. The prototype only ever accepts the word up next; the engine aligns with skips, strays, implied and confirmed words. Recommendation: the view follows the engine; a skipped word is left blank with its dashes, the fuse moves on, and a stray drops as a ghost.
3. Where the three-pass loop lives. Either the existing three-attempt exercises get the letterpress presentation (least change), or a new exercise runs one chunk through full, first letters and blank as the prototype does (closest feel). Recommendation: start with the existing exercises and the New boxes; add the three-pass exercise only if a reader asks for it.
4. Settings storage. Animation taste could be per device (browser storage, no API) or per account (the `settings` table). Recommendation: per account, so a phone and a desk match, with the input mode remembered per device.
5. Sample texts. Whether the 41 verses and 48 lines ship as importable samples; they were typed from memory and need proofreading either way.
6. Typing and the rules. Typing in the letterpress view is the keys engine; past New only spoken attempts count. Keep that, or let a typed attempt count: a rule change, not a view change.
7. Hide delay. The 1.9 s pause before the hiding animation (and the 700 ms flight, 3.5 s allowance) are tuned for the prototype's feel; keep them unless a device says otherwise.

**Limits carried from the prototype.**

- The keyboard's prediction bar sees the text so far and may offer the next word (section 4). The possible mitigation is untested.
- Chrome on Android plays its start sound at every recognizer restart. This affects only the browser fallback; the stream never ends on a pause.
- Voice was tested against a stubbed recognizer only, and never on iOS; the tray-and-flight path on a real phone is the first thing to check.
- The microphone is refused inside embedded frames without a permission policy, and needs HTTPS.
- The prototype parses English letters only and flows every text as prose; internalize has Spanish and lined forms (section 10).
- Ink creates nine droplets per letter; a forty-word chunk lands a few hundred elements per word in sequence. Fine in testing; watch long chunks on older phones.
- Light theme only, by design; no dark mode is planned.
- Accessibility is minimal in the prototype (a labelled hidden input, `aria-live` on the text, labelled radios and the stage). `MaskedText` already labels hidden words; keep that in `TypeText`.

## Appendix

**Constants**

| Name | Value | Where |
| --- | --- | --- |
| FLIGHT\_MS | 700 ms | entering flight (Strike uses 240 ms) |
| letter stagger | min(70, 400 / letters) ms; Stamp 0; Strike 55 | entering |
| HESITATION\_MS | 3500 ms (replace with the engine's allowance) | fuse |
| hide delay | 1900 ms after the last word lands | completion |
| tray-in | 200 ms | new tray letter |
| ghost shake / drop | 320 ms; drop after 420 ms, 260 ms; fast drop 180 ms | miss |
| word rule | 1300 ms | after a word is set |
| recognizer restart | 300 ms; stalled after 3 quick ends (< 1 s, no result) | voice |
| leniency | one edit on targets of 7+ letters; joins of 2 or 3 heard words; one heard word for 2 targets | voice |
| text size search | 9 steps between 14 and 72 px | layout |
| sheet | open 300 ms, close 220 ms; preview beat 350 to 450 ms; restore after 700 ms | settings |

**Copy.** The prototype's lines, and a glossary-conformant line for each (plain statement, no metaphor, no "I", digits for counts). The glossary wins where they differ.

| Prototype | Proposed for internalize |
| --- | --- |
| Type a word, then space | Type a word, then press space. |
| Tap anywhere to begin | Tap anywhere to open the keyboard. |
| Tap anywhere to listen | Tap anywhere to start the microphone. |
| Listening | Listening (the existing status line) |
| Voice unavailable | the capability messages in `capabilities.ts` |
| Type each word in order. Space sends it to its place. | Type each word in order, then press space. |
| Say the verse. Each word finds its place as you go. | Say the chunk from the start. Each word appears as you say it. |
| Only the initials remain. | Only the first letters are shown. |
| From memory. Hesitate, and the word is shown. | Nothing is shown. Stop too long on a word and it is shown; it then counts as a missed word. |
| Not quite. | That is not the next word. |
| Heard “x”. Not quite. | Heard “x”. That is not the next word. |
| Revealed. Set it in place. | The word is shown and counts as a missed word. Say it to go on. |
| Once more, with less to go on. | Correct. Again, with fewer words shown. |
| Well set. A new verse is coming. | Correct. The next chunk follows. |
| Voice input isn't available in this browser. Typing still works. | the capability reason for no recognition |
| The microphone is blocked here. Allow it in the browser, or open the page in a browser tab. | the `not-allowed` message |
| No microphone was found. / Voice recognition needs a connection. | the `audio-capture` and `network` messages |
| Listening keeps stopping. Tap to try again. | Listening stopped 3 times in a row. Tap to start it again. |

Settings labels: Settings; Done; Text (what you set): Scripture, Shakespeare; Entering (a word lands in the verse): Arc, Stamp, Ink, Strike; Revealing (a word you hesitated on is shown): Unfold, Develop, Bleed, Turn; Hiding (the words clear for the next pass): Fade, Erase, Scatter, Sink; Tap to replay. Option notes: Letters lob in from the tray and settle with a ring. / Pressed straight in, with a bloom of ink. / Droplets gather and pool into each letter. / Straight in, fast, landing with a jolt. / Letters tumble up into place, one by one. / The word sharpens as a light passes over it. / Ink rises through the letters from the baseline. / Letters turn face up, like type set in the chase. / Letters dissolve, word by word. / An eraser sweeps across the page. / The type loosens and falls away. / Letters sink into the paper, one after another. / Well-known verses from the King James Bible. / Famous lines from the plays and sonnets. These describe motion and may keep their figures of speech if the review allows; otherwise shorten to the literal motion.

**Alias classes** (a heard word matches a target in the same class): the, thee · thy, thigh · o, oh · handywork, handiwork · saith, sayeth · shewed, showed · sheweth, showeth · labour, labor · neer, never · oer, over · em, them · int, init ("in't" heard as "in it") · valour, valor · glisters, glitters. Digits 0 to 10 become zero to ten before matching.

**Scripture (KJV), 41 verses.** Genesis 1:1 In the beginning God created the heaven and the earth. · Joshua 1:9 Have not I commanded thee? Be strong and of a good courage; be not afraid, neither be thou dismayed: for the LORD thy God is with thee whithersoever thou goest. · Psalm 19:1 The heavens declare the glory of God; and the firmament sheweth his handywork. · Psalm 23:1 The LORD is my shepherd; I shall not want. · Psalm 27:1 The LORD is my light and my salvation; whom shall I fear? the LORD is the strength of my life; of whom shall I be afraid? · Psalm 34:8 O taste and see that the LORD is good: blessed is the man that trusteth in him. · Psalm 37:4 Delight thyself also in the LORD; and he shall give thee the desires of thine heart. · Psalm 46:10 Be still, and know that I am God: I will be exalted among the heathen, I will be exalted in the earth. · Psalm 51:10 Create in me a clean heart, O God; and renew a right spirit within me. · Psalm 118:24 This is the day which the LORD hath made; we will rejoice and be glad in it. · Psalm 119:105 Thy word is a lamp unto my feet, and a light unto my path. · Proverbs 3:5 Trust in the LORD with all thine heart; and lean not unto thine own understanding. · Proverbs 3:6 In all thy ways acknowledge him, and he shall direct thy paths. · Proverbs 18:10 The name of the LORD is a strong tower: the righteous runneth into it, and is safe. · Ecclesiastes 3:1 To every thing there is a season, and a time to every purpose under the heaven: · Isaiah 40:31 But they that wait upon the LORD shall renew their strength; they shall mount up with wings as eagles; they shall run, and not be weary; and they shall walk, and not faint. · Isaiah 41:10 Fear thou not; for I am with thee: be not dismayed; for I am thy God: I will strengthen thee; yea, I will help thee; yea, I will uphold thee with the right hand of my righteousness. · Jeremiah 29:11 For I know the thoughts that I think toward you, saith the LORD, thoughts of peace, and not of evil, to give you an expected end. · Micah 6:8 He hath shewed thee, O man, what is good; and what doth the LORD require of thee, but to do justly, and to love mercy, and to walk humbly with thy God? · Matthew 5:9 Blessed are the peacemakers: for they shall be called the children of God. · Matthew 5:14 Ye are the light of the world. A city that is set on an hill cannot be hid. · Matthew 6:33 But seek ye first the kingdom of God, and his righteousness; and all these things shall be added unto you. · Matthew 11:28 Come unto me, all ye that labour and are heavy laden, and I will give you rest. · John 1:1 In the beginning was the Word, and the Word was with God, and the Word was God. · John 3:16 For God so loved the world, that he gave his only begotten Son, that whosoever believeth in him should not perish, but have everlasting life. · John 11:35 Jesus wept. · John 14:6 Jesus saith unto him, I am the way, the truth, and the life: no man cometh unto the Father, but by me. · Romans 3:23 For all have sinned, and come short of the glory of God; · Romans 6:23 For the wages of sin is death; but the gift of God is eternal life through Jesus Christ our Lord. · Romans 8:28 And we know that all things work together for good to them that love God, to them who are the called according to his purpose. · 1 Corinthians 13:13 And now abideth faith, hope, charity, these three; but the greatest of these is charity. · Galatians 5:22 But the fruit of the Spirit is love, joy, peace, longsuffering, gentleness, goodness, faith, · Ephesians 2:8 For by grace are ye saved through faith; and that not of yourselves: it is the gift of God: · Philippians 4:13 I can do all things through Christ which strengtheneth me. · 1 Thessalonians 5:17 Pray without ceasing. · 2 Timothy 1:7 For God hath not given us the spirit of fear; but of power, and of love, and of a sound mind. · Hebrews 11:1 Now faith is the substance of things hoped for, the evidence of things not seen. · Hebrews 13:8 Jesus Christ the same yesterday, and to day, and for ever. · James 1:5 If any of you lack wisdom, let him ask of God, that giveth to all men liberally, and upbraideth not; and it shall be given him. · 1 Peter 5:7 Casting all your care upon him; for he careth for you. · 1 John 1:9 If we confess our sins, he is faithful and just to forgive us our sins, and to cleanse us from all unrighteousness.

**Shakespeare, 48 lines.** Hamlet 3.1 To be, or not to be, that is the question. · Hamlet 1.3 This above all: to thine own self be true. · Hamlet 1.3 Neither a borrower nor a lender be. · Hamlet 1.3 Give every man thy ear, but few thy voice. · Hamlet 1.5 There are more things in heaven and earth, Horatio, than are dreamt of in your philosophy. · Hamlet 2.2 Though this be madness, yet there is method in't. · Hamlet 2.2 Brevity is the soul of wit. · Hamlet 1.4 Something is rotten in the state of Denmark. · Hamlet 3.2 The lady doth protest too much, methinks. · Macbeth 5.5 Tomorrow, and tomorrow, and tomorrow, creeps in this petty pace from day to day. · Macbeth 4.1 Double, double toil and trouble; fire burn and cauldron bubble. · Macbeth 1.1 Fair is foul, and foul is fair. · Macbeth 5.5 Life's but a walking shadow, a poor player that struts and frets his hour upon the stage. · Romeo and Juliet 2.2 But, soft! what light through yonder window breaks? It is the east, and Juliet is the sun. · Romeo and Juliet 2.2 What's in a name? That which we call a rose by any other name would smell as sweet. · Romeo and Juliet 2.2 Good night, good night! Parting is such sweet sorrow. · Romeo and Juliet 1.5 Did my heart love till now? Forswear it, sight! For I ne'er saw true beauty till this night. · Julius Caesar 1.2 The fault, dear Brutus, is not in our stars, but in ourselves. · Julius Caesar 3.2 Friends, Romans, countrymen, lend me your ears. · Julius Caesar 2.2 Cowards die many times before their deaths; the valiant never taste of death but once. · Julius Caesar 1.2 Beware the ides of March. · As You Like It 2.7 All the world's a stage, and all the men and women merely players. · As You Like It 5.1 The fool doth think he is wise, but the wise man knows himself to be a fool. · Twelfth Night 2.5 Some are born great, some achieve greatness, and some have greatness thrust upon 'em. · Twelfth Night 1.1 If music be the food of love, play on. · Twelfth Night 1.5 Better a witty fool than a foolish wit. · The Tempest 4.1 We are such stuff as dreams are made on, and our little life is rounded with a sleep. · The Tempest 5.1 O brave new world, that has such people in't! · A Midsummer Night's Dream 1.1 The course of true love never did run smooth. · A Midsummer Night's Dream 3.2 Though she be but little, she is fierce. · A Midsummer Night's Dream 3.2 Lord, what fools these mortals be! · The Merchant of Venice 4.1 The quality of mercy is not strained; it droppeth as the gentle rain from heaven upon the place beneath. · The Merchant of Venice 3.1 If you prick us, do we not bleed? If you tickle us, do we not laugh? · The Merchant of Venice 2.7 All that glisters is not gold. · King Lear 1.1 Nothing will come of nothing. · King Lear 1.4 How sharper than a serpent's tooth it is to have a thankless child! · Othello 3.3 O, beware, my lord, of jealousy; it is the green-eyed monster which doth mock the meat it feeds on. · Henry V 3.1 Once more unto the breach, dear friends, once more. · Henry V 4.3 We few, we happy few, we band of brothers. · 1 Henry IV 5.4 The better part of valour is discretion. · 2 Henry IV 3.1 Uneasy lies the head that wears a crown. · Richard III 1.1 Now is the winter of our discontent made glorious summer by this sun of York. · Richard III 5.4 A horse! a horse! my kingdom for a horse! · Much Ado About Nothing 2.3 Sigh no more, ladies, sigh no more, men were deceivers ever. · Measure for Measure 1.4 Our doubts are traitors, and make us lose the good we oft might win by fearing to attempt. · All's Well That Ends Well 1.1 Love all, trust a few, do wrong to none. · Sonnet 18 Shall I compare thee to a summer's day? Thou art more lovely and more temperate. · Sonnet 116 Love is not love which alters when it alteration finds.
