# Mod API (version 1)

Header: `sdk/include/yeeps_mod_api.h` (C) and `sdk/include/ymsdk.hpp` (C++ helpers).

## Entry points your mod exports

```cpp
#include "ymsdk.hpp"

YEEPS_MOD("My Mod")                                  // name shown in the game log

YM_EXPORT int YeepsMod_Init(const YeepsModApi *api) {
    if (!ym::init(api)) return -1;                   // sets up BNM through the game
    ym::onUpdate(OnUpdate);                          // every frame, main thread
    ym::onRoomChange(OnRoomChange);                  // optional
    return 0;                                        // 0 = loaded
}
```

`YeepsMod_Init` runs once at game start, **before any scene exists**. Only set things
up there; touch Unity objects only from `onUpdate`.

## ym:: helpers

| Function | What it does |
|---|---|
| `ym::init(api)` | Sets up BNM. Call first. |
| `ym::onUpdate(fn)` | `fn(float dt)` every frame on the Unity main thread. Errors are caught; after 30 errors that callback stops. |
| `ym::onRoomChange(fn)` | `fn()` when the local player changes (room/scene load). |
| `ym::log(fmt, ...)` | Game log (all mods share 20 lines a second). |
| `ym::hasRole("role")` | Whether the server gave this account a role. Use it to keep features for trusted roles. |
| `ym::api()` | The raw `YeepsModApi*`. |

## YeepsModApi fields

| Field | Notes |
|---|---|
| `version`, `size` | API version of the game. Check `version >= 1`. |
| `resolve(name)` | Real il2cpp function name -> pointer. Works on every Yeeps version (the game handles scrambled names). BNM uses it for you. |
| `il2cppHandle` | `libil2cpp.so` handle. |
| `hook(target, replacement, &original)` | Hooks through the game's one Dobby. `-1` if already hooked by the game or another mod. BNM's `BasicHook` uses it for you. |
| `unhook(target)` | Remove your hook. |
| `onUpdate`, `onRoomChange` | Use the `ym::` versions (they catch errors). |
| `log`, `hasRole` | Use the `ym::` versions. |
| `gameVersion`, `buildId`, `modsDir` | Strings, e.g. `"2.32.0"`, `"live84"`. |
| `localPlayer()` | The local `MasterPlayer` object, or null before the first frame. |

## Useful game pieces (MasterPlayer)

Fields: `leftHandTransform`, `rightHandTransform`, `headTransform`, `headCollider`,
`bodyCollider`, `bodyTransform`, `mainCamera`. The player's `Rigidbody` is on its
GameObject. Controller buttons: `UnityEngine.XR.InputDevices.GetDeviceAtXRNode`
(4 = left hand, 5 = right hand) + `TryGetFeatureValue_bool(deviceId, "PrimaryButton" /
"SecondaryButton" / "GripButton" / "TriggerButton", out value)`. See the starter menu.
