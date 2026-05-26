# scribblotto

A single-page lottery number picker that conditions its output on mouse-movement data hashed with SHA-512. Supports Powerball and Mega Millions.

The mouse motion is the entropy source. A PRNG is not used at any point.

Try here 

## How it works

1. Each `mousemove` inside the capture area is recorded as a 7-field event: relative time, delta time, x, y, dx, dy, button state. Packed into 36 bytes per event (little-endian `<QQiiiiI`).
2. Per ticket, 64 events (2304 bytes) are concatenated with the optional note text (UTF-8) and hashed with SHA-512 via `crypto.subtle.digest`.
3. The 512-bit digest is read as 16-bit chunks. Each chunk is compared against `floor(65536 / N) * N` and rejected if above that limit, then taken mod N and shifted to the game's range. This is unbiased rejection sampling.
4. Five distinct main numbers, then one bonus number. If the digest runs short (rare — 32 chunks per digest, ~6 needed), it is extended by hashing `digest || counter`.
5. Tickets are computed up front before any animation starts, so copy-to-clipboard works the moment a draw begins.

Game ranges hardcoded:

- Powerball: 5 of 1–69 + 1 of 1–26
- Mega Millions: 5 of 1–70 + 1 of 1–24

## What it does not do

- Improve odds of winning. Output is uniform over the legal range; the lottery's odds are unchanged.
- Use `Math.random()`, `crypto.getRandomValues`, system entropy, or any seeded PRNG. The script will produce no output until enough mouse events are collected.
- Persist tickets. Reload clears state. Theme preference is the only thing stored in `localStorage`.
- Send anything to a server. All computation is local.
