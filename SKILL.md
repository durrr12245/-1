---
name: moka-portrait-agent
description: Continue and manage the user's staged Moka portrait projects in conversation, including identity references, paired front/profile beauty anchors, character views, environment, shot plans, pilots, series, and finishing. Use when the user asks to start, inspect, continue, generate for, select, or confirm a 模卡/写真 project. Do not use for unrelated image generation.
---

# Moka Portrait Agent

Operate the user's Moka portrait workflow from the current Codex conversation while keeping the local web app as the project record.

## Project source

The default project root is `C:\Users\HP\WorkBuddy\2026-09-16-19-57-06`. Respect a different root when the user provides one.

Resolve the requested project by exact ID first, then exact name. Run `scripts/moka_context.py --root <root> --project <id-or-name>` before deciding what to do. Use its absolute reference paths instead of rediscovering IDs manually.

## Operating rules

- Follow the eight stages in order: identity, beauty, character, environment, plan, pilot, series, finish.
- Treat the first unconfirmed stage as the default next action. Do not skip an unmet prerequisite.
- Generate with the built-in `image_gen` tool under the current ChatGPT account. Never use the skill's project API key, CLI image fallback, `api.openai-next.com`, or another external image service.
- Inspect every local reference image with `view_image` before generation. State each image's role in the prompt.
- Generate one candidate per call. Preserve identity and stage-specific invariants from `references/workflow.md`.
- Show the candidate to the user before confirmation. Importing or selecting a candidate is reversible and may proceed; confirming a stage requires the user's explicit acceptance of that stage's visible result.
- Do not claim biometric certainty or 3D reconstruction. Describe visual consistency and visible differences plainly.
- Never automatically retry a failed generation more than once. Do not switch to an API-key fallback unless the user explicitly asks.

## Generation and writeback

1. Load project context and the relevant reference images.
2. Read `references/workflow.md` for the current stage's reference roles and acceptance criteria.
3. Use built-in `image_gen` with local `referenced_image_paths`.
4. Inspect the generated output. If it satisfies the request, import it with:

   `scripts/moka_bridge.py import --project <id> --stage <stage> --key <field> --image <generated-file> [--shot-index <index>] [--source-photo <id>]`

   Use `front`/`side` for beauty, `front`/`side`/`back` for character, `photo` for pilot, and `photos` for series or finish.
5. Report that the image is a candidate and ask the user to inspect it. Do not confirm on their behalf.
6. After explicit acceptance, run:

   `scripts/moka_bridge.py confirm --project <id> --stage <stage> --accepted`

If no compatible local Moka server is running, tell the user the candidate remains generated but was not written back; do not edit `.moka-data` JSON directly while a server may be active.

## Conversational shortcuts

- “继续我的模卡项目” → list projects if selection is ambiguous; otherwise continue the first unconfirmed stage.
- “生成侧脸基准” → use the approved frontal anchor plus original side views, then import to `beauty.side`.
- “确认这一阶段” → confirm only when the user has just reviewed the current stage output and the intended project is unambiguous.
- “状态” → summarize confirmed stages, missing inputs, and the next concrete action.
