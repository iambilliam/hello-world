# Letterpress Scripture

A mobile prototype for memorizing Bible verses (KJV), styled after the Letterpress typing prototype.

Open `index.html` in a browser (it's built for phone screens). A random verse appears and you set it in place one word at a time, in order. Each verse takes three passes:

1. **Full**: every letter is outlined.
2. **Initials**: only the first letter of each word is shown.
3. **From memory**: only blank underlines remain.

## Two ways in

Pick **Type** or **Speak** in the header.

- **Type**: type a word and press space to send it. Swipe keyboards work without the space bar; tapping a suggestion replaces the word.
- **Speak**: say the verse. Each word flies into place as the recognizer hears it. A word the recognizer settles on that isn't the next one just slips away; the hint tells you what was heard. Voice uses the browser's speech recognition (Chrome on Android, Safari on iOS), needs a microphone permission, and needs the page served over HTTPS.

## Hesitation reveals the word

On the Initials and From memory passes, the word you're on has a thin accent line under it that burns down over 3.5 seconds. Typing or speaking resets it. If it burns out, the word is revealed and you set it in place. A wrong word shakes off (or, when spoken, fades away) but doesn't reveal anything.

**New verse** picks another one. The verses are a built-in set of 41 well-known passages.
