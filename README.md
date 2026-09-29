# Orenate Macro — bug report & fixes (Prospecting)

## This repository

| | |
|---|---|
| **Download the build** | [OrnateMacroV1.4.8-mod.zip](OrnateMacroV1.4.8-mod.zip) or the [release](../../releases/latest) |
| **What was changed** | [MODIFICATIONS.md](MODIFICATIONS.md) - every edit, the before/after, and what was deliberately left alone |
| **Before you run it** | [REQUIREMENTS.md](REQUIREMENTS.md) - the display, camera and launcher settings that decide whether it works at all |
| **Testing it** | [TESTING.md](TESTING.md) - what to run and what to report back |

> This is a **modified copy** of Orenate Macro v1.4.8, changed locally. It is not an official Orenate release. Get the macro from its authors for updates.

---


Independent bug report for **Orenate Macro v1.4.8**, the AutoHotkey v1.1 macro for the
Roblox game *Prospecting*.

These two bugs were found by **reading the source**, not by dynamic analysis. Both are
quiet failures: nothing crashes, nothing is logged, and the macro keeps running while
doing the wrong thing.

> **No source code from Orenate Macro is redistributed in this repository.** Only the
> single offending lines are quoted, for identification. The fixes below are written
> fresh. Orenate Macro is the work of its authors — this repo is not affiliated with,
> endorsed by, or derived from their distribution. If you are a maintainer and want
> something changed or removed here, open an issue and it will be done.

---

## Summary

| # | File | Function | Effect |
|---|------|----------|--------|
| 1 | `Modules/helpers.ahk` | `CenterOfT()` | Undefined variables — the **dig trigger** samples the wrong pixel on some resolutions |
| 2 | `Modules/Controller.ahk` | `GetHoldTime()` | String truncation — dig hold silently ~10× too short at low Dig Speed |

Both only bite in specific conditions, which is why they survived to release.

---

## Bug 1 — `CenterOfT()` writes to variables that don't exist

**Where:** `Modules/helpers.ahk`, inside `CenterOfT()`. `CenterOfT()` is the **dig
trigger** — the macro calls it to decide when it is standing on diggable ground and the
dig may begin.

The function declares its position as `depX` / `depY`, but two of its resolution
adjustments refer to names that are declared **nowhere in the entire project**:

```ahk
if ( A_ScreenWidth = 1856 && A_ScreenDPI = 96 ) {
    baseX += 0.006          ; baseX is never declared or assigned
}
if ( A_ScreenHeight = 953 && A_ScreenDPI = 96 ) {
    baseY -= 0.006          ; baseY is never declared or assigned
}
```

**Why it silently does nothing:** in AHK v1 an undefined variable is an empty string.
`"" += 0.006` evaluates to `0.006` — the arithmetic succeeds, the result is thrown away,
and no error is raised. The adjustment the author intended never reaches `depX`/`depY`.

**Impact:** on **1856-wide** or **953-tall** screens the probe samples the unadjusted
position. Since this function gates digging, a mis-sampled probe means the dig never
triggers — or triggers in the wrong place.

**Evidence:** `baseX` and `baseY` occur exactly twice each in the whole project — the two
lines above. Nothing reads them, so they cannot have been intentional state.

**Fix:** they mean `depX` / `depY`, the variables this function actually uses.

```ahk
if ( A_ScreenWidth = 1856 && A_ScreenDPI = 96 ) {
    depX += 0.006
}
if ( A_ScreenHeight = 953 && A_ScreenDPI = 96 ) {
    depY -= 0.006
}
```

---

## Bug 2 — `GetHoldTime()` takes the first three *characters*

**Where:** `Modules/Controller.ahk`.

```ahk
GetHoldTime(digSpeed) {
    if (digSpeed <= 0)
        digSpeed := 1
    fullResult := Round(66900 / (digSpeed + 9))
    lastResult := SubStr(fullResult, 1, 3)
    return lastResult
}
```

`SubStr(fullResult, 1, 3)` coerces the number to a string and keeps its first three
characters. That is correct only while the result is at most three digits.

- At the shipped default `DigspeedInput=100` the result is `614` → 3 digits → **no visible
  problem**.
- At **low Dig Speed** the result is 4 digits and gets silently cut:
  `digSpeed = 1` → `6690` → **`"669"`** — a hold **ten times too short**.

**Impact:** the dig hold is far too brief, so digging fails, for exactly the players with
the weakest gear — the ones most likely to be looking for a macro. It also caps the hold
at 999 ms, so a legitimately long hold is impossible.

**When it bites:** the result stays ≤ 999 while `66900 / (digSpeed + 9) < 1000`, i.e.
`digSpeed > ~58`. Above that, the truncation is harmless and the bug is invisible.

**Fix:** return the full value, and clamp it so a mistyped stat can't produce a 0 ms or
absurd hold.

```ahk
GetHoldTime(digSpeed) {
    if (digSpeed <= 0)
        digSpeed := 1
    ms := Round(66900 / (digSpeed + 9))
    if (ms < 60)
        ms := 60
    if (ms > 4000)
        ms := 4000
    return ms
}
```

---

## Applying the fixes

Both are small, local edits — one line each for Bug 1, one function for Bug 2. Edit the
two files in your Orenate install; nothing else depends on them.

After editing, the dig trigger (`CenterOfT`) will use the corrected position on affected
resolutions, and the dig hold will be correct at every Dig Speed.

---

## Notes on method

- Found by static reading of `Modules/helpers.ahk` and `Modules/Controller.ahk`.
  No memory editing, no binary patching, nothing dynamic.
- Bug 1 was confirmed by counting every occurrence of `baseX`/`baseY` in the project.
- Bug 2 was confirmed by evaluating the formula across a range of Dig Speed values and
  checking at which point the 3-character cut starts discarding digits.
- Bug 1 is the more serious of the two: it gates *when the macro digs at all*, and it
  fails only on specific (less common) display sizes, so most users would never see it —
  they would just find the macro unreliable.

## Reporting

Please report these upstream to the Orenate authors through their own distribution
channels (their Discord), so the fixes can land in the official build rather than living
in a fork.
