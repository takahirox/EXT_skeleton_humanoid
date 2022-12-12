# EXT_skin_humanoid

## Contributors

* Takahiro Aoyagi, Mozilla, [@takahirox](https://github.com/takahirox)

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec

## Overview

This `EXT_skin_humanoid` extension allows `skin` to indicate that it has predefined standard
humanoid bone set and to specify the map from predefined humanoid bone names to the humanoid
bones (joint nodes).

This extension helps applications to find a certain predefined humanoid bone and
to apply a consistent skinning animation regardless of 3D models if they define this extension.

This extension also allows `animation.channel.target` to specify a target node with
a predefined bone name. It enables for example to make humanoid animation library (eg. Walk, Run,
Jump) that can be applied to any glTF assets having `skin` that defines this extension.

## Skin

T.B.D.

```json
"skins": [
    {
        "joints": [ 0, 1, 2, ... ],
        "extensions": {
            "EXT_skin_humanoid": {
                "humanoidBones": {
                    "neck": 0,
                    "leftHand": 1,
                    "rightHand": 2,
                    ...
                },
            },
        },
    },
],
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

Schema: [skin.EXT_skin_humanoid.schema.json](./schema/skin.EXT_skin_humanoid.schema.json)

## Default pose

T-Pose?

## Animation

T.B.D.

```json
"animations": [
    {
        "channels": [
            {
                "sampler": 0,
                "target": {
                    "path": "rotation",
                    "extensions": {
                        "EXT_skin_humanoid": {
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
                        "EXT_skin_humanoid": {
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
                        "EXT_skin_humanoid": {
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

TODO: Allow only rotation?

Schema: [animation.channel.target.EXT_skin_humanoid.schema.json](./schema/animation.channel.target.EXT_skin_humanoid.schema.json)
