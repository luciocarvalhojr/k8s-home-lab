# Pre-commit Hooks

This repository uses [pre-commit](https://pre-commit.com/) to catch issues locally — before they reach CI or the cluster.

The same checks that run in GitHub Actions (lint, secret detection, K8s validation) run automatically on every `git commit`.

---

## Installation

```sh
brew install pre-commit   # macOS
# or: pip install pre-commit

# Install the git hook (run once per clone)
pre-commit install
```

After `pre-commit install`, the hooks run automatically on every `git commit`. You don't need to remember to run them.

---

## What Runs on Every Commit

| Hook | Tool | What it catches |
| ---- | ---- | --------------- |
| Trailing whitespace | `pre-commit-hooks` | Stray spaces at end of lines |
| End-of-file fixer | `pre-commit-hooks` | Missing newline at EOF |
| Merge conflict markers | `pre-commit-hooks` | Accidentally committed `<<<<<<` markers |
| Large files | `pre-commit-hooks` | Files > 500 KB |
| Private key detection | `pre-commit-hooks` | PEM/SSH private keys |
| Block direct push to `main` | `pre-commit-hooks` | Prevents committing directly to main branch |
| YAML lint | `yamllint` | Syntax errors and style issues (config: `.yamllint.yaml`) |
| Secret leak detection | `gitleaks` | API keys, tokens, passwords in staged files |
| K8s manifest validation | `kubeconform` | Invalid resources, wrong API versions |
| IaC security scan | `checkov` | Security misconfigurations in K8s manifests |

---

## Common Workflows

### Run against all files (first time, or after updating hooks)

```sh
pre-commit run --all-files
```

### Run a specific hook only

```sh
pre-commit run gitleaks
pre-commit run yamllint
pre-commit run kubeconform
```

### Skip hooks for a single commit (emergency only)

```sh
git commit --no-verify -m "your message"
```

> Use sparingly. Skipped commits still go through GitHub Actions CI.

### Update hooks to their latest versions

```sh
pre-commit autoupdate
git add .pre-commit-config.yaml
git commit -m "chore: update pre-commit hook versions"
```

---

## Troubleshooting

### `kubeconform` fails on non-K8s YAML files (workflows, config files)

Files like `.github/workflows/*.yaml`, `.pre-commit-config.yaml`, and `.yamllint.yaml` are valid YAML but not K8s manifests. Add them to the `exclude` pattern in `.pre-commit-config.yaml`:

```yaml
exclude: >-
  (?x)(
    \.github/|
    \.pre-commit-config\.yaml$|
    \.yamllint\.yaml$|
    \.kube-linter\.yaml$|
    values\.yaml$|
    ...
  )
```



### `gitleaks` flags a sealed secret value

Sealed secret values (the long `AgB...` strings) are encrypted ciphertext — not real secrets. If gitleaks false-positives on them, add a `.gitleaks.toml` at the repo root:

```toml
[allowlist]
  description = "Sealed secret ciphertext is not a real secret"
  regexes = ['''AgB[A-Za-z0-9+/=]{20,}''']
```

### `kubeconform` fails on a CRD (unknown schema)

CRDs not in the [datreeio catalog](https://github.com/datreeio/CRDs-catalog) will be skipped (`-ignore-missing-schemas`). If you need strict validation for a custom CRD, add its schema URL to the `-schema-location` list in `.pre-commit-config.yaml`.

### `checkov` reports a false positive

Find the check ID in the output (e.g. `CKV_K8S_14`) and add it to the `--skip-check` list in `.pre-commit-config.yaml`:

```yaml
args:
  - --skip-check=CKV_K8S_8,CKV_K8S_9,CKV_K8S_14
```

---

## Reusing This Pattern in Other Projects

Copy these three files to any k8s GitOps repository:

```text
.pre-commit-config.yaml   ← hook definitions
.yamllint.yaml            ← YAML lint rules
.kube-linter.yaml         ← kube-linter check exclusions
```

Then run:

```sh
pre-commit install
pre-commit autoupdate   # pin hooks to latest versions
```

Adjust the `exclude` patterns in `.pre-commit-config.yaml` to match the new repo's directory layout.
