# Cirrus's MODs of Hadar's plugins

## Hadar-Mods: changes to SpellupRecast ([Hadar_Spellups.xml](https://github.com/aard-cirrus/Hadar-Mods/blob/master/SpellupRecast/Hadar_Spellups.xml))

This fork is based on [zzyzzyzzx/Hadar](https://github.com/zzyzzyzzx/Hadar) **v1.17**. All credit for the original plugin goes to Hadar. The modified version is **v1.18**.

### Added

#### New commands

| Command | What it does |
|---|---|
| `hsp diag` | Dumps the plugin's internal state in one go: what's blocking casts, settings, recast queue, active recoveries, sanctuary/aura state, and per-skill tracking with live timers. |
| `hsp refresh` | Sends the server `spellup` command **and** re-reads `slist` so recovery timers resync with the server. Fixes cooldowns that got stuck. |
| `hsp sanc` | Forces a sanctuary/aura restore attempt, clearing any backoff. |
| `hsp room` | Dumps GMCP `room.info`, including room flags. |
| `hsp death mana <amount>` | Resumes spell autocast at a fixed mana value instead of a percentage. `hsp death mana off` goes back to percentage mode. |

#### Auto-recast for skills the server `spellup` command skips

| Command | Skill | Requirement |
|---|---|---|
| `hsp quickstab on\|off` | Quickstab | Ninja |
| `hsp stealth on\|off` | Stealth | Ninja |
| `hsp stalk on\|off` | Stalk | Ninja |
| `hsp shadow on\|off` | Shadow form | Must be learned (no class check, so it works with psi as a secondary) |

- Quickstab, stealth and stalk have cooldowns that last longer than the buff. When the buff drops, the plugin waits and recasts once the recovery clears.
- Shadow form has no cooldown, so it's recast right away to keep it up all the time.
- Skill numbers and recovery groups are read from `slist`, not hardcoded.
- The toggle has the final say. A skill that's switched off won't be recast by any path.

#### Sanctuary + aura management
Requires `hsp autoaura on` with an aura item set.

- The aura goes on **the moment sanctuary drops**, in or out of combat.
- When you can cast again, the plugin puts your normal float item back on and recasts sanctuary. If the recast fails for any reason, the aura goes straight back on.
- Handles nocast rooms: it waits for you to change rooms instead of retrying and briefly taking your cover off.
- Uses the sanctuary fall-off message as a backup to GMCP.
- Does nothing if sanctuary isn't learned.

#### Class checks
- Totem (Shaman), elemental focus/ward (Elementalist), Gaia's focus (Ranger), prophecy and eye (Oracle), and wraith form (Sorcerer/Elementalist/Enchanter) can't be enabled on the wrong class.
- On login, the plugin turns off any class-specific setting left over from another class.

### Modified

| Area | Before (1.17) | Now (1.18) |
|---|---|---|
| Recasting after death | Plugin could be left thinking you were asleep, so nothing recast while you stood idle. | A short hold that ends as soon as GMCP reports your real state. |
| "Can't cast while resting" spam | Every queued spell fired into a rejection. | The first rejection pauses casting until your state catches up. |
| Mud-side batch spellup | Cleared the whole queue on the assumption the server cast everything, so skills like quickstab were silently lost. | Skills are sent one by one first, and the queue is only cleared as each spell actually lands. |
| Nocast room / resting / out of moves | The failed spell was dropped and never recast. | The spell stays queued and is recast when possible. |
| "Already affected" rejections | Ignored, so the plugin kept retrying buffs you already had (common after relogging). | Treated as confirmation that the buff is up. |
| Stuck affect/recovery tracking | A missed `{affoff}`/`{recoff}` could block a recast forever. | Entries expire on their own timers. |
| Spells disabled after mana failures | The re-enable path couldn't be reached, so spells stayed off permanently, even across sessions. | Re-enables once mana is back above your threshold. |
| `hsp disable check` | Only reported the main toggle. | Also reports when spell casting is suppressed, and why. |
| Debug output | `We are attempting to cast_spells` printed on every combat round. | `cast_spells called, N queued`. |

### Removed

No user-facing features were removed. The code taken out was 1.17 logic that the fixes above replaced:

- The aura swap-back that took the aura off without checking that sanctuary was back.
- The hardcoded `currentState = 11` on death.
- The queue wipe after a mud-side spellup.
- The unreachable `failed >= 5` / `died2` re-enable conditions.
