# Vivo Performance, Battery & Debloat Guide

A community-tested collection of performance tweaks, debloating steps, and battery optimizations for **Vivo Devices**. -specific results may still vary.

> Primarily tested on **Vivo T3 5G**.

## Table of Contents
- [Disclaimer](#disclaimer)
- [Prerequisites](#prerequisites)
- [ADB Setup](#adb-setup)
- [Debloating](#debloating)
- [GameWatch Fix — Unlock 120Hz for Restricted Apps](#gamewatch-fix--unlock-120hz-for-restricted-apps)
- [Performance Settings](#performance-settings)
- [Network & Background Data Control](#network--background-data-control)
- [UI Smoothness](#ui-smoothness)
- [App Timer 0 Trick (Soft-Freeze System Apps)](#app-timer-0-trick-soft-freeze-system-apps)
- [Background Power Restriction](#background-power-restriction)
- [Battery Health](#battery-health)
- [Gaming Optimizations](#gaming-optimizations)
- [Credits](#credits)
- [License](#license)

---

## Disclaimer

This is not an official guide. The author is **not responsible** for bricked devices, boot loops, lost data, or any other issue arising from following these steps. Test one change at a time, keep a backup, and understand a command before you run it. Prior familiarity with ADB is assumed.

## Prerequisites

**Desktop tools**
- [Platform Tools (ADB/Fastboot)](https://developer.android.com/tools/releases/platform-tools) — official Google binaries
- [ADB App Control](https://github.com/mightysyntax/ADB-App-Control) — GUI for enabling/disabling packages
- [UAD-NG](https://github.com/Universal-Debloater-Alliance/universal-android-debloater-next-generation) — cross-platform debloat tool

**On-device tools (no PC required)**
- [Shizuku](https://shizuku.rikka.app/) — grants elevated permissions to apps without root
- [Canta](https://github.com/w568w/Canta) — Shizuku-powered uninstaller
- Any Shizuku-based app/background manager for freezing or force-stopping apps

## ADB Setup

**Windows (WinGet)**
```
winget install --id Google.PlatformTools --source winget
```

**Linux**
```
# Ubuntu/Debian
sudo apt install android-tools-adb

# Fedora
sudo dnf install android-tools

# Arch
sudo pacman -S android-tools
```

**macOS (Homebrew)**
```
brew install android-platform-tools
```

Enable **Developer Options → USB Debugging** on the phone, connect via USB, and accept the RSA prompt.

---

## Debloating

Removing unused OEM system apps reduces background wake-ups, notification spam, and storage bloat.

See [`debloat-list.txt`](./debloat-list.txt) for a categorized starter list of commonly-reported-safe Vivo/BBK packages.

**Before removing anything:**
1. Pull your own installed package list so you know what actually exists on your device/region variant:
   ```
   adb shell pm list packages -f > packages.txt
   ```
2. Cross-check package names against `debloat-list.txt` — regional firmware builds vary, so not every package will exist on every unit.
3. Disable, don't uninstall, on your first pass:
   ```
   adb shell pm disable-user --user 0 <package.name>
   ```
4. If everything is stable after a few days, move to a full removal:
   ```
   adb shell pm uninstall --user 0 <package.name>
   ```
5. To restore a package if something breaks:
   ```
   adb shell cmd package install-existing <package.name>
   ```

**Method A — Desktop tools:** Import the package list into ADB App Control or UAD-NG, review the selection, then disable/uninstall.

**Method B — On-device (no PC):** Set up Shizuku, grant access to Canta, select packages carefully, and uninstall.

**Avoid touching:** anything under `com.android.*` core system packages, your input method (keyboard), SIM/telephony services, and SystemUI — these are not bloat and removing them can soft-brick the UI.

---

## Fix 120Hz Not Working — Unlock 120Hz for Restricted Apps

This is a fix I found through my own testing on the T3 5G — Vivo subsequently restricted the ability to remove this app in later patches.

**The problem:** Some games/apps don't appear in the system's "Apps running at higher refresh rate" list at all (e.g. **MLBB Global**), so they're silently capped at 60Hz. Others show a 120Hz toggle that appears "on" but the game is still visibly capped at 60fps in practice (e.g. **BGMI**).

**The cause:** `com.vivo.gamewatch`, a system service that manages per-game refresh-rate profiles, was silently excluding or mismanaging certain packages.

**The fix:**
```
adb shell pm uninstall --user 0 com.vivo.gamewatch
```
or disable it via the [App Timer 0 trick](#app-timer-0-trick-soft-freeze-system-apps) if uninstall is blocked on your firmware version.

**Confirmed to work on:** FuntouchOS 14, and early FuntouchOS 15 patches on some units. Vivo has since patched later builds to prevent removing/disabling this package, so results depend on your current patch level(if u can still manage to remove this in latest fos 15 or Origin6 patches feel free to contribute).

**Known side effect:** once `com.vivo.gamewatch` is gone, the entire "Apps running at higher refresh rate" settings page will appear empty — this is expected, not a bug. You lose the per-app automatic profile list, but you can still manually switch the display between 60Hz/120Hz any time from the refresh rate settings.

---

## Performance Settings

| Setting | Recommendation | Why |
|---|---|---|
| AI Acceleration Engine | Off | In many cases, disabling it reduces AI-driven task scheduling overhead — measurably lower heat and better sustained performance for some users. Test on your unit. |
| ART++ Turbo | Test both | Some users see better app-launch performance with it on, others get more consistent frame pacing with it off. Many Users Reported That Turning Off Ai acceleration engine & ART++ Turbo Has Decreased Heating and Increased Battery backup as well No difference in Performance at all. |
| Extended RAM (Virtual RAM) | Off | Extends storage flash wear and adds swap latency for marginal RAM gains — turning it off tends to help both storage longevity and real-world responsiveness. |
| Haptic Feedback | Off (optional) | Removes a small but constant CPU/vibration-motor tax on every touch interaction; matters most on longer gaming sessions. |

---

## Network & Background Data Control

Under **Settings → Apps → Data usage** (or per-app battery/data settings), disable **background data** for apps that don't need to phone home when closed — e.g. Compass, V-Appstore, Play Store, and similar apps with no real background use case.

Why this helps: apps like the Play Store or V-Appstore, and Other system or Installed apps Uses Data In Background. Cutting their background data (with **Data Saver mode/Data Saving Mode: On**) removes periodic background pings, which in turn keeps your regular network ping more stable since fewer processes are competing for radio/data access in the background.

## UI Smoothness

**Developer Options → Allow window-level blurs** — this is **on by default**. Turning it **off** has shown a snappier, less laggy feel during app switching and animations on some units, at the cost of blur visual effects. Test it yourself — re-enabling developer options after a reset will turn this back on by default, so you'll need to reapply it if you ever reset dev options.

## App Timer 0 Trick (Soft-Freeze System Apps)

For OEM system apps that refuse to be disabled/uninstalled on newer patches, Digital Wellbeing can be repurposed as a soft-freeze:

1. Open **Digital Wellbeing**. or **Apps -> app info -> Screen Time -> App Timer**
2. Find the system app you want restricted, set its **App Timer to 0 minutes**.
3. The icon turns grayscale and the app is blocked from launching.
4. It may still run in the background — pair this with [background power restriction](#background-power-restriction) below.
5. Before doing this, clear the app's data and revoke its permissions from **App Info** for a cleaner freeze.

## Background Power Restriction

For apps you can't fully remove but rarely use: **Settings → Battery → Background power consumption management** (or per-app **Restrict battery usage**) — restrict background activity for non-essential system and user apps.
OR for Installed apps just **Hold them Go To app Info -> Battery Usage -> Background Power Control**

⚠️ Skip this for messaging apps or anything you rely on for real-time notifications — restricting background activity can delay or block push notifications entirely.

## Battery Health

**Settings → Battery → Optimized battery charging → Charging upper limit: 90%.** Keeping daily charge below 100% is one of the most effective ways to slow long-term battery capacity loss, at the cost of a small amount of daily capacity.

## Gaming Optimizations

if u are a Gamer and just want Priorities gaming then:
Disable All the Fancy settings & accessibilities and other stuff like:

- Gestures (use standard navigation instead)
- Smart Window
- Smart Sidebar
- Split-screen shortcuts

**Ultra Game Mode:** counter-intuitively, some users (including reports on Reddit/community forums) have found **better in-game performance with a game removed from Ultra Game Mode and the feature turned off entirely**, rather than left on. This isn't universal — test both states yourself. If you turn Ultra Game Mode off, you can still use DND-mode for call/notification silencing manually, instead of relying on Game Mode for it.

---

## Credits

Maintained by [deepanshu-meena](https://github.com/deepanshu-meena). Structure loosely inspired by the [Better Vivo](https://github.com/pikawee/better-vivo) guide for the Vivo X200 FE — this repo is an independent project focused on the Vivo T3 5G and broader OriginOS/FuntouchOS devices, built from my own testing.

## License

Released under the [MIT License](./LICENSE) — use, share, and modify freely with attribution.
