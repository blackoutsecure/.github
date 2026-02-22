# Contributing

Thanks for helping improve the blackoutsecure organization repositories.

## Before You Start

- Search existing issues and pull requests.
- For security issues, follow the process in SECURITY.md.

## Scope of Changes

- Keep changes focused and easy to review.
- Prefer explicit, readable implementations over clever shortcuts.
- Avoid introducing new dependencies unless necessary.

## Development Standards

- Keep changes scoped and explicit.
- Use clear names for functions, variables, and files.
- Validate inputs and avoid hard-coded secrets.
- Use secure defaults (HTTPS, TLS, encryption at rest where applicable).

## Documentation

- Update README or other docs when behavior changes.
- Add docstrings for public modules, classes, and functions.
- Include examples for new APIs or usage guidance.

## Tests

- Add or update tests for new or changed behavior.
- Use the preferred framework for the language:
  - Python: pytest
  - JavaScript/TypeScript: Jest
  - Go: testing

## Security Review

- Call out security-sensitive changes in the PR description.
- Avoid logging secrets or sensitive payloads.
- Redact or anonymize data in examples and logs.

## Pull Requests

- Describe the problem and the solution.
- Include steps to test.
- Note security considerations and potential risks.

## Code Style

Follow language-specific style guides and formatting tools used in each repository.
