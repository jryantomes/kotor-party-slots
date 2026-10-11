# Party Slots for KotOR I

One shared way for companion mods to take a party slot, so they never collide.

Kotor Patch Manager's **Party Roster Limit 32** gives KotOR I party slots 9 to 31 as well as the nine the game ships with. If every companion mod picks its own fixed number, two of them will eventually pick the same one and overwrite each other's companion. And without the roster patch there are no slots past 8 at all.

With Party Slots, a mod asks for a slot instead of choosing one. It gets the first slot from 9 up that no companion holds and no other mod has reserved, and keeps that slot for the rest of the game. On a game without the roster patch it gets a clear "no", so it can fall back to whatever it did before.

## For players

Party Slots is not something you install yourself. It's built into the companion mods that use it. To give those companions their own slots you need:

1. **[Kotor Patch Manager](https://github.com/LaneDibello/Kotor-Patch-Manager) 0.7.3 or later** with **Party Roster Limit 32 (K1)** applied.
2. **Party Clears Reach All Slots**, a small Kotor Patch Manager patch, also recommended. Without it, a companion in slots 9 to 31 stays in your party in places where the game expects you to be alone: boarding the Ebon Hawk, the Leviathan capture, the Taris dueling ring, the Manaan arrest, the Star Forge. Source: [kmdlswap, tools/kpm_patches/PartyClearsAllSlotsK1](https://github.com/jryantomes/kmdlswap/tree/claude/kmdlswap-droid-editing-pt3u7r/tools/kpm_patches/PartyClearsAllSlotsK1). Reported to Kotor Patch Manager as [issue #330](https://github.com/LaneDibello/Kotor-Patch-Manager/issues/330).

Without the roster patch, mods that use Party Slots fall back to their own behaviour and nothing breaks.

## For modders

### What's here

| File | What it is |
|---|---|
| `ps_slots.nss` | The include. Copy it next to your scripts and `#include "ps_slots"`. |
| `examples/ps_example.nss` | A recruit script that uses it, with a fallback. |
| `holopatcher/changes_globalcat.ini` | The HoloPatcher (TSLPatcher) lines for the globals it needs. |

### Three steps

1. **Pick a global name for your companion's slot,** for example `MYMOD_PARTY_SLOT`, and add it to `globalcat.2da` as a Number, together with the four shared globals `PS_SLOTS_A` to `PS_SLOTS_D`. `holopatcher/changes_globalcat.ini` has the lines. The shared ones use `ExclusiveColumn`, so they're added only if missing and any number of mods can ship them (tested: two mods installed one after the other give one row each). As with any HoloPatcher mod, keep an `info.rtf` beside your `changes.ini`.
2. **Add your companion through Party Slots:**

```c
#include "ps_slots"

void main()
{
    int nSlot = PS_AddByTemplate("MYMOD_PARTY_SLOT", "my_companion");
    if (nSlot == -1)
    {
        // no roster patch, or every slot is taken: your mod's own fallback
        return;
    }
    SetNPCSelectability(nSlot, TRUE);
}
```

3. **Use the slot wherever the game wants an NPC number** (`AddPartyMember`, `IsNPCPartyMember`, `SetNPCSelectability`, `RemoveAvailableNPC` and so on). Later scripts get it back with `PS_SlotOf("MYMOD_PARTY_SLOT")`.

### The functions

| Function | What it does |
|---|---|
| `PS_AddByTemplate(sGlobal, sTemplate)` | Puts the companion in your slot from a template, claiming the first free slot if you have none yet. Returns the slot, or -1 with nothing changed. |
| `PS_AddByObject(sGlobal, oCreature)` | The same with an existing creature, keeping its levels, gear and choices. |
| `PS_SlotOf(sGlobal)` | Your slot, or -1. |
| `PS_Claim(sGlobal)` | Reserves a slot without adding anyone. A reservation only: it doesn't check for the roster patch. |
| `PS_Release(sGlobal)` | Gives your slot back, for a companion who has left the game for good. |
| `PS_FindFree()` | The first free slot, or -1. |
| `PS_IsClaimed(nSlot)` | Whether any mod has reserved a slot. |

### Converting a mod that uses a fixed slot

Most companion mods today either swap their companion into a vanilla slot or use a fixed number. The change is small. A mod that shares T3-M4's slot 7, for example:

```c
// before
AddAvailableNPCByTemplate(7, "p_mycompanion");

// after: their own slot when the roster patch is there, slot 7 as before when it isn't
int nSlot = PS_AddByTemplate("MYMOD_PARTY_SLOT", "p_mycompanion");
if (nSlot == -1)
{
    nSlot = 7;
    AddAvailableNPCByTemplate(nSlot, "p_mycompanion");
}
```

Then every later reference to slot 7 asks `PS_SlotOf("MYMOD_PARTY_SLOT")` first and uses 7 only if it returns -1. Games already under way keep working: until the recruit script runs again, the companion stays where it was.

If your companion already lives in a fixed slot past 8 on players' games, record it the first time your scripts run, so other mods see it taken:

```c
if (PS_SlotOf("MYMOD_PARTY_SLOT") == -1 && IsAvailableCreature(10))
{
    SetGlobalNumber("MYMOD_PARTY_SLOT", 10);
    PS_SetClaimed(10, TRUE);
}
```

### How it keeps mods apart

Each mod keeps its slot in its own global. The four shared globals hold one bit per slot from 9 to 31, seven slots in each (KotOR's global numbers are signed bytes, so the top bit is left alone), marking every slot some mod has claimed. A slot stays reserved even while its companion is out of the roster for a while, so another mod can't take it in the meantime. A slot that already holds a companion is never handed out, even to a mod that doesn't use Party Slots.

### Safe without the roster patch

The game itself refuses slots past 8: they read as not available, and adding a companion to one fails. So `PS_AddByTemplate` and `PS_AddByObject` return -1 and change nothing, and your fallback runs.

### Notes

* The KotOR I script compiler (nwnnsscomp) has no `const`; the include uses plain variables for that reason.
* Keep the global names exactly as they are. Future versions of Party Slots will keep `PS_SLOTS_A` to `PS_SLOTS_D` and every function name working.

## Tested

In game, KotOR I 1.03 with Kotor Patch Manager 0.7.3 and Party Roster Limit 32, run unattended by a test harness that loads a save, sets the game state and reads the results back from the game's own saves:

* **Claiming slots:** with one companion already in slot 10, a second mod's companion added by template got slot 9, a third's added by object got 11, and a fourth's claim was given back cleanly. With every slot marked taken, a claim returned -1 and changed nothing.
* **Saving and loading:** after the save was loaded again, every claim was still there, the next free slot was 12, and the second mod got its own slot 9 back instead of a new one.
* **A companion that already had a fixed slot:** recorded and marked taken the first time its scripts ran.
* **Recruiting into the first free slot,** and **moving a companion from a shared vanilla slot to its own** (from slot 5 to slot 9).
* **The party screen** shows companions in slots 9, 10 and 11 with their portraits.
* **Without the roster patch:** nothing changed, and the companion stayed in its vanilla slot.
* **The HoloPatcher lines:** two mods installed one after the other gave one row of each shared global and one of each mod's own.

## Mods using Party Slots

Using it in your mod? Open an issue or a pull request to be added here.

## License

MIT. See [LICENSE](LICENSE). Use it, change it, and ship it with your mod.
