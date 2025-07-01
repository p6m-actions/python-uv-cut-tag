# Python UV Cut Tag

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/python-uv-cut-tag?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action for cutting versioned tags for Python projects using UV. This action will:

1. Bump the version in `pyproject.toml` using UV's native version management
2. Create a Git tag using the format "vMajor.Minor.Patch"
3. Push the new version and tag to the repository

This action uses UV's `uv version` command to handle version bumping and assumes your project uses a standard `pyproject.toml` file for version management.

## Usage

Add this action to your workflow after checking out your code:

```yaml
- name: Cut Tag
  uses: p6m-actions/python-uv-cut-tag@v1
  with:
    version-level: "patch" # Options: patch, minor, major
```

## Inputs

| Input | Description | Required | Default |
| ----- | ----------- | -------- | ------- |
| `version-level` | Version level to bump (patch, minor, or major) | No | `patch` |
| `pyproject-path` | Path to pyproject.toml file | No | `pyproject.toml` |

## Outputs

| Output | Description |
| ------ | ----------- |
| `version` | The new version number |
| `tag` | The new tag that was created |

## Examples

### Example 1: Build and Release Workflow

This example shows how to use this action as part of a workflow that builds a project and then cuts a new tag:

```yaml
name: Build and Release

on:
  push:
    branches: [main]

permissions:
  contents: write # Required for pushing tags

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup UV
        uses: p6m-actions/python-uv-setup@v1
        with:
          python-version: "3.12"

      - name: Install Dependencies
        run: uv sync

      - name: Run Tests
        run: uv run pytest

      - name: Cut Tag
        uses: p6m-actions/python-uv-cut-tag@v1
        with:
          version-level: "patch"
```

### Example 2: Complete UV Project Setup and Release

This example shows how to set up a complete UV project workflow with testing and release:

```yaml
name: Test and Release

on:
  workflow_dispatch:
    inputs:
      version-level:
        description: "Version level to bump"
        required: true
        default: "patch"
        type: choice
        options:
          - patch
          - minor
          - major

permissions:
  contents: write # Required for pushing tags

jobs:
  test-and-release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Setup UV with Python and caching
      - name: Setup UV
        uses: p6m-actions/python-uv-setup@v1
        with:
          python-version: "3.12"
          version: "latest"

      # Install project dependencies
      - name: Install Dependencies
        run: uv sync --all-extras --dev

      # Run linting
      - name: Run Linting
        run: uv run ruff check .

      # Run type checking
      - name: Run Type Checking
        run: uv run mypy .

      # Run tests
      - name: Run Tests
        run: uv run pytest --cov=. --cov-report=xml

      # Only cut tag if all tests pass
      - name: Cut Tag
        id: cut-tag
        uses: p6m-actions/python-uv-cut-tag@v1
        with:
          version-level: ${{ github.event.inputs.version-level }}

      - name: Output Version Info
        run: |
          echo "New version: ${{ steps.cut-tag.outputs.version }}"
          echo "Created tag: ${{ steps.cut-tag.outputs.tag }}"
```

### Example 3: Simple Manual Release

This example shows a minimal workflow for manually releasing a new version:

```yaml
name: Release New Version

on:
  workflow_dispatch:
    inputs:
      version-level:
        description: "Version level to bump"
        required: true
        default: "patch"
        type: choice
        options:
          - patch
          - minor
          - major

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Cut Tag
        uses: p6m-actions/python-uv-cut-tag@v1
        with:
          version-level: ${{ github.event.inputs.version-level }}
```

### Custom pyproject.toml Path

```yaml
- name: Cut Tag
  uses: p6m-actions/python-uv-cut-tag@v1
  with:
    version-level: "minor"
    pyproject-path: "src/pyproject.toml"
```

## Prerequisites

Your repository must have a `pyproject.toml` file with a version field. The version can be specified in either format:

**Standard format:**
```toml
[project]
name = "my-package"
version = "1.0.0"
```

**Alternative format:**
```toml
[tool.setuptools]
name = "my-package"
version = "1.0.0"
```

## Required Permissions

This action requires write permissions to the repository contents to push commits and tags. Make sure to include the following permissions in your workflow:

```yaml
permissions:
  contents: write
```

## How It Works

1. **UV Installation**: UV is automatically installed if not already available
2. **Version Reading**: Uses `uv version` to get the current project version from pyproject.toml
3. **Version Bumping**: Uses `uv version --bump [level]` to increment the version:
   - `patch`: Increments the patch version (1.0.0 → 1.0.1)
   - `minor`: Increments the minor version and resets patch (1.0.1 → 1.1.0)
   - `major`: Increments the major version and resets minor and patch (1.1.0 → 2.0.0)
4. **File Update**: UV automatically updates the pyproject.toml file with the new version
5. **Git Operations**: Commits the version change and creates a new tag (e.g., `v1.0.1`)
6. **Push**: Pushes both the commit and the new tag to the repository