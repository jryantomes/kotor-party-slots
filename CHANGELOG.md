# Changelog

## 1.0.1 (2026-10-10)

* No change to `ps_slots.nss`.
* README: the full in-game test results, and a note that a HoloPatcher mod needs an `info.rtf` beside `changes.ini` (without one, the command line install waits forever).
* `holopatcher/changes_globalcat.ini`: the same note.

## 1.0.0 (2026-10-10)

First release.

* `ps_slots.nss`: `PS_AddByTemplate`, `PS_AddByObject`, `PS_SlotOf`, `PS_Claim`, `PS_Release`, `PS_FindFree`, `PS_IsClaimed`, `PS_SetClaimed`.
* Shared globals `PS_SLOTS_A` to `PS_SLOTS_D` (slots 9 to 31, seven to a global).
* HoloPatcher lines for `globalcat.2da`, and an example recruit script.
