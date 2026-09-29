# Wanted: testers for the modified Orenate build

This build of Orenate Macro v1.4.8 for **Prospecting!** has been changed by reading code
only. **Nobody has played it.** Every change is a static edit; the behaviour is unproven.
I am looking for people who play the game to find out what it actually does.

You do not need to be a programmer. You need to play the game for ten minutes and report
what happened.

---

## Before you start — these are not optional

The macro's own guidance, and it is correct:

- Windows display scaling at **100%** (not 125%, not 150%)
- Game resolution **1920x1080**
- Roblox: **Screen Shake OFF**
- Roblox: camera **Classic**
- Roblox: **Shift Lock DISABLED** (not enabled)
- Roblox installed from **roblox.com/download**, not the Microsoft Store build
  (the Store build fails to boot with error `0x1023`)
- **AutoHotkey v1.1** installed

Launch with `RUN ORNATE.bat`, which sits beside the `OrnateMacroV1.4.8` folder. It calls
the v1.1 engine directly. **F6 starts, F8 stops.**

If it does not open at all, report that — do not try to fix it.

---

## What to test, in order

**Test 1 — does it open and take the hotkeys?**
Press F6. Does the window respond? Press F8. Does it stop? Report yes/no.

**Test 2 — the dig trigger.** This is the one I most need.
Walk to a **diggable node** and let it dig. Then walk to a **lake or open water** and let
it run. Report:

- Did it dig on the node?
- **Did it try to dig on the water?** (This is the known failure — it currently decides
  "diggable" from ONE pixel, so it can mistake water for ground.)
- Anything it tried to dig that you could not dig by hand.

**Test 3 — the panning.** Did the repeated clicks fill and shake the pan?
Did it stop when the pan was done, or keep clicking?

**Test 4 — movement.** Did it walk in a straight sensible line, or drift, spin,
walk into a wall, or walk off a ledge?

**Test 5 — long run.** Leave it for five minutes. Did anything get stuck, hang, or
stop responding?

---

## What to send back

1. **The Roblox Output window, copied as text.** View → Output. This is the single most
   useful thing — errors name the script and line.
2. **What you actually saw**, in your own words. "It dug on the lake and kept clicking"
   is a perfect report.
3. **Your screen setup**: resolution and display scaling.
4. **The macro's own log**: `Modules/OrnateMacro_Settings.ini` is settings; the session log
   is in the macro folder. Attach it if it exists.

Screenshots help too.

## What NOT to report

- "It doesn't work." That tells me nothing I can act on — I need *what* it did.
- Bugs in the ORIGINAL Orenate that exist in this build too. I only want to know about
  things this build does. (Known and unfixed, no need to report: dead code in
  `RightOfPan`, a wasted pixel read in `LeftOfPan`.)

---

## What was changed, so you know what you're testing

1. `#Requires AutoHotkey v1.1` added to all 22 launched scripts. Without it the Windows
   `.ahk` association hands v1 code to the v2 interpreter, which cannot parse it, so the
   macro would not open **at all** even with v1.1 installed. This is why it opens now.
2. `CenterOfT()` — the **dig trigger** — was adjusting variables (`baseX`/`baseY`) that are
   declared nowhere, so the adjustment was silently discarded on some screens. Fixed.
3. `GetHoldTime()` cut the dig hold to a tenth of its value at low Dig Speed (it took the
   first three *characters* of the number). Fixed, and clamped to 60–4000 ms.
4. The Ko-fi button ran a variable that never existed; it now says so instead of doing
   nothing.
5. A dead branch and a duplicated pixel read removed.

A previous edit had also broken 1920x1080 by applying the 768-pixel offsets on top of a
table that already handled 1080. **That has been reverted**, so 1920x1080 behaves as the
original author intended.

## Verification status, stated plainly

The code **compiles** under AutoHotkey v1.1. That is all that is proven. It has not been
run. Anything reported here is expected to include real bugs — that is what testing is for.

## Reporting upstream

This is a modified copy and is not affiliated with the Orenate authors. Genuine Orenate
bugs should go to their Discord; the changes listed above are mine.
