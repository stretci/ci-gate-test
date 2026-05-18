# ci-gate-test

Experimental repository for validating ci-gate.

## Structure

Three independent components, each with its own path-filtered GitHub Actions workflow:

| Directory | Workflow | Triggers on |
|---|---|---|
| `alpha/` | `.github/workflows/alpha.yaml` | changes to `alpha/**` |
| `beta/` | `.github/workflows/beta.yaml` | changes to `beta/**` |
| `gamma/` | `.github/workflows/gamma.yaml` | changes to `gamma/**` |

## How to test ci-gate

### Scenario 1 — PR touches one component (two workflows skipped)

```bash
git checkout -b test/alpha-only
echo "change" >> alpha/main.go
git commit -am "change alpha"
git push origin test/alpha-only
# open a PR
```

Expected: ci-gate immediately marks Beta and Gamma as skipped. Waits for Alpha to
complete, then resolves `ci-gate` to success.

### Scenario 2 — PR touches no components (all workflows skipped)

```bash
git checkout -b test/docs-only
echo "# note" >> README.md
git commit -am "update readme"
git push origin test/docs-only
# open a PR
```

Expected: ci-gate immediately resolves to success with all three workflows skipped.

### Scenario 3 — PR touches all components

```bash
git checkout -b test/all-components
echo "change" >> alpha/main.go
echo "change" >> beta/main.go
echo "change" >> gamma/main.go
git commit -am "change all"
git push origin test/all-components
# open a PR
```

Expected: ci-gate stays in_progress while all three workflows run (~15s each),
then resolves to success once all complete.

## Branch protection setup

Once ci-gate is running and receiving webhook events from this repo:

1. **Settings → Branches → Add rule** for `main`
2. Enable **Require status checks to pass before merging**
3. Add **`ci-gate`** as the required check
4. Enable **Require branches to be up to date** (optional, for merge queue)
