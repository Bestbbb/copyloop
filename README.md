# CopyLoop

Rebuild a scoped website workflow from browser observations, then compare the result against the reference.

**Status: specification stage.** This repository contains the MVP design, the first reference case, seed data, and acceptance criteria. The CLI, generation skill, browser harness, and reconstruction are **not implemented yet**. There is no install command or released package.

CopyLoop is intended to produce an editable local website plus evidence showing which visual, interaction, and persistence requirements passed. It does not promise complete recovery of an arbitrary product or its backend.

## Start here

- [MVP technical and product specification (中文)](docs/MVP_SPEC.zh-CN.md)
- [Case 001: linkding bookmark workspace (中文)](docs/CASE_001_LINKDING.zh-CN.md)
- [Acceptance contract and release gates (中文)](docs/ACCEPTANCE.zh-CN.md)
- [Machine-readable case definition](cases/linkding-v1/case.json)
- [Synthetic seed data](cases/linkding-v1/seed.json)

## Planned workflow

1. Define the reference URL, scoped journeys, and allowed observations.
2. Capture states, interactions, layout, and motion with a pinned browser.
3. Generate a scoped implementation in a separate workspace.
4. Replay the same journeys against both versions.
5. Compare independent functional results, screenshots, and motion evidence.
6. Repair discrepancies within a recorded budget and publish the remaining gaps.

The proposed implementation is a local TypeScript CLI with Playwright/Chromium, a coding-agent skill, and a static HTML/JSON report. The first generated target is a React/Vite application using IndexedDB. Those are design decisions, not shipped components.

## First case

The reference is [linkding v1.47.0](https://github.com/sissbruecker/linkding/releases/tag/v1.47.0), a self-hosted bookmark manager. The case covers listing, adding, searching, tag filtering, editing, duplicate handling, cancellation, deletion, keyboard use, and persistence. It runs against an isolated local instance; the shared public demo is not a test fixture.

Reference source commit: `24b5ad6cc9bde497b5d1b1e86aed5a2fb7d25c2f`.

The reference runner may use upstream code to start and seed the application. The reconstruction agent receives browser observations and approved assets, not the upstream implementation. Source-assisted experiments must be labeled separately.

## What counts as done

Functional, visual, motion, independence, and delivery gates must all pass. A weighted score cannot hide a failed workflow. Missing evidence remains `unknown` or `not_run`. Passing this one case supports a case-specific alpha claim, not general website-cloning performance.

Acceptance targets are specified in the linked contract; no measurements have been reported yet.

## Contributing

Start with the spec and acceptance contract. Implement the milestones in order, preserve failed runs, and keep private browser sessions and runtime captures out of Git. Changes to acceptance thresholds must precede a new candidate run and be recorded in the report.

## License

CopyLoop-authored material is licensed under [MIT](LICENSE). linkding is a separate MIT-licensed project by Sascha Ißbrücker; its name and source links identify the reference, not an affiliation. No linkding application source or brand assets are bundled here. Any later redistributed third-party assets must retain their license and attribution.
