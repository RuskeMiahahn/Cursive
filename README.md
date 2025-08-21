# Cursive

> [!IMPORTANT]
>
> **This addon requires you to have [SuperWoW](https://github.com/balakethelock/SuperWoW) installed.**
>
> It won't work without it. Really.

![image](https://github.com/pepopo978/Cursive/assets/149287158/801511af-29c7-4baf-b1ac-5e8c52f0f846)

Cursive combines ShaguScan unit scanning with curse tracking similar to DotTimer and the ability to automatically curse
targets similar to Decursive.

### Recommended setup

Move the Cursive label to the desired position on the screen. You can turn on 'Show Frame Background' in the settings to
help with this. Once you have it where you want it, turn off 'Show Frame Background' and turn on 'Allow clickthrough'
so that it doesn't block your clicks when it's not displaying mobs.

## Commands

`/cursive` for commands, minimap icon to edit options.

### Curse

`/cursive curse <spellName:str>|<guid?:str>|<options?:comma separated str>`: Casts spell if not already on target/guid

EXAMPLE: `/cursive curse Corruption|target` will attempt to cast Corruption on your target if it's not already on them
and they aren't cc'ed.

### Multicurse

`/cursive multicurse <spellName:str>|<priority?:str>|<options?:comma separated str>`: Picks target based on priority and
casts spell if not already on target and they aren't cc'ed.

EXAMPLE: `/cursive multicurse Corruption|HIGHEST_HP` will attempt to cast Corruption picking the target with the highest
HP that doesn't already have it and will warn you if it does nothing.

### Target

`/cursive target <spellName:str>|<priority?:str>|<options?:comma separated str>`: Targets unit based on priority if spell in range and not already on target

EXAMPLE: `/cursive target Icicles|HIGHEST_HP` will target the enemy with the highest HP in range of the spell Icicles.

can also do it only if you don't have a target already:
`/run if not UnitName("target") then SlashCmdList.CURSIVE("target Icicles|HIGHEST_HP") end`

There is also 
`/script targetGuid = Cursive:GetTarget("str", "str", {})`: Returns guid of the unit that would be targeted based on priority and options.

EXAMPLE: `/script targetGuid = Cursive:GetTarget("Corruption", "HIGHEST_HP", {})` will return guid of the enemy with the highest HP in range of the spell Corruption that doesn't already have Corruption.

## Priority Options

- HIGHEST_HP - Target highest HP enemy without a curse first.
- LOWEST_HP - Target lowest HP enemy without a curse first.
- RAID_MARK - Target largest raid mark enemy without a curse first. Priority is as follows:
    - Skull = 8 (1st priority)
    - Cross = 7
    - Square = 6
    - Moon = 5
    - Triangle = 4
    - Circle = 3
    - Star = 2
    - Diamond = 1 (8th priority)
    - No mark = 0
- RAID_MARK_SQUARE - Target largest raid mark but ignore Skull and Cross. Priority is as follows:
    - Square = 6 (1st priority)
    - Moon = 5
    - Triangle = 4
    - Circle = 3
    - Star = 2
    - Diamond = 1
    - No mark = 0
    - Cross = -1
    - Skull = -2 (9th priority)
- INVERSE_RAID_MARK - Target lowest raid mark enemy without a curse first. (reverse of RAID_MARK)

- HIGHEST_HP_RAID_MARK - Target highest HP enemy with a raid (use raid mark priorities for identical hp), then highest
  HP enemy without a mark.
- HIGHEST_HP_RAID_MARK_SQUARE - Same as HIGHEST_HP_RAID_MARK but with RAID_MARK_SQUARE priority for marked enemies.
- HIGHEST_HP_INVERSE_RAID_MARK - Same as HIGHEST_HP_RAID_MARK but with INVERSE_RAID_MARK priority for marked enemies.

## Command Options

All commands can take the following options separated by commas:

- `warnings` : "Display text warnings when a curse fails to cast.",
- `resistsound` : "Play a sound when a curse is resisted.",
- `expiringsound` : "Play a sound when a curse is about to expire.",
- `allowooc` : "Allow out of combat targets to be multicursed. Would only consider using this solo to avoid potentially
  griefing raids/dungeons by pulling unintended mobs.",
- `priotarget` : "Always prioritize current target when choosing target for multicurse. Does not affect 'curse'
  command.",
- `ignoretarget` : "Ignore the current target when choosing target for multicurse. Does not affect 'curse' command.",
- `playeronly` : "Only choose players and ignore npcs when choosing target for multicurse. Does not affect 'curse'
  command.",
- `minhp=<number>` : "Minimum HP for a target to be considered.",
- `refreshtime=<number>` : "Time threshold at which to allow refreshing a curse. Default is 0 seconds.",
- `name=<str>` : "Filter targets by name. Can be a partial match. If no match is found, the command will do nothing.",
- `ignorespellid=<number>` : "Ignore targets with the specified spell id already on them. Useful for ignoring targets
  that already have a shared debuff.",
- `ignorespelltexture=<number>` : "Ignore targets with the specified spell texture already on them. Useful for ignoring
  targets that already have a shared debuff.",

EXAMPLE: `/cursive multicurse Corruption|HIGHEST_HP|warnings,resistsound,expiringsound,minhp=10000,refreshtime=2`

EXAMPLE:
`/cursive multicurse Curse of Recklessness|RAID_MARK|name=Touched Warrior,ignorespelltexture=Spell_Shadow_UnholyStrength,resistsound,expiringsound`

## Macro examples

You can just put multiple commands in a macro like this
```
/cursive curse Curse of Recklessness|target|refreshtime=1
/cursive curse Corruption|target|refreshtime=3
/cursive curse Siphon Life|target|refreshtime=1
```
but the game won't always execute them in the order you want

if you want more control of the order you can do something like
```
/script if not Cursive:Curse("Curse of Recklessness", "target", {refreshtime=1}) then if not Cursive:Curse("Corruption", "target", {refreshtime=3}) then Cursive:Curse("Siphon Life", "target", {refreshtime=1}) end end
```
This also works for Multicurse:
```
/script if not Cursive:Multicurse("Curse of Recklessness", "HIGHEST_HP", {refreshtime=1}) then if not Cursive:Multicurse("Corruption", "HIGHEST_HP", {refreshtime=3}) then Cursive:Multicurse("Siphon Life", "HIGHEST_HP", {refreshtime=1}) end end
```

All cursive commands will return true only if it attempted to cast or it found a target.

## Shared Debuffs

Shared debuffs applied by other players will appear greyed out on targets.

Currently only Faerie Fire is supported as I felt it warranted special handling.  It works by looking for other players casting faerie fire and then checking if the mob has the debuff, so if someone refreshes faerie fire and gets resisted it will incorrectly restart the timer.  However, it should still correctly remove faerie fire if it falls off that target.  I didn't want to impact performance to handle this edge case.

## Accessing curse data in other addons

Cursive data can be accessed in other addons 

You can check if curse is active with
`Cursive.curses:HasCurse(lowercaseSpellNameNoRank, targetGuid, minRemaining)`

You can get raw curse data using 
`Cursive.curses:GetCurseData(spellName, guid)`

Curse data is a table with the following structure:
```
{
		rank = int,
		duration = float,
		start = float,
		spellID = int,
		targetGuid = int,
		currentPlayer = bool,
}
```

Here's an example that gets the time left on Corruption on the current target:
```
/run _, guid = UnitExists("target"); local data = Cursive.curses:GetCurseData("Corruption", guid); print(Cursive.curses:TimeRemaining(data))
```

## Important info

If you have my latest nampower, it will use the SpellInRange function from that to provide improved range checking.

Otherwise, all commands will prioritize targets within 28 yards of you first to have a better chance of being in range.

All commands will ignore targets with the following CCs on them:

- Sleep
- Entangling Roots
- Shackle Undead
- Polymorph
- Turn Undead
- Blind
- Sap
- Gouge
- Freezing Trap
- Banish

Multicurse will only ever target enemies that are already in combat (except if you target a mob directly first) to
prevent pulling things you didn't intend like marked patrols.

Mobs with raid marks will be displayed first.

Mobs will the top 3 max hps will always display next. I may make this configurable in the future.

There is an option "always show current target" that will display your current target in the last slot if they aren't already being displayed.

You can ignore mobs based on their unit name using the ignored mob list filter.  It is comma separated and you need to press enter to get it to save.  For example can do:
`whelp,scarab` to ignore all mobs with those strings in their name.

