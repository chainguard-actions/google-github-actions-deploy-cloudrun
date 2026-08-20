<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--deploy-cloudrun/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--deploy-cloudrun/v3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable version tags (@v2, @v3) instead of immutable full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved. Affected references:
- cleanup.yml: `google-github-actions/auth@v3` and `google-github-actions/setup-gcloud@v2`
- draft-release.yml: `google-github-actions/.github/.github/workflows/draft-release.yml@v3`
- integration.yml: `google-github-actions/auth@v3` (appears 3 times)
- release.yml: `google-github-actions/.github/.github/workflows/release.yml@v3`

Locations:

- `.github/workflows/cleanup.yml:22`
- `.github/workflows/cleanup.yml:27`
- `.github/workflows/draft-release.yml:14`
- `.github/workflows/integration.yml:42`
- `.github/workflows/integration.yml:88`
- `.github/workflows/integration.yml:148`
- `.github/workflows/release.yml:9`

### script-injection (severity: high)

Sub-rule (a) violation: The 'Set service name in metadata YAML' step in the `metadata` job of integration.yml directly interpolates a GitHub Actions expression inside a `run:` shell command. The offending line is:
  `sed -i "s/run-full-yaml/${{ env.SERVICE_NAME }}/" ./tests/fixtures/service.yaml`
The expression `${{ env.SERVICE_NAME }}` is expanded by the Actions template engine before the shell ever sees the command, allowing an attacker who can influence `env.SERVICE_NAME` to inject arbitrary shell metacharacters into the `sed` argument string.

Locations:

- `.github/workflows/integration.yml:96`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 7 unpinned action references by resolving their full commit SHAs via lookup_action_sha: google-github-actions/auth@v3 → SHA 7c6bc770..., google-github-actions/setup-gcloud@v2 → SHA e427ad8a..., and google-github-actions/.github reusable workflows @v3 → SHA 29c6d38e.... Fixed the script injection in integration.yml's 'Set service name in metadata YAML' step by moving ${{ env.SERVICE_NAME }} into the step's env: block and referencing it as the plain $SERVICE_NAME environment variable in the sed command.

