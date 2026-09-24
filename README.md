<h1 align="center">13.52</h1>
Host of RAWGAME's 11.00-13.52 Webkit exploit chain for launching the latest GoldHen through the console's web browser.

Based on [SLOPKIT](https://github.com/jordyidk/slopkit) by Jordy, originally written for the PS5. [RAWGAME](https://github.com/rawgame4/rawgame4.github.io) ported it to PS4 and brought the lapse and poops kernel exploits onto it to cover firmware up to 13.52. 

## Firmware support

The chain is selected automatically from the browser's User-Agent.

| Firmware | Chain | Tested on hardware |
| -------- | ----- | ------------------ |
| 11.00 | lapse | Yes |
| 11.50 | lapse | Yes |
| 12.00 | lapse | Yes |
| 12.02 | lapse | Yes |
| 12.50 | poops | Yes |
| 12.52 | poops | Yes |
| 13.00 | poops | Yes |

## Usage

1. Open the browser on your PS4 and go to https://actuallyniaxx.github.io/
2. Wait for `CACHED (first run)`. This stores everything in AppCache so later
   runs work offline.
3. Press X to start.

The exploit is not deterministic. A failed attempt usually crashes the browser
or reboots the console, so just reload and try again. If it keeps failing after
a lot of reloads, close the browser fully and reopen it.


Options:

| Parameter | Effect |
| --------- | ------ |
| `?bug=lapse` / `?bug=poops` | Force a chain instead of detecting firmware |
| `?verbose=1` | Full log lines instead of the compacted form |
| `?slots=N` | Override the carrier array size |
| `?payload=1` | Run the payload even if kernel patching was skipped |

## How it works

`core.js` builds an addrof/fakeobj pair in JavaScriptCore and turns it into
arbitrary read/write inside the WebProcess. `mem.js` and `int64.js` wrap that
into a 64-bit memory interface.

From there `chain_lapse.js` and `chain_poops.js` hit the kernel's async I/O
syscalls (`aio_submit_cmd`, `aio_multi_delete`, `aio_multi_poll`) for a
use-after-free, then reclaim the freed object with IPv6 socket options
(`IPV6_RTHDR`, `IPV6_2292PKTOPTIONS`) to reach kernel read/write.

Post-exploitation runs in order, each step gated on the previous one verifying:
escalate to root, escape the sandbox through `fd_rdir`/`fd_jdir`, apply the
firmware's patch blob from `patches/` (read back and byte-checked before it is
enabled), then map and run `payload.bin`.

## Credits

- **RawGame** - Exploit Port
- **Jordy** — SLOPKIT, the PS5 webkit.
- **GoldHEN team** — `payload.bin` is
  [GoldHEN](https://github.com/GoldHEN/GoldHEN), redistributed unmodified.
- The `aio` use-after-free and the IPv6 `pktopts` reclaim strategy behind
  `lapse` and `poops` are public PS4/PS5 community research. The kernel patch
  blobs come from published patch sources.
- Our team — the PS4 port, the kernel chains, the offset tables and the
  delivery layer.

## License

The port work is MIT, see [LICENSE](LICENSE). Bundled third-party components
keep their own terms and are excluded from it: the SLOPKIT primitive layer,
GoldHEN, and the kernel patch blobs.

## Disclaimer

For research and homebrew on hardware you own. Kernel exploits can crash a
console or corrupt storage, so don't run this on a system holding data you
can't lose. Nothing here enables piracy and no game content is included.
