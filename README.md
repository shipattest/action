# ShipAttest GitHub Action

Generate a CycloneDX SBOM for every release and upload it to [ShipAttest](https://shipattest.com), which re-checks it daily against OSV, CISA's Known Exploited Vulnerabilities catalogue and ENISA's EU Vulnerability Database. When a shipped component becomes actively exploited, ShipAttest opens an EU Cyber Resilience Act Article 14 case with the 24-hour, 72-hour and final-report deadlines.

## Usage

1. In ShipAttest, create a token under **Settings → API tokens**.
2. In your repository, add it as a secret named `SHIPATTEST_TOKEN` (**Settings → Secrets and variables → Actions**).
3. Add a workflow:

```yaml
name: SBOM
on:
  release:
    types: [published]

jobs:
  sbom:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: shipattest/action@v1
        with:
          product: smart-hub
          token: ${{ secrets.SHIPATTEST_TOKEN }}
```

The version defaults to the tag or branch name (`github.ref_name`).

## Inputs

| Input | Required | Default | Meaning |
|---|---|---|---|
| `product` | yes | | Product identifier in ShipAttest (created on first upload) |
| `token` | yes | | ShipAttest API token; store it as a secret |
| `version` | no | `github.ref_name` | Release version |
| `path` | no | `.` | Directory to scan for lockfiles |
| `sbom` | no | | Upload this CycloneDX or SPDX JSON file instead of reading lockfiles (firmware, containers, Java, C/C++) |
| `name` | no | | Product display name |
| `with-usage` | no | `false` | Send short excerpts of where direct dependencies are imported, for AI triage |
| `include-dev` | no | `false` | Include development-only dependencies |
| `fail-on-exploited` | no | `false` | Fail the step when the release contains an actively exploited vulnerability |
| `url` | no | `https://shipattest.com` | ShipAttest server URL |

## GitLab CI

```yaml
include:
  - remote: "https://raw.githubusercontent.com/shipattest/action/main/gitlab/shipattest.gitlab-ci.yml"

shipattest:
  extends: .shipattest-upload
  variables:
    SHIPATTEST_PRODUCT: smart-hub
  rules:
    - if: $CI_COMMIT_TAG
```

Set `SHIPATTEST_TOKEN` as a masked CI/CD variable.

## Without an account

The bundled CLI also scans locally for free: `npx @shipattest/cli scan . --fail-on exploited`. See [`@shipattest/cli`](https://www.npmjs.com/package/@shipattest/cli).

Licensed under Apache-2.0.
