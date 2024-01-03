# Anti-cheat

Before version 0.11.7.4xxx, EFT used their own in-house anti-cheat named
Saber, better known as `SClient.dll`. It was only used during online raids and
to detect cheat program signatures. It didn't have to be patched out for
running EFT offline.

Starting version 0.11.7.4xxx, EFT migrated to using BattlEye. It can detect
dll injection from BepInEx and other sources, but not from NLog targets. It can
also detect deobfuscation of `Assembly-CSharp.dll`. It is required to be
disabled for running EFT offline.

In all versions of the game, the sole function with the name `RunValidation` is
responsible for all anti-cheat checks that happen in offline raids and in the
main menu.
