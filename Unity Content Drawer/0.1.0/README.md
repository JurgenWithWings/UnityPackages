# Unity Content Drawer

```
Created by: JurgenWithWings
Version: 0.1.0
```

An editor tool that adds a pop-up **Project window** to Unity, akin to the Unreal Engine Content Drawer. Press a shortcut and it slides up
from the bottom of the editor, so you can find an asset without keeping a Project window docked, freeing up that precious screen space.

## Setup

1. Import the `.unitypackage` (**Assets** -> **Import Package** -> **Custom Package...**).
2. Press **Ctrl + Space** (**Cmd + Space** on macOS) to open the drawer.

## Documentation

<details><summary><h3>Usage</h3></summary>

| Action | How |
|--------|-----|
| Open / close the drawer | **Ctrl + Space** (**Cmd + Space** on macOS), or **Window** -> **Toggle Content Drawer** |
| Resize the drawer | Drag its top edge. The new height is used the next time it opens. |

The drawer is a normal Project window, so you can browse, search, and drag assets from it into the Scene, the Hierarchy,
or the Inspector.

When it closes, focus returns to the Scene view if one is open.

Shortcut can be changed Unity's settings, go to **Edit** -> **Shortcuts...** and search for `Toggle Content Drawer`.

</details>

---

<details><summary><h3>Known Bugs</h3></summary>

- Sometimes pressing the shortcut to close the drawer will bring up a new one instead. You can just close the old one.
- Unity the windows close icon instead of the shortcut causes any changes to the window to not be saved (e.g. height, zoom).

</details>

---

© JurgenWithWings 2026
