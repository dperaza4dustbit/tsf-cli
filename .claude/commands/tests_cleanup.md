# Cleanup E2E Test Resources

Clean up leftover resources from previous e2e test runs that may prevent new runs from succeeding.

## Steps

### 1. Verify Cluster Access

Source `e2e/my-test.env.template` (or `e2e/my-test.env` if it exists) to get environment variables.
Verify `oc whoami` works. If not, ask the user to log in first.

### 2. Identify Leftover Resources

Check for leftover resources on the cluster. Show the user what was found and ask for confirmation before deleting anything.

**Namespaces:**
- Check if `$E2E_APPLICATIONS_NAMESPACE-managed` namespace exists (`oc get namespace <name>`)
- List any namespaces matching the pattern `e2e-tsf-demo-*` which are generated test namespaces (`oc get namespaces | grep e2e-tsf-demo`)

**Cluster resources in `$E2E_APPLICATIONS_NAMESPACE`:**
- Check for leftover ReleasePlan: `oc get releaseplan tsf-release -n $E2E_APPLICATIONS_NAMESPACE`

**GitHub resources** (only if `GITHUB_TOKEN` and `MY_GITHUB_ORG` are set):
- List any open PRs in `$MY_GITHUB_ORG/testrepo` that have branches starting with `appstudio-` (PaC branches): `gh pr list --repo "$MY_GITHUB_ORG/testrepo" --state open`
- List branches starting with `appstudio-` or `base-` in `$MY_GITHUB_ORG/testrepo`: `gh api repos/$MY_GITHUB_ORG/testrepo/branches --paginate -q '.[].name' | grep -E '^(appstudio-|base-)'`
- List webhooks on `$MY_GITHUB_ORG/testrepo`: `gh api repos/$MY_GITHUB_ORG/testrepo/hooks -q '.[].config.url'`

### 3. Present Findings and Confirm

Present a summary of all found resources in a clear table/list. Use AskUserQuestion to ask the user which categories of resources to delete. Options should be:
- "All found resources" (recommended)
- "Only namespaces and cluster resources"
- "Only GitHub resources (PRs, branches, webhooks)"
- "Let me pick individually"

If "Let me pick individually" is chosen, ask about each resource separately.

### 4. Delete Confirmed Resources

For each confirmed category:

**Namespaces:**
- `oc delete namespace <name>` for each namespace
- Wait briefly and confirm deletion started (namespaces may take a while to terminate)

**Cluster resources:**
- `oc delete releaseplan tsf-release -n $E2E_APPLICATIONS_NAMESPACE` if it exists

**GitHub PRs:**
- Close any open PaC PRs: `gh pr close <number> --repo "$MY_GITHUB_ORG/testrepo"`

**GitHub branches:**
- Delete branches: `gh api -X DELETE repos/$MY_GITHUB_ORG/testrepo/git/refs/heads/<branch>`

**GitHub webhooks:**
- For each webhook, show its URL and delete if confirmed: `gh api -X DELETE repos/$MY_GITHUB_ORG/testrepo/hooks/<id>`

### 5. Verify

After cleanup, re-check that the key blocker (`$E2E_APPLICATIONS_NAMESPACE-managed` namespace) is gone or terminating. Report the final status.
