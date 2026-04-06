# Adapt Action Example Workflows

## Overview

This repository provides reference implementations of GitHub Actions workflows that integrate the [SecurityJourney/adapt-action@v1](https://github.com/SecurityJourney/adapt-action) with industry-standard security scanning tools. These examples illustrate best practices for extracting Common Weakness Enumeration (CWE) identifiers from scanner outputs and transmitting them to the Security Journey platform via the adapt-action.

## Example Workflows

The adapt-action is scanner-agnostic — it works with any tool capable of outputting CWE identifiers. Many scanners support the [SARIF](https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html) standard. Scanner outputs may require light transformation before being passed to the action. The examples here use [jq](https://jqlang.org/), a lightweight JSON processor pre-installed on all GitHub-hosted runners, to handle that reshaping.

This repository includes examples for a few popular scanners; this is not an exhaustive list:

- **GitHub Advanced Security (CodeQL)** - [`examples/ghas.yml`](examples/ghas.yml)
- **Snyk** - [`examples/snyk.yml`](examples/snyk.yml)

## Prerequisites

### General Requirements

All workflows require:

- A valid Security Journey API key
- GitHub Actions enabled on your repository

### Scanner-Specific Requirements

- **GitHub Advanced Security (CodeQL)**: Requires GitHub Advanced Security to be enabled on your organization or repository. This feature requires a separate license purchase for GitHub Enterprise Cloud. It can be enabled on public repositories for free.
- **Snyk**: Requires a Snyk account and API token. Sign up at [snyk.io](https://snyk.io) to obtain your API key.

## Installation

To implement these workflows in your repository:

1. **Copy the workflow file** corresponding to your security scanner from this repository to your `.github/workflows/` directory.

2. **Configure repository secrets** via Settings > Secrets and variables > Actions:
   - `SECURITY_JOURNEY_API_KEY` - Your Security Journey API key (required for all workflows)
   - `SNYK_TOKEN` - Your Snyk API token (required only for Snyk workflow)

   Note: The `GITHUB_TOKEN` is automatically provided by GitHub Actions but must still be explicitly passed as an environment variable in your workflow step (e.g. `env: GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}`).

3. **Configure branch triggers** in the workflow file to align with your branching strategy:

   ```yaml
   on:
     pull_request:
       branches:
         - main # Modify to match your target branch
   ```

4. **Customize scanner configuration** parameters as necessary to meet your project's security requirements.

## Architecture

Each workflow implements a two-stage pipeline:

1. **Scan Stage**: Executes the security scanner and extracts CWE identifiers from the analysis results
2. **Integration Stage**: Transmits the extracted CWE data to the Security Journey API via the adapt-action

The adapt-action requires a Security Journey API key and an array of CWE identifiers. Any security scanner capable of producing CWE output in this format can be integrated with the action.

## Integration Requirements

To integrate a custom security scanner with the adapt-action, the scanner must produce an array of CWE identifiers that can be passed to the action:

```yaml
- name: Process extracted CWEs
  uses: SecurityJourney/adapt-action@v1
  with:
    api_key: ${{ secrets.SECURITY_JOURNEY_API_KEY }}
    cwes: "${{ needs.scan.outputs.cwes }}" # Array of CWE IDs
```

The `cwes` parameter expects a JSON array of CWE identifiers. The API accepts several formats — the prefix and casing are normalized automatically:

```json
["CWE-79", "cwe-89", "CWE_502", "200"]
```

As long as your scanner can produce or be transformed to produce an array of CWE identifiers in any of these formats, it can be integrated with the Security Journey platform using this action.
