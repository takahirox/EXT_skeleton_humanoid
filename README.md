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
models that use the extension. The basic idea consists of

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

This example video shows that the same waving hand keyframe animation data is
applied to two different humanoid models.

![example](./images/example.gif)

The models are originally from 

* https://hub.vroid.com/en/characters/287819523106027526/models/7392039141849953586
* https://hub.vroid.com/en/characters/2843975675147313744/models/5644550979324015604

and a bit edited to support the extension.

## Predefined skeleton

### Humanoid bone set and hierarchy

T.B.D.

TODO: Finalize the humanoid bone set and hierarchy [#10](https://github.com/takahirox/EXT_skeleton_humanoid/issues/10)

Currently based on [VRM humanoid bone set](https://github.com/vrm-c/vrm-specification/blob/master/specification/VRMC_vrm-1.0/humanoid.md).

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

### Default pose

T.B.D.

TODO: Finalize the default pose restriction [#11](https://github.com/takahirox/EXT_skeleton_humanoid/issues/11)

Currently based on [Godot Engine](https://docs.godotengine.org/en/latest/tutorials/assets_pipeline/retargeting_3d_skeletons.html#rest-fixer)

The extension adds some restrictions to the default pose to ease the animation
retargeting.

* T-Pose
* Facing +Z
* Directs the +Y axis from the parent joint to the child joint
* +X rotation bends the joint like a muscle contracting

## Skeleton definition

The `EXT_skeleton_humanoid` extension in root `glTF` allows to define humanoid
skeletons. `glTF.EXT_skeleton_humanoid.humanoidSkeletons` takes an array of
humanoid skeleton definitions.

`glTF.EXT_skeleton_humanoid.humanoidSkeleton.humanoidBones` defines a map from
the predefined humanoid bone names to `nodes`.
`glTF.EXT_skeleton_humanoid.humanoidSkeleton.rootNode` specifies a skeleton
root `node`.

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

Schema: [glTF.EXT_skeleton_humanoid.schema.json](./schema/glTF.EXT_skeleton_humanoid.schema.json)

### EXT_skeleton_humanoid property

| Property | Type | Description | Requires |
|:------|:------|:------|:------|
| `humanoidSkeletons` | `humanoidSkeleton [1-*]` | An array of skeleton definitions | :white_check_mark: Yes |

### humanoidSkeleton property

| Property | Type | Description | Requires |
|:------|:------|:------|:------|
| `rootNode` | `integer` | `node` index as root node of this skeleton | :white_check_mark: Yes |
| `humanoidBones` | `Object` | A map from the predefined bone names to `node` indices | :white_check_mark: Yes |

## Animation definition

The `EXT_skeleton_humanoid` extension in `animation.channel.target` allows to
specify the target node with a humanoid bone name predefined in the
"Humanoid bone and hierarchy" section.

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

`animation.sampler` reffered by a `animation.channel` that defines this
extension **MUST NOT** be reffered by another `animation.channel` that
does not define this extension.

Schema: [animation.channel.target.EXT_skeleton_humanoid.schema.json](./schema/animation.channel.target.EXT_skeleton_humanoid.schema.json)

### Animation property

| Property | Type | Description | Requires |
|:------|:------|:------|:------|
| `humanoidBoneName` | `string` | A predefined bone name to specify a target node | :white_check_mark: Yes |

## Animation retargeting

An animation channel that defines the `EXT_skeleton_humanoid` extension is
expected to be retargeted to a humanoid model that defines the
`EXT_skeleton_humanoid` extension.

The implementation can find a target node of a humanoid model by accessing
`EXT_skeleton_humanoid.humanoidSkeleton.humanoidBoneName` as like the following
pseudo code.

```
boneName = animation.channel.target.extensions.EXT_skeleton_humanoid.humanoidBoneName;
targetNodeIndex = glTF.extensions.EXT_skeleton_humanoid.humanoidSkeletons[skeletonIndex].humanoidBones[boneName];
```

`animation.sampler` referred by `animation.channel` whose `target` defines
the `EXT_skeleton_humanoid` extension is expected to hold keyframe animation data
that represents relative transform from target node's default pose.

The default poses are specified with
[skin.inverseBindMatrices](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#reference-skin)
so retargeted animation can be calculated as like the following pseudo code.

```
bindWorldMatrix = invertMatrix(inverseBindMatrix);
bindLocalMatrix = parentInverseBindMatrix * bindWorldMatrix;
animatedLocalMatrix = bindLocalMatrix * composeMatrix(translation, rotation, scale);
(animatedTranslation, animatedRotation, animatedScale) = decomposeMatrix(animatedLocalMatrix);
```

## Appendix - What is the difference from [VRM](https://vrm.dev/)

*Non-normative extension proposal author comment*

[VRM](https://vrm.dev/) may be a well known humanoid avatar format based on
glTF and it has a glTF humanoid skeleton extension. The readers who know
VRM may wonder why we need another humanoid skeleton extension.

One of the major motivation of proposing the `EXT_skeleton_humanoid`
extension is to free from the limitations of VRM.

VRM has a strong restriction that it allows only one model in a single file.
Its humanoid skeleton extension follows this restriction so that only one
skeleton can be defined in a single file.

It may be a bit too strong limitation and some workflows may not be
compatible with VRM due to the limitation.

The `EXT_skeleton_humanoid` extension does not add such a limitation.
Multiple humanoid models and multiple skeletons can be included in a
single glTF file.

Another disadvantage of VRM is it does not specify keyframe animation
or animation retargeting although they are important features for
humanoid models. The `EXT_skeleton_humanoid` extension supports them.

VRM is primarily designed as VR avatar model format and has unique
features and restrictions. I think it is not good to be tied to their
restrictions and their specific ecosystems just for creating a
humanoid model in glTF. I hope we have a humanoid skeleton
multi-vendor extension that is simpler and has less restrictions.

If this extension will be accepted, the VRM humanoid extension
(and other rich humanoid extensions) may extend this extension.
