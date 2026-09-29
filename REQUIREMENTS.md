# REQUIREMENTS — read before you run this

These are not preferences. Get them wrong and the macro appears broken: the character
walks into walls, panning misses the water, or the script does not start at all. Two
of the four problems below are environmental, and no code change can fix them.

---

## 1. Character walks into walls, or despawns

**Cause:** Windows display scaling. Scaling shifts every screen coordinate, so
detection that samples fixed positions reads the wrong pixels.

**Fix:**
- Windows desktop → right-click → Display settings → set **Scale to 100%**.
  Not 125%, not 150%.
- Set the **game resolution to 1920x1080**.

**Caveat worth knowing (code-level, not fixed here).** The macro's pixel probes are
hand-tuned for heights `768 / 953 / 1050` and widths `1366 / 1680 / 1856`. There is
**no 1920x1080 case** — which is the very resolution the guidance above recommends. On
1920x1080 every probe therefore runs on its base ratio. That is a plausible cause of
"walks into walls" that scaling alone does not explain, and it is left documented
rather than guessed at. See MODIFICATIONS.md, "Known issues NOT fixed".

## 2. Macro misses the water, or fails to pan

**Cause:** high-frequency visual disturbance. Screen shake stops the macro from
finding its targets, and it defeats any "wait until the view settles" test, so steps
run to their time ceiling instead of stopping when they should.

**Fix (in-game Roblox settings):**
- **Screen Shake: OFF**
- **Camera mode: Classic**
- **Shift Lock: DISABLED**

> Correction to earlier advice: an earlier version of this project suggested turning
> Shift Lock **ON** for camera stability. For this game that is wrong. Shift Lock must
> be **OFF** with the camera on Classic.

## 3. The script fails to boot — error 0x1023

**Cause:** you are running the **Microsoft Store** build of Roblox.

**Fix:** install the official desktop launcher from **roblox.com/download**. The macro
only works with that build.

## 4. AHK crash, or keys feel laggy

**Cause:** the script running under **AutoHotkey v2**, or too many overlay programs
competing for input.

**Fix:**
- Use **AutoHotkey v1.1** (the legacy/deprecated branch) — this macro is v1.1 code.
- Close aggressive overlays such as Discord or GeForce Experience if input feels
  delayed.

**Already handled in this build.** On Windows the `.ahk` association is usually the
AutoHotkey **UX launcher**, which picks the interpreter from a script's `#Requires`
header and **defaults to v2 when there is none** — so this macro could fail to open
even with v1.1 installed. This copy has `#Requires AutoHotkey v1.1` on all 22 launched
scripts, and `RUN ORNATE.bat` calls the v1.1 engine by absolute path as a fallback.
If it still will not start, your AutoHotkey v1.1 install itself is the problem.

---

## Checklist

- [ ] Windows display scaling at **100%**
- [ ] Roblox game resolution **1920x1080**
- [ ] Roblox: **Screen Shake OFF**
- [ ] Roblox: camera **Classic**
- [ ] Roblox: **Shift Lock DISABLED**
- [ ] Roblox installed from **roblox.com/download**, not the Microsoft Store
- [ ] **AutoHotkey v1.1** installed
- [ ] Launch with **RUN ORNATE.bat** (it sits beside the `OrnateMacroV1.4.8` folder)

Starts on **F6**, stops on **F8**.
