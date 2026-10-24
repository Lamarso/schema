---
name: pstack-engineering
description: Use for code changes, debugging, API integration and architecture work that needs root-cause investigation and observable behavioral verification.
---

# Engineering workflow

Read the project's P-stack profile and existing build/test runbook. Follow the project's language and tools; retain the runtime's configured model and permissions.

- Trace the relevant entry point, data flow and external boundary. Use history or design records when they explain a real constraint. For a known small fix, inspect only what is needed.
- For a defect, establish a reproducible failure or a precise observation. Add a regression test when it protects meaningful behavior. Test observable contracts rather than mirroring implementation. Do not require tests for trivial prose or reversible cosmetic edits.
- Fix the first incorrect state or decision. Keep types and validation honest at boundaries. Remove obsolete callers only after migrating and checking them. Prefer simplification when a new abstraction does not earn its maintenance cost.
- For cross-cutting design or uncertain alternatives, compare the viable options and their operational consequences. Prototype the uncertain boundary when a small experiment can decide it. Use independent review for material risk; verify findings against source before accepting them.
- Run the focused checks warranted by the change, then the repository's required checks. Separate syntax/type checks, fixture tests and real integration evidence. Never claim deployment from a local pass.

For a project lacking a reliable verification recipe, maintain a project-local verification contract: prerequisites; exact launch and readiness check; isolated data/account/port; an actual user/API/CLI journey; observable expected result; evidence location; teardown of only resources started by this run. Inspect dry-run code before trusting it to avoid network or writes. Prove the recipe once before labeling it verified; otherwise mark it draft with the specific missing prerequisite.

For multi-surface projects, keep a small feature map tied to source entry points: feature, route/command, prerequisite, check, expected observable outcome and coverage status. Start with the important journeys, update affected rows with behavior changes, and distinguish a product regression from documentation drift. Preserve evidence through cleanup. Avoid launching shared production services or reinstalling dependencies solely to make a workflow document look complete.
