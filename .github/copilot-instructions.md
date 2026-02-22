# GitHub Copilot - Organization Instructions

## Purpose
This document defines how GitHub Copilot should assist contributors across all repositories in the
blackoutsecure organization. It ensures consistent code quality, security posture, and documentation
standards.

---

## 1. Coding Style & Conventions

### General
- Prefer clear, explicit code over clever or compact code.
- Follow language-specific style guides:
  - Python: PEP 8
  - JavaScript/TypeScript: ESLint + Prettier defaults
  - Go: gofmt + idiomatic Go patterns
  - Shell: POSIX-compliant where possible

### Comments & Documentation
- Generate docstrings for all public functions, classes, and modules.
- Use descriptive variable names.
- Include inline comments when logic is non-obvious.

---

## 2. Security Requirements

### Always:
- Validate inputs.
- Avoid hard-coded secrets, tokens, or credentials.
- Use environment variables for configuration.
- Recommend secure defaults (HTTPS, TLS, encryption at rest, etc.).
- Follow least-privilege principles.

### Never:
- Suggest insecure patterns (e.g., `eval`, weak crypto, unsanitized SQL).
- Generate example secrets or placeholder passwords.

---

## 3. GitHub Actions & CI/CD

When Copilot generates workflows:
- Use pinned action versions (e.g., `actions/checkout@v4`).
- Avoid untrusted third-party actions unless explicitly approved.
- Include security scanning steps when relevant (CodeQL, Trivy, etc.).
- Prefer reusable workflows from `.github/workflow-templates/` when available.

---

## 4. Repository Structure Guidance

Copilot should:
- Suggest using the organization's standard layout for:
  - `README.md`
  - `SECURITY.md`
  - `CONTRIBUTING.md`
  - Issue templates
  - PR templates
- Encourage modular, testable code organization.

---

## 5. Testing Expectations

Copilot should:
- Always generate tests when creating new modules or functions.
- Use the organization's preferred frameworks:
  - Python: pytest
  - JavaScript/TypeScript: Jest
  - Go: built-in `testing` package
- Include meaningful test cases, not trivial ones.

---

## 6. Documentation & Markdown

Copilot should:
- Produce clear, concise documentation.
- Use consistent Markdown formatting.
- Include examples when generating API docs or usage guides.

---

## 7. Communication Style

Copilot should:
- Provide helpful, context-aware suggestions.
- Prefer clarity over verbosity.
- Avoid generating speculative or incorrect information.
- When unsure, suggest asking for clarification rather than guessing.

---

## 8. Project-Specific Rules

If a repository contains a `.copilot` or `.github/copilot.json` file, those rules override this document for that repo.

---

## 9. Licensing & Attribution

Copilot must:
- Avoid generating copyrighted code unless it is:
  - Public domain
  - Under a compatible open-source license
  - Already present in the repository
- Prefer generating original code.

---

## 10. Security-First Mindset

This organization prioritizes security. Copilot should:
- Default to secure patterns.
- Highlight potential risks in generated code.
- Prefer dependency-minimal solutions.
- Suggest scanning tools or secure configurations when appropriate.

---

# End of Copilot Instructions
