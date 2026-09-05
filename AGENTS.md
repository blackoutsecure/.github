# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## What this is

`blackoutsecure/.github` is GitHub's organization-level special repository. GitHub treats a repo
literally named `.github` as the org's default provider for three unrelated features: community
health files inherited by sibling repos, the org profile page rendered from `profile/README.md`,
and starter workflows offered under Actions -> New workflow from `workflow-templates/`. There is
no application code here, and none should be added.

The content is Markdown, YAML, and JSON: policy documents, issue and pull-request templates, lint
configuration, two hub-managed kicker workflows, one locally-authored drift check, and one starter
workflow template. It also dogfoods the hygiene baseline `bos-marketplace-kit` recommends to
consumers (`DP001`, `LT001`-`LT005`); that config applies to this repo only and is inherited by
nothing. Most of the policy text is not authored here either — `bos-automation-hub` owns it under
`sync-files/` and pushes it in through `bos-managed-file-sync-action`, via the `org_defaults`,
`license_service`, `common`, `lf_line_endings`, `coverage_artifacts`, `shellcheck`, and `yamllint`
services. Check [Source of truth](#source-of-truth) before editing anything.

## Commands

There is no `package.json`, `pyproject.toml`, `Makefile`, or lockfile here, so no local package
manager and no repo-defined script. Validation is invoking the linters directly.

```bash
yamllint -c .yamllint.yml .
markdownlint-cli2 "**/*.md"
actionlint .github/workflows/*.yml workflow-templates/*.yml

python3 -m json.tool bos-universal-config.json >/dev/null
python3 -m json.tool .github/bos-universal-config.json >/dev/null
jq empty bos-launchpad-config.json
jq empty workflow-templates/bos-workflow-gatekeeper-kicker.properties.json
```

There are no standalone shell scripts, so `shellcheck` has no direct target; `.shellcheckrc`
exists because `actionlint` shells out to `shellcheck` for `run:` blocks and reads that config.

## Validating changes

- `.github/workflows/bos-universal-security-kicker.yml` (hub-managed) runs on push and pull
  request against `main`, on `merge_group`, daily on cron, and on dispatch. It calls the hub's
  `bos-universal-sync.yml` in `commit` mode, resolves a hub ref, then calls the hub's reusable
  `bos-universal-security.yml` with `config_authoritative: true`. That is the single required
  gate: markdownlint, yamllint, actionlint, shellcheck, `bos-code-scanning-kit`, CodeQL,
  dependency review, the canonical README header check, and the conventional-commit title check.
- `.github/workflows/bos-universal-sync-kicker.yml` (hub-managed) reconciles managed files on a
  weekly cron, on a push touching `.github/bos-universal-config.json`, or on dispatch with
  `mode: commit|check`.
- `.github/workflows/check-kicker-template-sync.yml` (authored here) fetches
  `blackoutsecure/bos-workflow-gatekeeper@main:examples/kicker.yml` and fails when the `on`,
  `permissions`, or `jobs` sections of `workflow-templates/bos-workflow-gatekeeper-kicker.yml`
  differ. Header comments may differ; functional shape may not.

Locally, narrowest first: lint the file you touched, then the whole file type, then the JSON parse
checks if a config changed, then `actionlint` if a workflow or template changed. If you edited the
gatekeeper template, diff it against upstream `examples/kicker.yml` yourself — the drift check is
the only thing that catches it, and it fails the whole gate.

## Architecture

```text
README.md                     This repo's own docs: inheritance contract, template rules.
CODE_OF_CONDUCT.md CONTRIBUTING.md SECURITY.md SUPPORT.md   Org defaults. Inherited org-wide.
LICENSE                       Apache-2.0, distributed by the hub's `license_service`.
profile/README.md             Rendered at https://github.com/blackoutsecure as the org profile.
bos-universal-config.json     Root universal config; adds a `launchpad` block disabling stages.
bos-launchpad-config.json     Launchpad sync services (`sync_files.services`) for this repo.
workflow-templates/           Starter workflows offered under Actions -> New workflow.
.github/bos-universal-config.json   The config the sync engine discovers first.
.github/CODEOWNERS            Review owners for this repo only; CODEOWNERS never inherits.
.github/dependabot.yml        Weekly `github-actions` bumps; managed `dependabot_actions` block.
.github/FUNDING.yml PULL_REQUEST_TEMPLATE.md ISSUE_TEMPLATE/   Org defaults. Inherited org-wide.
.github/workflows/            Two hub-managed kickers plus the local template drift check.
.editorconfig .gitattributes .gitignore .markdownlint.yaml .yamllint.yml .shellcheckrc
                              Local hygiene and lint config; the first three carry managed blocks.
.vscode/extensions.json       Recommended editor extensions.
```

### Branch model

`main` is the default branch and the only long-lived one; `git branch -a` shows `main`,
`origin/HEAD -> origin/main`, and a pending `chore/seed-bos-universal-gatekeeper-kicker` branch.
This repo does not use the `dev` -> `main` promote flow the code repos use, and carries no version
tags, because nothing here is consumed by SHA or tag. Work lands on `main` through a pull request.

### Inheritance semantics

GitHub applies a community health file from this repo to a sibling repo only when that repo does
not define its own copy, at its root or under its own `.github/`. If the sibling defines one,
GitHub uses that file verbatim and ignores the org default entirely — no merging, no field-level
fallback, no partial inheritance. Inheritable here: `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`,
`SECURITY.md`, `SUPPORT.md`, `.github/FUNDING.yml`, `.github/PULL_REQUEST_TEMPLATE.md`, and
`.github/ISSUE_TEMPLATE/`. These never inherit and must live in the consuming repo:
`.github/workflows/**`, `.github/dependabot.yml`, `.github/CODEOWNERS`, `LICENSE`, `NOTICE`, the
repo `README.md`, and all repo settings, branch protection, and secrets. The hygiene config here
is likewise local-only.

### Workflow templates

`workflow-templates/` is a separate feature from inheritance and copies nothing automatically.
Each `<name>.yml` needs a matching `<name>.properties.json`; the pair appears as a suggested
starter under Actions -> New workflow for every repo in the org. A maintainer must pick it, and it
is then written into that repo's `.github/workflows/` as an ordinary independent file with no
ongoing link back here. An `iconName` in the properties file requires a matching `<name>.svg` here
or the reference is broken.

- `bos-workflow-gatekeeper-kicker` — a `workflow_dispatch` front door that authorizes the
  triggering actor with the published `bos-workflow-gatekeeper` action, optionally narrows the
  requested operation against `.github/kicker-policy.json`, and routes to a backend
  `workflow_call` workflow. It is a relayed copy of that repo's `examples/kicker.yml`; the hub's
  managed-file sync flows hub -> consumers and does not cover this shape, so
  `check-kicker-template-sync.yml` is the guardrail instead. Change one, change both.

### Org configuration

`.github/bos-universal-config.json` is the repo-owned override layer the sync engine discovers
first. It sets `general.target_repo_role` to `org-default-repo` and enables `common`,
`lf_line_endings`, `bos_universal_gatekeeper_kicker`, and `org_defaults` in `commit` mode. Service
lists append across tiers rather than replacing, so the hub global config at
`bos-automation-hub/sync-files/config/managed-file-sync-global-config.json` adds `shellcheck`,
`yamllint`, `coverage_artifacts`, `license_service`, and `security_readme_pointer` on top. The
root `bos-universal-config.json` is a near-duplicate that also disables every launchpad stage, and
`bos-launchpad-config.json` declares this repo's launchpad sync services.
`bos_universal_gatekeeper_kicker` is enabled but its workflow is not present yet, which is what
the pending seed branch is for. Precedence matches the rest of the org: bundled marketplace
baseline, hub global config, repo config here, then any workflow input; mappings deep-merge and
scalars replace. Change gate behaviour in the repo config, never in a kicker.

### Source of truth

Synced from `bos-automation-hub/sync-files/` and verified byte-identical to the hub source. Edit
the hub, not the copy here:

- `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md`, and `.github/FUNDING.yml`
  from `community-health/`; `.github/PULL_REQUEST_TEMPLATE.md` and `.github/ISSUE_TEMPLATE/*` from
  `github-meta/`; `profile/README.md` from `org-profile/README.md`
- `LICENSE` from `legal/LICENSE` via `license_service`; the local copy is currently reformatted and
  will be overwritten on the next `commit`-mode sync
- the `# >>> managed-file-sync:<service> >>>` blocks inside `.editorconfig`, `.gitattributes`,
  `.gitignore`, `.shellcheckrc`, `.yamllint.yml`, and `.github/dependabot.yml`
- `.github/workflows/bos-universal-security-kicker.yml` and `bos-universal-sync-kicker.yml`

Authored here: `README.md`, `.markdownlint.yaml`, `.vscode/extensions.json`, `.github/CODEOWNERS`,
`.github/workflows/check-kicker-template-sync.yml`, everything under `workflow-templates/`, both
`bos-universal-config.json` files, `bos-launchpad-config.json`, and the prose outside the managed
blocks in `.editorconfig`, `.gitattributes`, `.gitignore`, and `.github/dependabot.yml`.

## Conventions

Markdown wraps at roughly 72 columns in the community health files, which is why `MD013` is off in
`.markdownlint.yaml`; `MD024` is `siblings_only`, and `MD028`, `MD033`, `MD034`, `MD041` are
relaxed for admonitions, inline HTML, bare URLs, and templates starting at `H2`. YAML is two-space
indented with LF endings, `document-start` disabled, `truthy` relaxed so a bare `on:` key does not
fail, and a 200-column soft limit — the kickers quote it as `"on":` to sidestep the YAML 1.1
boolean resolver. Every `uses:` in `.github/workflows/` is a 40-character commit SHA with a
trailing version comment; inside `workflow-templates/` a readable `@v1`-style ref is deliberate,
since that file is a starter a human copies and adapts. The canonical README header the org gate
enforces on code repos is `# Blackout Secure <Name>` followed by
`**Copyright © <year[-year]> Blackout Secure | Apache License 2.0**`; it does not apply to this
repo's `README.md`, which is internal documentation rather than a published listing. Every
template needs its paired properties file, and the shape is small:

```json
{
  "name": "Blackout Secure Kicker",
  "description": "workflow_dispatch front door: authorize with bos-workflow-gatekeeper, allowlist-narrow the requested operation, and route to a backend workflow.",
  "categories": ["Deployment", "Utilities"]
}
```

## Boundaries

### Always

- Establish whether a file is hub-synced before editing it; make the change at
  `bos-automation-hub/sync-files/` when it is.
- Keep `.yml` and `.properties.json` paired for every template, and update the template and
  `bos-workflow-gatekeeper`'s `examples/kicker.yml` together.
- Run `yamllint`, `markdownlint-cli2`, `actionlint`, and the JSON parse checks on what you changed.
- Pin every `uses:` in `.github/workflows/` to a commit SHA with a trailing version comment.
- Keep policy text generic enough to apply org-wide; repo-specific process belongs in that repo.

### Ask first

- Changing a community health file that every repo in the org inherits. One edit to `SECURITY.md`,
  `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SUPPORT.md`, `FUNDING.yml`, the PR template, or the
  issue templates silently changes the published policy of every repo that has not overridden it.
- Changing `profile/README.md`. It is the public organization landing page.
- Changing a workflow template that repos have already copied. Copies are independent, so an edit
  here does not reach them and creates two divergent versions of the same starter.
- Changing the service list, `mode`, or `target_repo_role` in either `bos-universal-config.json`,
  or the launchpad services in `bos-launchpad-config.json`.
- Adding a workflow to `.github/workflows/`, loosening the drift check's compared sections, or
  widening `.github/CODEOWNERS` so the inherited surface loses maintainer review.

### Never

- Never commit secrets, tokens, private keys, certificates, internal URLs, customer data, or PII.
- Never add application code, a backend, a build step, or a runtime dependency here. This repo is
  documentation and configuration only.
- Never use an unpinned `uses:` reference in `.github/workflows/`.
- Never edit a file synced from the hub, or text inside a `# >>> managed-file-sync:<service> >>>`
  block; the next sync run overwrites it. Edit the hub source instead.
- Never push directly to `main`; land changes through a pull request so the security gate runs.
- Never weaken or disable a check to get a green run, and never assume a sibling repo picks up a
  change here when that repo already ships its own copy of the file.
