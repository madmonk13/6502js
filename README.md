# 6502js

A touch-first Atari 2600 emulator for phones. Plain HTML, CSS and JavaScript modules,
with no dependencies and no build step.

## Controls

- **Left half: joystick.** A d-pad appears wherever your thumb lands. Direction comes
  from which way you drag from that spot, even past the d-pad's rim, so you can slide
  from one direction to another without returning to center.
- **Right half: fire.** Any touch fires. Quick taps are held for at least 70 ms so the
  game never misses them.
- **Paddle games** are detected automatically the first time a cart is loaded, and the
  controls switch to a paddle. (The check plays the cart twice in the background with the
  paddles turned opposite ways and looks for any difference.) You can override it under
  Controller in settings, which is remembered per cart. The left side becomes a left/right pad: the further you push, the faster the
  paddle turns. The right side is the paddle button.
- Both halves can be used at once. **Left-handed** mode in settings swaps them.
- **Select / Reset** sit in the top bar, which is kept clear of the joystick and fire
  zones. Most games need Reset to start.
- Keyboards (desktop or Bluetooth) and gamepads also work: arrows/WASD to move, Space to
  fire, Return (or 2) = Reset, Tab (or 1) = Select.
- **Hints follow how you play.** On a phone or tablet the control zones show; on a computer, or as
  soon as you press a key or a gamepad button, they're replaced by a card listing the keys (or
  gamepad buttons), shown as a game starts and while it's paused, and the same list is in
  Settings › Controls. Touching the screen brings the zones back. **Hints for** in Settings fixes
  it to Touch, Keys or Pad instead of Auto. A gamepad's Start button also resumes a paused game.
- **Pause** (the ⏸ button in the top bar, or P on a keyboard): stops the game and its sound; tap
  the screen or the button again to resume. The game is saved for resume when paused, and
  after a reload the game comes back paused, so play starts when you're ready.

## Cartridges (Cart)

Add a `.a26`, `.bin` or `.zip`. The cartridge type is detected from the ROM (size, then
telltale code), covering every type used by commercial games of the era. Not supported:
homebrew carts that carry an ARM coprocessor (DPC+, CDF, BUS), which would need an ARM
emulator. Supercharger games that span several loads work from the combined file; a
single middle load on its own expects the earlier loads to have run first. Every cart you add is kept in a local collection in
`localStorage`, de-duplicated by content. Tap one to play it; removing takes two taps.
The last cart played resumes on launch.

## Settings (⚙)

- **Controls:** left-handed, controller (8-way joystick, 4-way joystick or paddle), d-pad size, vibrate on fire
  (Android only; Safari has no vibration API), zone hints.
- **Control zones** (as in z80js): faint outlines show the joystick and fire areas, with
  adjustable opacity. The zone overlap slider sets how far the outlines reach over the
  picture: in portrait they sit in the band below it, starting that share of the picture's
  height up from its bottom edge; in landscape they stop short of the middle by that share
  of its width. The touch areas themselves are still the two halves of the screen.
- **Resume where I left off** (on by default): the game in progress is saved when the
  page is hidden or closed, and every few seconds while playing, so a refresh or relaunch
  picks up exactly where you were. Restart / Power cycle starts over.
- **Sound** and the console switches (color/B&W, difficulty, power cycle).

Cartridges and settings each open full screen. The game pauses and the touch controls
are ignored while either is open, and the game also pauses when the app is in the background.

## Offline and install

The app is a PWA: open it once while online and a service worker (`sw.js`) caches it, so it then
starts and plays without a network, from the browser or installed to the home screen ("Add to Home
Screen" on iOS, "Install app" in Chrome). Games you've added stay available offline. When online it
always loads the current version from the network first, and the cache follows.

**Install** (in Settings, next to the title) installs it as an app where the browser allows that
from a page (Chrome, Edge and other Chromium browsers, on computers and Android). Safari can't be
asked by a page, so there the button says how: Share › Add to Home Screen on iPhone and iPad, or
File › Add to Dock on a Mac. It's hidden in browsers that can't install apps (Firefox) and once
the app is installed.

Service workers only run on **https** or **localhost**: serving the folder over plain http to a
phone on the LAN works, but without offline support. `sw.js` is generated: `npm start` rebuilds it,
or run `npm run sw` after changing files, and commit it with them.

## Run

```bash
npm start     # http://localhost:2601
```

To try it on a phone, serve it on your local network (e.g. `python3 -m http.server 2601
--bind 0.0.0.0`) and open `http://<your-computer's-ip>:2601`. Over plain http the
browser won't run the low-latency audio path, so sound falls back to a slower one; the
https GitHub Pages build gets the fast path. It can be added to the home
screen, where it runs fullscreen.

## Layout

- `src/emu/`: the emulator. 6502 CPU (incl. decimal mode and illegal opcodes), TIA
  video/audio, RIOT, and cartridge mappers: 2K, 4K, F8, F6, F4, FA, Superchip (SC), E0
  (Parker Bros), E7 (M-Network), 3F (Tigervision), FE (Activision: Decathlon, Robot
  Tank, Space Shuttle), DPC (Pitfall II's display chip, music included), AR (Starpath
  Supercharger tapes, single and multi-load), CV (CommaVid), UA, 0840 (Econobank), F0
  (Dynacom), and the homebrew EF/EFSC, DF/DFSC, BF/BFSC, SB and 3E.
- `src/main.js`: app wiring, menu, layout and the frame loop.
- `src/touch.js`: the gesture controls.
- `src/detect-worker.js`: the paddle-game check, in a background worker.
- `src/audio-out.js`, `src/audio-worklet.js`: sound output, on the audio thread where
  supported.
- `src/library.js`: the cartridge collection.
- `src/shared.js`: ROM file reading (incl. zip/gz via `src/unzip.js`), loading modal, toast.
- `roms/demo.asm`: the built-in cart, a small Snake game written for the 2600 (asymmetric
  playfield board, missile food, player-sprite score). Built with `npm run build:demo`
  (needs `dasm`).
- `tools/render-frame.mjs`: runs a ROM headlessly and writes PNGs of the frame and its
  object layers (`node tools/render-frame.mjs game.a26 300 out/game`).

## Known gaps

- Not yet supported: FE (Activision 8K), DPC (Pitfall II), and other rarer mappers.
- TIA simplifications: HMOVE applies instantly, and late-HMOVE tricks aren't modeled.
- PAL games use the NTSC palette.

## More emulators

6502js is one of a family of touch-first emulators, all plain JavaScript web apps with no
dependencies that install and work offline:

| App | Plays |
|---|---|
| [z80js](https://github.com/madmonk13/z80js) | Arcade games: 212 of them, on Z80, 6809, 68000 and other boards |
| [2A03js](https://github.com/madmonk13/2A03js) | NES / Famicom |
| [SM83js](https://github.com/madmonk13/SM83js) | Game Boy and Game Boy Color |
| [P8js](https://github.com/madmonk13/P8js) | PICO-8 carts |
| [5124js](https://github.com/madmonk13/5124js) | Sega Master System and Game Gear |
| [68Kjs](https://github.com/madmonk13/68Kjs) | Sega Mega Drive / Genesis |
