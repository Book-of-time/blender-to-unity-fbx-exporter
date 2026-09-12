
# Blender To Unity FBX Exporter

FBX exporter add-on for Blender 3.2+ compatible with Unity's coordinate and scaling system. Exported FBX files are imported into Unity with the correct rotations and scales.

## How to install

1. Clone the repository or download the add-on file [`blender-to-unity-fbx-exporter.py`](https://raw.githubusercontent.com/EdyJ/blender-to-unity-fbx-exporter/master/blender-to-unity-fbx-exporter.py) to your device.
2. In Blender go to Edit > Preferences > Add-ons, then use the Install… button and use the File Browser to select the add-on file.
3. Enable the add-on by checking the enable checkbox.

<p align="center">
<img src="/img/blender-to-unity-fbx-exporter-addon.png" alt="Blender To Unity FBX Exporter Add-On">
</p>

## How to use

**File > Export > Unity FBX (.fbx)**

Exports all Empty, Mesh and Armature objects in the current scene except those in excluded collections. The export may be limited to the active collection, the selected objects or the visible objects. The full hierarchy is properly preserved and exported, including local positions and rotations.

<p align="center">
<img src="/img/blender-to-unity-fbx-exporter-menu.png" alt="Blender To Unity FBX Exporter Menu">
</p>

The File Browser exposes selection, mesh, file and armature options:

<p align="center">
<img src="/img/blender-to-unity-fbx-exporter-options.png" alt="Blender To Unity FBX Exporter Options">
</p>

### Export options

Selection:

| Option | Default | Description |
|---|---|---|
| Active Collection Only | Off | Exports the objects in the active collection only (and its children). |
| Selected Objects Only | Off | Exports the selected objects only. |
| Visible Objects Only | Off | Exports the visible objects only. Objects and collections hidden or disabled in the outliner (eye and monitor icons) are not exported. |

These three options may be combined; only the objects matching all the enabled filters are exported. By default the exporter includes the objects hidden or disabled in the outliner, so enable **Visible Objects Only** to leave them out of the FBX file entirely.

Meshes:

| Option | Default | Description |
|---|---|---|
| Apply Modifiers | On | Applies the modifiers of the objects before exporting. Objects driven by an armature modifier are excluded, as their deformation is exported with the armature. Disable it to export the base meshes with their modifiers unapplied. |
| Export tangents | Off | Adds binormal and tangent vectors, together with the normal they form the tangent space (tris/quads only). Meshes with N-gons won't export tangents unless the option Triangulate Faces is enabled. |
| Triangulate Faces | Off | Converts all faces to triangles. This is necessary for exporting tangents in meshes with N-gons. Otherwise Unity will show a warning when importing tangents in these meshes. |

Files:

| Option | Default | Description |
|---|---|---|
| Embed Textures | Off | Embeds the texture files inside the FBX. |

Armatures:

| Option | Default | Description |
|---|---|---|
| Only Deform Bones | Off | Writes deforming bones only (and non-deforming ones when they have deforming children). |
| Add Leaf Bones | Off | Appends a final bone to the end of each chain to specify the last bone length (use this when you intend to edit the armature from the exported data). |
| Bone Axes (Primary / Secondary) | Y / X | Orientation of the bone axes in the exported file. |

### Applying modifiers

Modifiers are applied to the objects before exporting only when the **Apply Modifiers** option is enabled. This is the behaviour of the previous versions of this add-on, so the option defaults to On.

When the option is disabled the meshes are exported as they are, with their modifiers unapplied, and the modifiers are not baked into the FBX file either — the add-on takes care of disabling them in the built-in FBX exporter as well. Note that Curve, Surface and Font objects are always converted to meshes regardless of this option, as the FBX format cannot represent them otherwise. Objects driven by an armature modifier are not converted by the add-on in any case, as their deformation is exported with the armature.

### Exporting visible objects only

The add-on makes all the collections and objects visible while preparing the scene, so hidden objects are correctly transformed, and then restores the original visibility right before writing the FBX file. This is what allows the **Visible Objects Only** option to work: it's evaluated at that very moment, against the visibility state of the original scene.

Objects excluded from the view layer (unchecked collections in the outliner) are not exported in any case.

## How it works

The exporter modifies the objects in the Blender scene right before exporting the FBX file, then reverts the modifications afterwards.

Every object to be exported receives a rotation of +90 degrees around the X axis in their transform _without_ actually modifying the visual pose of its geometry and children. This is done in the root objects, then recursively propagated to their children (as they inherit a -90 rotation after transforming their parent). The modified scene is then exported to FBX using Blender's built-in FBX exporter with the proper options applied. Finally the scene is restored to the state before the modifications.

When Unity imports the FBX file all objects receive a rotation of -90 degrees in the X axis to preserve their visual pose. As the objects in the FBX already have a rotation of X+90 then the undesired rotation is canceled and everything gets imported correctly.

#### Why not use the "Experimental - Apply Transform" option of the default FBX Exporter?

This option doesn't work with object hierarchies of more than 2 levels. Objects beyond the 2nd level keep receiving unwanted rotation and scaling when imported into Unity.

#### Why not use the "Bake Axis Conversion" option in the Unity Import Settings?

Doesn't seem to work properly with Blender-generated FBX.

#### Why not import the .blend file directly in the Unity project?

Requires Blender to be installed in the system, so:

- it's a no-go for publishing packages in the Asset Store.
- .blend files don't work with Unity Cloud Build.

## Known issues

- Negative scaling is imported with a different but equivalent transform in Unity. Example: scale (-1, 1, 1) and no rotation is imported as scale (-1, -1, -1) and rotation (-180, 0, 0). In Unity this is equivalent, and may be changed to, the original scale (-1, 1, 1) and rotation (0, 0, 0).
- Child objects in instanced collections receive a rotation of 90 degrees in the X axis. Clearing this rotation in Unity gives the expected result. ([#3](https://github.com/EdyJ/blender-to-unity-fbx-exporter/issues/3))

#### Tested and working:

- Mixed EMPTY and MESH hierarchies with depth > 3.
- Local rotations are preserved.
- Non-uniform scaling.
- Mesh modifiers.
- Meshes exported with their modifiers applied (Apply Modifiers enabled, the default) or unapplied (Apply Modifiers disabled).
- Animations.
- Multi-user meshes and linked objects, with and without modifiers.
- Armatures and Armature modifier.
- Partial selections (Selected Objects Only).
- Limiting the export to the visible objects (Visible Objects Only).
- Hidden objects and collections (eye icon in the outliner).
- Disabled objects (monitor icon in the outliner). Imported with MeshRenderer disabled in Unity.
- Disabled collections (monitor icon in the outliner).
- Excluded collections (unchecked in the outliner). Won't be exported.
- Nested collections.
- Objects with their parent in a disabled/excluded collection.
- Custom object properties.

## Version history

**1.4.4**

- New **Apply Modifiers** option in the Meshes group. The modifiers are applied before exporting only when this option is enabled. It's on by default, which matches the behaviour of the previous versions; disable it to export the meshes with their modifiers unapplied.
- New **Visible Objects Only** option in the Selection group. When enabled, only the objects visible in the scene are exported; objects and collections hidden or disabled in the outliner (eye and monitor icons) are left out of the FBX file. It's off by default, so the exported objects are the same as in the previous versions.

## About the author

Angel "Edy" Garcia<br>
[@VehiclePhysics](https://twitter.com/VehiclePhysics)<br>
https://vehiclephysics.com
