# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## What this is

`blackoutsecure/.github` is GitHub's organization-level special repository. GitHub treats a repo
literally named `.github` as the org's default provider for three unrelated features: community
health files inherited by sibling repos, the org profile page rendered from `profile/README.md`,
and starter workflows offered under Actions -> New workflow from `workflow-templates/`. There is
no application code here, and none should be added.

The content is Markdown, YAML, and JSON: policy documents, issue and pull-request templates, lint
configuration, the canonical Gatekeeper receiver, legacy security/sync callers, one locally-authored
drift check, and one starter workflow template. It also dogfoods the hygiene baseline `bos-marketplace-kit` recommends to
consumers (`DP001`, `LT001`-`LT005`); that config applies to this repo only and is inherited by
nothing. Most policy text is sourced from `bos-automation-hub/sync-files/` through
`bos-managed-file-sync-action`; generic dotfile services also come from that action's bundled
catalogue. Check [Source of truth](#source-of-truth) before editing anything.

## Commands

There is no `package.json`, `pyproject.toml`, `Makefile`, or lockfile here, so no local package
manager or standalone build/test runner. Workflows contain automation glue; local validation
invokes the linters directly.

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

- `.github/workflows/bos-universal-gatekeeper-kicker.yml` is the installed canonical managed
  receiver. It runs on pushes to `main` and `dev`, schedules, and manual dispatch, but not on
  pull requests. Authorization precedes its managed sync and routed operations. Use this
  authorized front door for new manual operations.
- `.github/workflows/bos-universal-security-kicker.yml` is a legacy hub caller. It runs on
  pushes and pull requests targeting both `main` and `dev`, with the workflow's documented
  push exclusions, plus `merge_group`, daily cron, and dispatch. It calls the hub's
  `bos-universal-sync.yml` in `commit` mode, resolves a hub ref, then calls the hub's reusable
  `bos-universal-security.yml` with `config_authoritative: true`. That is the single required
  gate for main-targeted changes: `security (main) / Security summary`. It aggregates applicable
  linters, `bos-code-scanning-kit`, dependency review, README-header and PR-title checks.
  Repository-configured CodeQL also appears in PR checks; it is not a job in that aggregate.
- `.github/workflows/bos-universal-sync-kicker.yml` is another legacy caller. It reconciles files on a
  weekly cron, on a push touching `.github/bos-universal-config.json`, or on dispatch with
  `mode: commit|check`.
- `.github/workflows/check-kicker-template-sync.yml` (authored here) fetches
  `blackoutsecure/bos-workflow-gatekeeper@main:examples/kicker.yml` and fails when the `on`,
  `permissions`, or `jobs` sections of `workflow-templates/bos-workflow-gatekeeper-kicker.yml`
  differ. Header comments may differ; functional shape may not. This is a separate online
  comparison, not the required Security summary.

Locally, narrowest first: lint the file you touched, then the whole file type, then the JSON parse
checks if a config changed, then `actionlint` if a workflow or template changed. If you edited the
starter template, verify the upstream ref/path configured by the comparison workflow before
comparing functional sections. An unavailable upstream or HTTP 404 is a failed comparison, not
evidence of a matching template. Do not weaken the check to hide that failure.

## Architecture

```text
README.md                     This repo's own docs: inheritance contract, template rules.
AGENTS.md                     Locally authored guidance for this repository.
CODE_OF_CONDUCT.md CONTRIBUTING.md SECURITY.md SUPPORT.md   Org defaults. Inherited org-wide.
LICENSE                       Apache-2.0, distributed by the hub's `license_service`.
profile/README.md             Rendered at https://github.com/blackoutsecure as the org profile.
bos-universal-config.json     Legacy root config; not the active managed caller's config path.
bos-launchpad-config.json     Historical Launchpad service configuration.
workflow-templates/           Starter workflows offered under Actions -> New workflow.
.github/bos-universal-config.json   The config the sync engine discovers first.
.github/CODEOWNERS            Review owners for this repo only; CODEOWNERS never inherits.
.github/dependabot.yml        Managed updater blocks; includes a legacy pip entry despite no Python manifest.
.github/FUNDING.yml PULL_REQUEST_TEMPLATE.md ISSUE_TEMPLATE/   Org defaults. Inherited org-wide.
.github/workflows/            Canonical Gatekeeper, two legacy callers, and the template drift check.
.editorconfig .gitattributes .gitignore .markdownlint.yaml .yamllint.yml .shellcheckrc
                              Per-repo hygiene from managed services; Markdown config is whole-file managed.
.vscode/extensions.json       Recommended editor extensions.
```

### Branch model

`main` is the current default and long-lived branch. Create a feature branch from it and land
changes through a pull request with the required checks and review; do not bypass them.
This repository does not use the code repositories' `dev` -> `main` runtime promotion flow.
Workflow declarations may support both branch names even though this repo currently uses `main`.
Do not infer repository architecture from temporary or pending PR branches.

### Inheritance semantics

This `.github` repository must remain public for GitHub's default community-health behavior.
For supported file types with multiple valid locations, GitHub searches the consumer's `.github/`
directory, then its root, then `docs/`, before falling back to this repository. A repository-local
file overrides the corresponding org default rather than merging with it.

A valid local `.github/ISSUE_TEMPLATE/` template or configuration overrides the default issue
template directory as a whole; do not assume missing individual templates will be inherited.
Defaults are displayed by GitHub, not copied into consumers' clones or Git history.
See [GitHub's default community-health documentation](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file).

Inheritable here: `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`,
`SECURITY.md`, `SUPPORT.md`, `.github/FUNDING.yml`, `.github/PULL_REQUEST_TEMPLATE.md`, and
`.github/ISSUE_TEMPLATE/`. These never inherit and must live in the consuming repo:
`.github/workflows/**`, `.github/dependabot.yml`, `.github/CODEOWNERS`, `AGENTS.md`, `LICENSE`, `NOTICE`, the
repo `README.md`, and all repo settings, branch protection, and secrets. The hygiene config here
is likewise local-only.

### Workflow templates

`workflow-templates/` is a separate feature from inheritance and copies nothing automatically.
Each `<name>.yml` needs a matching `<name>.properties.json`; the pair appears as a suggested
starter under Actions -> New workflow for every repo in the org. A maintainer must pick it, and it
is then written into that repo's `.github/workflows/` as an ordinary independent file with no
ongoing link back here. `iconName` may name a local SVG without its extension
(`example-icon` means `workflow-templates/example-icon.svg`), or an Octicon such as
`octicon smiley`, which needs no local SVG. A custom icon's name does not have to match
the workflow template's name. See [GitHub's template metadata documentation](https://docs.github.com/en/actions/how-tos/reuse-automations/create-workflow-templates).

- `bos-workflow-gatekeeper-kicker` — a `workflow_dispatch` front door that authorizes the
  triggering actor with the published `bos-workflow-gatekeeper` action, optionally narrows the
  requested operation against `.github/kicker-policy.json`, and routes to a backend
  `workflow_call` workflow. It is a relayed copy of that repo's `examples/kicker.yml`; the hub's
  managed-file sync flows hub -> consumers and does not cover this shape, so
  `check-kicker-template-sync.yml` is the guardrail instead. Coordinate functional changes with
  the upstream source, and verify its current location rather than assuming the historical
  `@main:examples/kicker.yml` path remains available.

### Org configuration

`.github/bos-universal-config.json` is the repo-owned override layer the sync engine discovers
first. It sets `general.target_repo_role` to `org-default-repo` and enables `common`,
`lf_line_endings`, `bos_universal_gatekeeper_kicker`, and `org_defaults` in `commit` mode. Service
lists normally append across tiers with duplicates removed; `use_marketplace_services: false`
replaces inherited selections, while `disabled_services` removes services after resolution.
The currently promoted hub global config at
`bos-automation-hub/sync-files/config/managed-file-sync-global-config.json` selects `shellcheck`,
`yamllint`, `coverage_artifacts`, `license_service`, `security_readme_pointer`, and
`bos_universal_gatekeeper_kicker`. Inspect that file at the selected runtime ref when changing
policy; do not treat an old seed branch as the current global tier.

The Gatekeeper workflow is installed. The root `bos-universal-config.json` is a legacy
near-duplicate with disabled Launchpad stages, and `bos-launchpad-config.json` is historical.
Current callers explicitly read `.github/bos-universal-config.json`; editing a root legacy file
does not change their configuration.

Precedence is bundled Marketplace defaults, hub global config, repository config, then explicit
workflow inputs. Scalars replace earlier values; service definitions merge by name, with a
same-named definition replacing the inherited definition. Change repository policy in the
active config, not in a generated receiver. Do not add a Python manifest merely to satisfy the
legacy pip updater in this documentation-only repository.

### Source of truth

This is an ownership map, not a guarantee that every checkout is currently byte-identical to
its source. Whole-file services replace their targets; block services preserve surrounding
prose. Verify the applicable source and selected ref before editing.

Hub-owned payloads under `bos-automation-hub/sync-files/`:

- `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md`, and `.github/FUNDING.yml`
  from `community-health/`; `.github/PULL_REQUEST_TEMPLATE.md` and `.github/ISSUE_TEMPLATE/*` from
  `github-meta/`; `profile/README.md` from `org-profile/README.md`
- `LICENSE` from `legal/LICENSE` via `license_service`; local formatting edits will be replaced
- `.github/workflows/bos-universal-gatekeeper-kicker.yml` from `workflows/`
- the managed `security_readme_pointer` block in `README.md`

Generic dotfile services, including `common`, `lf_line_endings`, `markdownlint`, `shellcheck`, `yamllint`, and
Dependabot blocks, come from the published sync action's catalogue, with hub-level overrides and
patches where configured. Follow that catalogue to the owning source; do not assume every
generated block has a file under the hub's `sync-files/`. The bundled default service selection
includes `markdownlint`, which owns `.markdownlint.yaml` in whole-file mode even though it
has no managed-block markers.

The legacy security/sync callers retain hub-managed headers but are not the canonical receiver
selected by the current hub service list. Coordinate their maintenance or retirement with the
hub; do not use a legacy entrypoint to bypass the canonical authorization job.

Authored here: `AGENTS.md`, the non-managed prose in `README.md`, `.vscode/extensions.json`, `.github/CODEOWNERS`,
`.github/workflows/check-kicker-template-sync.yml`, everything under `workflow-templates/`, both
`bos-universal-config.json` files, `bos-launchpad-config.json`, and the prose outside the managed
blocks in `.editorconfig`, `.gitattributes`, `.gitignore`, and `.github/dependabot.yml`.

## Conventions

Markdown wraps at roughly 72 columns in the community health files, which is why `MD013` is off in
`.markdownlint.yaml`; `MD024` is `siblings_only`, and `MD028`, `MD033`, `MD034`, `MD041` are
relaxed for admonitions, inline HTML, bare URLs, and templates starting at `H2`. YAML is two-space
indented with LF endings, `document-start` disabled, `truthy` relaxed so a bare `on:` key does not
fail, and a 200-column soft limit — the kickers quote it as `"on":` to sidestep the YAML 1.1
boolean resolver. Remote step-level actions in active workflows are SHA-pinned with version
comments. Hub reusable-workflow calls deliberately use literal `@main`/`@dev` routing, while
checked-out local actions use relative paths; do not rewrite those managed references to satisfy
a blanket pinning rule. Starter templates may use readable `@v1` examples, which must be reviewed
and pinned as appropriate when adopted into an active consumer workflow.

The Marketplace README profile requires the canonical title/copyright/license badges. This
repository uses the generic profile instead, which still requires the `Made by BlackoutSecure`
badge in the first 30 lines. Internal documentation is not exempt from that check. Every
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

- Establish the owning service before editing a generated file. Change hub payloads at
  `bos-automation-hub/sync-files/`, or the published action's catalogue for its bundled services.
- Keep `.yml` and `.properties.json` paired for every template, and update the template and
  `bos-workflow-gatekeeper`'s `examples/kicker.yml` together.
- Run `yamllint`, `markdownlint-cli2`, `actionlint`, and the JSON parse checks on what you changed.
- Pin new remote step-level actions to a full commit SHA with a version comment; preserve
  intentional hub workflow branch refs and relative local-action calls.
- Keep policy text generic enough to apply org-wide; repo-specific process belongs in that repo.

### Ask first

- Changing a community health file that every repo in the org inherits. One edit to `SECURITY.md`,
  `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SUPPORT.md`, `FUNDING.yml`, the PR template, or the
  issue templates silently changes the published policy of every repo that has not overridden it.
- Changing `profile/README.md`. It is the public organization landing page.
- Changing a workflow template that repos have already copied. Copies are independent, so an edit
  here does not reach them and creates two divergent versions of the same starter.
- Changing the service list, `mode`, or `target_repo_role` in the active
  `.github/bos-universal-config.json`, or migrating/removing legacy root configuration.
- Adding a workflow to `.github/workflows/`, loosening the drift check's compared sections, or
  widening `.github/CODEOWNERS` so the inherited surface loses maintainer review.

### Never

- Never commit secrets, tokens, private keys, certificates, internal URLs, customer data, or PII.
- Never add application code, a backend, a build step, or a runtime dependency here. This repo is
  documentation and configuration only.
- Never introduce mutable refs for third-party actions in active workflows. Do not replace
  intentional hub-managed `@main`/`@dev` workflow refs or relative action paths.
- Never edit a file synced from the hub, or text inside a `# >>> managed-file-sync:<service> >>>`
  block; the next sync run overwrites it. Edit the hub source instead.
- Never push directly to `main`; land changes through a pull request so the security gate runs.
- Never weaken or disable a check to get a green run, and never assume a sibling repo picks up a
  change here when that repo already ships its own copy of the file.
- Never report a skipped check as passed, bypass required approvals, or use an unguarded legacy
  caller to evade the canonical authorization boundary.
