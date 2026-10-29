# Scriptable Input System (SIS)

```
Created by: JurgenWithWings
Version: 0.3.0
```

This is a simple input wrapper for Unity that allows you to easily manage input actions and bindings. It provides a 
clean way to create and modify input captured by Unity for use in your game.

## Dependencies

>**Input System**
>  - Name: [com.unity.inputsystem](https://link.unity.com/hub/package-manager/install/com.unity.inputsystem)
>  - Version: 1.19.0 or higher

## Setup

1. Navigate to the **Input Manager Settings** in **Packages** -> **JurgenWithWings** -> **Scriptable Input System** -> **Resources** and assign the projects **Input Action Asset**. If one is not there you can create one inside a `Resources` folder (right-click in Project window -> **Create** -> **Scriptable Input** -> **Input Manager Settings**, it must be named `InputManagerSettings`).
2. Create **Input Channel** assets for each action you want to read (defaults are provided in **Packages** -> **JurgenWithWings** -> **Scriptable Input System** -> **Input Channels** -> **Channels**), you can also create by right-clicking in the project window -> **Create** -> **Scriptable Input** -> **ScriptableObjects** -> **Input Channel** and selecting one of the desired types. 
3. Add your **Input Channels** to the `inputChannels` list in the **Input Manager Settings**.
4. Optionally attach **Input Processors** to channels to transform their values (defaults are provided in **Packages** -> **JurgenWithWings** -> **Scriptable Input System** -> **Input Processors** -> **Processors**), you can create your own by right-clicking in the project window -> **Create** -> **Scriptable Input** -> **ScriptableObjects** -> **Input Processor**.
5. Assign **Input Device Profiles** to the `deviceProfiles` list in **Input Manager Settings** (defaults are provided in **Packages** -> **JurgenWithWings** -> **Scriptable Input System** -> **Input Device Profiles**). Optionally assign a `fallbackDeviceProfile` for when no profile matches the active device.

`InputManager` initializes automatically before any scene loads — no MonoBehaviour setup is required.

## Documentation

<details><summary><h3>Input Channels</h3></summary>

Each class that inherits from `InputChannel<T>` represents a type of input that can be received from Unity's Input System and determines how it is read and parsed.

To create a new Input Channel asset (e.g. Jump, Shoot, Move), right-click in the Project window -> **Create** -> **Scriptable Input** -> **ScriptableObjects** -> **Input Channel** -> and choose the value type you want to parse (e.g. `BoolInput`, `Vector2Input`). Then assign the corresponding `InputActionReference` in the inspector.

#### Built-in Types

| Type | Value Type | Notes |
|------|------------|-------|
| `BoolInput` | `bool` | Has a `considerCanceledAsTrue` field that also returns `true` when the action is canceled |
| `FloatInput` | `float` | Standard float axis read |
| `Vector2Input` | `Vector2` | Standard 2D axis read |

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `Value` | `T` | The processed value after all processors in the chain have run |
| `RawValue` | `T` | The unprocessed value read directly from the input action |
| `Triggered` | `bool` | `true` only on the first frame the action is performed (rising edge); reset each frame automatically |
| `Context` | `InputAction.CallbackContext` | The most recent callback context from the Input System |
| `OnEventRaised` | `event Action<T>` | Fired each time the action is performed or canceled |

#### Custom Types

To create a new Input Channel class, right-click in the Project window -> **Create** -> **Scriptable Input** -> **Classes** -> **New Input Channel**. Override `ReadValue` to control how the raw value is extracted from the `CallbackContext`.

</details>

---

<details><summary><h3>Input Processors</h3></summary>

Input Processors are ScriptableObjects that sit in a channel's `Processors` list and transform values in sequence. Each processor receives the output of the previous one, allowing modifications to be chained.

To create a new Input Processor asset, right-click -> **Create** -> **Scriptable Input** -> **ScriptableObjects** -> **Input Processor** -> choose the processor type. Add it to the `Processors` list on an Input Channel asset in the inspector.

#### Built-in Processors

| Type | Value Type | Description |
|------|------------|-------------|
| `PlayerLookSensitivityProcessor` | `Vector2` | Scales the value by `mouseSensitivity` or `controllerSensitivity` based on the currently active control device |

#### Custom Types

To create a new Processor class, right-click -> **Create** -> **Scriptable Input** -> **Classes** -> **New Input Processor**. Implement `Process(T value, InputAction.CallbackContext context)` and return the modified value.

</details>

---

<details><summary><h3>Input Manager</h3></summary>

`InputManager` is a static class that initializes automatically at runtime before any scene loads. It registers all channels from `InputManagerSettings`, resets one-frame flags each frame, and tracks the active device profile.

#### Reading Input

```csharp
// Retrieve a channel by the name of its ScriptableObject asset
InputChannel<bool> jump = InputManager.GetInput<bool>("PlayerJump");
if (jump.Triggered) { /* executes only on the first frame of the jump */ }

InputChannel<Vector2> move = InputManager.GetInput<Vector2>("PlayerMove");
Vector2 movement = move.Value; // already processed (e.g. sensitivity applied)
```

#### Device Detection

```csharp
// Check the current device profile
InputDeviceProfile profile = InputManager.CurrentDeviceProfile;
Debug.Log(profile.profileName); // e.g. "Playstation", "Keyboard&Mouse"
Debug.Log(profile.glyphCode);   // e.g. "PS", "KBM"

// Subscribe to device change events
InputManager.OnDeviceProfileChanged += OnDeviceChanged;

void OnDeviceChanged(InputDeviceProfile newProfile) {
    // Update UI prompts, swap glyph icons, etc.
}
```

#### Action Map Slots

Action maps are controlled through named **slots** (e.g. `"Main"`, `"UI"`). Each slot holds any number of action maps, each with a priority:

- Only the map(s) with the **highest priority** in a slot are active. Maps tied on priority are all active.
- A whole slot can be enabled or disabled. A disabled slot activates nothing.
- A map is enabled if it is active in **any** enabled slot. Every other map in the asset is disabled.

Slots are created on first use and start enabled.

```csharp
InputManager.AddToSlot("Main", "Player");          // priority 0
InputManager.AddToSlot("UI", "UI");

// Override the player's controls completely
InputManager.AddToSlot("Main", "Debug", 10);       // Debug active, Player inactive
InputManager.RemoveFromSlot("Main", "Debug");      // Player active again

// Same priority = both active
InputManager.AddToSlot("Main", "Vehicle", 0);      // Player and Vehicle active

// Toggle whole slots, e.g. while paused
InputManager.SetSlotEnabled("Main", false);
InputManager.SetSlotEnabled("Main", true);

// Read-only info about a slot
if (InputManager.GetSlot("Main")?.Enabled == true) { /* ... */ }
```

Starting slots are configured in `InputManagerSettings.defaultSlots`. Slots created there start enabled.

#### API Reference

**Methods**

| Method | Description |
|--------|-------------|
| `GetInput<T>(string name)` | Returns the `InputChannel<T>` whose asset name matches `name`. Logs an error if the type does not match. |
| `GetInput(string name)` | Returns the untyped `InputChannelBase` whose asset name matches `name`. |
| `AddToSlot(string slot, string mapName, int priority = 0)` | Adds a map to a slot, or updates its priority if already present. Warns if the map does not exist. |
| `RemoveFromSlot(string slot, string mapName)` | Removes a map from a slot. |
| `ClearSlot(string slot)` | Removes every map from a slot. |
| `SetSlotEnabled(string slot, bool enabled)` | Enables or disables a slot. Creates the slot if it does not exist. |
| `SetAllSlotsEnabled(bool enabled)` | Enables or disables every existing slot. |
| `GetSlot(string slot)` | Returns the `InputSlot` (its maps, priorities and active maps), or `null`. |
| `IsActionMapActive(string mapName)` | Whether the named action map is currently enabled. |

**Properties & Events**

| Member | Type | Description |
|--------|------|-------------|
| `CurrentDeviceProfile` | `InputDeviceProfile` | The profile that matched the most recently used input device. |
| `OnDeviceProfileChanged` | `event Action<InputDeviceProfile>` | Raised whenever the active device profile changes. |
| `Slots` | `IEnumerable<InputSlot>` | Every slot that has been created. |
| `OnActionMapsChanged` | `event Action` | Raised whenever slot changes enable or disable any action map. |

</details>

---

<details><summary><h3>Input Manager Settings</h3></summary>

`InputManagerSettings` is a ScriptableObject that **must** reside inside a `Resources` folder and **must** be named `InputManagerSettings`. `InputManager` loads it automatically at startup.

| Field | Description |
|-------|-------------|
| `inputAsset` | The `InputActionAsset` containing all action maps and actions |
| `inputChannels` | List of all Input Channel assets to register and reset each frame |
| `defaultSlots` | Action maps loaded into slots on startup. Each entry has a `slot` name, `actionMap` name, and `priority` |
| `deviceProfiles` | List of `InputDeviceProfile` assets used for device detection |
| `fallbackDeviceProfile` | Profile used when no entry in `deviceProfiles` matches the active device |

> **Note:** Channels are keyed by their ScriptableObject asset name. Pass that same name to `InputManager.GetInput<T>(name)`.

</details>

---

<details><summary><h3>Input Device Profiles</h3></summary>

`InputDeviceProfile` is a ScriptableObject that describes a class of input device. When a new device produces input, `InputManager` checks all profiles sorted by `priority` (highest first) and activates the first one whose `matchingLayouts` covers that device's layout hierarchy.

To create a new profile, right-click in the Project window -> **Create** -> **Scriptable Input** -> **Input Device Profile**.

#### Fields

| Field | Description |
|-------|-------------|
| `profileName` | Human-readable name (e.g. `"PlayStation 5 Controller"`) |
| `glyphCode` | Short code used to fetch UI glyphs (e.g. `"PS"`, `"KBM"`, `"GP"`) |
| `controlSchemes` | Control scheme names this profile belongs to (e.g. `"Gamepad"`, `"Keyboard&Mouse"`) |
| `matchingLayouts` | Unity device layout names to match against (e.g. `"DualShockGamepad"`, `"Keyboard"`). Uses Unity's layout inheritance tree, so targeting `"DualShockGamepad"` also matches `"DualShock4GamepadHID"`. |
| `priority` | Higher values are checked first. Give specific controllers (e.g. PlayStation) higher priority than generic ones (e.g. Gamepad). |

#### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `MatchesDevice(InputDevice device)` | `bool` | Returns `true` if `device` derives from any layout in `matchingLayouts`, using Unity's full layout inheritance tree. |
| `IsInControlScheme(string scheme)` | `bool` | Returns `true` if `scheme` is present in `controlSchemes`. Use this to check whether the current profile applies to a given control scheme (e.g. `"Gamepad"`, `"Keyboard&Mouse"`). |

#### Built-in Profiles

| Asset | Glyph Code | Matching Layouts | Priority |
|-------|-----------|-----------------|----------|
| `Keyboard&Mouse` | `KBM` | `Keyboard`, `Mouse` | 0 |
| `Gamepad` | `GP` | `Gamepad` | 0 |
| `Playstation` | `PS` | `DualShockGamepad` | 10 |

#### Usage Example

```csharp
// Inspect the glyph code to choose the right icon set
string code = InputManager.CurrentDeviceProfile.glyphCode; // "PS", "KBM", "GP", ...

// Check whether the current profile belongs to a specific control scheme
bool isGamepad = InputManager.CurrentDeviceProfile.IsInControlScheme("Gamepad");
```

</details>

---

© JurgenWithWings 2026