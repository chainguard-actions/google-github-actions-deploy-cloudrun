<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--deploy-cloudrun/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--deploy-cloudrun/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

Failing references:
- cleanup.yml: `google-github-actions/auth@v3` and `google-github-actions/setup-gcloud@v2`
- draft-release.yml: `google-github-actions/.github/.github/workflows/draft-release.yml@v3`
- integration.yml: `google-github-actions/auth@v3` (appears three times)
- release.yml: `google-github-actions/.github/.github/workflows/release.yml@v3`

Locations:

- `.github/workflows/cleanup.yml:29`
- `.github/workflows/cleanup.yml:32`
- `.github/workflows/draft-release.yml:16`
- `.github/workflows/integration.yml:53`
- `.github/workflows/integration.yml:83`
- `.github/workflows/integration.yml:121`
- `.github/workflows/release.yml:10`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. In the `metadata` job's 'Set service name in metadata YAML' step, `${{ env.SERVICE_NAME }}` is embedded directly in a `sed` command:

```
sed -i "s/run-full-yaml/${{ env.SERVICE_NAME }}/" ./tests/fixtures/service.yaml
```

The `env.*` context is resolved by the GitHub Actions template engine before the shell ever sees the string, meaning a malicious value in `SERVICE_NAME` could inject arbitrary shell metacharacters into the `sed` argument and break out of the quoted string. The value should be passed via an `env:` variable and referenced as `"$SERVICE_NAME"` inside the script.

Locations:

- `.github/workflows/integration.yml:105`

### github-env-injection (severity: high)

In the `deploy` job's 'Compute service name' step, the env var `MATRIX_NAME` is set from `${{ matrix.name }}` (a workflow-controllable value) and then written unsanitized to `$GITHUB_ENV`:

```yaml
env:
  MATRIX_NAME: '${{ matrix.name }}'
run: |-
  echo "SERVICE_NAME=${GITHUB_JOB}-${MATRIX_NAME}-${GITHUB_SHA::7}-${GITHUB_RUN_NUMBER}" >> "${GITHUB_ENV}"
```

Routing through an `env:` variable does not sanitize the value. A newline character embedded in `matrix.name` could inject additional key=value pairs into `$GITHUB_ENV`. The value must be sanitized with `printf '%s' "$MATRIX_NAME" | tr -d '\n\r'` before the write.

Locations:

- `.github/workflows/integration.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 7 unpinned action references by resolving to full commit SHAs: google-github-actions/auth@v3 → 7c6bc770dae815cd3e89ee6cdf493a5fab2cc093, google-github-actions/setup-gcloud@v2 → e427ad8a34f8676edf47cf7d7925499adf3eb74f, google-github-actions/.github@v3 → 29c6d38eeb974133b4b66401985f7c70cf4a6681. Fixed script-injection in integration.yml metadata job by moving ${{ env.SERVICE_NAME }} into an env: block and referencing it as ${SERVICE_NAME} in the sed command. Fixed github-env-injection in integration.yml deploy job by sanitizing MATRIX_NAME with printf '%s' | tr -d '\n\r' before writing to GITHUB_ENV.

