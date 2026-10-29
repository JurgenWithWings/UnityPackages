# Hierarchy Icons

```
Created by: JurgenWithWings
Version: 0.2.0
```

An editor tool that replaces the default GameObject icon in the **Hierarchy** with the icon of the object's main
component, so you can tell what each object is at a glance. Empty GameObject with 'f:' at the start of the name will show a folder icon.

Works with both the original (IMGUI) Hierarchy window and the new UI Toolkit Hierarchy window added in Unity 6.3.

## Setup

1. Import the `.unitypackage` (**Assets** -> **Import Package** -> **Custom Package...**).

## Documentation

<details><summary><h3>Component Icons</h3></summary>

Each GameObject in the Hierarchy shows the icon of its main or highest component in the inspector:

| GameObject | Icon shown |
|------------|------------|
| Has components besides `Transform` | The icon of the first component after `Transform` (e.g. `Camera`, `Light`, `MeshRenderer`) |
| UI object (`RectTransform` + `CanvasRenderer` + ...) | The icon of the component after `CanvasRenderer` (e.g. `Image`, `TextMeshProUGUI`) instead of the Canvas Renderer's |
| Only a `Transform` | The `Transform` icon |

Hover over an icon to see the component's type name as a tooltip.

If a component has no icon, Unity's default icon is kept.

</details>

---

<details><summary><h3>Folders</h3></summary>

To show a GameObject as a folder, leave it with only a `Transform` and start its name with `f:`, e.g. `f:Environment`.
This is handy for grouping objects in the Hierarchy.

A GameObject named `f:...` that has other components shows its main component's icon as usual.

</details>

---

<details><summary><h3>Prefabs</h3></summary>

Prefab instances also get component icons, replacing the blue prefab cube. To keep Unity's prefab icon, open
`HierarchyIconDisplay.cs` and uncomment the line below `// Comment out to make prefabs have Icons` in both places it
appears (`ApplyIcon` for the new Hierarchy, `OnHierarchyWindowItemOnGUI` for the original one):

```csharp
// Comment out to make prefabs have Icons
//if (PrefabUtility.GetCorrespondingObjectFromSource(go) != null) return;
```

</details>

---

© JurgenWithWings 2026
