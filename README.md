# Class1 (Internal name: abc7d) — Project controls and cost-risk estimation for AI systems

Class1 is a high-performance cost-engineering tool for LLM-based systems. Unlike typical observability tools that look backward at historical spend, Class1 focuses on **estimation**: providing a risk-adjusted forecast of a code change's monthly cost delta **in the pull request, before merge**, with a declared AACE estimate class and a calibration loop.

Cost engineering, not dashboards.

---

## How it works

When a developer submits a PR, the Class1 GitHub Action:
1. **Scans the diff** to identify added, modified, or removed LLM callsites (Python AST + TS/JS tree-sitter).
2. **Translates code changes** (model swaps, token changes, tool changes) into baseline vs. head scenarios.
3. **Runs a Monte Carlo simulation** using Common Random Numbers (CRN) to estimate the monthly cost delta.
4. **Applies a budget policy** to automatically warn or block the PR if the P90 tail risk exceeds your threshold.
5. **Posts a detailed cost-risk report** directly to the PR comments.

---

## Getting Started: GitHub Action Setup

Add the Class1 check to your repository in three simple steps:

### 1. Create a budget policy
Add a `.class1.json` file at the root of your repository:
```json
{
  "fail_pr_if": {
    "delta_p90_usd": 500.0,
    "warn_at_fraction": 0.8
  }
}
```
*If a PR's estimated monthly P90 cost increase exceeds $500, the GitHub status check will fail, blocking the merge.*

### 2. Configure the GitHub Actions workflow
Create `.github/workflows/class1.yml`:
```yaml
name: Class1 Cost-Risk Guard

on:
  pull_request:
    branches: [ main ]

jobs:
  estimate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Required to scan history and base ref diffs

      - name: Run Class1 Guard
        uses: 1snob/class1-action@latest
        with:
          license_key: ${{ secrets.CLASS1_LICENSE_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### 3. Add your License Key (Premium features)
To unlock blocking budget gates, custom price overrides, and data persistence for post-merge calibration, get a license key from [https://class1.dev](https://class1.dev) and add it as a Repository Secret named `CLASS1_LICENSE_KEY`.

*Class1 remains completely **free and advisory** (posting comments without blocking) for open-source repositories and individual developers.*

---

## License

Class1 is licensed under the Business Source License 1.1 (BSL) — see [LICENSE](https://github.com/1snob/abc7d/blob/main/LICENSE) for details. The project relies on price and capability dataset snapshots under their respective open-source licenses.
