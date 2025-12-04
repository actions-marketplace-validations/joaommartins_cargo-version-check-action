# Cargo Version Check Action

A GitHub Action to check if the version in `Cargo.toml` has changed compared to a base branch or previous commit. Useful for triggering release workflows only when the version number is bumped.

## Features

- Compares current `Cargo.toml` version against a base branch or previous commit
- Outputs boolean flag for conditional workflow steps
- Returns both current and previous versions
- Supports custom `Cargo.toml` paths (e.g., for workspaces)
- Works with pull requests and manual triggers
- Provides helpful notices and error messages

## Usage

### Basic Usage (Pull Request)

```yaml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  check-version:
    runs-on: ubuntu-latest
    outputs:
      version-changed: ${{ steps.version-check.outputs.version-changed }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required for git history comparison

      - name: Check if version changed
        id: version-check
        uses: joaommartins/cargo-version-check-action@v1
        with:
          base-ref: ${{ github.base_ref }}

      - name: Show version info
        run: |
          echo "Version changed: ${{ steps.version-check.outputs.version-changed }}"
          echo "Current: ${{ steps.version-check.outputs.current-version }}"
          echo "Previous: ${{ steps.version-check.outputs.previous-version }}"

  build-if-version-changed:
    needs: check-version
    if: needs.check-version.outputs.version-changed == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build release
        run: cargo build --release
```

### Workspace with Custom Cargo.toml Path

```yaml
- name: Check if version changed
  id: version-check
  uses: joaommartins/cargo-version-check-action@v1
  with:
    base-ref: main
    cargo-toml-path: 'crates/my-crate/Cargo.toml'
```

### Compare Against Previous Commit

If you don't specify a `base-ref`, the action will compare against the previous commit (`HEAD~1`):

```yaml
- name: Check if version changed
  id: version-check
  uses: joaommartins/cargo-version-check-action@v1
  # No base-ref specified - compares with previous commit
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `base-ref` | Base branch reference for comparison (e.g., `main`). If not provided, compares against previous commit. | No | `''` |
| `cargo-toml-path` | Path to `Cargo.toml` file relative to repository root | No | `Cargo.toml` |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `version-changed` | Whether the version has changed (`true` or `false`) | `true` |
| `current-version` | Current version in Cargo.toml | `0.2.0` |
| `previous-version` | Previous version in Cargo.toml | `0.1.0` |

## Common Patterns

### Conditional Release Builds

Only build release binaries/wheels when version changes:

```yaml
jobs:
  check-version:
    runs-on: ubuntu-latest
    outputs:
      version-changed: ${{ steps.check.outputs.version-changed }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - id: check
        uses: joaommartins/cargo-version-check-action@v1
        with:
          base-ref: ${{ github.base_ref }}

  release-build:
    needs: check-version
    if: needs.check-version.outputs.version-changed == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cargo build --release
      # ... upload artifacts, publish, etc.
```

### Manual Workflow Dispatch

```yaml
on:
  workflow_dispatch:
  pull_request:

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: joaommartins/cargo-version-check-action@v1
        with:
          # For PRs, uses github.base_ref
          # For manual dispatch, compares with previous commit
          base-ref: ${{ github.base_ref }}
```

## Requirements

- Your workflow must check out the repository with `fetch-depth: 0` to access git history
- The action requires `bash` shell (available on all GitHub-hosted runners)

## Error Handling

The action will fail with clear error messages if:

- `Cargo.toml` is not found at the specified path
- Version cannot be extracted from `Cargo.toml`

It will provide warnings if:

- The specified base ref doesn't exist (falls back to previous commit comparison)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - see LICENSE file for details

## Author

Created by [@joaommartins](https://github.com/joaommartins)
