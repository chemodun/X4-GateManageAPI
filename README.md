# X4-GateManageAPI

Provides an in‑game API for managing Jump Gates and Accelerators in X4: Foundations. The API supports creating, connecting or disconnecting, and destroying gates and accelerators.

## Features

- Create Jump Gates and Accelerators at specified positions and orientations, with optional ownership assignment.
- Connect or disconnect existing Jump Gates and Accelerators.
- Destroy existing Jump Gates and Accelerators.
- Provides lists of available gate and accelerator macros id's.
- Asynchronous command processing with callback support.

## Installation

You can download the latest version via Steam client - [Gate Manager](https://steamcommunity.com/sharedfiles/filedetails/?id=3588643062)
Or you can do it via the Nexus Mods - [Gate Manager](https://www.nexusmods.com/x4foundations/mods/1856)

## Usage

The API is designed to be used by other scripts or mods. It provides a set of functions that can be called to perform various operations related to Jump Gates and Accelerators.

### Checking Availability

Before using the API, you can check if it is available in the current context:

Simple create a `cue` with the reaction on `<event_cue_signalled cue="md.Gate_Manage_API.Reloaded" />`:

#### Example

```xml
<cue name="YouOnReloaded" instantiate="true">
    <conditions>
        <event_cue_signalled cue="md.Gate_Manage_API.Reloaded" />
    </conditions>
    <actions>
    <debug_text text="'Gate_Manage_API: Reloaded'" chance="100" filter="general" />
    ...
    </actions>
</cue>
```

### Main principles

In common you need to do only one thing - call the `md.Gate_Manage_API.Request` cue with a table of arguments. If you need to get the result of the command, you should provide a callback function in the arguments table. The callback function will be called with a result table when the command is completed.

#### Basic Example

```xml
<cue name="..." >
    <actions>
        ...
        <set_value name="$args"
            exact="table[
                $command  = 'destroy_gate',
                $gate     = @$gate,
                $callback = Your_Capture_Results,
                ]" />
        <signal_cue_instantly cue="md.Gate_Manage_API.Request" param="$args" />
        ...
    </actions>
</cue>

<cue name="Your_Capture_Results" instantiate="true">
    <conditions>
        <event_cue_signalled/>
    </conditions>
    <actions>
        ...

        <set_value name="$result" exact="@event.param" />
        <debug_text text="'Command %s completed with result %s (%s)'.[@$result.$command, @$result.$result, @$result.$info]" chance="100" filter="general" />
        ...
    </actions>
</cue>
```

### Standard Result Fields

All commands return a result table with the content of the input arguments and some additional standard fields:

- `$result`: string, either 'success' or 'error'.
- `$info`: string, additional information about the result, especially in case of an error.
- `$detail`: string, more detailed information about the result, especially in case of an error. Optional, may be not present.

## Commands

The following commands with appropriate arguments are supported:

### build_gate

Creates a new Jump Gate or Accelerator at a specified position and orientation within a sector. It can optionally assign an owner to the created gate or accelerator.

- `$command`: string, must be 'build_gate'.
- `$sector`: sector object (required).
- `$macroId`: string (required), the macro ID of the gate or accelerator to create.
- `$ownerId`: string (optional, default "'ownerless'"), string "'null'" is possible to indicate no owner.
- `$offset`: vector (required), the position offset within the sector.
- `$rotation`: quaternion (required if getRotationFromMap is false), the orientation of the gate or accelerator.
- `$getRotationFromMap`: boolean (optional, default false), if true, the rotation will be determined based on the sector's map data.
- `$callback`: function, the callback function to invoke with the result.

Returns the created gate or accelerator object in the result table on success. In addition to the input and standard result fields, the result table will contain:

- `$gate`: gate object, the created gate or accelerator (only on success).

### connect_gates

Connects two Jump Gates or Accelerators, allowing for instant travel between them.

- `$command`: string, must be 'connect_gates'.
- `$gateSource`: gate object (required), the first (source) gate to connect.
- `$gateTarget`: gate object (required), the second (target) gate to connect.
- `$callback`: function, the callback function to invoke with the result.

### disconnect_gates

Disconnects two connected Jump Gates or Accelerators.

- `$command`: string, must be 'disconnect_gates'.
- `$gateSource`: gate object (required), the first (source) gate to disconnect.
- `$gateTarget`: gate object (required), the second (target) gate to disconnect.
- `$callback`: function, the callback function to invoke with the result.

### destroy_gate

Destroys an existing Jump Gate or Accelerator.

- `$command`: string, must be 'destroy_gate'.
- `$gate`: gate object (required), the gate or accelerator to destroy.
- `$callback`: function, the callback function to invoke with the result.

In addition to the input and standard result fields, the result table will contain:

- `$name`: gate object name, the name of the destroyed gate or accelerator.
- `$sector`: sector object, the sector where the destroyed gate or accelerator was located.

### mark_gate

Marks a Jump Gate or Accelerator on a map for easier identification.

- `$command`: string, must be 'mark_gate'.
- `$gate`: gate object (required), the gate or accelerator to mark.
- `$callback`: function, the callback function to invoke with the result.

### unmark_gate

Removes a previously marked selection of an Jump Gate or Accelerator on a map.

- `$command`: string, must be 'unmark_gate'.
- `$gate`: gate object (required), the gate or accelerator to unmark.
- `$callback`: function, the callback function to invoke with the result.

### get_macro_tables

Retrieves the available macros for Jump Gates and Accelerators as a tables.

- `$command`: string, must be 'get_macro_tables'.
- `$callback`: function, the callback function to invoke with the result.

In addition to the input and standard result fields, the result table will contain:

- `$gatesTable`: list of tables for the available Jump Gates macros.
- `$acceleratorsTable`: list of tables for the available Accelerators macros.

Each table in the lists will contain:

- `$name`: string, the name of the macro.
- `$macroId`: string, the ID of the macro.
- `$icon`: string, the icon associated with the macro.
- `$isAccelerator`: boolean, true if the macro is for an Accelerator

Unfortunately, due to a limitation in the game's lua engine, at least for now, the only way to identify if a macro is for an Accelerator or a Jump Gate is by checking the icon field. If the icon is `mapob_transorbital_accelerator`, then it is an Accelerator, if it is `mapob_jumpgate`, then it is a Jump Gate.
So, if some mod adds new gates or accelerators with different icons, they will not be correctly identified by this API.

*If someone knows a better way to identify the type of macro, please let me know.*

## Reference

Currently, there is only one mod that uses this API - `Gate Manager`, from version 1.16 and above:

- [On Steam](https://steamcommunity.com/sharedfiles/filedetails/?id=3577926900)
- [On Nexus Mods](https://www.nexusmods.com/x4foundations/mods/1842)

## Credits

- Author: Chem O`Dun, on [Nexus Mods](https://next.nexusmods.com/profile/ChemODun/mods?gameId=2659) and [Steam Workshop](https://steamcommunity.com/id/chemodun/myworkshopfiles/?appid=392160)
- *"X4: Foundations"* is a trademark of [Egosoft](https://www.egosoft.com).

## Acknowledgements

- [EGOSOFT](https://www.egosoft.com) - for the game itself (In fact - for the series of games)!
- Enormous big thank to [Forleyor](https://github.com/Forleyor), for his help with Lua! For make me curious about it and for his patience! Without him I will never touched the Lua and started to make this mod!
- Thanks to `cheapman44` for the discussion on Discord about the gate management which pushed me to make this API.
- Thanks to all members of the [X4 modding channel](https://discord.com/channels/337098290917146624/502057640877228042) on [Egosoft Discord](https://discord.com/invite/zhs8sRpd3m).

## Changelog

### [1.00] - 2024-10-17

- Added
  - Initial public version