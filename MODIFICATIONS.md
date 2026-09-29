# What was changed

`OrnateMacroV1.4.8.zip` in this repository is a **modified copy** of Orenate Macro
v1.4.8. Four changes were made, all listed here so the modification is auditable
rather than invisible.

**Provenance:** Orenate Macro is the work of its own authors, distributed by them
through their Discord. The files carry their notice: "Copyright (c) ORNATE MACRO
(PROSPECTING!)" and "Please do not steal our code". This repository holds an
independently modified copy for archival use. It is not affiliated with, endorsed by,
or maintained by the Orenate authors. If you want the macro, get it from them — that
is where fixes and updates actually land.

---

## 1. #Requires AutoHotkey v1.1 added to every launched script (22 files)

**Why:** the macro would not open. On Windows, `.ahk` is usually associated with the
AutoHotkey **UX launcher**, which picks the interpreter from the script's `#Requires`
header — and **defaults to v2 when there is none**. Orenate is v1.1 code (`Gui, Add`,
`SetTimer,`, `Send,`), so v2 cannot parse it and no window ever appears. Having v1.1
installed is not enough on its own; nothing told Windows to use it.

**Fix:** `#Requires AutoHotkey v1.1` as the first line of all 22 scripts that get
**launched** — the entry point, `Controller.ahk`, and all 20 `Script*.ahk` workers.
Not just the entry point: `Ornate.ahk` does `Run, Controller.ahk` and Controller
spawns the workers the same way, so every one of them re-enters the association and
every one needed the header.

`helpers.ahk` and `CurrentVersion.ahk` are `#Include`d rather than launched and inherit
the parent's engine, so they were left alone.

`RUN ORNATE.bat` is included at the archive root as a fallback: it calls
`AutoHotkeyU64.exe` (v1.1) by absolute path, bypassing the association entirely. It
must sit beside the `OrnateMacroV1.4.8` folder — it does `cd /d "%~dp0OrnateMacroV1.4.8"`.

## 2. CenterOfT() wrote to variables that do not exist

**Why:** `CenterOfT()` is the **dig trigger** — the macro calls it to decide when it is
standing on diggable ground. Two of its resolution adjustments referenced `baseX` and
`baseY`, which are declared nowhere in the project. In AHK v1 an undefined variable is
an empty string, so `"" += 0.006` evaluates to `0.006`: the arithmetic succeeds and the
result is discarded, silently. On **1856-wide** or **953-tall** displays the probe
therefore sampled the unadjusted position.

**Evidence:** `baseX` and `baseY` occur exactly twice each in the whole project — the
two offending lines. Nothing reads them, so they cannot have been intentional state.

**Fix:** the variables this function actually uses are `depX` and `depY`.

## 3. GetHoldTime() returned the first three CHARACTERS

```
fullResult := Round(66900 / (digSpeed + 9))
lastResult := SubStr(fullResult, 1, 3)      <- takes CHARACTERS
return lastResult
```

**Why:** `SubStr(n, 1, 3)` coerces the number to a string and keeps its first three
characters, which is only correct while the result is three digits or fewer. At the
shipped default `DigspeedInput=100` the result is `614`, so the bug is invisible. At low
Dig Speed the result is four digits and is silently cut:

```
digSpeed = 1  ->  6690  ->  "669"     a hold TEN TIMES too short
```

It also caps the hold at 999 ms, so a legitimately long hold is impossible. This hits
exactly the players with the weakest gear.

**When it bites:** the cut only discards digits while `66900 / (digSpeed + 9) >= 1000`,
i.e. below roughly dig speed 58.

**Fix:** return the full value, clamped to 60..4000 ms so a mistyped stat cannot produce
a zero-length or absurd hold.

## 4. The Ko-fi button ran a variable that was never assigned

```
674: global paypal_link := "https://paypal.me/OrnateMacro"
...
OpenKoFi:
    Run, %link%          <- 'link' is assigned nowhere in the project
```

**Why:** pressing the Ko-fi button did nothing at all and gave no feedback. The author
has a `paypal_link` global but never created the matching Ko-fi one.

**Fix:** the handler now fails visibly when the link is unset, instead of silently
running an empty command. No URL was invented — the real one is the author's to supply.

---

## What was NOT changed

Everything else is as shipped, including the balance numbers, the per-location
navigation scripts and the pixel-probe tuning. Those constants were left alone
deliberately: they are the author's tuning, and changing them with no way to test in
game would be guessing.

## Known issues NOT fixed

* **`RightOfPan()` contains dead code.** Both branches of an if/else assign the same
  colour (`0xF9E36E`), so the 768-height special case does nothing — almost certainly
  meant to be a different value.
* **`LeftOfPan()` wastes a pixel read.** It assigns `clr := getPixelColorRelative(...)`
  and then returns `IsPixelColorClose(depX, depY, ...)`, which reads the same pixel
  again.
* **Detection is hand-tuned per resolution** for heights 768/953/1050 and widths
  1366/1680/1856. There is **no 1920x1080 case**, which is a very common display, so on
  those screens every probe runs on its base ratio.

## Verification

Fixes 2 and 3 were confirmed by reading the code and counting symbol occurrences; the
truncation was confirmed by evaluating the formula across a range of Dig Speed values
and finding where the 3-character cut starts discarding digits. **None of this was
verified by running the macro.** It is a static analysis, and the author's own warning
about bugs in this version should be taken seriously.