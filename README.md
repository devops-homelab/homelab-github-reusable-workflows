# Homelab GitHub Reusable Workflows

This repository contains reusable GitHub Actions workflows designed to streamline CI/CD processes for various projects in the homelab ecosystem. These workflows can be referenced across multiple repositories to ensure consistency and reduce duplication of effort.

## Repository Structure

```
.github/
    workflows/
        helm-release.yaml
        helm-test.yaml
```

### `helm-release.yaml`
This workflow automates the release process for Helm charts. It handles versioning, packaging, and publishing charts to a specified repository.

### `helm-test.yaml`
This workflow performs linting and testing of Helm charts to ensure they meet quality standards before being merged or released.

## Usage

To use these reusable workflows in your repository, reference them in your GitHub Actions workflow files. For example:

### Example: Using `helm-release.yaml`
```yaml
name: Chart Release

on:
  push:
    branches:
      - main

jobs:
  release-chart:
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/helm-release.yaml@main
    with:
      REPO_URL: https://raw.githubusercontent.com/devops-homelab/homelab-helm-charts/gh-pages
    secrets:
      GITHUB_PAT: ${{ secrets.GH_PAT }}
```

### Example: Using `helm-test.yaml`
```yaml
name: Chart Test

on:
  pull_request:
    types:
      - opened
      - reopened
      - labeled
      - synchronize

jobs:
  lint-test:
    uses: devops-homelab/homelab-github-reusable-workflows/.github/workflows/helm-test.yaml@main
```

## Contributing

Contributions are welcome! If you have suggestions for improvements or new workflows, feel free to open an issue or submit a pull request.

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

Happy automating!
