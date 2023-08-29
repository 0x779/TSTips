
# Conversions and issues tips, tricks, scripts & everything in between

This is a series of useful tips & tricks, software quirks, good practices and limitations for converting various 3D formats.

Table of contents:

[3ds Max](#3ds-max)

[Blender](#blender)

[Lightwave](#lightwave)

[Maya](#maya)

[Unity](#unity)

[Unreal Engine](#unreal-engine)

[General](#general)

&nbsp;

## 3ds Max

### Batch converting 3ds Max files

For converting multiple 3ds Max files to various exchange formats (3DS, OBJ, DWG, FBX, etc.), there's an [excellent little script from bodylcg](https://bodyulcg.com/tools/batch-convert-max-files/) which does exactly that. 

### Performance improvement for large scenes

A very straightforward alleviation to the issue of an unresponsive or sluggish 3ds Max when working with very large scenes is to lower the texture resolution in the viewport.

<img width="433" alt="max_texturesize" src="https://user-images.githubusercontent.com/1897654/213131366-09123845-3a7a-499e-a6b6-f30c92dd45b1.png">

### Revert to standard materials from physical materials

Newer versions of 3ds Max (2021 and later) will default to physical materials instead of standard (scanline) materials when importing/exporting files. This is sometimes unwanted behavior, as not many software packages can interpret these phyiscal materials. Luckly, you can revert to standard scanline materials by editing the `3dsmax.ini` file in the your preferences folder (usually `C:\users\<username>\AppData\Local\Autodesk\3dsmax\<versionnumber>\ENU`) and changing the following lines:

In `[LegacyMaterial]`:
```
StandardMtlFBXImport=1
OBJImport=1
SceneConverter=1
3DSImport=1
ATFImporter=1
```

In `[ExportPhysicalMaterial]`:
```
PhysicalMtlAsLambert=1
```

&nbsp;

## Blender

### FBX transform in groups bug

There is an [ongoing unresolved bug](https://developer.blender.org/T51807) with the FBX importer in which objects nested 2 levels down in a group or object lose transforms. This means that any object that is in a hierarchy deeper than 1 level (eg. an object in a group which is in another group) will have incorrect location, rotation and scale information. This is especially prevalent in FBXs exported from 3ds Max.

**A workaround would be to either export in a different format (DAE works well) or explode the groups or remove objects from hierarchies where possible.**

&nbsp;

### FBX armature bugs

Unfortunately, the FBX Blender bugs don't stop there, as yet another [ongoing unresolved bug](https://developer.blender.org/T53620) in the FBX importer is wreaking havoc with the way Blender imports bones/rigs. This time, it has to do with fancy-pants matrix multiplications. 

Suffice to say that some rigged models do not import properly in Blender. One workaround is to export a GLTF/GLB and import it into Blender, but YMMV.

&nbsp;

### DAE vs FBX

On the topic of FBX, the current implementation of this importer is written in Python, which makes importing/exporting FBX files from/to Blender _very_ slow. On the other side of the spectrum, there is the DAE plugin, which is written in C++ and performs extremely well.

Whenever possible, try to use the DAE plugin, as it is fast, doesn't have the aforementioned transform bug and generally handles materials way better.

&nbsp;

### Increase TDR timeout to prevent crashes on larger scenes

When working on large scenes and either Blender or your GPU crashes, it helps to [increase the TDR timeout](https://substance3d.adobe.com/documentation/spdoc/gpu-drivers-crash-with-long-computations-tdr-crash-128745489.html). This tells Windows to increase the time it considers before considering a GPU hang (and subsequently restarts it).

&nbsp;


### Simple glass in EEVEE

Here's a quick and easy way of simulating semi-realistic glass in EEVEE (don't forget to set the blend mode to Alpha Blend). One thing to not is that due to how EEVEE renders transparent surfaces, there will be some z-fighting going on with complex objects. This method works flawlessly for flat objects, though.

![Simple glass in EEVEE](https://user-images.githubusercontent.com/1897654/195306029-cd2ad3fe-8687-4812-95d8-7b152ba27d80.png)


&nbsp;

### Clear all Custom Split Normal Data
Sometimes, imported meshes do not retain proper normals and are incorrectly shaded in Blender (this is especially prevalent in OBJs exported from Lightwave). This can be easily circumvented by using the "**Clear Custom Split Normal Data**" button, however it does not work for multiple meshes (the classic ALT+LMB does not work), thus the need for this script.

![enter image description here](https://i.imgur.com/JiUK6X9.png)

```python
import bpy

for o in bpy.data.objects:
    try:
        bpy.context.view_layer.objects.active = o
        bpy.ops.mesh.customdata_custom_splitnormals_clear()
    except:
        print("Object has no custom split normals: " + o.name + ", skipping")
```

&nbsp;

### Set property for all materials
Most of the times, when importing FBX files from 3ds Max, the materials come in with the Metallic property cranked up. Having multiple materials makes it a chore to modify each one, so here's a script that modifies all materials at once. 
You can change the `Metallic` property to anything else (eg. `Specular`, `Roughness`, etc.) with the same effect.

![enter image description here](https://i.imgur.com/iQimy20.gif)

```python
import bpy

for mat in bpy.data.materials:
    if not mat.use_nodes:
        mat.metallic = 0
        continue
    for n in mat.node_tree.nodes:
        if n.type == 'BSDF_PRINCIPLED':
            n.inputs["Metallic"].default_value = 0
```

&nbsp;


### Set Blend Mode for multiple materials
A quick an easy method of chaning the Blend Mode for materials. Change `OPAQUE` for anything else, such as `ALPHA_BLEND` or `ALPHA_HASHED`.

![enter image description here](https://i.imgur.com/IitCVHo.gif)

```python
import bpy

for mat in bpy.data.materials:
    mat.blend_method = 'OPAQUE'
```

&nbsp;

### Remove all Subdivision modifiers from all objects
An easier method to remove all of the subdivision modifiers from all objects without removing all other modifiers.

![enter image description here](https://i.imgur.com/CdsBP4j.gif)

```python
import bpy

for o in bpy.data.objects:
    for m in o.modifiers:
        if(m.type == "SUBSURF"):
            o.modifiers.remove(m)
```

[Back to top](#conversions-and-issues-tips-tricks-scrips--everything-in-between)

&nbsp;

---

&nbsp;

## Lightwave

### Preserving UVs

For the best results, export as OBJ where possible. This retains UVs, as opposed to other formats which either don’t or create additional UV channels.

[Back to top](#conversions-and-issues-tips-tricks-scrips--everything-in-between)

&nbsp;

---

&nbsp;

## Maya
### Shave and a Haircut

As Shave and a Haircut was acquired by Epic Games, it must be downloaded via Epic's GitHub repo. [Here](https://www.unrealengine.com/en-US/blog/shave-and-a-haircut-v9-6-for-maya) are the instructions on how to get access to the repo.

SaaH has some quirks when using Arnold. For version 9.6, the supported Arnold versions are: 2.0.1.0 to 3.0.9.9, 3.1.0.0 to 3.3.9.9. Anything else will not render.

&nbsp;

### Xgen

Most of Xgen's woes boils down to paths not being read correctly. First and foremost, if the model is in a project, it must be set via `File > Set Project`. 

Maya's log offers very good insights into why Xgen might not render. Be sure to check it in the Script Editor, which is accessible by going to `Windows > General Editors > Script Editor`.

[Back to top](#conversions-and-issues-tips-tricks-scrips--everything-in-between)

&nbsp;

---

&nbsp;

## Unity

### Exporting a scene from Unity

Exporting a whole Unity scene is easy, and while most things do carry over, it's usually a good idea to only select GameObjects that are or contain meshes. Cameras and light might carry over, but due to how Unity handles FOV, clipping limits, light intensities, etc., it might not be optimal.

![unityExport](https://user-images.githubusercontent.com/1897654/158948030-81b788bf-ee96-4a3a-8209-547c5eec18c3.gif)



&nbsp;

### Importing textures before the model

A good workflow for importing models into Unity is to place the model's textures first, and then import the asset. That way, Unity can find the textures it needs, without having to manually relink them.

![unityTextures](https://user-images.githubusercontent.com/1897654/158947357-3c77dc0f-3ad4-4586-ac0d-5c9f6a1d5f31.gif)



&nbsp;

### Extracting materials from the model

Continuing from the above workflow, you can extract the materials from the model without having to manually create materials, assign them and then relinking textures.

![unityMaterials](https://user-images.githubusercontent.com/1897654/158947374-4f589d00-f415-4336-8ff4-cc057ea78269.gif)



&nbsp;

### How Unity handles PBR textures

Unity handles texturing a bit different than, say, Unreal Engine.

- The **transparency** map is stored in the **diffuse** texture's alpha channel.
- The **glossiness** map is stored in the **metalness** texture's alpha channel.

Note that for the alpha, you can have a transparent PNG for the diffuse map, as it will correctly assume that the PNG's transparency is the opacity map.

As for the metalness/glossiness map, it will need to be in a format that has an alpha (not transparency) channel, such a 32-bit TGA (lower bit precision such as 8, 16 or 24 do not contain the alpha channel).


NOTE: Later versions of Unity contain an "Autodesk Interactive" material, which enables you to use Roughness maps, however it might not be part of the Universal Render Pipeline.


[Back to top](#conversions-and-issues-tips-tricks-scrips--everything-in-between)

&nbsp;

---

&nbsp;

## Unreal Engine

### Splitting imported models into original individual objects (UE4/UE5)
By default, Unreal Engine 4 combines all meshes into one object, which can be undesirable. To prevent this, when importing the model, uncheck the `Combine Meshes` box in the FBX Import Options dialog.

![enter image description here](https://i.imgur.com/CB05kIq.png)


&nbsp;

### How Unreal Engine 4 handles pivots (UE4/UE5)
It's a bit counterintuitive, but UE4 considers the scene center, or the 0,0,0 position as the model's pivot point. This makes the object's pivot point relative to its position to the scene center. Here is an example: 

![enter image description here](https://i.imgur.com/3kYny7K.png)

While in Blender, the object's pivot point is at the center of the object, UE4 considers the pivot as being lower, as that is the scene's center. In raw numbers, the object's position in Blender is `X: 0, Y: 0, Z: 10` and in UE4 it's `X: 0, Y:0, Z:0`

![enter image description here](https://i.imgur.com/CGDrN9M.png)

Moving the object in Blender and re-exporting it in UE4 causes it to move itself away from its "pivot point".

A good analogy for this is moving faces/vertices in 3ds Max in the Edit Poly mode. While the faces themselves move, the pivot point stays in the same place.

![enter image description here](https://i.imgur.com/qtBTGl2.gif)


&nbsp;

### Importing a model for Nanite (UE5)
Unreal Engine 5 now has a brand new system called Nanite which can process close to a bajillion polygons. Importing a model for Nanite is pretty straightforward: just check the `Build Nanite` box in the FBX Import Options.

![enter image description here](https://i.imgur.com/9mWrCKj.png)

**Please do note, that Nanite does not currently support skeletal meshes, so any kind of rig or animation will not carry over.**


&nbsp;

### Datasmith for large scenes (UE4)
Large FBX scenes usually send UE in a memory eating spiral and lead to inevitable crashes. This is where the Datasmith plugin comes in. It not only enables you to move very large scenes to UE, it also transfers CAD data with ease.
You can get the plugins [here](https://www.unrealengine.com/en-US/datasmith/plugins) and you can read about how to use them [here](https://docs.unrealengine.com/4.27/en-US/WorkingWithContent/Importing/Datasmith/SoftwareInteropGuides/CAD/).


&nbsp;

### Exporting a scene from UE

As with Unity, you can export a whole scene from UE4, with a few caveats. Depending on how the level is made, there will be a lot of unusable objects which the FBX can't parse (atmospheric fog, post-process volumes, blueprints, etc.) which come in as empty objects. Also, UE might export collision meshes (objects prefixed with `UCX_`) which are simplified versions of the main mesh generated by either the artist or the engine itself.

![ue4export](https://user-images.githubusercontent.com/1897654/158764291-832658d8-c8cc-4793-974a-a7b41fff3ddc.gif)


&nbsp;

---

[Back to top](#conversions-and-issues-tips-tricks-scrips--everything-in-between)


# General

## Rigs

Generally, the only rigs that *might* carry over to other software are 3ds Max's biped rig and simple bone rigs. Anything involving custom controllers or IK will most likely fail.

A good note on 3ds Max is that only the `Skin` modifier will correctly bind bones to the mesh in an exchange format. `Skin Wrap`, for instance, will not.

&nbsp;

## Animations

In the majority of cases, animations are bone-based, which are dependent on the rigs. In special cases, animation can be made with modfiers (eg. FFD in 3ds Max), which can only be exported via Alembic (`ABC`).

While Alembic usually exports any type of animation (as they are exporting each frame as separate geometry), some caveats include:
- Very large file size for long animations in complex models
- No UVs*
- No materials*
- No textures*

*this is highly dependent on the Alembic exporter used (eg. 3ds Max's Alembic exporter does not support materials, but does seem to export UVs).

&nbsp;

## USDZ

USDZ is a specialized format mostly used for AR in iOS devices. While it is based on USD, it can't be imported or exported by most software.

Various exchange formats such as OBJ, GLTF and USD can be converted to USDZ using [Reality Converter](https://developer.apple.com/augmented-reality/tools/), however this is Mac-only.

&nbsp;

## Lumion

Currently, [nothing can be exported from Lumion](https://support.lumion.com/hc/en-us/articles/360003475333-Can-you-export-3D-models-from-Lumion-), be it geometry, materials or textures.
&nbsp;

## UDIMs

TODO
