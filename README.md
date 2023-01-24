# EXT_skeleton_humanoid

## Contributors

* Takahiro Aoyagi, Mozilla, [@takahirox](https://github.com/takahirox)

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec

## Overview

This extension defines a default humanoid skeleton in glTF, allows to retarget
glTF animations to any humanoid model that follows the standard skeleton, and
improves the reusability of humanoid animations.

Humanoid model is one of the popular 3D assets. You will find a lot of humanoid
models in glTF assets download websites.

[The glTF 2.0 core specification supports Linear Blend Skinning.](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#skins)
Humanoid animations, eg: Walk, Run, and Jump, are generally achieved with
skinning. Reusing humanoid animation that is made for a certain humanoid model
to other humanoid models is not easy in glTF because of lack of animation
retargeting mechanism.

What makes the retargeting difficult is that the glTF 2.0 core does not
specify default humanoid skeleton (a series of bones and their hierarchy)
therefore different humanoid models may have different formed skeletons.

Another reason is that animation target in the glTF 2.0 core is tied to a
certain node in the same glTF file and there is no way to target a node
in other glTF files.

Because of these two reasons there is no simple, clear, and common workflow for
humanoid animation retargeting in glTF. Currently even common humanoid
animations like Walk need to be made for each humanoid model. It is very costly.
Humanoid animation retargeting is important for saving cost.

This extension allows easy humanoid animation retargeting across humanoid
models that use the extension. The basic　idea consists of

* Predefine a default humanoid skeleton (a series of bones, their hierarchy,
and default pose)
* Allow animation channel to specify a target node with a predefined bone name
* Represent animation data as relative transform from the default pose

The implementations will be able to easily achieve humanoid animation
retargeting by

* Loading an animation data
* Loading a humanoid model
* Finding an animation target node by a predefined bone name
* Applying retargeted animation calculated from the default pose

Also, this extension allows to create common humanoid animation library because
animation target is no longer tied to a certain node in the same glTF file.

## Example

This example video shows that the same waving hand keyframe animation data is applied to two
models.

![example](./images/example.gif)

The models are originally from 

* https://hub.vroid.com/en/characters/287819523106027526/models/7392039141849953586
* https://hub.vroid.com/en/characters/2843975675147313744/models/5644550979324015604

The asset files are a bit edited from original to support the extension.

## Skeleton definition

The `EXT_skeleton_humanoid` extension in root `glTF` allows to define humanoid skeletons.
`glTF.EXT_skeleton_humanoid.humanoidSkeletons` takes an array of humanoid skeleton
definitions. The elements of the array have a skeleton root `node` index as `rootNode`
and a mapping from predefined humanoid bone names to `node` indices as `humanoidBones`.

```json
"extensions": {
    "EXT_skeleton_humanoid": {
        "humanoidSkeletons": [
            {
                "rootNode": 0,
                "humanoidBones": {
                    "neck": 1,
                    "leftHand": 2,
                    "rightHand": 3,
                    ...
                },
            },
        ],
    },
},
```

### Humanoid bones

T.B.D.

Based on [VRM humanoid bone set](https://github.com/vrm-c/vrm-specification/blob/master/specification/VRMC_vrm-1.0/humanoid.md).

```
└─ hips
   ├─ spine
   │  └─ chest
   │     └─ upperChest
   │        ├─ neck
   │        │  └─ head
   │        └─ [left|right]Shoulder
   │           └─ [left|right]UpperArm
   │              └─ [left|right]LowerArm
   │                 └─ [left|right]Hand
   │                    ├─ [left|right]ThumbMetacarpal
   │                    |  └─ [left|right]ThumbProximal
   │                    |     └─ [left|right]ThumbDistal
   │                    └─ [left|right][Index|Middle|Ring|Little]Proximal
   │                       └─ [left|right][Index|Middle|Ring|Little]Intermediate
   │                          └─ [left|right][Index|Middle|Ring|Little]Distal
   └─ [left|right]UpperLeg
      └─ [left|right]LowerLeg
         └─ [left|right]Foot
            └─ [left|right]Toes
```

<img src="./images/body.png" width="480" alt="body">

<img src="./images/hand.png" width="480" alt="hand">

TODO: Add bone names into the images.

Schema: [glTF.EXT_skeleton_humanoid.schema.json](./schema/glTF.EXT_skeleton_humanoid.schema.json)

*A non-normative extension proposal author comment:*

The current status of this extension proposal is as experiment to know whether the key
concept mentioned in the overview is good enough for humanoid animation retargeting. I
don't have any strong opinion and even don't have good knowledge to determine how the
actual bone set and hierarchy should be yet (for example how many spine bones should
be).

[VRM](https://vrm.dev/) may be one of the well known avatar formats based on glTF. And
there seems to be many VRM models. So perhaps it is good that the extension deifnes
bone set and hierarchy that are compatible with VRM humanoid skeleton for now.

### Default pose

The extension adds some rules for the default pose to ease the animation retargeting.

* T-Pose
* Facing +Z
* Directs the +Y axis from the parent joint to the child joint
* +X rotation bends the joint like a muscle contracting

Based on [Godot Engine](https://docs.godotengine.org/en/latest/tutorials/assets_pipeline/retargeting_3d_skeletons.html#rest-fixer)

*A non-normative extension proposal author comment:*

Similar to the author comment in the Humanoid bones section, I don't have any 
strong opinion and knowledge to determine the actual default pose yet. I reuse the
default pose defined in Godot engine for now because it looks good.

## Animation

`EXT_skeleton_humanoid` extension in `animation.channel.target` allows to specify the
target node with a humanoid bone name defined in the "Humanoid bones" section.

```json
"animations": [
    {
        "channels": [
            {
                "sampler": 0,
                "target": {
                    "path": "rotation",
                    "extensions": {
                        "EXT_skeleton_humanoid": {
                            "humanoidBoneName": "neck",
                        },
                    },
                }
            },
            {
                "sampler": 1,
                "target": {
                    "path": "rotation"
                    "extensions": {
                        "EXT_skeleton_humanoid": {
                            "humanoidBoneName": "leftHand",
                        },
                    },
                }
            },
            {
                "sampler": 2,
                "target": {
                    "path": "rotation"
                    "extensions": {
                        "EXT_skeleton_humanoid": {
                            "humanoidBoneName": "rightHand",
                        },
                    },
                }
            },
            ...
        ],
    },
],
```

`animation.sampler` reffered by a `animation.channel` that define this extension
**MUST NOT** be reffered by another `animation.channel` that doesn't define this
extension.

Schema: [animation.channel.target.EXT_skeleton_humanoid.schema.json](./schema/animation.channel.target.EXT_skeleton_humanoid.schema.json)

### Animation retargeting

Animation channel having `EXT_skeleton_humanoid` extension definition is expected
to be retargeted to a humanoid model having `EXT_skeleton_humanoid` extension
definition.

The implementation can find a target node of the humanoid model by accessing
`EXT_skeleton_humanoid.humanoidSkeletons[skeletonIndex].humanoidBoneName` as like the
following pseudo code.

```
boneName = animation.channel.target.extensions.EXT_skeleton_humanoid.humanoidBoneName;
targetNodeIndex = glTF.extensions.EXT_skeleton_humanoid.humanoidSkeletons[skeletonIndex].humanoidBones[boneName];
```

`animation.sampler` specified by `animation.channel` whose `target` has
`EXT_skeleton_humanoid` extension definition is expected to hold keyframe animation
data that represents relative translation/rotation/scale from target node
translation/rotation/scale in the skeleton's default pose.

The default pose can be calculated with [skin.inverseBindMatrices](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#reference-skin) so animated translation/rotation/scale can be calculated as like the following pseudo code.

```
bindWorldMatrix = invertMatrix(inverseBindMatrix);
bindLocalMatrix = parentInverseBindMatrix * bindWorldMatrix;
animatedLocalMatrix = bindLocalMatrix * composeMatrix(translation, rotation, scale);
(animatedTranslation, animatedRotation, animatedScale) = decomposeMatrix(animatedLocalMatrix);
```

## Appendix - What is the difference from [VRM](https://vrm.dev/)

*Non-normative extension proposal author comment*

[VRM](https://vrm.dev/) may be a well known humanoid model format based on glTF. It consists of
some glTF extensions and some restrictions against glTF, for example VRM allows only one 
humanoid model in a single VRM file. VRM has a glTF humanoid skeleton extension but it is based
on the restriction that VRM allows only one humanoid model in VRM file so there is no way to
define multiple humanoid skeletons in a single VRM file.

VRM is primarily designed as a VR avatar format so their extension might be richer than
general humanoid model use requirement.

VRM doesn't specify any keyframe animation although keyframe animation
is one of the important stuffs in humanoid models.

Humanoid skeleton is a popular use so a humanoid skeleton extension may not really need to
be a vender specific extension. Ideally it is at least a multi-vendor extension.

I started to write this draft to know if it is good to revisit and rewrite a standard
glTF humanoid skeleton extension that may be simpler, may be no or less restriction as
glTF (ex: Allow define multiple humanoid skeletons in a single glTF file), may cover
animation retargeting, and may be as a multi-vendor extension.

In this scenario VRM (and other richer vendor specific humanoid skeleton extensions)
can be written against this extension.
