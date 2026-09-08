# Frontend Delivery Skill

[简体中文](./README.md) | [English](./README_EN.md)

`frontend-delivery` is a general frontend delivery skill for Codex. It selects a lightweight change, regular delivery, or full workflow according to impact and uncertainty. Small edits stay lightweight, while complex work retains requirement review, scoped approval, recovery, verification, and delivery controls.

Current stable version: **v2.2.0**

## Key capabilities

- Three workflow depths: lightweight change, regular delivery, and full workflow.
- Three execution endpoints: review, planning, and full delivery.
- Separate completion of the current code handoff from overall acceptance.
- Task ID and version based approval, blocking, and scoped invalidation.
- UI recreation from images, screenshots, and Figma MCP.
- Motion recreation and Figma source change detection.
- Scenario based frontend risk checks loaded only when relevant.
- Text, newline, and path guidance for Windows, macOS, and Linux.
- Integration with existing code review, CI, and release requirements.

## Versions

| Version | Main changes | Browse | Download |
| --- | --- | --- | --- |
| 2.2.0 | UI and motion recreation, conditional risks, implementation handoff, and sequential routing | [View](https://github.com/new-forever/frontend-delivery-skill/tree/v2.2.0/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v2.2.0.zip) |
| 2.1.0 | Image, screenshot, and Figma MCP UI recreation | [View](https://github.com/new-forever/frontend-delivery-skill/tree/v2.1.0/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v2.1.0.zip) |
| 2.0.0 | Three workflow depths, task scoped approval, recovery, and risk checks | [View](https://github.com/new-forever/frontend-delivery-skill/tree/v2.0.0/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v2.0.0.zip) |
| 1.0.0 | Original five stage workflow with two gates | [View](https://github.com/new-forever/frontend-delivery-skill/tree/v1.0.0/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v1.0.0.zip) |

See [CHANGELOG.md](./skills/frontend-delivery/CHANGELOG.md) for details.

## Install with Codex

Send the following request to Codex to install the current stable version:

```text
Install https://github.com/new-forever/frontend-delivery-skill/tree/v2.2.0/skills/frontend-delivery
```

To install an older version, replace the version in the URL with the required tag:

```text
Install https://github.com/new-forever/frontend-delivery-skill/tree/v2.1.0/skills/frontend-delivery
```

The skill becomes available on the next turn after installation.

## Manual installation

1. Download and extract the ZIP for the required version.
2. Locate the `skills/frontend-delivery` directory.
3. Copy it to the Codex skills directory:

Windows:

```text
%USERPROFILE%\.codex\skills\frontend-delivery
```

macOS / Linux:

```text
~/.codex/skills/frontend-delivery
```

Do not install multiple versions with the same skill name in one environment. To switch versions, back up or remove the existing `frontend-delivery` directory, then copy the selected version.

## Usage examples

Review a requirement:

```text
Use $frontend-delivery to review this frontend requirement and report only issues and risks.
```

Create a plan:

```text
Use $frontend-delivery to create a frontend implementation plan without changing code.
```

Complete a lightweight change:

```text
Use $frontend-delivery to update this button label and perform the necessary static checks.
```

Recreate a page from Figma:

```text
Use $frontend-delivery to recreate this page from the following Figma node: <Figma node URL>
```

## Repository structure

```text
skills/frontend-delivery/
├─ SKILL.md
├─ CHANGELOG.md
├─ references/
├─ assets/templates/
├─ workflow-diagram.md
└─ workflow-diagram.html
```

- `SKILL.md`: entrypoint, workflow depth, and conditional reference routing.
- `references/`: detailed requirement, planning, implementation, UI, risk, recovery, and delivery rules.
- `assets/templates/`: records used by regular and full workflows when needed.
- [workflow-diagram.md](./skills/frontend-delivery/workflow-diagram.md): workflow diagrams for the current version.
- `workflow-diagram.html`: a browser view of the workflow diagrams after download.

The diagrams explain the structure. The versioned `SKILL.md` and reference files remain authoritative for execution conditions, permissions, and completion criteria.

## Versioning

This project follows semantic versioning:

- Major: incompatible workflow or state changes.
- Minor: backward compatible capability additions.
- Patch: rule or documentation corrections that preserve compatibility.

Published tags remain immutable. When a historical version needs a correction, publish a new patch version such as `v2.2.1`.
