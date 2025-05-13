# Best Practices for Engineering Contribution Cycle

This document outlines the recommended best practices for the engineering teams at Dentscape. It covers the entire contribution cycle, focusing on consistency, maintainability, operational ease, and modularity.

## Table of Contents

- [Consistency of Development](#consistency-of-development)
  - [IDE / Editor Setup](#ide-editor-setup)
  - [Pre-commit Hooks (Linters, Formatters, Type Checking, Pytest)](#pre-commit-hooks-linters-formatters-type-checking-pytest)
  - [GitHub Actions (Lint & Test)](#github-actions-lint-test)
  - [Git Flow (Branching Strategy)](#git-flow-branching-strategy)
- [Maintainability](#maintainability)
  - [README](#readme)
  - [Reference Prompt for README Generation](#reference-prompt-for-readme-generation)
  - [Commit Message Template (.gitmessage.txt)](#commit-message-template-gitmessagetxt)
  - [Commit Message Guidelines](#commit-message-guidelines)
- [Ease of Operations](#ease-of-operations)
  - [UV (Python Environment)](#uv-python-environment)
  - [Dockerfile](#dockerfile)
  - [GitHub Actions (Build & Deploy)](#github-actions-build-deploy)
- [Modularity](#modularity)
  - [Polyrepo Approach](#polyrepo-approach)

## Consistency of Development

### IDE / Editor Setup

- Editor choice is up to you. Popular options include:
  - VSCode — extensive plugin ecosystem.
  - PyCharm — rich Python-specific support.
  - Cursor — explore rules at [`PatrickJS/awesome-cursorrules`](https://github.com/PatrickJS/awesome-cursorrules)
  - Neovim — example config at [`stmharry/stmharry-config:astronvim/nvim`](https://github.com/stmharry/stmharry-config/tree/main/astronvim/nvim)
- Recommended features (via plugins/extensions):
  - Linting (e.g., flake8, pylint) for code quality.
  - Formatting (e.g., black, isort) for consistent style.
  - AI-powered code assistance (e.g., GitHub Copilot, OpenAI Codex).
- Note: actual enforcement of formatting, type checking, and local testing is handled by pre-commit hooks. Development-phase tooling is flexible.

### Pre-commit Hooks (Linters, Formatters, Type Checking, Pytest)

Pre-commit hooks ensure code quality and consistency before each commit.

#### Example `.pre-commit-config.yaml`:

```yaml
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.6.0
  hooks:
    - id: end-of-file-fixer
    - id: trailing-whitespace

- repo: https://github.com/myint/autoflake
  rev: v2.3.1
  hooks:
    - id: autoflake
      exclude: .*/__init__.py
      args:
        - --in-place
        - --remove-all-unused-imports
        - --expand-star-imports
        - --remove-duplicate-keys
        - --remove-unused-variables

- repo: https://github.com/pre-commit/mirrors-isort
  rev: v5.10.1
  hooks:
    - id: isort
      args:
        - --profile
        - black

- repo: https://github.com/psf/black
  rev: 25.1.0
  hooks:
    - id: black
      args:
        - --preview
        - --enable-unstable-feature=string_processing
        - --enable-unstable-feature=hug_parens_with_braces_and_square_brackets
        - --line-length=88

- repo: https://github.com/pre-commit/mirrors-mypy
  rev: v1.14.1
  hooks:
    - id: mypy
      language: system
      types_or:
        - python
        - pyi
      args:
        - --ignore-missing-imports

- repo: local
  hooks:
    - id: pytest
      name: pytest
      entry: pytest
      language: system
      types: [python]
```

Brief explanations:
- **pre-commit-hooks**: basic file cleanup (EOF fixes, whitespace).
- **autoflake**: remove unused imports and variables.
- **isort**: sort imports (Black profile).
- **black**: enforce code formatting.
- **mypy**: static type checking.
- **pytest**: run tests before commit.

### GitHub Actions (Lint & Test)

- Use GitHub Actions to run pre-commit hooks and pytest on each push or pull request. This provides remote enforcement of linting and testing, but may be overkill at our current scale.

### Git Flow (Branching Strategy)

- Branch naming convention using `<type>/<description>`:
  - `dev/<name>` for general development.
  - `feature/<feature-name>` for new features.
  - `fix/<issue>` for bug fixes.
  - `refactor/<area>` for code refactoring.
  - `chore/<task>` for maintenance tasks.
  - `release/<version>` to prepare a release.
- Use `main` branch for stable releases.
- After PR approval and passing checks, merge into `main`.
- Tag release commits on `main` with semantic versioning (e.g., `v1.0.0`, `v1.2.3`).

## Maintainability

### README

- Include project overview, installation instructions, usage examples, and contribution guidelines.
- Use AI tools to draft and iterate on documentation.
- Example: [`AIDirect/py-clr:README.md`](https://github.com/AIDirect/py-clr/blob/main/README.md)

#### Reference Prompt for README Generation

Use the following prompt with OpenAI Codex to generate a README file:

```md
You are an AI assistant with full access to the codebase at the repository root. Your task is to generate a complete, well-structured README.md file that follows open-source best practices. The README might include the following sections, but they are not all mandatory. Use your judgment to include only the most relevant sections based on the codebase content.

1. **Project Title and Badges**
2. **One-Sentence Tagline**
3. **Overview / Description** – What the project does and why it exists.
4. **Table of Contents** with links to each major section.
5. **Installation** – Step-by-step instructions for setup, including prerequisites and commands.
6. **Usage** – Examples and common workflows, with code snippets.
7. **Configuration** – Any environment variables or settings.
8. **API / CLI Reference** – Key commands, endpoints, or functions.
9. **Examples** – Minimal working examples or links to demos.
10. **Testing** – How to run the test suite.
11. **Contributing** – Guidelines for submitting issues and pull requests, code style, and branch strategy.
12. **Roadmap** – Planned features or future work.
13. **Authors and Acknowledgments**
14. **License** – SPDX identifier and link to the full license text.
15. **Contact / Support** – How to get help or report bugs.

Write the README in Markdown, with clear section headings, concise text, and fenced code blocks where appropriate. Ensure all links and examples are syntactically correct.
```

### Commit Message Template (`.gitmessage.txt`)

Store a commit message template in `.gitmessage.txt`. Example:
```txt
# Gitmoji Commit Message Template
# <emoji> [(scope)]: <subject>

# ✨ feat       Introduce new features
# 🐛 fix        Fix a bug
# 📝 docs       Documentation only changes
# 💄 style      Code style changes (formatting, whitespace)
# ♻️ refactor   Code refactoring without behavior change
# ⚡️ perf       Improve performance
# ✅ test       Add or update tests
# 🔧 build      Changes to build system or dependencies
# 👷 ci         CI configuration and scripts
# 🔒 chore      Other changes not affecting src or tests
# ⏪️ revert     Revert a previous commit

# Body (optional)
# - Explain *what* and *why*, not *how*
# - Wrap at 72 characters

# Footer (optional)
# - Issue tracker ID (e.g., JIRA-1234)
# - Co-authored-by: Name <email@example.com>
```

### Commit Message Guidelines

- Use Gitmoji-style commit messages.
- Subject should be concise and imperative (≤ 50 characters).
- Optionally use AI (e.g., Copilot Chat). See [COMMIT_MESSAGE_PROMPT.md](COMMIT_MESSAGE_PROMPT.md) for the full prompt.

#### Reference Prompt for Commit Message Generation

```markdown
# Gitmoji-Style Commit Message Generator Prompt

You are an AI assistant that writes Gitmoji-style commit messages.

---

## Input
<REPO_ROOT>
<git diff --staged or unified diff output>

## Task
1. Analyze the diff and identify the **primary change type** (e.g., feature, fix, docs, style, refactor, perf, test, build, ci, chore, revert).
2. Select the appropriate **Gitmoji** for that change:
   - ✨ `feat` — new feature
   - 🐛 `fix` — bug fix
   - 📝 `docs` — documentation only
   - 💄 `style` — formatting, whitespace, etc.
   - ♻️ `refactor` — code refactoring
   - ⚡️ `perf` — performance improvement
   - ✅ `test` — tests added or updated
   - 🔧 `build` — build system or dependencies
   - 👷 `ci` — CI configuration
   - 🔒 `chore` — other changes not affecting src or tests
   - ⏪️ `revert` — revert a previous commit
3. Optionally infer a **scope** from file paths or module names.
4. Craft a **concise, imperative** subject line (≤ 50 characters).
5. If needed, write an optional **body** (wrapped at 72 characters) explaining *what* and *why*.
6. If applicable, include an optional **footer** for issue IDs or co-authors.

## Output Format
<emoji> [(scope)]: <imperative subject>

[optional body]

[optional footer]

## Example
**Input:**
```diff
diff --git a/src/parser.js b/src/parser.js
index 3e5e1f0..4b8c7a2 100644
--- a/src/parser.js
+++ b/src/parser.js
@@ -10,6 +10,10 @@ function parse(input) {
     // existing code
}
+
+// Added JSON support
+export function parseJSON(input) {
+  return JSON.parse(input);
}

**Expected Output:**
✨ (parser): add support for JSON parsing
```

## Ease of Operations

### UV (Python Environment)

- Use [UV Package Manager](https://github.com/indygreg/uv) to initialize projects, manage Python versions, create virtualenvs, and handle dependencies in `pyproject.toml`.
- Example: [`AIDirect/flask-server:pyproject.toml`](https://github.com/AIDirect/flask-server/blob/main/pyproject.toml)

### Dockerfile

- Write a `Dockerfile` for deployment or handoff to operations.
- Use UV inside the Dockerfile for environment and dependencies.

### GitHub Actions (Build & Deploy)

- Configure workflows to build and publish Docker images on tag events.

#### Example GitHub Actions Workflow

```yaml
on:
  push:
    tags:
      - 'v*.*.*'
jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Login to registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}/
```

Brief explanations:
- `on`: triggers the workflow on tag pushes.
- `jobs`: defines the build-and-push job.
  - `actions/checkout`: checks out the code.
  - `actions/setup-python`: sets up the Python environment.
  - `docker/login-action`: logs into the Docker registry.
  - `docker/build-push-action`: builds and pushes the Docker image to the registry.

## Modularity

### Polyrepo Approach

- Maintain separate repositories for core libraries and services:
  - [`AIDirect/dentscape-core`](https://github.com/AIDirect/dentscape-core): Mesh operations, AI model inference, Apache Beam integration.
  - [`AIDirect/py-clr`](https://github.com/AIDirect/py-clr): Python bindings for 3shape DLLs.
- Benefits: separation of concerns, independent versioning, easier dependency management.
