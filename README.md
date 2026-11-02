# Letterpress Scripture

A mobile prototype for memorizing text by setting it in type, styled after the Letterpress typing prototype. It comes with Bible verses (KJV) and Shakespeare lines.

Open `index.html` in a browser (it's built for phone screens). A random verse or line appears and you set it in place one word at a time, in order. Each one takes three passes:

1. **Full**: every letter is outlined.
2. **Initials**: only the first letter of each word is shown.
3. **From memory**: only blank underlines remain.

## Two ways in

Pick **Type** or **Speak** in the header.

- **Type**: type a word and press space to send it. Swipe keyboards work without the space bar; tapping a suggestion replaces the word.
- **Speak**: say the verse. Each word flies into place as the recognizer hears it. A word the recognizer settles on that isn't the next one just slips away; the hint tells you what was heard. Voice uses the browser's speech recognition (Chrome on Android, Safari on iOS), needs a microphone permission, and needs the page served over HTTPS.

## Hesitation reveals the word

On the Initials and From memory passes, the word you're on has a thin accent line under it that burns down over 3.5 seconds. Typing or speaking resets it. If it burns out, the word is revealed and you set it in place. A wrong word shakes off (or, when spoken, fades away) but doesn't reveal anything.

## Settings

The sliders button in the header opens a sheet.

**Text**: *Scripture* (41 well-known KJV verses) or *Shakespeare* (48 famous lines from the plays and sonnets). Switching loads a new one straight away.

Three animation choices, each previewed on a sample word as you pick it:

- **Entering**, how a word lands: *Arc* (letters lob in from the tray), *Stamp* (pressed straight in with a bloom of ink), *Ink* (droplets gather into each letter), *Strike* (straight in, fast, landing with a jolt).
- **Revealing**, how a word you hesitated on is shown: *Unfold* (letters tumble up one by one), *Develop* (the word sharpens as a light passes over it), *Bleed* (ink rises from the baseline), *Turn* (letters turn face up like type tiles).
- **Hiding**, how the words clear for the next pass: *Fade* (word by word), *Erase* (an eraser sweeps across), *Scatter* (the type loosens and falls), *Sink* (letters sink into the paper one after another).

Choices are remembered in the browser.

**New verse** (or **New line**) picks another one.
