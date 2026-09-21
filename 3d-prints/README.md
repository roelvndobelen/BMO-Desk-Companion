# BMO 3D-print files

The files are separated into **current custom parts** and **original reference parts**. Do not print both versions of a replacement part by mistake.

These are verified byte-for-byte copies of the locally saved STL files. Copy verification is not a new mesh validation or a guarantee of physical fit.

## Current custom parts

| File | Purpose | Dimensions and notes |
| --- | --- | --- |
| [Modified faceplate](current/faceplate_BMO_scherm_5mm_omhoog_AI_camera_mm.stl) | Replaces the original faceplate. | 140 × 178 × 16.5 mm. Screen opening moved 5 mm upward from the original; opening 109.5 × 65.5 mm. Camera posts 2.95 mm high with 1.8 mm pilot holes. Includes the USB fitting modifications. |
| [Rear cover with 12 × 12 mm opening](current/hatchdoor_BMO_USB_rechtsonder_12x12mm_v3.stl) | Replaces the original rear cover. | Opening at the bottom right when viewed from the smooth exterior, with 5 mm edge margins. This is a cable-clearance opening, not a standardized panel-mount USB socket. |
| [Front-slot cover](current/BMO_sleuf_afdekplaatje_73x14x1.6mm_v1.stl) | Glues behind the horizontal front slot to hide the interior. | 73 × 14 × 1.6 mm. Approximately 3 mm overlap around the 67 × 8 mm slot. |

These three custom files use **millimetre-sized coordinates** and should be imported in millimetres at **100% scale**. Earlier opening sizes and fit-test revisions are intentionally not included in this folder.

For the slot cover, print the broad flat surface on the bed and use opaque material. Dry-fit before gluing, and apply glue only to the overlapping border. Covering the slot also reduces ventilation through that opening.

## Original downloaded model set

The original files are preserved without rescaling or geometry changes. They are not all required for this customized build.

| File | Part |
| --- | --- |
| [body_bmo.stl](originals/body_bmo.stl) | Main body. |
| [arms_bmo.stl](originals/arms_bmo.stl) | Arms. |
| [legs_bmo.stl](originals/legs_bmo.stl) | Legs. |
| [hatchframe_bmo.stl](originals/hatchframe_bmo.stl) | Rear access frame. |
| [dpad_bmo.stl](originals/dpad_bmo.stl) | Directional-pad piece. |
| [buttonbigcircle_bmo.stl](originals/buttonbigcircle_bmo.stl) | Large round button piece. |
| [buttoncircle_bmo.stl](originals/buttoncircle_bmo.stl) | Small round button piece. |
| [buttontriangle_bmo.stl](originals/buttontriangle_bmo.stl) | Triangular button piece. |
| [text_bmo.stl](originals/text_bmo.stl) | BMO lettering. |
| [faceplate_bmo.stl](originals/faceplate_bmo.stl) | Original faceplate, retained as a reference; use the custom faceplate above for this revision. |
| [hatchdoor_bmo.stl](originals/hatchdoor_bmo.stl) | Original rear cover, retained as a reference; use the custom cover above for the larger cable opening. |
| [battery_bmo_case_v2.stl](originals/battery_bmo_case_v2.stl) | Additional source-model part; inclusion does not mean a battery is installed in this build. |
| [heartdiplomo_bmo_case_v2.stl](originals/heartdiplomo_bmo_case_v2.stl) | Additional source-model part. |
| [infinitybox_bmo_case_v2.stl](originals/infinitybox_bmo_case_v2.stl) | Additional source-model part. |
| [medal_bmo_case_v2.stl](originals/medal_bmo_case_v2.stl) | Additional source-model part. |

## Important scale and printing notes

STL files do not store an explicit unit. The original faceplate and rear-cover files were previously found to use metre-sized coordinates, unlike the millimetre-sized custom parts. **Do not apply one blanket scale setting to both folders.**

Check every original part's dimensions in the slicer before printing. If a metre-coordinate part is interpreted as millimetres, its dimensions need a factor of 1000, equivalent to 100,000% of that incorrect import scale. Do not apply that factor if your slicer has already converted the units. The original faceplate should be 140 mm wide and 178 mm high; the original rear cover should be approximately 105.5 × 65.5 × 3 mm.

Other original parts have not been newly checked for scale in this packaging step. Verify mating dimensions against the correctly sized faceplate and body. Check orientation, unsupported regions, filament profile, and the sliced preview for each part. No universal support or print-speed setting is supplied here.

## Attribution and redistribution

The source enclosure is **BMO from Adventure Time — Local AI Agent Project**, by **brenpoly**:

- [Original model and authoritative license information](https://www.printables.com/model/1582055-bmo-from-adventure-time-local-ai-agent-project)
- [Original build video](https://www.youtube.com/watch?v=l5ggH-YhuAw)
- [Original software project](https://github.com/brenpoly/be-more-agent)

The custom faceplate and rear cover are modifications of the source models. The simple slot-cover rectangle was created for this customized build. No new license is granted here for third-party files, and the software repository's license must not be assumed to cover the STL models.

**Before making this package public, confirm the current source-model license and comply with its attribution, modification, and redistribution requirements.** The Printables license page could not be retrieved during preparation, so this local package is not a confirmation of permission for unrestricted redistribution or commercial use.

BMO and Adventure Time belong to their respective rights holders. This is an unofficial fan build.
