# blackoutsecure .github

Organization-wide community-health defaults for the blackoutsecure
GitHub organization. Files here are inherited automatically by every
repo in the org that does NOT supply its own version.

## What lives here

| File / folder                          | Purpose                                          |
|----------------------------------------|--------------------------------------------------|
| `CODE_OF_CONDUCT.md`                   | Default code of conduct                          |
| `CONTRIBUTING.md`                      | Default contribution guidance                    |
| `SECURITY.md`                          | Default security policy + reporting flow         |
| `SUPPORT.md`                           | Default support guidance                         |
| `FUNDING.yml`                          | Default GitHub Sponsors / funding links          |
| `.github/ISSUE_TEMPLATE/`              | Default bug + feature-request templates + config |
| `.github/PULL_REQUEST_TEMPLATE.md`     | Default PR template                              |
| `profile/README.md`                    | Org profile page (rendered on the org landing)   |

These files are intentionally generic so they apply across the board
— for public repositories, published GitHub Marketplace actions, and
internal-leaning public repos alike. Per-repo specifics (release
process, tool surface, branching strategy, etc.) belong in each
repo's own `README.md` or a repo-local override of the file in
question.

## How GitHub's inheritance works

GitHub applies a community-health file from this `.github` repo to a
sibling repo when **all** of the following are true:

1. The repo does NOT define its own copy of the file (either at the
   repo root or under its own `.github/` folder).
2. The file is one of the inheritable types in the table above.

If a repo defines its own file, GitHub uses that one verbatim — the
org default is ignored for that repo (no merging).

### What does NOT inherit

These are per-repo by design and must live inside the consuming repo
itself:

- `.github/workflows/**` — workflows do not inherit.
- `.github/dependabot.yml` — Dependabot config is per-repo.
- `.github/CODEOWNERS` — code-owner rules are per-repo.
- `LICENSE`, `NOTICE`, repo `README.md` — per-repo.
- Branch protection, repo settings, secrets — per-repo / org config.

## Hygiene for this repo itself

This repo also ships its own dev-hygiene configuration so it
dogfoods the standards the
[`bos-marketplace-kit`](https://github.com/blackoutsecure/bos-marketplace-kit)
hygiene rules (`DP001`, `LT001`–`LT005`) recommend for consumers.
These files apply only to THIS repo — they are NOT inherited by
sibling repos (GitHub's inheritance contract above covers only the
community-health surface):

| File                          | Purpose                                                    |
|-------------------------------|------------------------------------------------------------|
| `.github/dependabot.yml`      | Weekly bumps for any future `github-actions` workflows     |
| `.github/CODEOWNERS`          | Required maintainer review on community-health changes     |
| `.editorconfig`               | Indentation / encoding / EOL defaults                      |
| `.gitattributes`              | Force LF line endings + tag common binary types            |
| `.gitignore`                  | Ignore editor noise, OS noise, env files, private keys     |
| `.markdownlint.yaml`          | Markdown lint settings tuned for community-health docs     |
| `.yamllint.yml`               | YAML lint settings tuned for community-health + workflows  |

## Conventions for repos in this org

- Prefer the inherited defaults. Override locally only when a repo
  genuinely needs a different policy (e.g. a CONTRIBUTING.md that
  documents a repo-specific release flow).
- Never commit secrets, internal URLs, customer data, or PII to any
  file in this repo or to the override files in any public repo.
- For security reports, route through the per-repo Security Advisory
  workflow when available; otherwise fall back to the policy here.

## Links

- Website: <https://blackoutsecure.app>
- Sponsors: <https://github.com/sponsors/blackoutsecure>
- Security policy: [SECURITY.md](SECURITY.md)
- Support: [SUPPORT.md](SUPPORT.md)
- Contributing: [CONTRIBUTING.md](CONTRIBUTING.md)
- Code of Conduct: [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
